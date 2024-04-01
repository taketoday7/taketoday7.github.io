---
layout: post
title:  Java IO与NIO
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

处理输入和输出是Java程序员的常见任务。在本教程中，我们将**介绍原始的[java.io](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/package-summary.html)([IO](https://www.baeldung.com/java-io))库和更新的[java.nio](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/package-summary.html)([NIO](https://www.baeldung.com/tag/java-nio/))库**，以及它们在通过网络通信时的不同之处。

## 2. 主要特点

让我们先看看这两个包的主要功能。

### 2.1 IO-java.io

**Java 1.0中引入了java.io包**，Java 1.1中引入了Reader。它提供：

-   InputStream和[OutputStream](https://www.baeldung.com/java-outputstream)：一次提供一个字节的数据
-   Reader和Writer：流的便利包装器
-   阻塞模式：等待完整的消息

### 2.2 NIO-java.nio

**java.nio包在Java 1.4中引入并在Java 1.7(NIO 2)中进行了更新**，具有[增强的文件操作](https://www.baeldung.com/java-nio-2-file-api)和[ASynchronousSocketChannel](https://www.baeldung.com/java-nio2-async-socket-channel)。它提供：

-   缓冲区：一次读取数据块
-   CharsetDecoder：用于将原始字节映射到可读字符/从可读字符映射
-   Channel：与外界通信
-   [Selector](https://www.baeldung.com/java-nio-selector)：在SelectableChannel上启用多路复用，并提供对任何准备好I/O的Channel的访问
-   非阻塞模式：读取任何准备就绪的内容

现在让我们看看在向服务器发送数据或读取其响应时如何使用这些包中的每一个。

## 3. 配置我们的测试服务器

在这里，我们将使用[WireMock](https://www.baeldung.com/introduction-to-wiremock)模拟另一台服务器，以便我们可以独立运行测试。

我们将配置它来监听我们的请求并向我们发送响应，就像真正的Web服务器一样。我们还将使用动态端口，这样我们就不会与本地机器上的任何服务发生冲突。

让我们为具有测试范围的WireMock添加Maven依赖项：

```xml
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.26.3</version>
    <scope>test</scope>
</dependency>
```

在测试类中，让我们定义一个JUnit @Rule以在空闲端口上启动WireMock。然后，我们将其配置为在请求预定义资源时返回HTTP 200响应，消息正文为JSON格式的一些文本：

```java
@Rule public WireMockRule wireMockRule = new WireMockRule(wireMockConfig().dynamicPort());

private String REQUESTED_RESOURCE = "/test.json";

@Before
public void setup() {
    stubFor(get(urlEqualTo(REQUESTED_RESOURCE))
        .willReturn(aResponse()
        .withStatus(200)
        .withBody("{ \"response\" : \"It worked!\" }")));
}
```

现在我们已经设置了模拟服务器，我们准备运行一些测试。

## 4. 阻塞IO-java.io

让我们通过从网站读取一些数据来了解原始阻塞IO模型的工作原理。我们将使用[java.net.Socket](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/Socket.html)来访问操作系统的一个端口。

### 4.1 发送请求

在此示例中，我们将创建一个GET请求来检索我们的资源。首先，让我们创建一个Socket来访问我们的WireMock服务器正在监听的端口：

```java
Socket socket = new Socket("localhost", wireMockRule.port())
```

对于正常的HTTP或HTTPS通信，端口将为80或443。但是，在这种情况下，我们使用wireMockRule.port()来访问我们之前设置的动态端口。

现在让我们**在套接字上打开一个OutputStream**，包装在一个OutputStreamWriter中并将它传递给PrintWriter来编写我们的消息。让我们确保刷新缓冲区以便发送我们的请求：

```java
OutputStream clientOutput = socket.getOutputStream();
PrintWriter writer = new PrintWriter(new OutputStreamWriter(clientOutput));
writer.print("GET " + TEST_JSON + " HTTP/1.0\r\n\r\n");
writer.flush();
```

### 4.2 等待响应

让我们在套接字上打开InputStream以访问响应，使用[BufferedReader](https://www.baeldung.com/java-buffered-reader)读取流，并将其存储在StringBuilder中：

```java
InputStream serverInput = socket.getInputStream();
BufferedReader reader = new BufferedReader(new InputStreamReader(serverInput));
StringBuilder ourStore = new StringBuilder();
```

让我们使用reader.readLine()来阻塞，等待一个完整的行，然后将该行追加到我们的ourStore。我们将继续读取，直到得到一个null，这表明流的结束：

```java
for (String line; (line = reader.readLine()) != null;) {
   ourStore.append(line);
   ourStore.append(System.lineSeparator());
}
```

## 5. 非阻塞IO-java.nio

现在，让我们看看nio包的非阻塞IO模型如何使用相同的示例。

这一次，我们将**创建一个[java.nio.channel.SocketChannel](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/SocketChannel.html)来访问我们服务器上的端口(而不是java.net.Socket)**，并传递给它一个InetSocketAddress。

### 5.1 发送请求

首先，让我们打开我们的SocketChannel：

```java
InetSocketAddress address = new InetSocketAddress("localhost", wireMockRule.port());
SocketChannel socketChannel = SocketChannel.open(address);
```

现在，让我们使用标准的UTF-8字符集来编码和写入我们的消息：

```java
Charset charset = StandardCharsets.UTF_8;
socket.write(charset.encode(CharBuffer.wrap("GET " + REQUESTED_RESOURCE + " HTTP/1.0\r\n\r\n")));
```

### 5.2 读取响应

发送请求后，我们可以使用原始缓冲区以非阻塞模式读取响应。

由于我们将处理文本，因此我们需要一个用于原始字节的[ByteBuffer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/ByteBuffer.html)和一个用于转换字符的[CharBuffer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/CharBuffer.html)(由[CharsetDecoder](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/charset/CharsetDecoder.html)辅助)：

```java
ByteBuffer byteBuffer = ByteBuffer.allocate(8192);
CharsetDecoder charsetDecoder = charset.newDecoder();
CharBuffer charBuffer = CharBuffer.allocate(8192);
```

如果数据以多字节字符集发送，我们的CharBuffer将有剩余空间。

请注意，如果我们需要特别快的性能，我们可以使用ByteBuffer.allocateDirect()在本机内存中创建一个[MappedByteBuffer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/MappedByteBuffer.html)。但是，在我们的例子中，从标准堆中使用allocate()足够快了。

**在处理缓冲区时，我们需要知道缓冲区有多大(容量)，我们在缓冲区的什么位置(当前位置)，以及我们能走多远(限制)**。

因此，让我们从SocketChannel中读取数据，将其传递给我们的ByteBuffer以存储我们的数据。我们从SocketChannel的读取将**以ByteBuffer的当前位置设置为要写入的下一个字节(紧接在写入的最后一个字节之后)完成，但其limit不变**：

```java
socketChannel.read(byteBuffer)
```

**我们的SocketChannel.read()返回可以写入缓冲区的读取字节数**。如果套接字断开连接，这将为-1。

当我们的缓冲区因为我们还没有处理它的所有数据而没有剩余空间时，那么SocketChannel.read()将返回0字节读取但我们的buffer.position()仍然大于0。

为确保我们从缓冲区中的正确位置开始读取，我们将**使用Buffer.flip()将ByteBuffer的当前位置设置为0，并将其limit设置为SocketChannel写入的最后一个字节**。然后我们将使用我们的storeBufferContents方法保存缓冲区内容，稍后我们将查看。最后，我们将使用buffer.compact()来压缩缓冲区并设置当前位置以备下次从SocketChannel读取。

由于我们的数据可能会分段到达，让我们将读取缓冲区的代码包装在一个带有终止条件的循环中，以检查我们的套接字是否仍处于连接状态，或者我们是否已断开连接但缓冲区中仍有数据：

```java
while (socketChannel.read(byteBuffer) != -1 || byteBuffer.position() > 0) {
    byteBuffer.flip();
    storeBufferContents(byteBuffer, charBuffer, charsetDecoder, ourStore);
    byteBuffer.compact();
}
```

不要忘记close()我们的套接字(除非我们在try-with-resources块中打开它)：

```java
socketChannel.close();
```

### 5.3 存储缓冲区中的数据

来自服务器的响应将包含标头，这可能会使数据量超过我们缓冲区的大小。因此，我们将使用StringBuilder在消息到达时构建完整的消息。

为了存储我们的消息，我们首先**将原始字节解码为CharBuffer中的字符**。然后我们将翻转指针，以便我们可以读取我们的字符数据，并将其附加到我们的可扩展StringBuilder。最后，我们将清除CharBuffer，为下一个写/读周期做好准备。

所以现在，让我们实现完整的storeBufferContents()方法，传入我们的缓冲区、CharsetDecoder和StringBuilder：

```java
void storeBufferContents(ByteBuffer byteBuffer, CharBuffer charBuffer, CharsetDecoder charsetDecoder, StringBuilder ourStore) {
    charsetDecoder.decode(byteBuffer, charBuffer, true);
    charBuffer.flip();
    ourStore.append(charBuffer);
    charBuffer.clear();
}
```

## 6. 总结

在本文中，我们了解了原始java.io模型如何阻塞、等待请求并使用流来操作它接收到的数据。

相比之下，java.nio库允许使用Buffer和Channel进行非阻塞通信，并且可以提供直接内存访问以获得更快的性能。然而，这种速度带来了处理缓冲区的额外复杂性。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-2)上获得。
