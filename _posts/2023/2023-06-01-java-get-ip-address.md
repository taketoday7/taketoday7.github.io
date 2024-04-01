---
layout: post
title:  使用Java获取当前机器的IP地址
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

[IP地址](https://www.baeldung.com/cs/ipv4-vs-ipv6)或互联网协议地址唯一标识互联网上的设备。因此，了解运行我们应用程序的设备的身份是某些应用程序的关键部分。

在本教程中，我们将检查使用Java检索计算机IP地址的各种方法。

## 2. 查找本地IP地址

首先，让我们来看一些获取本机本地IPv4地址的方法。

### 2.1 Java Net库

此方法使用Java Net库建立UDP连接：

```java
try (final DatagramSocket datagramSocket = new DatagramSocket()) {
    datagramSocket.connect(InetAddress.getByName("8.8.8.8"), 12345);
    return datagramSocket.getLocalAddress().getHostAddress();
}
```

在这里，为简单起见，我们使用Google的主DNS作为我们的目标主机并提供IP地址8.8.8.8。Java Net库此时仅检查地址格式的有效性，因此地址本身可能无法访问。此外，我们使用随机端口12345通过socket.connect()方法创建UDP连接。在底层，它设置发送和接收数据所需的所有变量，包括机器的本地地址，而不实际向目标主机发送任何请求。

虽然此解决方案在Linux和Windows计算机上运行良好，但在macOS上存在问题并且不会返回预期的IP地址。

### 2.2 带套接字连接的本地地址

或者，**我们可以通过可靠的互联网连接使用套接字连接来查找IP地址**：

```java
try (Socket socket = new Socket()) {
    socket.connect(new InetSocketAddress("google.com", 80));
    return socket.getLocalAddress().getHostAddress();
}
```

在这里，同样为了简单起见，我们使用google.com和端口80上的连接来获取主机地址。我们可以使用任何其他URL来创建套接字连接，只要它是可访问的。

### 2.3 复杂网络情况的注意事项

上面列出的方法在简单的网络情况下效果很好。但是，在计算机具有更多网络接口的情况下，行为可能无法预测。

换句话说，**从上述函数返回的IP地址将是机器上首选网络接口的地址。因此，它可能与我们期望的不同**。对于特定需求，我们可以[查找连接到服务器的客户端的IP地址](https://www.baeldung.com/java-client-get-ip-address)。

## 3. 查找公共IP地址

类似于本地IP地址，我们可能想知道当前机器的公网IP地址。**公共IP地址是可从互联网访问的IPv4地址**。此外，它可能无法唯一标识查找地址的机器。例如，同一路由器下的多台主机具有相同的公网IP地址。

简单地说，我们可以连接到Amazon AWS checkip.amazonaws.com URL并读取响应：

```java
String urlString = "http://checkip.amazonaws.com/";
URL url = new URL(urlString);
try (BufferedReader br = new BufferedReader(new InputStreamReader(url.openStream()))) {
    return br.readLine();
}
```

这在大多数情况下都很有效。但是，我们明确依赖于无法保证可靠性的外部来源。因此，作为回退，我们可以使用这些URL中的任何一个来检索公共IP地址：

-   https://ipv4.icanhazip.com/
-   http://myexternalip.com/raw
-   http://ipecho.net/plain

## 4. 总结

在本文中，我们学习了如何查找当前机器的IP地址以及如何使用Java检索它们。我们还研究了检查本地和公共IP地址的各种方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-3)上获得。
