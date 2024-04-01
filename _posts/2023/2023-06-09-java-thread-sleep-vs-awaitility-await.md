---
layout: post
title:  Thread.sleep()与Awaitility.await()
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将比较两种在Java中处理异步操作的方法。**首先，我们将了解sleep()方法如何在[Thread](https://www.baeldung.com/java-start-thread)上工作。然后，我们将尝试使用[Awaitility](https://www.baeldung.com/awaitility-testing)库提供的功能实现相同的目标**。在此过程中，我们可以看到这些解决方案的比较情况以及哪种解决方案更适合我们的用例。

## 2. 用例

**当我们想要等待异步操作完成时，sleep()和await()方法特别有用**。例如，我们的应用程序可能会将消息发送到消息代理或队列。在这种情况下，我们无法准确知道另一端何时收到消息。另一个用例是调用API端点并等待特定结果。例如，我们向服务发送请求，它启动一个长时间运行的任务，然后等待它完成。

**在我们的示例应用程序中，我们将创建一个简单的服务来跟踪请求的状态。我们将在给定时间后检查请求是否处于所需状态**。

## 3. 应用程序设置

让我们创建一个处理请求的异步服务。我们还需要一种方法来获取这些请求的状态，以便之后能够对其进行验证：

```java
public class RequestProcessor {

    private Map<String, String> requestStatuses = new HashMap<>();

    public String processRequest() {
        String requestId = UUID.randomUUID().toString();
        requestStatuses.put(requestId, "PROCESSING");

        ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
        executorService.schedule((() -> {
            requestStatuses.put(requestId, "DONE");
        }), getRandomNumberBetween(500, 2000), TimeUnit.MILLISECONDS);

        return requestId;
    }

    public String getStatus(String requestId) {
        return requestStatuses.get(requestId);
    }

    private int getRandomNumberBetween(int min, int max) {
        Random random = new Random();
        return random.nextInt(max - min) + min;
    }
}
```

此服务使用[Java的ScheduledExecutorService](https://www.baeldung.com/java-executor-service-tutorial#ScheduledExecutorService)来延迟将请求状态更改为“DONE”的命令，它等待半秒到两秒之间的随机时间。

**我们将创建单元测试来展示检查异步操作结果的两种方法**。

## 4. 纯Java

首先，让我们使用普通的Java方法并暂停线程上的执行。

**在这种情况下，我们可以设置我们想要等待的时间量(以毫秒为单位)**。让我们创建我们的测试类和第一个单元测试：

```java
@DisplayName("Request processor")
public class RequestProcessorUnitTest {

    RequestProcessor requestProcessor = new RequestProcessor();

    @Test
    @DisplayName("Wait for completion using Thread.sleep")
    void whenWaitingWithThreadSleep_thenStatusIsDone() throws InterruptedException {
        String requestId = requestProcessor.processRequest();

        Thread.sleep(2000);

        assertEquals("DONE", requestProcessor.getStatus(requestId));
    }
}
```

在这个测试用例中，我们调用RequestProcessor的processRequest()方法来启动请求。然后，我们必须等待才能通过请求ID获取状态。我们正在等待状态更改，因为我们希望它完成。

**当我们使用Thread.sleep()时，我们必须确保在检查结果之前等待足够的时间**。在我们的例子中，我们知道我们的请求最多会在两秒内得到处理。然而，**在现实场景中，很难确定正确的等待时间**。

## 5. Awaitility

**我们还可以使用[Awaitility](https://www.baeldung.com/awaitility-testing)，这是一个提供易于阅读的API来测试此类代码的库**。

首先，我们将[awaitility依赖](https://search.maven.org/search?q=g:org.awaitilityANDa:awaitility)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>4.2.0</version>
    <scope>test</scope>
</dependency>
```

现在，我们可以创建利用新功能的测试用例：

```java
@Test
@DisplayName("Wait for completion using Awaitility")
void whenWaitingWithAwaitility_thenStatusIsDone() {
    String requestId = requestProcessor.processRequest();

    Awaitility.await()
        .until(() -> requestProcessor.getStatus(requestId), not(equalTo("PROCESSING")));

    assertEquals("DONE", requestProcessor.getStatus(requestId));
}
```

这个测试用例的启动方式与上一个相同。**但是，我们有一个await语句，而不是睡眠固定的两秒钟。在这种情况下，我们会休眠直到请求不再处于“PROCESSING”状态**。在此之后，我们使用相同的断言来确保状态具有预期值。

我们也可以提供其他选项。例如，我们可以配置最多等待两秒钟加上再轮询的时间量，以确保该过程已完成，并且等待轮询已获得更新的“DONE”值。

```java
Awaitility.await()
    .atMost(2101, TimeUnit.MILLISECONDS)
    .until(() -> requestProcessor.getStatus(requestId), not(equalTo("PROCESSING")));
```

**在后台，Awaitility使用[轮询](https://github.com/awaitility/awaitility/wiki/Usage#polling)来检查给定的语句是true还是false。我们可以增加或减少轮询间隔，但[默认值](https://github.com/awaitility/awaitility/wiki/Usage#defaults)为100毫秒**。换句话说，Awaitility每100毫秒检查一次条件。下面，让我们添加500毫秒的轮询延迟：

```java
Awaitility.await()
    .atMost(2501, TimeUnit.MILLISECONDS)
    .pollDelay(500, TimeUnit.MILLISECONDS)
    .until(() -> requestProcessor.getStatus(requestId), not(equalTo("PROCESSING")));
```

## 6. 比较

正如我们所见，这两种方法都适用于我们的用例。但是，我们应该注意一些优点和缺点。

**使用sleep()暂停一个线程相当简单，但是在我们将它送入睡眠后我们没有多少控制权**。我们正在等待的操作可能会在之后立即完成，但我们仍然需要等待整个预定义的持续时间。

**另一方面，Awaitility允许我们拥有更细粒度的配置**。一旦条件满足，线程就会恢复执行，这可以提高性能。

**sleep()方法默认在Java中可用，而Awaitility是一个需要添加到我们项目中的库**。在选择解决方案时，我们必须考虑到这一点。**使用内置方法更明显，但我们可以使用领域特定语言获得更具可读性的代码**。

## 7. 总结

在本文中，我们讨论了在Java中处理异步操作的两种不同方法。重点在于测试，但这些示例也可以用于代码的其他部分。

首先，我们使用Java的内置解决方案通过sleep()方法暂停线程的执行。它易于使用，但我们必须提前提供睡眠时长。

然后，我们将它与Awaitility库进行了比较，Awaitility库提供了一种用于处理这种情况的特定领域语言。它会带来更具可读性的代码，但我们必须学习如何使用它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上获得。