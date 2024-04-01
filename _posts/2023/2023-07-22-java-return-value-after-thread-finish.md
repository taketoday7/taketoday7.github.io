---
layout: post
title:  在Java中完成线程的任务后返回一个值
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

Java的主要特性之一是[并发性](https://www.baeldung.com/java-concurrency)，它允许多个线程运行和执行并行任务。因此，我们可以执行异步和非阻塞指令。这将优化可用资源，特别是当计算机具有多个CPU时。有两种类型的线程：有返回值或无返回值(在后一种情况下，我们说它将有一个void返回方法)。

在本文中，我们将重点介绍**如何从作业已终止的线程返回值**。

## 2. Thread和Runnable

**我们将Java线程称为轻量级进程**，让我们看一下Java程序通常是如何工作的：

![](/assets/images/2023/javaconcurrency/javareturnvalueafterthreadfinish01.png)

Java程序是一个正在执行的进程。线程是Java进程的子集，可以访问主内存。它可以与同一进程的其他线程进行通信。

线程有[生命周期](https://www.baeldung.com/java-thread-lifecycle)和不同的状态，实现它的常见方法是通过Runnable接口：

```java
public class RunnableExample implements Runnable {
    // ...
    @Override
    public void run() {
        // do something
    }
}
```

然后，我们可以启动线程：

```java
Thread thread = new Thread(new RunnableExample());
thread.start();
thread.join();
```

正如我们所看到的，我们无法从Runnable返回值。但是，我们可以使用[wait()和notify()](https://www.baeldung.com/java-wait-notify)与其他线程同步。[join()](https://www.baeldung.com/java-thread-join)方法将使执行保持等待状态，直到它完成。稍后当我们从异步执行中获取结果时，我们将看到这一点的重要性。

## 3. Callable

Java从1.5版本开始引入了[Callable](https://www.baeldung.com/java-runnable-callable)接口，让我们看一个返回[阶乘](https://en.wikipedia.org/wiki/Factorial)计算值的异步任务的示例。我们使用BigInteger，因为结果可能会很大：

```java
public class CallableFactorialTask implements Callable<BigInteger> {
    // fields and constructor
    @Override
    public BigInteger call() throws Exception {
        return factorial(BigInteger.valueOf(value));
    }
}
```

我们还创建一个简单的阶乘计算器：

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

Callable只有一个我们需要重写的方法call()，该方法将返回异步任务的对象。

Callable和Runnable都是@FunctionalInterface。Callable可以返回值并抛出异常，但是，它需要一个Future来完成任务。

## 4. 执行Callable

我们可以使用Future或Fork/Join来执行Callable。

### 4.1 使用Future执行Callable

**从1.5版本开始，Java有了[Future](https://www.baeldung.com/java-future)接口来创建包含异步处理响应的对象**。我们可以在逻辑上将Future与JavaScript中的[Promise](https://www.w3schools.com/js/js_promise.asp)进行比较。

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

**get方法对于我们来说有趣的是等待并获取执行结果**。

为了启动Future作业，我们将它的执行与[ThreadPool](https://www.baeldung.com/thread-pool-java-and-guava)关联起来。这样，我们将为这些异步任务分配一些资源。

让我们创建一个使用Executor的示例，该Executor对我们之前看到的Callable的阶乘数字进行求和。我们将使用Executor接口和[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)实现来创建ThreadPoolExecutor。我们可能倾向于使用[固定或缓存的线程池](https://www.baeldung.com/java-executors-cached-fixed-threadpool)，在本例中，我们将使用缓存线程池来演示：

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

我们可以用图表来表示此执行，在其中我们可以观察线程池和Callable如何交互：

![](/assets/images/2023/javaconcurrency/javareturnvalueafterthreadfinish02.png)

Executor将调用并收集Future对象中的所有内容。然后，我们可以从异步处理中获取一个或多个结果。

让我们通过相加两个阶乘数的结果来进行测试：

```java
@Test
void givenCallableExecutor_whenExecuteFactorial_thenResultOk() {
    BigInteger result = callableExecutor.execute(Arrays.asList(new CallableFactorialTask(5), new CallableFactorialTask(3)));
    assertEquals(BigInteger.valueOf(126), result);
}
```

### 4.2 使用Fork/Join执行Callable

**我们还可以选择使用[ForkJoinPool](https://www.baeldung.com/java-fork-join)**，它的工作方式仍然与ExecutorService类似，因为它扩展了[AbstractExecutorService](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/concurrent/AbstractExecutorService.html)类。但是，它具有不同的方式来创建和组织线程。**它将一个任务分解为更小的任务并优化资源，使它们永远不会闲置**。我们可以用图表来表示子任务：

![](/assets/images/2023/javaconcurrency/javareturnvalueafterthreadfinish03.png)

我们可以看到主任务将分叉为SubTask1、SubTask3和SubTask4作为最小的可执行单元。最后，他们将加入最终结果。

让我们使用ForkJoinPool转换前面的示例，我们可以将所有内容包装在执行器方法中：

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

在这种情况下，我们只需要创建一个不同的池来获取我们的Future。让我们用阶乘Callable列表来测试一下：

```java
@Test
void givenForkExecutor_whenExecuteCallable_thenResultOk() {
    assertEquals(BigInteger.valueOf(126), forkExecutor.execute(Arrays.asList(new CallableFactorialTask(5), new CallableFactorialTask(3))));
}
```

但是，我们也可能决定如何fork我们的任务，我们可能希望根据某些条件(例如输入参数或服务负载)来fork计算。

我们需要将任务重写为ForkJoinTask，因此我们将使用[RecursiveTask](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/concurrent/RecursiveTask.html)：

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
                    .multiply(new ForkFactorialTask(middle + 1, end, threshold).fork()
                            .join()));
        }

        return factorial.multiply(factorial(BigInteger.valueOf(start), BigInteger.valueOf(end)));
    }
}
```

如果某个阈值适用，我们将细分我们的主任务，然后我们可以使用invoke()方法来获取结果：

```java
ForkJoinPool forkJoinPool = ForkJoinPool.commonPool(); 
int result = forkJoinPool.invoke(forkFactorialTask);
```

此外，submit()或execute()也是一个选项。但是，我们总是需要join()命令来完成执行。

下面创建一个测试，在其中对阶乘执行进行子任务分解：

```java
@Test
void givenForkExecutor_whenExecuteRecursiveTask_thenResultOk() {
    assertEquals(BigInteger.valueOf(3628800), forkExecutor.execute(new ForkFactorialTask(10, 5)));
}
```

在本例中，我们将10的阶乘分为两个任务。第一个将从1计算到5，第二个将从6计算到10。

## 5. CompletableFuture

**自1.8版本以来，Java通过引入[CompletableFuture](https://www.baeldung.com/java-completablefuture)改进了多线程**。它从Future执行中删除了样板代码，并添加了链接或组合异步结果等功能。但是，最重要的是，我们现在可以对任何方法进行异步计算，因此我们不必绑定到Callable。此外，我们可以将多个语义不同的Future连接在一起。

### 5.1 supplyAsync()

使用CompletableFuture可以非常简单：

```java
CompletableFuture<BigInteger> future = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(10)));
...
BigInteger result = future.get();
```

我们不再需要Callable了，我们可以传递任何lambda表达式作为参数。让我们使用SupplyAsync()测试factorial方法：

```java
@Test
void givenCompletableFuture_whenSupplyAsyncFactorial_thenResultOk() throws ExecutionException, InterruptedException {
    CompletableFuture<BigInteger> completableFuture = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(10)));
    assertEquals(BigInteger.valueOf(3628800), completableFuture.get());
}
```

请注意，我们没有指定任何线程池。在这种情况下，将使用默认的ForkJoinPool。但是，我们可以指定一个Executor，例如使用固定的线程池：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(10)), Executors.newFixedThreadPool(1));
```

### 5.2 thenCompose()

我们还可以创建一个连续的Future链。假设我们有两个阶乘任务，第二个任务需要第一个任务的输入：

```java
CompletableFuture<BigInteger> completableFuture = CompletableFuture.supplyAsync(() -> factorial(BigInteger.valueOf(3)))
    .thenCompose(inputFromFirstTask -> CompletableFuture.supplyAsync(() -> factorial(inputFromFirstTask)));

BigInteger result = completableFuture.get();
```

我们可以使用thenCompose()方法在链的下一个中使用来自CompletableFuture的返回值。

让我们结合两个阶乘的执行。例如，我们从3开始，给出阶乘6作为下一个阶乘的输入：

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

从多次执行中收集异步结果就像添加到allOf()和join()来完成任务一样简单：

```java
BigInteger result = allOf(asyncTask1, asyncTask2)
    .thenApplyAsync(fn -> factorial(factorialTask1.join()).add(factorial(new BigInteger(factorialTask2.join()))), Executors.newFixedThreadPool(1)).join();
```

请注意，**allOf()具有void返回类型**。因此，我们需要手动获取单个Future的结果。此外，我们可以在同一次执行中运行具有不同返回类型的Future。

为了测试，让我们加入两个不同的阶乘任务。为了演示，一个接收数字输入，而第二个接收字符串：

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

在本教程中，我们了解了如何从线程返回对象。我们介绍了如何将Callable与Future和线程池结合使用，Future包装结果并等待所有任务完成。我们还看到了ForkJoinPool的示例，用于将执行优化为多个子任务。

Java 8中的CompletableFuture的工作原理类似，但还提供了新功能，例如执行任何lambda表达式的可能性。它还允许我们链接和组合异步任务的结果。

最后，我们使用Future、Fork和CompletableFuture测试了一个简单的阶乘任务。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上找到。