---
layout: post
title:  使用CompletableFuture重试逻辑
category: java-concurrency
copyright: java-concurrency
excerpt: Java CompletableFuture
---

## 1. 概述

在本文中，我们将学习如何将重试逻辑应用于[CompletableFuture](https://www.baeldung.com/java-completablefuture)对象。最初，我们将重试包含在CompletableFuture中的任务。接下来，我们将利用CompletableFuture API创建多个实例链，使我们能够在Future遇到异常完成时重新执行任务。

## 2. 重试任务

**重试任务的一个简单方法是利用[装饰器模式](https://www.baeldung.com/java-decorator-pattern)并使用带有类和接口的经典OOP风格来实现它**。另一方面，我们可以选择更简洁、更实用的方法，利用高阶函数。

最初，我们将声明一个函数，该函数将Supplier<T\>和最大调用次数作为参数。之后，如果需要，我们将使用while循环和try-catch块多次调用该函数。最后，我们将通过返回另一个Supplier<T\>来保留原始数据类型：

```java
static <T> Supplier<T> retryFunction(Supplier<T> supplier, int maxRetries) {
    return () -> {
        int retries = 0;
	    while (retries < maxRetries) {
	        try {
	            return supplier.get();
	        } catch (Exception e) {
	            retries++;
	        }
        }
	    throw new IllegalStateException(String.format("Task failed after %s attempts", maxRetries));
    };
}
```

我们可以通过允许重试特定异常的定义或在调用之间引入延迟来进一步改进此装饰器。但是，为了简单起见，让我们继续基于此函数装饰器创建CompletableFuture：

```java
static <T> CompletableFuture<T> retryTask(Supplier<T> supplier, int maxRetries) {
    Supplier<T> retryableSupplier = retryFunction(supplier, maxRetries);
    return CompletableFuture.supplyAsync(retryableSupplier);
}
```

现在，让我们继续为此功能编写测试。首先，我们需要一个由CompletableFuture重试的方法。为此，我们将设计一个方法，该方法通过抛出RuntimeExceptions失败四次，并在第五次尝试时成功完成，返回一个整数值：

```java
AtomicInteger retriesCounter = new AtomicInteger(0);

@BeforeEach
void beforeEach() {
    retriesCounter.set(0);
}

int failFourTimesThenReturn(int returnValue) {
    int retryNr = retriesCounter.get();
    if (retryNr < 4) {
        retriesCounter.set(retryNr + 1);
	throw new RuntimeException();
    }
    return returnValue;
}
```

现在，我们终于可以测试retryTask()函数并断言返回了预期值。此外，我们可以通过询问retriesCounter来检查调用次数：

```java
@Test
void whenRetryingTask_thenReturnsCorrectlyAfterFourInvocations() {
    Supplier<Integer> codeToRun = () -> failFourTimesThenReturn(100);

    CompletableFuture<Integer> result = retryTask(codeToRun, 10);

    assertThat(result.join()).isEqualTo(100);
    assertThat(retriesCounter).hasValue(4);
}
```

此外，如果我们使用较小的maxRetires参数值调用相同的函数，我们将期望Future能够异常完成。原始的IllegalStateException应包装到CompletionException中，但应保留原始的错误消息：

```java
@Test
void whenRetryingTask_thenThrowsExceptionAfterThreeInvocations() {
    Supplier<Integer> codeToRun = () -> failFourTimesThenReturn(100);

    CompletableFuture<Integer> result = retryTask(codeToRun, 3);

    assertThatThrownBy(result::join)
        .isInstanceOf(CompletionException.class)
        .hasMessageContaining("IllegalStateException: Task failed after 3 attempts");
}
```

## 3. 重试CompletableFuture

**CompletableFuture API提供了在异常发生时进行处理的选项。因此，我们可以使用exceptionally()等方法，而不是创建函数装饰器**。

### 3.1 不安全重试

exceptionally()方法使我们能够指定一个替代函数，当初始调用完成但出现异常时将调用该函数。例如，如果我们打算重试调用两次，我们可以利用流式API添加其中两个回退：

```java
static <T> CompletableFuture<T> retryTwice(Supplier<T> supplier) {
    return CompletableFuture.supplyAsync(supplier)
        .exceptionally(__ -> supplier.get())
        .exceptionally(__ -> supplier.get());
}
```

由于我们需要可变次数的重试，因此我们重构代码并使用for循环：

```java
static <T> CompletableFuture<T> retryUnsafe(Supplier<T> supplier, int maxRetries) {
    CompletableFuture<T> cf = CompletableFuture.supplyAsync(supplier);
    for (int i = 0; i < maxRetries; i++) {
        cf = cf.exceptionally(__ -> supplier.get());
    }
    return cf;
}
```

我们可以使用相同的测试工具来测试retryUnsafe()并预测类似的结果，尽管如此，如果初始Supplier在最终的CompletableFuture及其所有exceptionally()回退创建之前完成，将会有一个微妙的区别。**在这种情况下，该函数确实会重试指定的次数，但这个重试过程会发生在主线程上，导致[异步性](https://www.baeldung.com/java-completablefuture-non-blocking)的损失**。

为了说明这一点，我们可以在for循环之前插入一个100毫秒的暂停，该循环会迭代调用exceptionally()方法。

```java
static <T> CompletableFuture<T> retryUnsafe(Supplier<T> supplier, int maxRetries) {
    CompletableFuture<T> cf = CompletableFuture.supplyAsync(supplier);  
    sleep(100l);
    for (int i = 0; i < maxRetries; i++) {
        cf = cf.exceptionally(__ -> supplier.get());
    }
    return cf;
}
```

接下来，我们将修改failFourTimesThenReturn()测试方法以记录每次调用此方法时的尝试次数和当前线程名称。现在，让我们重新运行测试并检查控制台：

```text
invocation: 0, thread: ForkJoinPool.commonPool-worker-1
invocation: 1, thread: main
invocation: 2, thread: main
invocation: 3, thread: main
invocation: 4, thread: main
```

正如预期的那样，后续调用由主线程执行。如果初始调用速度很快，但后续调用预计会较慢，这可能会出现问题。

### 3.2 异步重试

我们可以通过确保后续调用异步执行来解决这个问题。为了实现这一点，从Java 12开始，API中引入了一种专用方法。**通过使用exceptionallyAsync()，我们将确保所有重试都将异步执行，无论初始CompletableFuture完成的速度如何**：

```java
static <T> CompletableFuture<T> retryExceptionallyAsync(Supplier<T> supplier, int maxRetries) {
   CompletableFuture<T> cf = CompletableFuture.supplyAsync(supplier);
   for (int i = 0; i < maxRetries; i++) {
      cf = cf.exceptionallyAsync(__ -> supplier.get());
   }
   return cf;
}
```

让我们快速运行测试并检查日志：

```text
invocation: 0, thread: ForkJoinPool.commonPool-worker-1
invocation: 1, thread: ForkJoinPool.commonPool-worker-1
invocation: 2, thread: ForkJoinPool.commonPool-worker-1
invocation: 3, thread: ForkJoinPool.commonPool-worker-2
invocation: 4, thread: ForkJoinPool.commonPool-worker-2
```

正如预期的那样，主线程没有执行任何调用。

### 3.3 嵌套CompletableFuture

**如果我们需要一个与Java 12之前版本兼容的解决方案，我们可以手动增强第一个示例以实现完全异步**。为了实现这一点，我们必须确保回退在新的CompletableFuture中异步执行：

```java
cf.exceptionally(__ -> CompletableFuture.supplyAsync(supplier))
```

然而，上面的代码不会编译，因为数据类型不匹配，但我们可以通过三个步骤修复它。首先，我们需要双重嵌套最初的Future，我们可以通过completedFuture()轻松做到这一点：

```java
CompletableFuture<CompletableFuture<T>> temp = cf.thenApply(value -> CompletableFuture.completedFuture(value));
```

现在类型是匹配的，所以我们可以安全地应用exceptionally()回退：

```java
temp = temp.exceptionally(__ -> CompletableFuture.supplyAsync(supplier));
```

最后，我们将使用thenCompose()来展平对象并返回到原始类型：

```java
cf = temp.thenCompose(t -> t);
```

最后，让我们将所有内容结合起来，创建一个具有可变数量的异步回退的CompletableFuture。此外，让我们利用流式API、方法引用和工具函数来保持代码简洁：

```java
static <T> CompletableFuture<T> retryNesting(Supplier<T> supplier, int maxRetries) {
    CompletableFuture<T> cf = CompletableFuture.supplyAsync(supplier);
    for (int i = 0; i < maxRetries; i++) {
        cf = cf.thenApply(CompletableFuture::completedFuture)
	        .exceptionally(__ -> CompletableFuture.supplyAsync(supplier))
	        .thenCompose(Function.identity());
    }
    return cf;
}
```

## 4. 总结

在本文中，我们探讨了在CompletableFuture中重试函数调用的概念。我们首先深入研究函数式风格的装饰器模式的实现，使我们能够重试函数本身。

随后，我们利用CompletableFuture API来完成相同的任务，同时保持异步流程。我们的发现包括Java 12中引入的exceptionallyAsync()方法，它非常适合此目的。最后，我们提出了一种替代方法，仅依赖于原始Java 8 API中的方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上获得。