---
layout: post
title:  使用Java HttpClient发送POST请求
category: java
copyright: java
excerpt: Java HttpClient
---

## 1. 概述

[Java HttpClient API](https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html)是在Java 11中引入的。**该API实现了最新HTTP标准的客户端**。它支持HTTP/1.1和HTTP/2，同步和异步编程模型。

我们可以使用它来发送HTTP请求并检索它们的响应。在Java 11之前，我们不得不依赖基本的[URLConnection](https://www.baeldung.com/java-http-request)实现或第三方库，例如[Apache HttpClient](https://www.baeldung.com/httpclient-guide)。

在本教程中，我们将介绍使用Java HttpClient发送POST请求。我们将展示如何发送同步和异步POST请求，以及并发POST请求。此外，我们将检查如何向POST请求添加身份验证参数和JSON主体。

最后，我们将看到如何上传文件和提交表单数据。因此，我们将涵盖大多数常见用例。

## 2. 准备POST请求

在发送HTTP请求之前，我们首先需要创建一个HttpClient实例。

**可以使用newBuilder方法从其构建器配置和创建HttpClient实例**。否则，如果不需要配置，我们可以使用newHttpClient实用程序方法来创建默认客户端：

```java
HttpClient client = HttpClient.newHttpClient();
```

HttpClient默认使用HTTP/2。如果服务器不支持HTTP/2，它也会自动降级到HTTP/1.1。

现在我们已准备好从其构建器创建HttpRequest的实例。稍后我们将使用客户端实例发送此请求。POST请求的最小参数是服务器URL、请求方法和正文：

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(serviceUrl))
    .POST(HttpRequest.BodyPublishers.noBody())
    .build();
```

请求正文需要通过BodyPublisher类提供。它是一个响应流发布者，可以按需发布请求主体流。在我们的示例中，我们使用了一个不发送请求主体的主体发布者。

## 3. 发送POST请求

现在我们已经准备好POST请求，让我们看看发送它的不同选项。

### 3.1 同步

我们可以使用此默认send方法发送准备好的请求。**此方法将阻塞我们的代码，直到收到响应为止**：

```java
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString())
```

BodyHandlers实用程序实现各种有用的处理程序，例如将响应主体作为字符串处理或将响应主体流式传输到文件。收到响应后，HttpResponse对象将包含响应状态、标头和正文：

```java
assertThat(response.statusCode())
    .isEqualTo(200);
assertThat(response.body())
    .isEqualTo("{\"message\":\"ok\"}");
```

### 3.2 异步

我们可以使用sendAsync方法异步发送上一示例中的相同请求。**这个方法不会阻塞我们的代码，而是会立即返回一个[CompletableFuture](https://www.baeldung.com/java-completablefuture)实例**：

```java
CompletableFuture<HttpResponse<String>> futureResponse = client.sendAsync(request, HttpResponse.BodyHandlers.ofString());
```

CompletableFuture在HttpResponse可用后完成：

```java
HttpResponse<String> response = futureResponse.get();
assertThat(response.statusCode()).isEqualTo(200);
assertThat(response.body()).isEqualTo("{\"message\":\"ok\"}");
```

### 3.3 并发

我们可以将[Stream](https://www.baeldung.com/java-8-streams)与CompletableFutures结合起来，以便**同时发出多个请求并等待它们的响应**：

```java
List<CompletableFuture<HttpResponse<String>>> completableFutures = serviceUrls.stream()
    .map(URI::create)
    .map(HttpRequest::newBuilder)
    .map(builder -> builder.POST(HttpRequest.BodyPublishers.noBody()))
    .map(HttpRequest.Builder::build)
    .map(request -> client.sendAsync(request, HttpResponse.BodyHandlers.ofString()))
    .collect(Collectors.toList());
```

现在，让我们等待所有请求完成，以便我们可以一次处理所有响应：

```java
CompletableFuture<List<HttpResponse<String>>> combinedFutures = CompletableFuture
    .allOf(completableFutures.toArray(new CompletableFuture[0]))
    .thenApply(future ->
        completableFutures.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList()));
```

当我们使用allOf和join方法组合所有响应时，我们得到一个新的CompletableFuture来保存我们的响应：

```java
List<HttpResponse<String>> responses = combinedFutures.get();
responses.forEach((response) -> {
    assertThat(response.statusCode()).isEqualTo(200);
    assertThat(response.body()).isEqualTo("{\"message\":\"ok\"}");
});
```

## 4. 添加认证参数

**我们可以在客户端级别设置一个身份验证器，用于对所有请求进行HTTP身份验证**：

```java
HttpClient client = HttpClient.newBuilder()
    .authenticator(new Authenticator() {
        @Override
        protected PasswordAuthentication getPasswordAuthentication() {
            return new PasswordAuthentication(
                "tuyucheng",
                "123456".toCharArray());
            }
    })
    .build();
```

但是，HttpClient不会发送基本凭据，直到使用来自服务器的WWW-Authenticate标头对它们进行质询。

要绕过这一点，我们始终可以手动创建和发送基本授权标头：

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(serviceUrl))
    .POST(HttpRequest.BodyPublishers.noBody())
    .header("Authorization", "Basic " + Base64.getEncoder().encodeToString(("tuyucheng:123456").getBytes()))
    .build();
```

## 5. 添加正文

到目前为止，在示例中，我们还没有向POST请求添加任何正文。但是，POST方法通常用于通过请求体向服务器发送数据。

### 5.1 JSON正文

BodyPublishers实用程序实现了各种有用的发布者，例如从字符串或文件发布请求正文。我们可以**将JSON数据发布为String**，使用UTF-8字符集转换：

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(serviceUrl))
    .POST(HttpRequest.BodyPublishers.ofString("{\"action\":\"hello\"}"))
    .build();
```

### 5.2 上传文件

让我们创建一个可用于通过HttpClient上传的[临时文件](https://www.baeldung.com/junit-5-temporary-directory)：

```java
Path file = tempDir.resolve("temp.txt");
List<String> lines = Arrays.asList("1", "2", "3");
Files.write(file, lines);
```

**HttpClient提供了一个单独的方法BodyPublishers.ofFile**，用于将文件添加到POST主体。我们可以简单地将我们的临时文件添加为方法参数，API会处理剩下的事情：

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(serviceUrl))
    .POST(HttpRequest.BodyPublishers.ofFile(file))
    .build();
```

### 5.3 提交表单

与文件相反，HttpClient不提供用于发布表单数据的单独方法。因此，我们将**再次需要使用BodyPublishers.ofString方法**：

```java
Map<String, String> formData = new HashMap<>();
formData.put("username", "tuyucheng");
formData.put("message", "hello");

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(serviceUrl))
    .POST(HttpRequest.BodyPublishers.ofString(getFormDataAsString(formData)))
    .build();
```

但是，我们需要使用自定义实现将表单数据从Map转换为String：

```java
private static String getFormDataAsString(Map<String, String> formData) {
    StringBuilder formBodyBuilder = new StringBuilder();
    for (Map.Entry<String, String> singleEntry : formData.entrySet()) {
        if (formBodyBuilder.length() > 0) {
            formBodyBuilder.append("&");
        }
        formBodyBuilder.append(URLEncoder.encode(singleEntry.getKey(), StandardCharsets.UTF_8));
        formBodyBuilder.append("=");
        formBodyBuilder.append(URLEncoder.encode(singleEntry.getValue(), StandardCharsets.UTF_8));
    }
    return formBodyBuilder.toString();
}
```

## 6. 总结

在本文中，我们探讨了使用Java 11中引入的Java HttpClient API发送POST请求。

我们学习了如何创建HttpClient实例并准备POST请求。我们看到了如何同步、异步和并发发送准备好的请求。接下来，我们还看到了如何添加基本身份验证参数。

最后，我们研究了向POST请求添加正文。我们介绍了JSON负载、上传文件和提交表单数据。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-httpclient)上获得。