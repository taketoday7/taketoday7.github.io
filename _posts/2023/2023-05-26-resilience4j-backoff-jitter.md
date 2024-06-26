---
layout: post
title:  使用指数退避和抖动进行更好的重试
category: designpattern
copyright: designpattern
excerpt: Design Pattern
---

## 1. 概述

在本教程中，我们将探讨如何使用两种不同的策略来改进客户端重试：指数退避和抖动。

## 2. 重试

在分布式系统中，众多组件之间的网络通信随时可能失败，**客户端应用程序通过实施重试来处理这些失败**。

假设我们有一个调用远程服务的客户端应用程序-PingPongService。

```java
interface PingPongService {
    String call(String ping) throws PingPongServiceException;
}
```

如果PingPongService返回PingPongServiceException，客户端应用程序必须重试。在以下部分中，我们将介绍实现客户端重试的方法。

## 3. Resilience4j重试

对于我们的示例，我们将使用[Resilience4j]()库，尤其是它的[retry](https://resilience4j.readme.io/docs/retry)模块，我们需要将resilience4j-retry模块添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-retry</artifactId>
</dependency>
```

有关使用重试的复习，请不要忘记阅读我们的[Resilience4j指南]()。

## 4. 指数退避

客户端应用程序必须负责任地实现重试，**当客户端重试失败的调用而不等待时，它们可能会使系统不堪重负**，并导致已经处于困境中的服务进一步降级。

指数退避是处理失败网络调用重试的常用策略。简而言之，**客户端在连续重试之间等待的时间间隔会逐渐延长**：

```java
wait_interval = base * multiplier^n
```

-   base为初始间隔，即等待第一次重试
-   n是已发生的失败次数
-   multiplier是一个任意的乘数，可以替换为任何合适的值

通过这种方法，我们为系统提供了一个喘息的空间，以便从间歇性故障甚至更严重的问题中恢复。

我们可以通过配置接收initialInterval和multiplier的IntervalFunction在Resilience4j retry中使用指数退避算法。

IntervalFunction被重试机制用作睡眠函数：

```java
IntervalFunction intervalFn = IntervalFunction.ofExponentialBackoff(INITIAL_INTERVAL, MULTIPLIER);

RetryConfig retryConfig = RetryConfig.custom()
    .maxAttempts(MAX_RETRIES)
    .intervalFunction(intervalFn)
    .build();
Retry retry = Retry.of("pingpong", retryConfig);

Function<String, String> pingPongFn = Retry
        .decorateFunction(retry, ping -> service.call(ping));
pingPongFn.apply("Hello");
```

让我们模拟一个真实的场景，并假设我们有多个客户端同时调用PingPongService：

```java
ExecutorService executors = newFixedThreadPool(NUM_CONCURRENT_CLIENTS);
List<Callable> tasks = nCopies(NUM_CONCURRENT_CLIENTS, () -> pingPongFn.apply("Hello"));
executors.invokeAll(tasks);
```

让我们看一下NUM_CONCURRENT_CLIENTS等于4的远程调用日志：

```shell
[thread-1] At 00:37:42.756
[thread-2] At 00:37:42.756
[thread-3] At 00:37:42.756
[thread-4] At 00:37:42.756

[thread-2] At 00:37:43.802
[thread-4] At 00:37:43.802
[thread-1] At 00:37:43.802
[thread-3] At 00:37:43.802

[thread-2] At 00:37:45.803
[thread-1] At 00:37:45.803
[thread-4] At 00:37:45.803
[thread-3] At 00:37:45.803

[thread-2] At 00:37:49.808
[thread-3] At 00:37:49.808
[thread-4] At 00:37:49.808
[thread-1] At 00:37:49.808
```

我们可以在这里看到一个清晰的模式-客户端等待呈指数增长的间隔，但它们在每次重试(冲突)时都在精确的同一时间调用远程服务。

![](/assets/images/2023/designpattern/resilience4jbackoffjitter01.png)

**我们只解决了问题的一部分-我们不再通过重试来打击远程服务，但我们没有随着时间的推移分散工作量，而是在工作时间段穿插了更多的空闲时间。这种行为类似于[Thundering Herd Problem](https://en.wikipedia.org/wiki/Thundering_herd_problem)**。

## 5. 引入抖动

在我们之前的方法中，客户端等待的时间逐渐变长，但仍处于同步状态。**添加抖动提供了一种中断客户端同步从而避免冲突的方法**。在这种方法中，我们为等待间隔添加了随机性。

```java
wait_interval = (base * 2^n) +/- (random_interval)
```

其中，增加(或减去)random_interval以中断客户端之间的同步。

我们不会深入探讨计算随机间隔的机制，但随机化必须将尖峰间隔开，以便更平滑地分配客户端调用。

我们可以在Resilience4j重试中使用带有抖动的指数退避，方法是配置一个指数随机退避IntervalFunction，该函数也接收randomizationFactor：

```java
IntervalFunction intervalFn = IntervalFunction.ofExponentialRandomBackoff(INITIAL_INTERVAL, MULTIPLIER, RANDOMIZATION_FACTOR);
```

回到我们的真实场景，看看引入抖动的远程调用日志：

```shell
[thread-2] At 39:21.297
[thread-4] At 39:21.297
[thread-3] At 39:21.297
[thread-1] At 39:21.297

[thread-2] At 39:21.918
[thread-3] At 39:21.868
[thread-4] At 39:22.011
[thread-1] At 39:22.184

[thread-1] At 39:23.086
[thread-5] At 39:23.939
[thread-3] At 39:24.152
[thread-4] At 39:24.977

[thread-3] At 39:26.861
[thread-1] At 39:28.617
[thread-4] At 39:28.942
[thread-2] At 39:31.039
```

现在我们的传播要好得多，我们已经**消除了冲突和空闲时间，并最终实现了几乎恒定的客户端调用率**，除了最初的激增。

![](/assets/images/2023/designpattern/resilience4jbackoffjitter02.png)

注意：为了便于说明，我们夸大了间隔，在真实场景中，我们的间隔会更小。

## 6. 总结

在本教程中，我们探讨了如何通过使用抖动增强指数退避来改进客户端应用程序重试失败调用的方式。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/design-patterns-modules)上获得。