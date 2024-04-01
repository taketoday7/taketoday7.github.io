---
layout: post
title:  Java 16中的Unix域套接字
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在本教程中，我们将学习Unix域[套接字通道](https://www.baeldung.com/a-guide-to-java-sockets)。

我们将介绍一些理论基础、优缺点，并构建一个简单的Java客户端-服务器应用程序，该应用程序使用Unix域套接字通道来交换文本消息。

我们还将了解如何使用Unix域套接字连接数据库。

## 2. Unix域套接字通道

传统的进程间通信涉及由IP地址和端口号定义的[TCP/IP](https://www.baeldung.com/cs/udp-vs-tcp#tcp)套接字。它们用于互联网或专用网络上的网络通信。

另一方面，Unix域[套接字](https://www.baeldung.com/cs/port-vs-socket#what-is-a-socket)仅限于同一物理主机上的进程之间的通信。几十年来，它们一直是Unix操作系统的一个功能，但最近才被添加到Microsoft Windows中。因此，它们不再局限于Unix系统。

Unix域套接字由看起来与其他文件名非常相似的[文件系统](https://www.baeldung.com/cs/files-file-systems#file-systems)路径名寻址，例如/folder/socket或C:\folder\socket。与TCP/IP连接相比，它们具有更快的设置时间、更高的输出数据量，并且在接受远程连接时没有安全风险。另一方面，最大的缺点是仅限于单个物理主机。

请注意，只要我们在共享卷上创建套接字，我们甚至可以使用Unix域套接字在同一系统上的[容器](https://www.baeldung.com/cs/containers-vs-virtual-machines)之间进行通信。

## 3. 套接字配置

正如我们之前所了解的，Unix域套接字基于文件系统路径名，因此首先，我们需要为我们的套接字文件定义一个路径并将其转换为UnixDomainSocketAddress：

```java
Path socketPath = Path
    .of(System.getProperty("user.home"))
    .resolve("tuyucheng.socket");
UnixDomainSocketAddress socketAddress = UnixDomainSocketAddress.of(socketPath);
```

在我们的示例中，我们在用户主目录下的tuyucheng.socket文件中创建套接字。

我们需要记住的一件事是在每次关闭我们的服务器后删除套接字文件：

```java
Files.deleteIfExists(socketPath);
```

不幸的是，它不会被自动删除，我们也无法将其重新用于进一步的连接。任何重用相同路径的尝试都将以异常结束，指出该地址已被使用：

```text
java.net.BindException: Address already in use
```

## 4. 接收消息

接下来我们可以做的是启动一个服务器，该服务器将从套接字通道接收消息。

首先，我们应该使用Unix协议创建一个服务器套接字通道：

```java
ServerSocketChannel serverChannel = ServerSocketChannel
    .open(StandardProtocolFamily.UNIX);
```

此外，我们需要将它与我们之前创建的套接字地址绑定：

```java
serverChannel.bind(socketAddress);
```

现在我们可以等待第一个客户端连接：

```java
SocketChannel channel = serverChannel.accept();
```

当客户端连接时，消息将进入字节缓冲区。要读取这些消息，我们需要构建一个无限循环来处理输入并将每条消息[打印到控制台](https://www.baeldung.com/java-console-input-output#writing-to-systemout)：

```java
while (true) {
    readSocketMessage(channel)
        .ifPresent(message -> System.out.printf("[Client message] %s", message));
    Thread.sleep(100);
}
```

在上面的示例中，方法readSocketMessage负责将套接字通道[缓冲区转换为String](https://www.baeldung.com/java-string-to-byte-array)：

```java
private Optional<String> readSocketMessage(SocketChannel channel) throws IOException {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    int bytesRead = channel.read(buffer);
    if (bytesRead < 0)
        return Optional.empty();

    byte[] bytes = new byte[bytesRead];
    buffer.flip();
    buffer.get(bytes);
    String message = new String(bytes);
    return Optional.of(message);
}
```

我们需要记住，服务器需要在客户端之前启动。在我们的示例中，它只能接受单个客户端连接。

## 5. 发送消息

发送消息比接收消息要简单一些。

我们唯一需要设置的是一个带有Unix协议的套接字通道，并将其连接到我们的套接字地址：

```java
SocketChannel channel = SocketChannel
    .open(StandardProtocolFamily.UNIX);
channel.connect(socketAddress);
```

现在我们可以准备一条消息：

```java
String message = "Hello from Tuyucheng Unix domain socket article";
```

将其转换为字节缓冲区：

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);
buffer.clear();
buffer.put(message.getBytes());
buffer.flip();
```

并将整个数据写入我们的套接字：

```java
while (buffer.hasRemaining()) {
    channel.write(buffer);
}
```

最后，服务器日志中会弹出以下输出：

```plaintext
[Client message] Hello from Tuyucheng Unix domain socket article!
```

## 6. 连接到数据库

Unix域套接字可用于连接数据库。许多流行的发行版，如[MongoDB](https://www.baeldung.com/java-mongodb)或PostgreSQL，都带有一个随时可以使用的默认配置。

例如，MongoDB在/tmp/mongodb-27017.sock中创建了一个Unix域套接字，我们可以在[MongoClient配置](https://www.baeldung.com/java-mongodb#1-make-a-connection-with-mongoclient)中直接使用它：

```java
MongoClient mongoClient = new MongoClient("/tmp/mongodb-27017.sock");
```

一个要求是将[jnr.unixsocket](https://mvnrepository.com/artifact/com.github.jnr/jnr-unixsocket)依赖项添加到我们的项目中：

```xml
<dependency>
    <groupId>com.github.jnr</groupId>
    <artifactId>jnr-unixsocket</artifactId>
    <version>0.38.13</version>
</dependency>
```

另一方面，PostgreSQL为我们提供了将Unix域套接字与[JDBC](https://www.baeldung.com/java-jdbc)标准一起使用的可能性。因此，我们只需要在创建连接时提供一个额外的socketFactory参数：

```java
String dbUrl = "jdbc:postgresql://databaseName?socketFactory=org.newsclub.net.unix.
    AFUNIXSocketFactory$FactoryArg&socketFactoryArg=/var/run/postgresql/.s.PGSQL.5432";
Connection connection = DriverManager
    .getConnection(dbUrl, "dbUsername", "dbPassword")
```

socketFactory参数应该指向一个扩展java.net.SocketFactory的类。此类将负责创建Unix域套接字而不是TCP/IP套接字。

在我们的示例中，我们使用了[junixsocket](https://mvnrepository.com/artifact/com.kohlschutter.junixsocket/junixsocket-core)库中的AFUNIXSocketFactory类：

```xml
<dependency>
    <groupId>com.kohlschutter.junixsocket</groupId>
    <artifactId>junixsocket-core</artifactId>
    <version>2.4.0</version>
</dependency>
```

## 7. 总结

在本教程中，我们学习了如何使用Unix域套接字通道。我们已经介绍了使用Unix域套接字发送和接收消息，并且学习了如何使用Unix域套接字连接数据库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-3)上获得。
