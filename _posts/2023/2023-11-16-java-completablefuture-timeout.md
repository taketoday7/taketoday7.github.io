---
layout: post
title:  如何为CompletableFuture管理超时
category: java-concurrency
copyright: java-concurrency
excerpt: Java CompletableFuture
---

## 1. 概述

当我们构建依赖于其他服务的服务时，我们经常需要处理依赖服务响应太慢的情况。

如果我们使用[CompletableFuture](https://www.baeldung.com/java-completablefuture)来异步管理对依赖项的调用，它的超时功能使我们能够设置结果的最大等待时间。如果预期结果没有在指定时间内到达，我们可以采取措施，例如提供默认值，以防止我们的应用程序陷入冗长的过程。

在本文中，我们将讨论在CompletableFuture中管理超时的三种不同方法。

## 2. 管理超时

想象一下一个电子商务应用程序需要调用外部服务来获取特殊产品优惠，**我们可以使用带有超时设置的CompletableFuture来保持响应能力**。如果服务未能足够快地响应，这可能会引发错误或提供默认值。

例如，在本例中，假设我们要向返回PRODUCT_OFFERS的API发出请求。我们称之为fetchProductData()，我们可以用CompletableFuture包装它，这样我们就可以处理超时：

```java
private CompletableFuture<String> fetchProductData() {
    return CompletableFuture.supplyAsync(() -> {
        try {
            URL url = new URL("http://localhost:8080/api/dummy");
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();

            try (BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()))) {
                String inputLine;
                StringBuffer response = new StringBuffer();

                while ((inputLine = in.readLine()) != null) {
                    response.append(inputLine);
                }

                return response.toString();
            } finally {
                connection.disconnect();
            }
        } catch (IOException e) {
            return "";
        }
    });
}
```

要使用[WireMock](https://www.baeldung.com/introduction-to-wiremock)测试超时，我们可以按照[WireMock使用指南](https://www.baeldung.com/introduction-to-wiremock)轻松配置模拟服务器以实现超时。假设典型互联网连接上合理的网页加载时间为1000毫秒，因此我们将DEFAULT_TIMEOUT设置为该值：

```java
private static final int DEFAULT_TIMEOUT = 1000; // 1 seconds
```

然后，我们将创建一个wireMockServer，它给出PRODUCT_OFFERS的正文响应，并设置5000毫秒或5秒的延迟，确保该值超过DEFAULT_TIMEOUT以确保发生超时：

```java
stubFor(get(urlEqualTo("/api/dummy"))
    .willReturn(aResponse()
        .withFixedDelay(5000) // must be > DEFAULT_TIMEOUT for a timeout to occur.
        .withBody(PRODUCT_OFFERS)));
```

## 3. 使用completeOnTimeout()

如果任务未在指定时间内完成，则completeOnTimeout()方法将使用默认值解析CompletableFuture。

通过这个方法，**我们可以设置超时时返回的默认值<T\>**。此方法返回调用此方法的CompletableFuture。

在此示例中，我们默认为DEFAULT_PRODUCT：

```java
CompletableFuture<Integer> productDataFuture = fetchProductData();
productDataFuture.completeOnTimeout(DEFAULT_PRODUCT, DEFAULT_TIMEOUT, TimeUnit.MILLISECONDS);
assertEquals(DEFAULT_PRODUCT, productDataFuture.get());
```

**如果我们的目标是即使在请求期间失败或超时的情况下结果仍然有意义，那么这种方法是合适的**。

例如在电商场景中，展示商品促销时，如果获取特价促销商品数据失败或超过超时时间，系统将显示默认商品。

## 4. 使用orTimeout()

如果Future没有在特定时间内完成，我们可以使用orTimeout()来增强CompletableFuture的超时处理行为。

**此方法返回应用此方法的相同CompletableFuture，并在超时的情况下抛出TimeoutException**。

然后，为了测试这个方法，我们应该使用assertThrows()来证明引发了异常：

```java
CompletableFuture<Integer> productDataFuture = fetchProductData();
productDataFuture.orTimeout(DEFAULT_TIMEOUT, TimeUnit.MILLISECONDS);
assertThrows(ExecutionException.class, productDataFuture::get);
```

如果我们的优先级是响应能力或耗时的任务，并且我们希望在发生超时时提供快速操作，那么这是一个合适的方法。

但是，为了获得良好的性能，需要正确处理这些异常，因为此方法会显式引发异常。

此外，该方法适用于各种场景，例如管理网络连接、处理IO操作、处理实时数据和管理队列。

## 5. 使用completeExceptionally()

CompletableFuture类的completeExceptionally()方法让我们可以通过特定的Exception异常地完成Future，对结果检索方法(如get()和join())的后续调用将引发指定的异常。

如果方法调用导致CompletableFuture转换为已完成状态，则此方法返回true。否则，它返回false。

在这里，我们将使用[ScheduledExecutorService](https://www.baeldung.com/java-executor-service-tutorial)，它是Java中的一个接口，用于在特定时间或延迟时安排和管理任务执行。它在并发环境中调度重复任务、处理超时和管理错误方面提供了灵活性：

```java
ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);
//...
CompletableFuture<Integer> productDataFuture = fetchProductData();
executorService.schedule(() -> productDataFuture.completeExceptionally(
    new TimeoutException("Timeout occurred")), DEFAULT_TIMEOUT, TimeUnit.MILLISECONDS);
assertThrows(ExecutionException.class, productDataFuture::get);
```

如果我们需要处理TimeoutException以及其他异常，或者我们想要自定义或特定它，也许这是一个合适的方法。我们通常使用这种方法来处理失败的数据验证、致命错误或任务没有默认值的情况。

## 6. 比较：completeOnTimeout()、orTimeout()、completeExceptionally()

通过所有这些方法，我们可以管理和控制CompletableFuture在不同场景中的行为，特别是在处理需要计时和处理超时或错误的异步操作时。

让我们比较一下completeOnTimeout()、orTimeout()和completeExceptionally()的优缺点：

| 方法            | 优点                                         | 缺点                 |
| ------------------- |--------------------------------------------| ------------------------ |
| completeOnTimeout()  | 如果长时间运行的任务花费太长时间，则允许替换默认结果，对于避免超时而不引发异常很有用 | 没有明确标记发生超时     |
| orTimeout()     | 发生超时时显式生成TimeoutException，可以以特定方式处理超时      | 不提供替换默认结果的选项 |
| completeExceptionally() | 允许使用自定义异常显式标记结果，对于指示异步操作失败很有用              | 比管理超时更通用         |

## 7. 总结

在本文中，我们研究了在CompletableFuture内的异步进程中响应超时的三种不同方法。

在选择方法时，我们应该考虑管理长时间运行的任务的需求。我们应该在默认值之间做出决定，指示具有特定异常的异步操作超时。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-simple)上获得。