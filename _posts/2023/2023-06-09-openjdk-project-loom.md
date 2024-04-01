---
layout: post
title:  OpenJDK Project Loom
category: java-new
copyright: java-new
excerpt: Java 19
---

## 1. 概述

在本文中，我们将快速浏览[Project Loom](https://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html)。**从本质上讲，Project Loom的主要目标是在Java中支持高吞吐量，轻量级的并发模型**。

## 2. Project Loom

Loom项目是OpenJDK社区向Java引入轻量级并发结构的一次尝试。到目前为止，Loom的原型已经对JVM和Java库进行了更改。

虽然Loom还没有计划发布，但我们可以在[Project Loom的wiki](https://wiki.openjdk.java.net/display/loom/Main)上访问最近的原型。

在讨论Loom的各种概念之前，让我们先讨论一下Java目前的并发模型。

## 3. Java的并发模型

目前，Thread代表了Java中并发的核心抽象。这种抽象以及其他[并发API](https://www.baeldung.com/java-util-concurrent)使得编写并发应用程序变得容易。

但是，由于Java使用[操作系统内核](https://www.baeldung.com/cs/os-kernel)线程来实现，因此它无法满足当今的并发要求。特别是有两个主要问题：

1.  线程无法与域的并发单位规模相匹配。例如，应用程序通常允许多达数百万个事务、用户或会话。但是，内核支持的线程数要少得多。因此，**针对每个用户、事务或会话的线程模型通常是不可行的**。
2.  大多数并发应用程序需要为每个请求在线程之间进行一些同步。因此，**操作系统线程之间会发生代价高昂的上下文切换**。

**此类问题的可能解决方案是使用异步并发API，常见的例子是[CompletableFuture](https://www.baeldung.com/java-completablefuture)和[RxJava](https://www.baeldung.com/rx-java)**。只要此类API不阻塞内核线程，它就会在Java线程之上为应用程序提供更细粒度的并发构造。

另一方面，**此类API更难调试和与遗留API集成**。因此，需要一种独立于内核线程的轻量级并发结构。

## 4. 任务和调度器

线程的任何实现，无论是轻量级还是重量级，都依赖于两个构造：

1.  任务(也称为延续-continuation)：可以为某些阻塞操作暂停自身的指令序列
2.  调度程序：用于将延续分配给CPU并从暂停的延续中重新分配CPU

目前，**Java的延续和调度程序都依赖于操作系统实现**。

现在，为了挂起延续，需要存储整个调用堆栈。同样，在恢复时检索调用堆栈。**由于延续的操作系统实现包括本机调用堆栈和Java的调用堆栈，因此会占用大量空间**。

不过，更大的问题是操作系统调度程序的使用。由于调度程序在内核模式下运行，因此线程之间没有区别。它以相同的方式处理每个CPU请求。

这种类型的调度对于Java应用程序来说并不是最佳的。

例如，考虑一个应用程序线程，它对请求执行某些操作，然后将数据传递给另一个线程以进行进一步处理。在这里，**最好将这两个线程安排在同一个CPU上**。但由于调度程序对请求CPU的线程是不可知的，因此无法保证这一点。

**Loom项目建议通过用户模式线程来解决这个问题，该线程依赖于延续和调度程序的Java运行时实现，而不是操作系统实现**。

## 5. 纤程

在OpenJDK的最新原型中，一个名为Fiber的新类与Thread类一起被引入到库中。

由于计划中的Fiber库与Thread相似，因此用户实现也应该保持相似。但是，有两个主要区别：

1.  **Fiber会将任何任务包装在内部用户模式延续中，这将允许任务在Java运行时而不是内核中挂起和恢复**
2.  将使用可插拔的用户模式调度程序(例如ForkJoinPool)

让我们详细了解这两项。

## 6. 延续

延续(或协程)是一系列指令，可以在稍后阶段由调用者产生和恢复。

每个延续都有一个入口点和一个屈服点，屈服点是它暂停的地方。每当调用者恢复延续时，控制将返回到最后一个屈服点。

**重要的是要意识到这种挂起/恢复现在发生在语言运行时而不是操作系统中**。因此，它可以防止内核线程之间昂贵的上下文切换。

与线程类似，Project Loom旨在支持嵌套纤程。由于纤程在内部依赖延续，因此它还必须支持嵌套延续。为了更好地理解这一点，考虑一个允许嵌套的类Continuation：

```java
Continuation cont1 = new Continuation(() -> {
    Continuation cont2 = new Continuation(() -> {
        //do something
        suspend(SCOPE_CONT_2);
        suspend(SCOPE_CONT_1);
    });
});
```

如上所示，嵌套延续可以通过传递作用域变量来暂停自身或任何封闭延续。因此，**它们被称为作用域延续**。

由于挂起延续也需要它存储调用堆栈，因此Loom项目的目标也是在恢复延续时添加轻量级堆栈检索。

## 7. 调度器

前面，我们讨论了操作系统调度程序在同一CPU上调度相关线程的缺点。

尽管Project Loom的目标是允许使用纤程的可插拔调度器，但**异步模式下的[ForkJoinPool](https://www.baeldung.com/java-fork-join)将用作默认调度器**。

ForkJoinPool使用**工作窃取算法**。因此，每个线程都维护一个任务双端队列并从其头部执行任务。此外，任何空闲线程都不会阻塞，而是**等待任务并将其从另一个线程的双端队列的尾部拉出**。

异步模式的唯一区别是**工作线程从另一个双端队列的头部窃取任务**。

**ForkJoinPool将另一个正在运行的任务调度的任务添加到本地队列。因此，在同一个CPU上执行它**。

## 8. 总结

在本文中，我们讨论了Java当前并发模型中存在的问题以及[Project Loom](https://cr.openjdk.java.net/~rpressler/loom/Loom-Proposal.html)提出的更改。

在此过程中，我们还定义了任务和调度程序，并研究了Fiber和ForkJoinPool如何使用内核线程提供Java的替代方案。