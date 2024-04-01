---
layout: post
title:  Java中的并行化for循环
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

有时，我们可能需要在[for循环](https://www.baeldung.com/java-for-loop)中处理大量元素。按顺序执行此操作可能会花费大量时间，并使系统无法得到充分利用。

在本教程中，我们将学习在Java中并行化for循环的不同方法，以提高此类情况下应用程序的性能。

## 2. 顺序处理

让我们首先看看如何在for循环中顺序处理元素并测量处理元素所需的时间。

### 2.1 使用for循环进行顺序处理

首先，我们将创建一个运行100次的for循环，并在每次迭代中执行繁重操作。

繁重操作的常见示例包括数据库调用、网络调用或CPU密集型操作。为了模拟繁重操作所花费的时间，让我们在每次迭代中调用[Thread.sleep()](https://www.baeldung.com/java-delay-code-execution#1-using-threadsleep)方法：

```java
public class Processor {

    public void processSerially() throws InterruptedException {
        for (int i = 0; i < 100; i++) {
            Thread.sleep(10);
        }
    }
}
```

在上面的代码中，我们在每次迭代中调用Thread.sleep()方法，这会导致执行暂停10毫秒。当我们运行processSerially()方法时，需要花费大量时间来顺序处理元素。

在接下来的部分中，我们将通过并行化for循环来优化此方法。最后，我们将比较顺序处理和并行处理所花费的时间。

## 3. 使用ExecutorService进行并行处理

[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)是一个代表异步执行机制的接口，**它允许我们提交要执行的任务并提供管理它们的方法**。

让我们看看如何使用ExecutorService接口来并行化for循环：

```java
void processParallelyWithExecutorService() throws InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    List<CompletableFuture<Void>> futures = new ArrayList<>();
    for (int i = 0; i < 100; i++) {
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, executorService);
        futures.add(future);
    }
    CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
    executorService.shutdown();
}
```

上面的代码中有几点需要注意：

- 我们**使用newFixedThreadPool()方法创建一个包含10个线程的线程池**。
- 接下来，我们**使用CompletableFuture.runAsync()方法将任务提交到线程池**，runAsync()方法确保提供给它的任务在单独的线程中异步运行。
- 该方法接收Callable或Runnable对象作为参数。在本例中，我们使用lambda表达式创建一个Runnable对象。
- runAsync()方法返回一个CompletableFuture对象，我们**将其添加到CompletableFuture对象列表中，以便稍后使用executorService实例中的线程池执行**。
- 接下来，我们使用CompletableFuture.allOf()方法组合CompletableFuture对象，并对它们调用join()操作。**执行join()时，进程会等待所有CompletableFuture任务并行完成**。
- 最后，我们使用shutdown()方法关闭ExecutorService，该方法释放线程池中的所有线程。

## 4. 使用流进行并行处理

Java 8引入了[Stream API](https://www.baeldung.com/java-8-streams-introduction)，它支持并行处理。让我们探讨一下Stream API如何并行化for循环。

### 4.1 使用并行流

让我们看看如何使用Stream API的parallel()方法来并行化for循环：

```java
void processParallelyWithStream() {
    IntStream.range(0, 100)
        .parallel()
        .forEach(i -> {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
}
```

在上面的代码中，我们使用IntStream.range()方法创建一个整数流。接下来，我们调用parallel()方法来并行化流。

最后，我们调用forEach()方法来处理流的元素。对于每个元素，我们调用Thread.sleep()方法来模拟繁重的操作。

### 4.2 使用StreamSupport

并行化for循环的另一种方法是使用[StreamSupport](https://www.baeldung.com/java-iterable-to-stream#converting-iterable-to-stream)类，让我们看一下相同的代码：

```java
void processParallelyWithStreamSupport() {
    Iterable<Integer> iterable = () -> IntStream.range(0, 100).iterator();
    Stream<Integer> stream = StreamSupport.stream(iterable.spliterator(), true);
    stream.forEach(i -> {
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
```

StreamSupport类提供了一个stream()方法，该方法接收Iterable对象作为参数。此外，它还需要一个布尔参数来指示流是否应该并行。

在这里，我们使用IntStream.range()方法创建一个Iterable对象。接下来，我们调用stream()方法来创建整数流。最后，我们调用forEach()方法来处理流的元素。

parallel()方法和StreamSupport类的工作方式类似，**它们在内部创建线程来处理流的元素，创建的线程数量取决于系统中可用的核心数量**。

## 5. 性能比较

现在我们已经了解了并行化for循环的不同方法，让我们比较一下每种方法的性能。为此，我们使用[Java Microbenchmark Harness](https://www.baeldung.com/java-microbenchmark-harness)(JMH)。首先，我们需要将[JMH依赖项](https://www.baeldung.com/java-microbenchmark-harness#start)添加到我们的项目中。

接下来，让我们将@BenchmarkMode注解添加到我们的方法中，并使它们能够针对平均时间进行基准测试：

```java
@Benchmark
@BenchmarkMode(Mode.AverageTime)
public void processSerially() throws InterruptedException {
    for (int i = 0; i < 100; i++) {
        Thread.sleep(10);
    }
}
```

同样，让我们对所有并行处理方法执行相同的操作。

为了运行基准测试，我们创建一个main()方法并设置JMH：

```java
class Benchmark {

    public static void main(String[] args) {
        try {
            org.openjdk.jmh.Main.main(new String[] { "cn.tuyucheng.taketoday.concurrent.parallel.Processor" });
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

从main()方法中，我们调用JMH的main()方法并将路径作为参数传递给Processor类。这告诉JMH对Processor类的方法运行基准测试。

当我们运行main()方法时，我们会看到以下结果：

![](/assets/images/2023/javaconcurrency/javaforloopparallel01.png)

从上面的结果可以看出，并行处理元素所花费的时间比顺序处理它们所花费的时间要少得多。

值得注意的是，**处理元素所花费的时间可能因系统而异**，这取决于系统中可用的核心数量。

此外，**每种并行方法每次运行所花费的时间可能会有所不同，并且这些数字并不是这些方法之间的精确比较**。

## 6. 总结

在本文中，我们研究了在Java中并行化for循环的不同方法。我们探讨了如何使用ExecutorService接口、Stream API和StreamSupport实用程序来并行化for循环。最后，我们使用JMH比较了每种方法的性能。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-2)上找到。