---
layout: post
title:  Java HttpClient超时
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在本教程中，我们将展示如何使用Java 11及更高版本提供的新Java HTTP客户端和Java设置超时。

## 2. 配置超时

首先，我们需要设置一个HttpClient以便能够发出HTTP请求：

```java
private static HttpClient getHttpClientWithTimeout(int seconds) {
    return HttpClient.newBuilder()
        .connectTimeout(Duration.ofSeconds(seconds))
        .build();
}
```

上面，我们创建了一个方法，该方法返回一个配置有超时定义为参数的HttpClient。然后，**我们使用[构建器设计模式](https://www.baeldung.com/creational-design-patterns#builder)实例化一个HttpClient并使用connectTimeout方法配置超时**。此外，使用静态方法ofSeconds，我们创建了一个Duration对象的实例，它以秒为单位定义超时。

之后，我们检查HttpClient超时是否配置正确：

```java
httpClient.connectTimeout().map(Duration::toSeconds)
    .ifPresent(sec -> System.out.println("Timeout in seconds: " + sec));
```

因此，我们使用connectTimeout方法来获取超时时间。结果，它返回一个Duration的Optional，我们将其映射到秒。

## 3. 处理超时

此外，我们需要创建一个HttpRequest对象，我们的客户端将使用它来发出HTTP请求：

```java
HttpRequest httpRequest = HttpRequest.newBuilder()
    .uri(URI.create("http://10.255.255.1")).GET().build();
```

为了模拟超时，我们调用一个不可路由的IP地址。换句话说，所有TCP数据包都会丢弃并在之前配置的预定义持续时间后强制超时。

现在，让我们更深入地了解如何处理超时。

### 3.1 处理同步调用超时

例如，要进行同步调用，请使用send方法：

```java
HttpConnectTimeoutException thrown = assertThrows(
    HttpConnectTimeoutException.class,
    () -> httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString()),
    "Expected send() to throw HttpConnectTimeoutException, but it didn't");
assertTrue(thrown.getMessage().contains("timed out"));
```

**同步调用强制捕获扩展了IOException的HttpConnectTimeoutException**。因此，在上面的测试中，我们期望HttpConnectTimeoutException带有错误消息。

### 3.2 处理异步调用超时

同样，要进行异步调用，请使用sendAsync方法：

```java
CompletableFuture<String> completableFuture = httpClient.sendAsync(httpRequest, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .exceptionally(Throwable::getMessage);
String response = completableFuture.get(5, TimeUnit.SECONDS);
assertTrue(response.contains("timed out"));
```

上面对sendAsync的调用返回一个CompletableFuture<HttpResponse\>。因此，我们需要定义如何在功能上处理响应。详细地说，当没有错误发生时，我们从响应中获取主体。否则，我们会从Throwable中获取错误消息。最后，我们通过等待最多5秒从CompletableFuture获得结果。同样，这个请求会在3秒后抛出一个HttpConnectTimeoutException，正如我们所期望的那样。

## 4. 在请求级别配置超时

上面，我们为同步和异步调用重用了相同的客户端实例。但是，我们可能希望针对每个请求以不同方式处理超时。同样，我们可以为单个请求设置超时：

```java
HttpRequest httpRequest = HttpRequest.newBuilder()
    .uri(URI.create("http://10.255.255.1"))
    .timeout(Duration.ofSeconds(1))
    .GET()
    .build();
```

同样，我们使用timeout方法来设置此请求的超时时间。在这里，我们为这个请求配置了1秒的超时。

**客户端和请求之间的最短持续时间设置请求的超时时间**。

## 5. 总结

在本文中，我们使用新的Java HTTP客户端成功配置了超时，并在超时溢出时优雅地处理请求。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-3)上获得。
