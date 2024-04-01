---
layout: post
title:  在Java中完成线程的作业后返回一个值
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

Java的主要特性之一是[并发性](https://www.baeldung.com/java-concurrency)，它允许多个线程运行并执行并行任务。因此，我们可以执行异步和非阻塞指令。这将优化可用资源，尤其是当计算机具有多个CPU时。有两种类型的线程：有返回值或没有返回值(在后一种情况下，我们说它将有一个void返回方法)。

在本文中，我们将重点介绍**如何从其作业已终止的线程返回一个值**。

## 2. Thread和Runnable

**我们将Java线程称为轻量级进程**，让我们看一下Java程序通常是如何工作的：

![](/assets/images/2023/javaconcurrency/javareturnvalueafterthreadfinish01.png)

Java程序是一个正在执行的进程。线程是Java进程的子集，可以访问主内存。它可以与同一进程的其他线程通信。

线程具有[生命周期](https://www.baeldung.com/java-thread-lifecycle)和不同的状态。实现它的一种常见方法是通过Runnable接口：

```java
public class RunnableExample implements Runnable {
    // ...
    @Override
    public void run() {
        // do something
    }
}
```

然后，我们可以开始我们的线程：

```java
Thread thread = new Thread(new RunnableExample());
thread.start();
thread.join();
```

如我们所见，我们无法从Runnable返回值。但是，我们可以使用[wait()和notify()](https://www.baeldung.com/java-wait-notify)与其他线程同步。[join()](https://www.baeldung.com/java-thread-join)方法将使执行处于等待状态，直到它完成。当我们从异步执行中获取结果时，我们稍后会看到这有多重要。

## 3. Callable

Java从1.5版本开始引入了[Callable](https://www.baeldung.com/java-runnable-callable)接口。让我们看一个异步任务返回[阶乘](https://en.wikipedia.org/wiki/Factorial)计算值的示例，我们使用BigInteger，因为结果可能是一个很大的数字：

```java
public class CallableFactorialTask implements Callable<BigInteger> {
    // fields and constructor
    @Override
    public BigInteger call() throws Exception {
        return factorial(BigInteger.valueOf(value));
    }
}
```

让我们也创建一个简单的阶乘计算器：

```java
public class FactorialCalculator {

    public static BigInteger factorial(BigInteger end) {
        BigInteger start = BigInteger.ONE;
        BigInteger res = BigInteger.ONE;

        for (int i = start.add(BigInteger.ONE).intValue(); i <= end.intValue(); i++) {
            res = res.multiply(BigInteger.valueOf(i));
        }

        return res;
    }

    public static BigInteger factorial(BigInteger start, BigInteger end) {
        BigInteger res = start;

        for (int i = start.add(BigInteger.ONE).intValue(); i <= end.intValue(); i++) {
            res = res.multiply(BigInteger.valueOf(i));
        }

        return res;
    }
}
```

Callable只有一个方法call()需要我们覆盖，该方法将返回我们的异步任务的对象。

Callable和Runnable都是@FunctionalInterface。Callable可以返回一个值并抛出异常。但是，它需要一个Future来完成任务。

## 4. 执行Callable

我们可以使用Future或Fork/Join执行Callable。

### 4.1 Callable和Future

**从1.5版本开始，Java具有[Future](https://www.baeldung.com/java-future)接口来创建包含我们异步处理的响应的对象**。我们可以在逻辑上将Future与Javascript中的[Promise](https://www.w3schools.com/js/js_promise.asp)进行比较。

例如，当我们想要从多个端点获取数据时，我们通常会看到Future。因此，我们需要等待所有任务完成才能收集响应数据。

Future包装响应并等待线程完成。但是，我们可能会遇到中断，例如超时或执行异常。

让我们看一下Future接口：

```java
public interface Future<V> {
    boolean cancel(boolean var1);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long var1, TimeUnit var3) throws InterruptedException, ExecutionException, TimeoutException;
}
```

**get方法很有趣，我们可以等待并获取执行的结果**。

要启动Future作业，我们将它的执行与[ThreadPool](https://www.baeldung.com/thread-pool-java-and-guava)相关联。这样，我们将为这些异步任务分配一些资源。

让我们创建一个使用Executor的示例，该Executor对我们之前看到的Callable中的阶乘数求和。我们将使用Executor接口和[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)实现来创建ThreadPoolExecutor。我们可能想使用[固定的或缓存的线程池](https://www.baeldung.com/java-executors-cached-fixed-threadpool)。在本例中，我们将使用缓存线程池来演示：

```java
public BigInteger execute(List<CallableFactorialTask> tasks) {

    BigInteger result = BigInteger.ZERO;

    ExecutorService cachedPool = Executors.newCachedThreadPool();

    List<Future<BigInteger>> futures;

    try {
        futures = cachedPool.invokeAll(tasks);
    } catch (InterruptedException e) {
        // exception handling example
        throw new RuntimeException(e);
    }

    for (Future<BigInteger> future : futures) {
        try {
            result = result.add(future.get());
        } catch (InterruptedException | ExecutionException e) {
            // exception handling example
            throw new RuntimeException(e);
        }
    }

    return result;
}
```

我们可以用一张图来表示这个执行，我们可以在其中观察线程池和Callable是如何交互的：

![](/assets/images/2023/javaconcurrency/javareturnvalueafterthreadfinish02.png)

Executor将在Future对象中调用和收集所有内容。然后，我们可以从异步处理中得到一个或多个结果。

让我们通过统计两个阶乘数的结果来测试：

```java
@Test
void givenCallableExecutor_whenExecuteFactorial_thenResultOk() {
    BigInteger result = callableExecutor.execute(Arrays.asList(new CallableFactorialTask(5), new CallableFactorialTask(3)));
    assertEquals(BigInteger.valueOf(126), result);
}
```

### 4.2 Callable和Fork/Join

**我们还可以选择使用[ForkJoinPool](https://www.baeldung.com/java-fork-join)**。它仍然与ExecutorService类似，因为它扩展了[AbstractExecutorService](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/AbstractExecutorService.html)类。但是，它有不同的方式来创建和组织线程。**它将任务分解为更小的任务并优化资源，使它们永远不会闲置**。我们可以用图表表示子任务：

![](/assets/images/2023/javaconcurrency/javareturnvalueafterthreadfinish03.png)

我们可以看到主任务将分叉为SubTask1、SubTask3和SubTask4作为最小的可执行块。最后，他们将合并最终结果。

让我们使用ForkJoinPool将前面的示例转换，我们可以将所有内容包装在一个execute方法中：

```java
public BigInteger execute(List<Callable<BigInteger>> forkFactorials) {
    List<Future<BigInteger>> futures = forkJoinPool.invokeAll(forkFactorials);

    BigInteger result = BigInteger.ZERO;

    for (Future<BigInteger> future : futures) {
        try {
            result = result.add(future.get());
        } catch (InterruptedException | ExecutionException e) {
            // exception handling example
            throw new RuntimeException(e);
        }
    }

    return result;
}
```

在这种情况下，我们只需要创建一个不同的池来获得我们的Future。让我们用阶乘Callable的列表来测试它：

```java
@Test
void givenForkExecutor_whenExecuteCallable_thenResultOk() {
    assertEquals(BigInteger.valueOf(126), forkExecutor.execute(Arrays.asList(new CallableFactorialTask(5), new CallableFactorialTask(3))));
}
```

然而，我们也可以决定如何分解我们的任务。我们可能希望根据某些标准来分解我们的计算，例如，根据输入参数或服务负载。

我们需要将任务重写为ForkJoinTask，因此我们将使用[RecursiveTask](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/RecursiveTask.html)：

```java
public class ForkFactorialTask extends RecursiveTask<BigInteger> {
    // fields and constructor

    @Override
    protected BigInteger compute() {

        BigInteger factorial = BigInteger.ONE;

        if (end - start > threshold) {

            int middle = (end + start) / 2;

            return factorial.multiply(new ForkFactorialTask(start, middle, threshold).fork()
                    .join()
                    .multiply(new ForkFactorialTask(middle + 1, end, threshold).fork().join()));
        }

        return factorial.multiply(factorial(BigInteger.valueOf(start), BigInteger.valueOf(end)));
    }
}
```

如果应用某个阈值，我们将细分我们的主要任务。然后我们可以使用invoke()方法来获取结果：

```java
ForkJoinPool forkJoinPool = ForkJoinPool.commonPool(); 
int result = forkJoinPool.invoke(forkFactorialTask);
```

此外，submit()或execute()也是一个选项。但是，我们总是需要join()命令来完成执行。

我们还创建一个测试，在其中执行阶乘子任务：

```java
@Test
void givenForkExecutor_whenExecuteRecursiveTask_thenResultOk() {
    assertEquals(BigInteger.valueOf(3628800), forkExecutor.execute(new ForkFactorialTask(10, 5)));
}
```

在本例中，我们将10的阶乘分成两个任务。第一个将从1计算到5，而第二个将从6计算到10。

## 5. CompletableFuture

**自1.8版本以来，Java通过引入[CompletableFuture](https://www.baeldung.com/java-completablefuture)改进了多线程**。它从Future执行中删除了样板代码，并添加了链接或组合异步结果等功能。但是，最重要的是，我们现在可以对任何方法进行异步计算，因此我们不受Callable的约束。此外，我们可以将语义不同的多个Future链接在一起。

### 5.1 supplyAsync()

使用CompletableFuture可以很简单：

```java
CompletableFuture<BigInteger> future = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(10)));
// ...
BigInteger result = future.get();
```

我们不再需要Callable，我们可以将任何Lambda表达式作为参数传递。让我们用supplyAsync()测试阶乘方法：

```java
@Test
void givenCompletableFuture_whenSupplyAsyncFactorial_thenResultOk() throws ExecutionException, InterruptedException {
    CompletableFuture<BigInteger> completableFuture = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(10)));
    assertEquals(BigInteger.valueOf(3628800), completableFuture.get());
}
```

请注意，我们没有指定任何线程池。在这种情况下，将使用默认的ForkJoinPool。但是，我们可以指定一个Executor，例如，使用固定线程池：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(10)), Executors.newFixedThreadPool(1));
```

### 5.2 thenCompose()

我们还可以创建一个连续的Future链。假设我们有两个阶乘任务，第二个需要第一个任务的输入：

```java
CompletableFuture<BigInteger> completableFuture = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(3)))
    .thenCompose(inputFromFirstTask -> CompletableFuture.supplyAsync(() -> factorial(inputFromFirstTask)));

BigInteger result = completableFuture.get();
```

我们可以使用thenCompose()方法在链的下一个中使用来自CompletableFuture的返回值。

让我们组合两个阶乘的执行。例如，我们从3开始，给出6的阶乘作为下一个阶乘的输入：

```java
@Test
void givenCompletableFuture_whenComposeTasks_thenResultOk() throws ExecutionException, InterruptedException {
    CompletableFuture<BigInteger> completableFuture = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(3)))
        .thenCompose(inputFromFirstTask -> CompletableFuture.supplyAsync(() -> factorial(inputFromFirstTask)));
    assertEquals(BigInteger.valueOf(720), completableFuture.get());
}
```

### 5.3 allOf()

有趣的是，我们可以使用接收输入可变参数的静态方法allOf()并行执行多个Future。

从多个执行中收集异步结果就像添加到allOf()和join()来完成任务一样简单：

```java
BigInteger result = allOf(asyncTask1, asyncTask2)
    .thenApplyAsync(fn -> factorial(factorialTask1.join()).add(factorial(new BigInteger(factorialTask2.join()))), Executors.newFixedThreadPool(1)).join();
```

请注意，**allOf()的返回类型为void**。因此，我们需要手动从单个Future中获取结果。此外，我们可以在同一执行中运行具有不同返回类型的Future。

为了进行测试，让我们加入两个不同的阶乘任务。为了演示，一个是数字输入，而第二个是字符串：

```java
@Test
void givenCompletableFuture_whenAllOfTasks_thenResultOk() {
    CompletableFuture<BigInteger> asyncTask1 = CompletableFuture.supplyAsync(() -> BigInteger.valueOf(5));
    CompletableFuture<String> asyncTask2 = CompletableFuture.supplyAsync(() -> "3");

    BigInteger result = allOf(asyncTask1, asyncTask2)
        .thenApplyAsync(fn -> factorial(asyncTask1.join()).add(factorial(new BigInteger(asyncTask2.join()))), Executors.newFixedThreadPool(1))
            .join();

    assertEquals(BigInteger.valueOf(126), result);
}
```

## 6. 总结

在本教程中，我们了解了如何从线程返回对象。我们看到了如何将Callable与Future和线程池结合使用，Future包装结果并等待所有任务完成。我们还看到了ForkJoinPool的一个例子，可以将我们的执行优化为多个子任务。

Java 8中的CompletableFuture的工作方式类似，但还提供了新功能，例如可以执行任何Lambda表达式。它还允许我们链接和组合异步任务的结果。

最后，我们用Future、Fork和CompletableFuture测试了一个简单的阶乘任务。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上获得。