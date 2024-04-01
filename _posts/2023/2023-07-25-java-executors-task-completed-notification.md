---
layout: post
title:  如何在Java Executor中的任务完成时接收通知
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

Java为我们提供了一系列异步运行任务的选项，例如使用[Executor](https://www.baeldung.com/java-executor-service-tutorial)。通常我们想知道任务何时完成，例如提醒用户或开始下一个任务。在本教程中，我们将研究用于接收任务完成通知的不同选项，具体取决于我们最初运行任务的方式。

## 2. 设置

首先，让我们定义一个要运行的任务和一个回调接口，任务完成时我们希望通过该回调接口收到警报。

**对于我们的任务，我们将实现[Runnable](https://www.baeldung.com/java-runnable-vs-extending-thread)**。Runnable是当我们希望某些东西在线程中运行时可以使用的接口，我们必须重写run()方法并将我们的业务逻辑放入其中。对于我们的示例，我们只需打印到控制台，以便我们知道它已运行：

```java
class Task implements Runnable{
    @Override
    public void run() {
        System.out.println("Task in progress");
        // Business logic goes here
    }
}
```

然后让我们创建回调接口，我们将有一个接收String参数的方法。这是一个简单的示例，但我们可以在这里拥有任何需要的东西，以使我们的警报尽可能有用。使用接口使我们能够以更通用的方式实现我们的解决方案，这意味着我们可以根据我们的用例传入不同的CallbackInterface实现：

```java
interface CallbackInterface {
    void taskDone(String details);
}
```

现在让我们实现该接口：

```java
class Callback implements CallbackInterface {
    void taskDone(String details) {
        System.out.println("task complete: " + details);
        // Alerts/notifications go here
    }
}
```

我们将在每个任务运行选项中使用它们，这将清楚地表明我们正在完成相同的任务，并且每次结束时我们都会收到来自相同回调的警报。

## 3. Runnable实现

**我们将看到的第一个示例是Runnable接口的简单实现**。我们可以通过构造函数提供任务和回调，然后在重写的run()方法中调用两者：

```java
class RunnableImpl implements Runnable {
    Runnable task;
    CallbackInterface callback;
    String taskDoneMessage;

    RunnableImpl(Runnable task, CallbackInterface callback, String taskDoneMessage) {
        this.task = task;
        this.callback = callback;
        this.taskDoneMessage = taskDoneMessage;
    }

    void run() {
        task.run();
        callback.taskDone(taskDoneMessage);
    }
}
```

我们在这里的设置运行我们提供的任务，然后在任务完成后调用我们的回调方法。以下是它的实际效果：

```java
@Test
void whenImplementingRunnable_thenReceiveNotificationOfCompletedTask() {
    Task task = new Task();
    Callback callback = new Callback();
    RunnableImpl runnableImpl = new RunnableImpl(task, callback, "ready for next task");
    runnableImpl.run();
}
```

如果我们检查此测试的日志，我们会看到我们期望的两条消息：

```text
Task in progress
task complete ready for next task
```

当然，在真实的应用程序中，我们会有业务逻辑并发生真实的警报。

## 4. 使用CompletableFuture

**也许异步运行任务并在完成时收到警报的最简单选项是使用[CompletableFuture](https://www.baeldung.com/java-completablefuture)类**。 

CompletableFuture是在Java 8中引入的Future接口的实现，它允许我们按顺序执行多个任务，每个任务在前一个Future完成后运行。这对我们来说非常有用，因为我们可以运行我们的任务，并指示CompletableFuture之后运行我们的回调方法。

为了将该计划付诸行动，我们可以首先使用CompletableFuture的runAsync()方法接收一个Runnable并为我们运行它，返回一个CompletableFuture实例。然后，我们可以将thenAccept()方法与Callback.taskDone()方法作为参数链接起来，一旦我们的任务完成，就会调用该方法：

```java
@Test
void whenUsingCompletableFuture_thenReceiveNotificationOfCompletedTask() {
    Task task = new Task();
    Callback callback = new Callback();
    CompletableFuture.runAsync(task)
        .thenAccept(result -> callback.taskDone("completion details: " + result));
}
```

运行此命令会生成任务的输出，然后按预期进行回调实现。我们的任务不会返回任何内容，因此result为null，但根据我们的用例，我们可以以不同的方式处理它。

## 5. 扩展ThreadPoolExecutor

对于某些用例，例如，如果我们要一次提交许多任务，我们可能需要使用[ThreadPoolExecutor](https://www.baeldung.com/thread-pool-java-and-guava)。ThreadPoolExecutor允许我们使用实例化时指定的线程数来执行所需数量的任务。

**我们可以扩展ThreadPoolExecutor类并重写afterExecute()方法，这样它将在每个任务之后调用我们的回调实现**：

```java
class AlertingThreadPoolExecutor extends ThreadPoolExecutor {
    CallbackInterface callback;

    public AlertingThreadPoolExecutor(CallbackInterface callback) {
        super(1, 1, 60, TimeUnit.SECONDS, new ArrayBlockingQueue<>(10));
        this.callback = callback;
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        callback.taskDone("runnable details here");
    }
}
```

在我们的构造函数中，我们使用硬编码值调用了父类构造函数，以便为我们提供一个可供使用的线程。我们还将空闲线程的保持活动时间设置为60秒，并给自己一个可容纳10个任务的队列。我们可以根据我们的特定应用程序对此进行微调，但这就是我们需要的一个简单示例。正如预期的那样，运行时会给出以下输出：

```text
Task in progress
task complete runnable details here
```

## 6. 扩展FutureTask

我们的最后一个选择是扩展[FutureTask](https://www.baeldung.com/java-future)类。**FutureTask是Future接口的另一个实现，它还实现了Runnable，这意味着我们可以将其实例提交给ExecutorService**。这个类提供了一个我们可以重写的方法，名为done()，它将在提供的Runnable完成时被调用。

因此，考虑到这一点，我们在这里需要做的就是扩展FutureTask，重写done()方法，并调用通过构造函数传入的回调实现：

```java
class AlertingFutureTask extends FutureTask<String> {
    CallbackInterface callback;

    public AlertingFutureTask(Runnable runnable, Callback callback) {
        super(runnable, null);
        this.callback = callback;
    }

    @Override
    protected void done() {
        callback.taskDone("alert alert");
    }
}
```

我们可以通过创建FutureTask的实例和ExecutorService的实例来使用FutureTask的这个扩展，然后我们将FutureTask提交给ExecutorService：

```java
@Test
void whenUsingFutureTask_thenReceiveNotificationOfCompletedTask(){
    Task task = new Task();
    Callback callback = new Callback();
    FutureTask<String> future = new AlertingFutureTask(task, callback);
    ExecutorService executor = Executors.newSingleThreadExecutor();
    executor.submit(future);
}
```

该测试的输出符合预期-首先记录任务，然后是我们的回调实现记录：

```text
Task in progress
task complete: task details here
```

## 7. 总结

在本文中，我们研究了异步运行同一任务，然后接收指示其完成的警报的四种方法。其中三个选项涉及扩展或实现现有的Java类或接口，它们是Runnable接口以及ThreadPoolExecutor和FutureTask类。这些选项中的每一个都为我们提供了高水平的控制和大量的选项来获得我们想要的警报行为。

最后一个选项是使用CompletableFuture，它更加简化。我们不需要单独的类，我们可以在一两行中执行所有操作。对于简单的用例或者当我们将所有内容都很好地包装在回调和任务类中时，这可以很好地完成工作。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-1)上找到。