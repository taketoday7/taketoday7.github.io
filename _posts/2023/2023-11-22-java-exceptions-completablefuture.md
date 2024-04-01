---
layout: post
title: 在Java CompletableFuture中处理异常
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

Java 8引入了一个基于Future的新抽象来运行异步任务-CompletableFuture类，它基本上是为了克服旧的[Future API](https://www.baeldung.com/java-future)的问题。

在本教程中，我们将研究使用CompletableFuture时处理异常的方法。

## 2. CompletableFuture回顾

首先，我们可能需要回顾一下CompletableFuture是什么。CompletableFuture是一个Future实现，它允许我们运行，最重要的是，链式异步操作。一般来说，异步操作完成可能有三种结果-正常、异常或可以从外部取消。CompletableFuture有各种API方法来解决所有这些可能的结果。

与CompletableFuture中的许多其他方法一样，这些方法具有[非异步、异步，并使用特定的Executor变体](https://www.baeldung.com/java-completablefuture-threadpool)进行异步。那么，事不宜迟，让我们一一看看CompletableFuture中处理异常的方法。

## 3. handle()

首先，我们有一个handle()方法。通过使用此方法，**我们可以访问和转换CompletionStage的整个结果，而不管结果如何**。也就是说，handle()方法接收BiFunction[函数接口](https://www.baeldung.com/java-8-functional-interfaces)。因此，此接口有两个输入。在handle()方法的情况下，参数将是上一个CompletionStage和发生的[Exception](https://www.baeldung.com/java-exceptions)的结果。

**重要的是这两个参数都是可选的，这意味着它们可以为null**。这在某种意义上是显而易见的，因为之前的CompletionStage已正常完成。那么Exception应该是null因为没有任何异常，类似于CompletionStage结果值为可空。

现在让我们看一下handle()方法用法的示例：

```java
@ParameterizedTest
@MethodSource("parametersSource_handle")
void whenCompletableFutureIsScheduled_thenHandleStageIsAlwaysInvoked(int radius, long expected) throws ExecutionException, InterruptedException {
    long actual = CompletableFuture
        .supplyAsync(() -> {
            if (radius <= 0) {
                throw new IllegalArgumentException("Supplied with non-positive radius '%d'");
            }
            return Math.round(Math.pow(radius, 2) * Math.PI);
        })
        .handle((result, ex) -> {
            if (ex == null) {
                return result;
            } else {
                return -1L;
            }
        })
        .get();

    Assertions.assertThat(actual).isEqualTo(expected);
}

static Stream<Arguments> parameterSource_handle() {
    return Stream.of(Arguments.of(1, 3), Arguments.of(1, -1));
}
```

这里需要注意的是handle()方法返回一个新的CompletionStage，无论之前的CompletionStage结果如何，它都将始终执行。因此，handle()将上一阶段的源值转换为某个输出值。因此，我们将通过get()方法获取的值是从handle()方法返回的值。

## 4. exceptionally()

handle()方法并不总是很方便，特别是如果我们只想在有异常的情况下处理异常。幸运的是，我们有一个替代方案-exceptionally()。

此方法允许我们提供一个回调，**仅当上一个CompletionStage以Exception结束时才执行**。如果没有抛出异常，则省略回调，并且执行链将继续执行下一个回调(如果有)，并使用前一个回调的值。

为了理解，让我们看一个具体的例子：

```java
@ParameterizedTest
@MethodSource("parametersSource_exceptionally")
void whenCompletableFutureIsScheduled_thenExceptionallyExecutedOnlyOnFailure(int a, int b, int c, long expected) throws ExecutionException, InterruptedException {
    long actual = CompletableFuture
        .supplyAsync(() -> {
            if (a <= 0 || b <= 0 || c <= 0) {
                throw new IllegalArgumentException(String.format("Supplied with incorrect edge length [%s]", List.of(a, b, c)));
            }
            return a * b * c;
        })
        .exceptionally((ex) -> -1)
        .get();

    Assertions.assertThat(actual).isEqualTo(expected);
}

static Stream<Arguments> parametersSource_exceptionally() {
    return Stream.of(
        Arguments.of(1, 5, 5, 25),
        Arguments.of(-1, 10, 15, -1)
    );
}
```

这里，它的工作方式与handle()相同，但我们有一个Exception实例作为回调的参数。此参数永远不会为null，因此我们的代码现在更简单了。

此处需要注意的重要一点是exceptionally()方法的回调仅在上一阶段以Exception完成时执行，这基本上意味着，如果异常发生在执行链中的某个地方，并且已经有一个handle()方法捕获了它，那么exceptionally()回调将不会在之后执行：


```java
@ParameterizedTest
@MethodSource("parametersSource_exceptionally")
void givenCompletableFutureIsScheduled_whenHandleIsAlreadyPresent_thenExceptionallyIsNotExecuted(int a, int b, int c, long expected) throws ExecutionException, InterruptedException {
    long actual = CompletableFuture
        .supplyAsync(() -> {
            if (a <= 0 || b <= 0 || c <= 0) {
                throw new IllegalArgumentException(String.format("Supplied with incorrect edge length [%s]", List.of(a, b, c)));
            }
            return a * b * c;
        })
        .handle((result, throwable) -> {
            if (throwable != null) {
                return -1;
            }
            return result;
        })
        .exceptionally((ex) -> {
            System.exit(1);
            return 0;
        })
        .get();

    Assertions.assertThat(actual).isEqualTo(expected);
}

```

这里，exceptionally()不会被调用，因为handle()方法已经捕获了Exception(如果有的话)。因此，除非Exception发生在handle()方法内部，否则此处的exceptionally()方法将永远不会执行。


## 5. whenComplete()

API中还有一个whenComplete()方法，它接收带有两个参数的BiConsumer：结果和上一阶段的异常(如果有)。然而，这种方法与上面的方法有很大不同。

区别在于whenComplete()不会转换之前阶段的任何异常结果。因此，即使考虑到whenComplete()的回调将始终运行，前一阶段的异常(如果有)也将进一步传播：

```java
@ParameterizedTest
@MethodSource("parametersSource_whenComplete")
void whenCompletableFutureIsScheduled_thenWhenCompletedExecutedAlways(Double a, long expected) {
    try {
        CountDownLatch countDownLatch = new CountDownLatch(1);
        long actual = CompletableFuture
            .supplyAsync(() -> {
                if (a.isNaN()) {
                    throw new IllegalArgumentException("Supplied value is NaN");
                }
                return Math.round(Math.pow(a, 2));
            })
            .whenComplete((result, exception) -> countDownLatch.countDown())
            .get();
        Assertions.assertThat(countDownLatch.await(20L, java.util.concurrent.TimeUnit.SECONDS));
        Assertions.assertThat(actual).isEqualTo(expected);
    } catch (Exception e) {
        Assertions.assertThat(e.getClass()).isSameAs(ExecutionException.class);
        Assertions.assertThat(e.getCause().getClass()).isSameAs(IllegalArgumentException.class);
    }
}

static Stream<Arguments> parametersSource_whenComplete() {
    return Stream.of(
        Arguments.of(2d, 4),
        Arguments.of(Double.NaN, 1)
    );
}
```

正如我们在这里看到的，whenCompleted()内部的回调在两个测试调用中都运行。但是，在第二次调用中，我们完成了ExecutionException，这导致了IllegalArgumentException。因此，正如我们所看到的，回调中的异常会传播到被调用者。我们将在下一节中介绍发生这种情况的原因。

## 6. 未处理的异常

最后，我们需要稍微讨论一下未处理的异常。**通常，如果异常仍未捕获，则CompletableFuture将完成，但不会传播到被调用方的Exception**。在上面的例子中，我们从get()方法调用中获取ExecutionException。因此，这是因为当CompletableFuture以异常结束时，我们尝试访问结果。

因此，我们需要在get()调用之前检查CompletableFuture的结果，有几种方法可以做到这一点。第一种方法，也可能是最熟悉的方法，是通过isCompletedExceptionally()/isCancelled()/isDone()方法。如果CompletableFuture完成但出现异常、从外部取消或成功完成，这些方法将返回布尔值。

不过，值得一提的是，还有一个state()方法，它返回一个State枚举实例。该实例表示CompletableFuture的状态，例如RUNNING、SUCCESS等。因此，这是访问CompletableFuture结果的另一种方式。

## 7. 总结

在本文中，我们探讨了处理CompletableFuture阶段中发生的异常的方法。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-advanced-5)上找到。