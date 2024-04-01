---
layout: post
title:  CompletableFuture allOf().join()与CompletableFuture.join()
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本文中，我们将探讨CompletableFuture.allOf()方法的细节，并了解使用它与在多个单独的CompletableFuture实例上调用join()之间的区别。我们会发现allOf()使我们能够以非阻塞的方式继续我们的流程，同时还确保原子性。

## 2. join()与allOf()

**CompletableFuture是Java 8中引入的一个强大功能，可促进和简化非阻塞代码的创建。在本文中，我们将重点介绍两种支持并行代码执行的方法：join()和allOf()**。

我们首先分析这两种方法的内部工作原理。接下来，我们将深入研究他们实现共同目标、并行执行代码以及随后合并结果的不同方法。对于本文的代码片段，我们将使用两个辅助函数，它们会阻塞线程一段时间，然后返回一些数据或引发异常：

```java
private CompletableFuture waitAndReturn(long millis, String value) {
    return CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(millis);
            return value;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    });
}

private CompletableFuture waitAndThrow(long millis) {
    return CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(millis);
        } finally {
            throw new RuntimeException();
        }
    });
}
```

### 2.1 join()

**CompletableFuture API公开join()方法，作为通过阻塞线程直到执行完成来检索Future对象值的一种方式**。我们应该注意到，即使CompletableFuture的执行发生在[不同的线程](https://www.baeldung.com/java-completablefuture-non-blocking)上，调用者线程也会被阻塞：

```java
CompletableFuture<String> future = waitAndReturn(1_000, "Harry");
assertEquals("Harry", future.join());
```

**此外，如果CompletableFuture完成时出现错误，join()会将其作为RuntimeException抛出**： 

```java
CompletableFuture<String> futureError = waitAndThrow(1_000);
assertThrows(RuntimeException.class, futureError::join);
```

### 2.2 allOf()

**静态方法allOff()允许我们组合多个CompletableFuture实例并返回一个CompletableFuture<Void\>**，结果对象的完成取决于所有后续Future的完成。而且，如果后续Future中任何一个异常完成，则整体结果也将被视为失败。重要的是要了解allOf()不是阻塞方法，这意味着它将立即执行：

```java
CompletableFuture<String> f1 = waitAndReturn(1_000, "Harry");
CompletableFuture<String> f2 = waitAndReturn(2_000, "Ron");

CompletableFuture<Void> combinedFutures = CompletableFuture.allOf(f1, f2);
```

但是，为了提取值，我们需要调用API的其他方法。例如，如果我们对生成的CompletableFuture<Void\>调用join()，线程将等待两个组合CompletableFuture对象完成-每个对象都在自己的线程中。换句话说，调用者线程将被阻塞的时间与长Future完成所需的时间相同：

```java
combinedFutures.join();
```

由于主线程已经等待了两秒，因此f1和f2现已完成，后续的join()或get()等调用将立即执行：

```java
assertEquals("Harry", f1.join());
assertEquals("Ron", f2.join());
```

### 2.3 并行执行代码

从前面的示例中我们可以注意到，**我们可以并行执行CompletableFutures并只需对每个结果调用join()即可组合结果**：

```java
CompletableFuture<String> f1 = waitAndReturn(1_000, "Harry");
CompletableFuture<String> f2 = waitAndReturn(2_000, "Ron");

sayHello(f1.join());
sayHello(f2.join());
```

或者通过迭代CompletableFuture的Collection或Stream，对它们中的每一个调用join()并消费它们的结果：

```java
Stream.of(f1, f2).map(CompletableFuture::join).forEach(this::sayHello);
```

现在的问题是在迭代和拼接所有CompletableFuture之前使用静态allOf()方法是否会对最终结果产生任何影响：

```java
CompletableFuture.allOf(f1, f2).join();
Stream.of(f1, f2).map(CompletableFututre::join).forEach(this::sayHello);
```

这两种方法之间有两个显著区别：错误处理和以非阻塞方式进行的能力。让我们深入研究它们并了解它们的特殊性。

## 3. 错误处理

**两种方法之间的主要区别之一是，如果我们省略调用allOf，我们将按顺序处理CompletableFuture的结果。因此，我们最终可以对值进行部分处理**。

换句话说，如果其中一个CompletableFuture抛出异常，它将破坏链并停止处理。在某些情况下，这可能会导致错误，因为前面的元素已经处理过：

```java
CompletableFuture<String> f1 = waitAndReturn(1_000, "Harry");
CompletableFuture<String> f2 = waitAndThrow(1_000);
CompletableFuture<String> f3 = waitAndReturn(1_000, "Ron");

Stream.of(f1, f2, f3)
    .map(CompletableFuture::join)
    .forEach(this::sayHello);
```

**另一方面，我们可以使用allOf()组合三个实例，然后调用join()方法来实现某种原子性。通过这样做，我们要么立即处理所有元素，要么不处理任何元素**：

```java
CompletableFuture.allOf(f1, f2, f3).join();
Stream.of(f1, f2, f3)
    .map(CompletableFuture::join) 
    .forEach(this::sayHello);
```

## 4. 非阻塞代码

allOf()的优点之一是它允许我们以非阻塞的方式继续我们的流程。**由于返回类型是CompletableFuture<Void\>，我们可以使用thenAccept()在数据到达时处理数据，而不会阻塞线程**：

```java
CompletableFuture.allOf(f1, f2, f3)
    .thenAccept(__ -> sayHelloToAll(f1.join(), f2.join(), f3.join()));
```

类似地，如果我们需要合并来自不同Future的数据，我们可以使用thenApply()方法。例如，我们可以拼接三个future的值，并使用结果字符串继续非阻塞流程：

```java
CompletableFuture<String> names = CompletableFuture.allOf(f1, f2, f3)
    .thenApply(__ -> f1.join() + "," + f2.join() + "," + f3.join());
```

**此外，如果我们不离开异步世界并继续CompletableFuture链，我们将能够利用它自己的机制进行错误处理和恢复，即exceptionally()方法**。例如，如果其中一个CompletableFuture完成时出现异常，我们可以简单地记录它并使用默认值继续流程：

```java
CompletableFuture<String> names = CompletableFuture.allOf(f1, f2, f3)
    .thenApply(__ -> f1.join() + "," + f2.join() + "," + f3.join())
    .exceptionally(err -> {
        System.out.println("oops, there was a problem! " + err.getMessage());
        return "names not found!";
    });
```

## 5. 总结

在本文中，我们学习了如何使用CompletableFuture的join()进行并行代码执行，同时无缝合并结果。我们还揭示了allOf()的优点，join()允许我们以原子方式处理数据。换句话说，只有当所有组成的CompletableFuture对象都成功完成时，流程才会继续。

最后，我们发现我们可以使用allOf()并省略调用join()。这将使我们能够使用多个CompletableFuture的结果，同时继续非阻塞流程。我们通过API的其他有用方法实现了这一点，例如thenApply()、theAccept()和exceptionally()。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上找到。