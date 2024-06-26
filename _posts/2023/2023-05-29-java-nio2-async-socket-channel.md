---
layout: post
title:  NIO 2 AsynchronousSocketChannel指南
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在本文中，我们将演示如何使用Java 7 NIO.2通道API构建一个简单的服务器及其客户端。

我们将查看AsynchronousServerSocketChannel和AsynchronousSocketChannel类，它们分别是用于实现服务器和客户端的关键类。

如果你不熟悉NIO.2通道API，我们在此站点上有一篇介绍性文章。你可以点击此[链接](https://www.baeldung.com/java-nio-2-async-channels)阅读它。

使用NIO.2通道API所需的所有类都捆绑在java.nio.channels包中：

```java
import java.nio.channels.*;
```

## 2. 使用Future的服务器

AsynchronousServerSocketChannel的实例是通过在其类上调用静态open API创建的：

```java
AsynchronousServerSocketChannel server = AsynchronousServerSocketChannel.open();
```

新创建的异步服务器套接字通道已打开但尚未绑定，因此我们必须将其绑定到本地地址并选择一个端口：

```java
server.bind(new InetSocketAddress("127.0.0.1", 4555));
```

我们也可以传入null以便它使用本地地址并绑定到任意端口：

```java
server.bind(null);
```

绑定后，accept API用于启动接受到通道套接字的连接：

```java
Future<AsynchronousSocketChannel> acceptFuture = server.accept();
```

与异步通道操作一样，上述调用立即返回并继续执行。

接下来，我们可以使用get API查询来自Future对象的响应：

```java
AsynchronousSocketChannel worker = future.get();
```

如果需要等待来自客户端的连接请求，此调用将阻塞。或者，如果我们不想永远等待，我们可以指定超时：

```java
AsynchronousSocketChannel worker = acceptFuture.get(10, TimeUnit.SECONDS);
```

在上述调用返回且操作成功后，我们可以创建一个循环，在该循环中，我们监听传入的消息并将其回显到客户端。

让我们创建一个名为runServer的方法，我们将在其中进行等待并处理任何传入消息：

```java
public void runServer() {
    clientChannel = acceptResult.get();
    if ((clientChannel != null) && (clientChannel.isOpen())) {
        while (true) {
            ByteBuffer buffer = ByteBuffer.allocate(32);
            Future<Integer> readResult  = clientChannel.read(buffer);
            
            // perform other computations
            
            readResult.get();
            
            buffer.flip();
            Future<Integer> writeResult = clientChannel.write(buffer);
 
            // perform other computations
 
            writeResult.get();
            buffer.clear();
        } 
        clientChannel.close();
        serverChannel.close();
    }
}
```

在循环内部，我们所做的就是创建一个缓冲区，根据操作进行读取和写入。

然后，每次我们进行读取或写入时，我们都可以继续执行任何其他代码，当我们准备好处理结果时，我们调用Future对象的get() API。

要启动服务器，我们调用其构造函数，然后在main中调用runServer方法：

```java
public static void main(String[] args) {
    AsyncEchoServer server = new AsyncEchoServer();
    server.runServer();
}
```

## 3. 使用CompletionHandler的服务器

在本节中，我们将了解如何使用CompletionHandler方法而不是Future方法来实现相同的服务器。

在构造函数中，我们创建一个AsynchronousServerSocketChannel并将其绑定到本地地址，方法与之前相同：

```java
serverChannel = AsynchronousServerSocketChannel.open();
InetSocketAddress hostAddress = new InetSocketAddress("localhost", 4999);
serverChannel.bind(hostAddress);
```

接下来，仍然在构造函数内部，我们创建一个while循环，在其中我们接受来自客户端的任何传入连接。**此while循环严格用于防止服务器在与客户端建立连接之前退出**。

**为了防止循环无休止地运行，我们在其末尾调用System.in.read()来阻塞执行，直到从标准输入流中读取传入连接**：

```java
while (true) {
    serverChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel,Object>() {
        @Override
        public void completed(AsynchronousSocketChannel result, Object attachment) {
            if (serverChannel.isOpen()){
                serverChannel.accept(null, this);
            }

            clientChannel = result;
            if ((clientChannel != null) && (clientChannel.isOpen())) {
                ReadWriteHandler handler = new ReadWriteHandler();
                ByteBuffer buffer = ByteBuffer.allocate(32);

                Map<String, Object> readInfo = new HashMap<>();
                readInfo.put("action", "read");
                readInfo.put("buffer", buffer);

                clientChannel.read(buffer, readInfo, handler);
             }
         }
        @Override
        public void failed(Throwable exc, Object attachment) {
            // process error
        }
    });
    System.in.read();
}
```

建立连接后，会调用accept操作的CompletionHandler中的completed回调方法。

它的返回类型是AsynchronousSocketChannel的实例。如果服务器套接字通道仍处于打开状态，我们将再次调用accept API以准备好接收另一个传入连接，同时重用相同的处理程序。

接下来，我们将返回的套接字通道分配给一个全局实例。然后我们检查它是否不为null并且在对其执行操作之前它是否已打开。

我们可以开始读写操作的点是在accept操作的处理程序的completed 回调API中。此步骤取代了之前我们使用get API轮询通道的方法。

请注意，**建立连接后服务器将不再退出，除非我们显式关闭它**。

另请注意，我们创建了一个单独的内部类ReadWriteHandler来处理读写操作。此时我们将看到attachment对象如何派上用场。

首先，让我们看一下ReadWriteHandler类：

```java
class ReadWriteHandler implements CompletionHandler<Integer, Map<String, Object>> {

    @Override
    public void completed(Integer result, Map<String, Object> attachment) {
        Map<String, Object> actionInfo = attachment;
        String action = (String) actionInfo.get("action");

        if ("read".equals(action)) {
            ByteBuffer buffer = (ByteBuffer) actionInfo.get("buffer");
            buffer.flip();
            actionInfo.put("action", "write");

            clientChannel.write(buffer, actionInfo, this);
            buffer.clear();

        } else if ("write".equals(action)) {
            ByteBuffer buffer = ByteBuffer.allocate(32);

            actionInfo.put("action", "read");
            actionInfo.put("buffer", buffer);

            clientChannel.read(buffer, actionInfo, this);
        }
    }

    @Override
    public void failed(Throwable exc, Map<String, Object> attachment) {
        // 
    }
}
```

ReadWriteHandler类中attachment的泛型类型是Map。我们特别需要通过它传递两个重要的参数-操作类型(action)和缓冲区。

接下来，我们将看到如何使用这些参数。

我们执行的第一个操作是read，因为这是一个只对客户端消息做出响应的回显服务器。在ReadWriteHandler的completed回调方法中，我们检索附加数据并决定相应地做什么。

如果是已完成的read操作，我们检索缓冲区，更改attachment的action参数并立即执行write操作以将消息回显给客户端。

如果是刚刚完成的write操作，我们再次调用read API以准备服务器接收另一条传入消息。

## 4. 客户端

设置服务器后，我们现在可以通过调用AsynchronousSocketChannel类上的open API来设置客户端。此调用创建一个新的客户端套接字通道实例，然后我们使用它来建立与服务器的连接：

```java
AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
InetSocketAddress hostAddress = new InetSocketAddress("localhost", 4999)
Future<Void> future = client.connect(hostAddress);
```

connect操作成功时不返回任何内容。但是，我们仍然可以使用Future对象来监视异步操作的状态。

让我们调用get API来等待连接：

```java
future.get()
```

在这一步之后，我们可以开始向服务器发送消息并接收回显。sendMessage方法如下所示：

```java
public String sendMessage(String message) {
    byte[] byteMsg = new String(message).getBytes();
    ByteBuffer buffer = ByteBuffer.wrap(byteMsg);
    Future<Integer> writeResult = client.write(buffer);

    // do some computation

    writeResult.get();
    buffer.flip();
    Future<Integer> readResult = client.read(buffer);
    
    // do some computation

    readResult.get();
    String echo = new String(buffer.array()).trim();
    buffer.clear();
    return echo;
}
```

## 5. 测试

为了确认我们的服务器和客户端应用程序是否按照预期执行，我们可以使用测试：

```java
@Test
public void givenServerClient_whenServerEchosMessage_thenCorrect() {
    String resp1 = client.sendMessage("hello");
    String resp2 = client.sendMessage("world");

    assertEquals("hello", resp1);
    assertEquals("world", resp2);
}
```

## 6. 总结

在本文中，我们探讨了Java NIO.2异步套接字通道API。我们已经能够逐步完成使用这些新API构建服务器和客户端的过程。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-1)上获得。
