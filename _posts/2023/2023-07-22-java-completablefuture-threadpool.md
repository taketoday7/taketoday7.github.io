---
layout: post
title:  Java中的CompletableFuture和ThreadPool
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

Java 8的并发API引入了CompletableFuture，这是一个用于简化异步和[非阻塞编程](https://www.baeldung.com/java-completablefuture-non-blocking)的宝贵工具。

在本文中，我们将讨论Java的CompletableFuture及其利用的线程池。我们将探讨其异步和非异步方法之间的差异，并学习如何最大限度地发挥CompletableFuture API的潜力。

## 2. 非异步方法

CompletableFuture提供了由50多种方法组成的广泛API，其中许多方法都有两种变体：非异步和异步。让我们从非异步对应项开始，并使用thenApply()方法深入研究实际示例：

![](/assets/images/2023/javaconcurrency/javacompletablefuturethreadpool01.png)

当使用thenApply()时，我们传递一个函数作为参数，该函数将CompletableFuture的先前值作为输入，执行操作并返回一个新值。因此，会创建一个新的CompletableFuture来封装结果值。为了说明这个概念，让我们考虑一个简单的示例，其中我们将字符串值转换为表示其长度的整数。此外，我们还将打印负责执行此操作的线程的名称：

```java
@Test
void whenUsingNonAsync_thenMainThreadIsUsed() throws Exception {
    CompletableFuture<String> name = CompletableFuture.supplyAsync(() -> "Tuyucheng");

    CompletableFuture<Integer> nameLength = name.thenApply(value -> {
        printCurrentThread(); // will print "main"
        return value.length();
    });

   assertThat(nameLength.get()).isEqualTo(9);
}

private static void printCurrentThread() {
    System.out.println(Thread.currentThread().getName());
}
```

**作为参数传递给thenApply()的函数将由直接与CompletableFuture的API交互的线程执行，在我们的例子中是main线程**。但是，如果我们提取与CompletableFuture的交互并从不同的线程调用它，我们应该注意到变化：

```java
@Test
void whenUsingNonAsync_thenUsesCallersThread() throws Exception {
    Runnable test = () -> {
        CompletableFuture<String> name = CompletableFuture.supplyAsync(() -> "Tuyucheng");

        CompletableFuture<Integer> nameLength = name.thenApply(value -> {
            printCurrentThread(); // will print "test-thread"
            return value.length();
        });

        try {
            assertThat(nameLength.get()).isEqualTo(9);
        } catch (Exception e) {
            fail(e.getMessage());
        }
    };

    new Thread(test, "test-thread").start();
    Thread.sleep(100l);
}
```

## 3. 异步方法

API中的大多数方法都具有异步对应方法，我们可以使用这些异步变体来确保中间操作在单独的线程池上执行。让我们更改前面的代码示例，从thenApply()切换到thenApplyAsync()：

```java
@Test
void whenUsingAsync_thenUsesCommonPool() throws Exception {
    CompletableFuture<String> name = CompletableFuture.supplyAsync(() -> "Tuyucheng");

    CompletableFuture<Integer> nameLength = name.thenApplyAsync(value -> {
        printCurrentThread(); // will print "ForkJoinPool.commonPool-worker-1"
        return value.length();
    });

    assertThat(nameLength.get()).isEqualTo(9);
}
```

**根据[官方文档](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/concurrent/CompletableFuture.html)，如果我们使用异步方法而不显式提供Executor，则函数将使用ForkJoinPool.commonPool()执行**。因此，如果我们运行代码片段，我们应该会看到常见的ForkJoinPool工作线程之一：在我的例子中是“ForkJoinPool.commonPool-worker-1”。

## 4. 使用自定义Executor的异步方法

**我们可以注意到所有异步方法都被重载，提供了一种接收要执行的代码以及Executor的替代方法，我们可以使用它来为异步操作使用显式线程池**。让我们进一步更新我们的测试并提供一个用于thenApplyAsync()方法的自定义线程池：

```java
@Test
void whenUsingAsync_thenUsesCustomExecutor() throws Exception {
    Executor testExecutor = Executors.newFixedThreadPool(5);
    CompletableFuture<String> name = CompletableFuture.supplyAsync(() -> "Tuyucheng");

    CompletableFuture<Integer> nameLength = name.thenApplyAsync(value -> {
        printCurrentThread(); // will print "pool-2-thread-1"
        return value.length();
    }, testExecutor);

   assertThat(nameLength.get()).isEqualTo(9);
}
```

正如预期的那样，当使用重载方法时，CompletableFuture将不再使用常规的ForkJoinPool。

## 5. 扩展CompletableFuture

**最后，我们可以扩展CompletableFuture并重写defaultExecutor()，封装自定义线程池。因此，我们将能够在不指定Executor的情况下使用异步方法，并且这些函数将由我们的线程池而不是常规的ForkJoinPool调用**。

让我们创建一个扩展CompletableFuture的CustomCompletableFuture，并使用newSingleThreadExecutor创建一个线程，其名称在测试时可以在控制台中轻松识别。此外，我们将重写defaultExecutor()方法，使CompletableFuture能够无缝地利用我们自定义的线程池：

```java
public class CustomCompletableFuture<T> extends CompletableFuture<T> {
    private static final Executor executor = Executors.newSingleThreadExecutor(
            runnable -> new Thread(runnable, "Custom-Single-Thread"));

    @Override
    public Executor defaultExecutor() {
        return executor;
    }
}
```

此外，我们添加一个遵循CompletableFuture模式的静态工厂方法，这将使我们能够轻松创建并完成CustomCompletableFuture对象：

```java
public static <TYPE> CustomCompletableFuture<TYPE> supplyAsync(Supplier<TYPE> supplier) {
    CustomCompletableFuture<TYPE> future = new CustomCompletableFuture<>();
    executor.execute(() -> {
        try {
            future.complete(supplier.get());
        } catch (Exception ex) {
            future.completeExceptionally(ex);
        }
    });
    return future;
}
```

现在，让我们创建CustomCompletableFuture的实例，并对thenSupplyAsync()内的String值执行相同的转换。不过，这一次，我们将不再指定Executor，但仍然期望该函数由我们的专用线程“Custom-Single-Thread”调用：

```java
@Test
void whenOverridingDefaultThreadPool_thenUsesCustomExecutor() throws Exception {
    CompletableFuture<String> name = CustomCompletableFuture.supplyAsync(() -> "Tuyucheng");

    CompletableFuture<Integer> nameLength = name.thenApplyAsync(value -> {
        printCurrentThread(); // will print "Custom-Single-Thread"
        return value.length();
    });

   assertThat(nameLength.get()).isEqualTo(9);
}
```

## 6. 总结

在本文中，我们了解到CompletableFuture的API中的大多数方法都允许异步和非异步执行。通过调用非异步变体，调用CompletableFuture的API的线程还将执行所有中间操作和转换。另一方面，异步对应项将使用不同的线程池，默认线程池是通用的ForkJoinPool。

之后，我们讨论了对执行的进一步自定义，为每个异步步骤使用自定义Executor。最后，我们学习了如何创建自定义CompletableFuture对象并重写defaultExecutor()方法，这使我们能够利用异步方法，而无需每次都指定自定义Executor。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上找到。