---
layout: post
title:  Java HttpClient连接管理
category: java
copyright: java
excerpt: Java HttpClient
---

## 1. 概述

我们的应用程序通常需要某种形式的连接管理来更好地利用资源。

**在本教程中，我们将了解在[Java 11的HttpClient](https://www.baeldung.com/java-9-http-client)中我们可以使用哪些连接管理支持**。我们将介绍使用系统属性来设置池大小和默认超时，以及使用WireMock来模拟不同的主机。

## 2. Java HttpClient的连接池

**Java 11 HttpClient有一个内部连接池**。默认情况下，它的大小是无限的。

让我们通过构建一个可用于发送请求的HttpClient来查看连接池的运行情况：

```java
HttpClient client = HttpClient.newHttpClient();
```

## 3. 目标服务器

我们将使用[WireMock](https://www.baeldung.com/introduction-to-wiremock)服务器作为我们的模拟主机，这使我们能够使用Jetty的调试日志记录来跟踪正在建立的连接。

首先，让我们看看HttpClient创建并重用缓存连接。让我们通过在动态端口上启动WireMock服务器来启动我们的模拟主机：

```java
WireMockServer server = new WireMockServer(WireMockConfiguration
    .options()
    .dynamicPort());
```

在我们的setup()方法中，让我们启动服务器并将其配置为以200响应任何请求：

```java
firstServer.start();
server.stubFor(WireMock
    .get(WireMock.anyUrl())
    .willReturn(WireMock
        .aResponse()
        .withStatus(200)));
```

接下来，让我们创建要发送的HttpRequest，配置为指向我们的WireMock端点：

```java
HttpRequest getRequest = HttpRequest.newBuilder()
    .uri(create("http://localhost:" + server.port() + "/first";))
    .build();
```

现在我们有一个客户端和一个要发送到的服务器，让我们发送我们的请求：

```java
HttpResponse<String> response = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
```

为了简单起见，我们在HttpResponse.BodyHandler内部类中使用了ofString工厂方法来创建我们的String响应处理程序。

当我们运行这段代码时，我们没有看到太多的事情发生，因此让我们打开一些调试来确定我们的连接是否真的建立和重用了。

## 4. Jetty调试日志配置

鉴于JDK 11的ConnectionPool类中日志记录的稀疏性，我们需要外部日志记录来帮助我们查看连接何时被重用或创建。

**因此，让我们通过在类路径中创建一个jetty-logging.properties来启用Jetty的调试日志记录**：

```properties
org.eclipse.jetty.util.log.class=org.eclipse.jetty.util.log.StrErrLog
org.eclipse.jetty.LEVEL=DEBUG
jetty.logs=logs
```

在这里，我们将Jetty的日志记录级别设置为DEBUG并将其配置为写入错误输出流。

创建新连接时，Jetty会记录一条“New HTTP Connection”消息：

```text
DBUG:oejs.HttpConnection:qtp2037764568-17-selector-ServerConnectorManager@34b9f960/0: New HTTP Connection HttpConnection@ba7665b{IDLE}
```

我们可以在运行测试时查找这些消息以确认连接创建活动。

## 5. 连接池-建立与复用

现在我们有了我们的客户端，以及一个在请求新连接时记录的服务器，我们准备运行一些测试。

首先，让我们验证HttpClient确实使用了内部连接池。如果有正在使用的连接池，我们只会看到一条“New HTTP Connection”消息。

因此，让我们向同一服务器发出两个请求，看看记录了多少新的连接消息：

```java
HttpResponse<String> firstResponse = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
HttpResponse<String> secondResponse = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
```

现在让我们检查“New HTTP Connection”消息的日志输出：

```text
DBUG:oejs.HttpConnection:qtp2037764568-17-selector-ServerConnectorManager@34b9f960/0: New HTTP Connection HttpConnection@ba7665b{IDLE}
```

我们看到只记录了一个新的连接请求。这告诉我们，我们发出的第二个请求不需要创建新连接。

**我们的客户端在第一次调用时建立了一个连接并将其放入池中，允许第二次调用重用相同的连接**。

那么，现在让我们看看连接池是客户端独有的还是跨客户端共享的。

让我们创建第二个客户端来检查：

```java
HttpClient secondClient = HttpClient.newHttpClient();
```

让我们从每个客户向同一服务器发送请求：

```java
HttpResponse<String> firstResponse = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
HttpResponse<String> secondResponse = secondClient.send(getRequest, HttpResponse.BodyHandlers.ofString());
```

当我们检查日志输出时，我们看到创建的不是一个而是两个新连接：

```text
DBUG:oejs.HttpConnection:qtp729218894-17-selector-ServerConnectorManager@51acdf2e/0: New HTTP Connection HttpConnection@3cc85dbb{IDLE}
DBUG:oejs.HttpConnection:qtp729218894-21-selector-ServerConnectorManager@51acdf2e/1: New HTTP Connection HttpConnection@6062141{IDLE}
```

**我们的第二个客户端导致创建到同一目的地的新连接。由此，我们可以推断出每个客户端都有一个连接池**。

## 6. 控制连接池大小

现在我们已经看到了连接的创建和重用，让我们看看如何控制池的大小。

**JDK 11 ConnectionPool在初始化时检查jdk.httpclient.connectionPoolSize系统属性，默认为0(无限制)**。

我们可以将系统属性设置为JVM参数或以编程方式设置。但是，此属性只会在初始化时读取，因此我们将使用JVM参数来确保在第一次建立任何连接时设置该值。

首先，让我们运行一个测试，首先调用一个服务器，然后是另一个，然后再次返回到第一个：

```java
HttpResponse<String> firstResponse = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
HttpResponse<String> secondResponse = client.send(secondGet, HttpResponse.BodyHandlers.ofString());
HttpResponse<String> thirdResponse = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
```

我们在日志中只看到两个连接请求，因为我们的池仍然包含我们第一个请求中建立的连接：

```text
DBUG:oejs.HttpConnection:qtp2037764568-17-selector-ServerConnectorManager@34b9f960/0: New HTTP Connection HttpConnection@1af88cae{IDLE}
DBUG:oejs.HttpConnection:qtp1932332324-26-selector-ServerConnectorManager@13d4992d/0: New HTTP Connection HttpConnection@71c7d4f{IDLE}
```

现在让我们通过将池大小设置为1来更改此默认行为：

```shell
-Djdk.httpclient.connectionPoolSize=1
```

我们通常不会将池大小设置为1，但在本例中，这样做使我们能够更快地达到最大池大小。

当我们使用我们的属性集再次运行测试时，我们看到创建了三个连接：

```text
DBUG:oejs.HttpConnection:qtp2104973502-22-selector-ServerConnectorManager@48b67364/0: New HTTP Connection HttpConnection@3da6a47f{IDLE}
DBUG:oejs.HttpConnection:qtp351877391-26-selector-ServerConnectorManager@3b8f0a79/0: New HTTP Connection HttpConnection@20b59486{IDLE}
DBUG:oejs.HttpConnection:qtp2104973502-18-selector-ServerConnectorManager@48b67364/1: New HTTP Connection HttpConnection@599a00c1{IDLE}
```

我们的属性达到了我们预期的效果！连接池大小仅为1，当调用第二个服务器时，第一个连接将从池中清除。因此，当我们的第三次调用返回到第一个服务器时，池中不再有条目，我们必须创建一个新的第三个连接。

## 7. 连接保活超时

建立连接后，它将保留在我们的池中以供重用。**如果连接闲置时间过长，那么它将从我们的连接池中清除**。

**JDK 11 ConnectionPool在初始化时检查jdk.httpclient.keepalive.timeout系统属性，默认为1200秒(20分钟)**。

请注意，keepalive超时系统属性不同于[HttpClient的connectTimeout](https://www.baeldung.com/java-httpclient-timeout)方法。连接超时决定了我们要等待多长时间才能建立新的连接，而**保持连接超时决定了连接建立后要保持多长时间**。

由于20分钟在现代架构中是一个很长的时间，JDK 20 Build26将默认值减少到30秒。

让我们通过JVM参数设置我们的keepalive系统属性，将它减少到2秒来测试这个设置：

```shell
-Djdk.httpclient.keepalive.timeout=2
```

现在让我们运行一个测试，该测试休眠足够的时间以便在调用之间断开连接：

```java
HttpResponse<String> firstResponse = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
Thread.sleep(3000);
HttpResponse<String> secondResponse = client.send(getRequest, HttpResponse.BodyHandlers.ofString());
```

正如预期的那样，我们看到为两个请求创建了一个新连接，因为第一个连接在2秒后断开。

```text
DBUG:oejs.HttpConnection:qtp729218894-17-selector-ServerConnectorManager@51acdf2e/0: New HTTP Connection HttpConnection@3cc85dbb{IDLE}
DBUG:oejs.HttpConnection:qtp729218894-21-selector-ServerConnectorManager@51acdf2e/1: New HTTP Connection HttpConnection@6062141{IDLE}
```

## 8. HttpClient的增强

**自JDK 11以来，HttpClient得到了进一步发展，例如各种[网络日志记录改进](https://docs.oracle.com/en/java/javase/19/core/java-networking.html)**。当我们针对Java 19运行这些测试时，我们可以探索HttpClient的内部日志来监控网络活动，而不是依赖WireMock的Jetty日志记录。还有一些关于[如何使用客户端的有用方法](https://openjdk.org/groups/net/httpclient/recipes.html)。

由于**HttpClient还支持使用连接多路复用的HTTP/2(H2)连接**，因此我们的应用程序可能不需要使用那么多连接。因此，在JDK 20 build25中，专门为H2池引入了一些额外的系统属性：

-   jdk.httpclient.keepalivetimeout.h2：设置此属性以控制H2连接的保持连接超时
-   jdk.httpclient.maxstreams：设置此属性以控制每个HTTP连接允许的最大H2流数(默认为100)

## 9. 总结

在本教程中，我们了解了Java HttpClient如何重用其内部连接池中的连接。我们使用带有Jetty日志记录的Wiremock来向我们展示新的连接请求何时发出。接下来，我们学习了如何控制连接池大小以及达到池限制时的效果。我们还学习了如何配置清除空闲连接的时间。

最后，我们研究了在更新的Java版本中所做的一些网络更改。

当我们使用Java 11之前的版本或需要不同的功能时，[Apache HttpClient4](https://www.baeldung.com/httpclient-connection-management)教程演示了Java 11客户端的替代方案。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-httpclient)上获得。