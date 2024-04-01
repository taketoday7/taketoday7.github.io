---
layout: post
title:  在Java中寻找空闲端口
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在我们的Java应用程序中启动套接字服务器时，java.net API要求我们指定一个要监听的可用端口号。端口号是必需的，以便TCP层可以识别传入数据的目标应用程序。

**明确指定端口号并不总是一个好的选择，因为应用程序可能已经占用了它**。这将导致我们的Java应用程序出现输入/输出异常。

**在本快速教程中，我们将了解如何检查特定端口状态以及如何使用自动分配的端口状态**。我们将研究如何使用纯Java和Spring框架完成此操作。我们还将研究其他一些服务器实现，例如嵌入式Tomcat和Jetty。

## 2. 检查端口状态

让我们看看如何使用java.net API检查特定端口是空闲还是已占用。

### 2.1 特定端口

我们将使用java.net API中的[ServerSocket](https://www.baeldung.com/a-guide-to-java-sockets)类来创建绑定到指定端口的服务器套接字。**在其构造函数中，ServerSocket接收一个明确的端口号。该类还实现了Closeable接口，因此可以在[try-with-resources](https://www.baeldung.com/java-try-with-resources)中使用它来自动关闭套接字并释放端口**：

```java
try (ServerSocket serverSocket = new ServerSocket(FREE_PORT_NUMBER)) {
    assertThat(serverSocket).isNotNull();
    assertThat(serverSocket.getLocalPort()).isEqualTo(FREE_PORT_NUMBER);
} catch (IOException e) {
    fail("Port is not available");
}
```

如果我们使用特定端口两次，或者它已经被另一个应用程序占用，ServerSocket构造函数将抛出IOException：

```java
try (ServerSocket serverSocket = new ServerSocket(FREE_PORT_NUMBER)) {
    new ServerSocket(FREE_PORT_NUMBER);
    fail("Same port cannot be used twice");
} catch (IOException e) {
    assertThat(e).hasMessageContaining("Address already in use");
}
```

### 2.2 端口范围

现在让我们检查如何利用抛出的IOException，使用给定端口号范围内的第一个空闲端口创建服务器套接字：

```java
for (int port : FREE_PORT_RANGE) {
    try (ServerSocket serverSocket = new ServerSocket(port)) {
        assertThat(serverSocket).isNotNull();
        assertThat(serverSocket.getLocalPort()).isEqualTo(port);
        return;
    } catch (IOException e) {
        assertThat(e).hasMessageContaining("Address already in use");
    }
}
fail("No free port in the range found");
```

## 3. 寻找空闲端口

使用明确的端口号并不总是一个好的选择，因此让我们研究一下自动分配空闲端口的可能性。

### 3.1 纯Java

我们可以在ServerSocket类构造函数中使用一个特殊的端口号0。因此，java.net API会自动为我们分配一个空闲端口：

```java
try (ServerSocket serverSocket = new ServerSocket(0)) {
    assertThat(serverSocket).isNotNull();
    assertThat(serverSocket.getLocalPort()).isGreaterThan(0);
} catch (IOException e) {
    fail("Port is not available");
}
```

### 3.2 Spring框架

Spring框架包含一个SocketUtils类，我们可以使用它来查找可用的空闲端口。它的内部实现使用ServerSocket类，如我们前面的示例所示：

```java
int port = SocketUtils.findAvailableTcpPort();
try (ServerSocket serverSocket = new ServerSocket(port)) {
    assertThat(serverSocket).isNotNull();
    assertThat(serverSocket.getLocalPort()).isEqualTo(port);
} catch (IOException e) {
    fail("Port is not available");
}
```

## 4. 其他服务器实现

现在让我们看一下其他一些流行的服务器实现。

### 4.1 Jetty

[Jetty](https://www.baeldung.com/jetty-embedded)是一种非常流行的用于Java应用程序的嵌入式服务器。**它会自动为我们分配一个空闲端口，除非我们通过ServerConnector类的setPort方法显式设置它**：

```java
Server jettyServer = new Server();
ServerConnector serverConnector = new ServerConnector(jettyServer);
jettyServer.addConnector(serverConnector);
try {
    jettyServer.start();
    assertThat(serverConnector.getLocalPort()).isGreaterThan(0);
} catch (Exception e) {
    fail("Failed to start Jetty server");
} finally {
    jettyServer.stop();
    jettyServer.destroy();
}
```

### 4.2 Tomcat

另一种流行的Java嵌入式服务器[Tomcat](https://www.baeldung.com/tomcat)的工作方式略有不同。我们可以通过Tomcat类的setPort方法指定一个明确的端口号。**如果我们提供的端口号为0，Tomcat将自动分配一个空闲端口**。但是，如果我们不设置任何端口号，Tomcat将使用其默认端口8080。请注意，默认Tomcat端口可能被其他应用程序占用：

```java
Tomcat tomcatServer = new Tomcat();
tomcatServer.setPort(0);
try {
    tomcatServer.start();
    assertThat(tomcatServer.getConnector().getLocalPort()).isGreaterThan(0);
} catch (LifecycleException e) {
    fail("Failed to start Tomcat server");
} finally {
    tomcatServer.stop();
    tomcatServer.destroy();
}
```

## 5. 总结

**在本文中，我们探讨了如何检查特定端口状态。我们还介绍了从一系列端口号中查找空闲端口，并解释了如何使用自动分配的空闲端口**。

在示例中，我们介绍了来自java.net API和其他流行服务器实现(包括Jetty和Tomcat)的基本ServerSocket类。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-3)上获得。
