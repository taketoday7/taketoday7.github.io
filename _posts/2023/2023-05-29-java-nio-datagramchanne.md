---
layout: post
title:  Java NIO DatagramChannel
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在本教程中，我们将探讨允许我们[发送和接收UDP数据包](https://www.baeldung.com/udp-in-java)的DatagramChannel类。

## 2. DatagramChannel

在互联网支持的各种协议中，[TCP和UDP](https://www.baeldung.com/cs/udp-vs-tcp)是最常见的。

TCP是一种面向连接的协议，而**UDP是一种面向数据报的协议，性能高但可靠性较低**。由于其不可靠的特性，**UDP通常用于发送广播或多播数据传输**。

**[Java的NIO模块](https://www.baeldung.com/tag/java-nio/)的[DatagramChannel](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/DatagramChannel.html)类为面向数据报的套接字提供了可选择的通道**。换句话说，它允许创建数据报通道来发送和接收数据报(UDP数据包)。

让我们使用DatagramChannel类创建一个通过本地IP地址发送数据报的客户端和一个接收数据报的服务器。

## 3. 打开并绑定

首先，让我们使用openChannel方法创建DatagramChannelBuilder类，该方法提供一个打开但未连接的数据报通道：

```java
public class DatagramChannelBuilder {
    public static DatagramChannel openChannel() throws IOException {
        DatagramChannel datagramChannel = DatagramChannel.open();
        return datagramChannel;
    }
}
```

然后，我们需要将一个打开的通道绑定到本地地址以监听入站UDP数据包。

因此，我们将添加将DatagramChannel绑定到提供的本地地址的bindChannel方法：

```java
public static DatagramChannel bindChannel(SocketAddress local) throws IOException {
    return openChannel().bind(local); 
}
```

现在，我们可以使用DatagramChannelBuilder类来创建在配置的套接字地址上发送/接收UDP数据包的客户端/服务器。

## 4. 客户端

首先，让我们使用startClient方法创建DatagramClient类，该方法使用已经讨论过的DatagramChannelBuilder类的bindChannel方法：


```java
public class DatagramClient {
    public static DatagramChannel startClient() throws IOException {
        DatagramChannel client = DatagramChannelBuilder.bindChannel(null);
        return client;
    }
}
```

**由于客户端不需要监听入站UDP数据包，因此我们在绑定通道时为地址提供了一个null值**。

然后，让我们添加sendMessage方法以在服务器地址上发送数据报：

```java
public static void sendMessage(DatagramChannel client, String msg, SocketAddress serverAddress) throws IOException {
    ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
    client.send(buffer, serverAddress);
}
```

就是这样！现在我们准备好使用客户端发送消息了：

```java
DatagramChannel client = startClient();
String msg = "Hello, this is a Tuyucheng's DatagramChannel based UDP client!";
InetSocketAddress serverAddress = new InetSocketAddress("localhost", 7001);

sendMessage(client, msg, serverAddress);
```

注意：因为我们已经将消息发送到localhost:7001地址，所以我们必须使用相同的地址启动我们的服务器。

## 5. 服务器

同样，让我们使用startServer方法创建DatagramServer类，以在localhost:7001地址上启动服务器：

```java
public class DatagramServer {
    public static DatagramChannel startServer() throws IOException {
        InetSocketAddress address = new InetSocketAddress("localhost", 7001);
        DatagramChannel server = DatagramChannelBuilder.bindChannel(address);
        System.out.println("Server started at #" + address);
        return server;
    }
}
```

然后，让我们添加receiveMessage方法，该方法从客户端接收数据报、提取消息并打印它：

```java
public static void receiveMessage(DatagramChannel server) throws IOException {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    SocketAddress remoteAdd = server.receive(buffer);
    String message = extractMessage(buffer);
    System.out.println("Client at #" + remoteAdd + "  sent: " + message);
}
```

此外，要从接收缓冲区中提取客户端消息，我们需要添加extractMessage方法：

```java
private static String extractMessage(ByteBuffer buffer) {
    buffer.flip();
    byte[] bytes = new byte[buffer.remaining()];
    buffer.get(bytes);
    String msg = new String(bytes);
    
    return msg;
}
```

在这里，我们在[ByteBuffer](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/ByteBuffer.html)实例上使用了flip方法，将其从读取I/O翻转为写入I/O。此外，flip方法将limit设置为当前位置并将position设置为0，以便我们从头开始读取。

现在，我们可以启动我们的服务器并接收来自客户端的消息：

```java
DatagramChannel server = startServer();
receiveMessage(server);
```

因此，当服务器收到我们的消息时，打印输出将是：

```text
Server started at #localhost/127.0.0.1:7001
Client at #/127.0.0.1:52580  sent: Hello, this is a Tuyucheng's DatagramChannel based UDP client!
```

## 6. DatagramChannelUnitTest

现在我们已经准备好了客户端和服务器，我们可以编写一个单元测试来验证端到端数据报(UDP数据包)的传递：

```java
@Test
public void whenClientSendsAndServerReceivesUDPPacket_thenCorrect() throws IOException {
    DatagramChannel server = DatagramServer.startServer();
    DatagramChannel client = DatagramClient.startClient();
    String msg1 = "Hello, this is a Tuyucheng's DatagramChannel based UDP client!";
    String msg2 = "Hi again!, Are you there!";
    InetSocketAddress serverAddress = new InetSocketAddress("localhost", 7001);
    
    DatagramClient.sendMessage(client, msg1, serverAddress);
    DatagramClient.sendMessage(client, msg2, serverAddress);
    
    assertEquals("Hello, this is a Tuyucheng's DatagramChannel based UDP client!", DatagramServer.receiveMessage(server));
    assertEquals("Hi again!, Are you there!", DatagramServer.receiveMessage(server));
}
```

首先，我们启动了绑定数据报通道的服务器以监听localhost:7001上的入站消息。然后，我们启动了客户端并发送了两条消息。

最后，我们在服务器上接收入站消息，并将它们与我们通过客户端发送的消息进行比较。

## 7. 其他方法

到目前为止，**我们已经使用了DatagramChannel类提供的open、bind、send和receive等方法**。现在，让我们快速浏览一下其他方便的方法。

### 7.1 configureBlocking

默认情况下，数据报通道是阻塞的。我们可以使用configureBlocking方法在传递false值时使通道非阻塞：

```java
client.configureBlocking(false);
```

### 7.2 isConnected

isConnected方法返回数据报通道的状态-也就是说，它是连接的还是断开的。

### 7.3 socket

socket方法返回与数据报通道关联的[DatagramSocket](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/DatagramSocket.html)类的对象。

### 7.4 close

此外，我们可以通过调用DatagramChannel类的close方法来关闭通道。

## 8. 总结

在这个快速教程中，我们探讨了Java NIO的DatagramChannel类，它允许创建数据报通道来发送/接收UDP数据包。

首先，我们检查了一些方法，如open和bind，它们同时允许数据报通道监听入站UDP数据包。

然后，我们创建了一个客户端和一个服务器来探索使用DatagramChannel类的端到端UDP数据包传输。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-2)上获得。
