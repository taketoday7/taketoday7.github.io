---
layout: post
title:  Java 20中的新特性
category: java-new
copyright: java-new
excerpt: Java 20
---

## 1. 概述

Java 20于2023年3月21日发布，是迄今为止基于Java 19构建的最新短期增量版本，它由[JEP 2.0](https://cr.openjdk.org/~mr/jep/jep-2.0-02.html)中提到的7个重要的JDK增强提案(JEP)组成。JEP流程用于评估JDK增强功能的提案，Java 20中的大部分更新都是对早期版本中引入的功能的改进或增强。

此外，Oracle JDK 20不是长期支持版本。因此，它将收到更新，直到Java 21发布。

在本文中，我们将探讨其中的一些新功能。

## 2. ScopedValue([JEP 429](https://openjdk.org/jeps/429))

大量Java应用程序都有需要在彼此之间共享数据的组件或模块。通常，这些模块是基于线程的，因此我们必须保护它们共享的数据免受任何更改。

我们一直在使用[ThreadLocal](https://www.baeldung.com/java-threadlocal)类型的变量来允许组件共享数据。

但它会带来一些后果：

- ThreadLocal变量是可变的，ThreadLocal API允许访问其变量类型的get()和set()方法。
- 我们可能会面临内存泄漏问题，因为ThreadLocal变量的值会一直保留，直到我们显式调用它的remove()方法或线程退出为止。因此，没有绑定到这些每线程(per-thread)变量的生存期。
- 如果使用大量线程，它们可能会导致内存占用过多。这是因为子线程会继承父线程的ThreadLocal变量，从而为每个ThreadLocal变量分配内存。

为了克服ThreadLocal变量的这些问题，Java 20引入了作用域值，用于在线程内和线程间共享数据。

**作用域值提供了一个简单、不可变且可继承的数据共享选项，特别是在我们使用大量线程的情况下**。

[ScopedValue](https://www.baeldung.com/java-20-scoped-values)是一个不可变的值，可供线程在有限的执行时间内读取。由于它是不可变的，因此它允许在有限的线程执行时间内安全且轻松地共享数据。此外，我们不需要将值作为方法参数传递。

**我们可以使用[ScopedValue](https://download.java.net/java/early_access/loom/docs/api/jdk.incubator.concurrent/jdk/incubator/concurrent/ScopedValue.html)类的where()方法来为线程执行的有限周期设置变量的值。而且，一旦我们使用get()方法获取了数据，我们就无法再次访问它**。

一旦线程的run()方法完成执行，作用域值将恢复为未绑定状态。我们可以使用get()方法来读取线程内作用域值变量的值。

## 3. 记录模式([JEP 432](https://openjdk.org/jeps/432))

JDK 19已经引入了[记录模式作为预览功能](https://www.baeldung.com/java-19-record-patterns)。

Java 20提供了记录模式的改进和精炼版本，让我们看看此版本中的一些改进：

- 添加了对泛型记录模式参数的类型推断的支持。
- **添加了对记录模式的支持，使其可在[增强for循环](https://www.baeldung.com/java-for-loop#foreach)的标头中使用**。
- 删除了对命名记录模式的支持，我们可以在其中为记录模式提供可用于引用记录模式的可选标识符。

此版本的本质目的是通过模式匹配来表达更改进的可组合数据查询，同时不更改类型模式的语法或语义。

## 4. Switch模式匹配([JEP 433](https://openjdk.org/jeps/433))

Java 20为[switch表达式和语句提供了模式匹配](https://www.baeldung.com/java-switch-pattern-matching)的改进版本，特别是关于switch表达式中使用的语法。它首先在[Java 17](https://www.baeldung.com/java-17-new-features#7-pattern-matching-for-switch-preview-jep-406)中交付，随后在Java 18和19中进行了一些改进，从而扩展了switch语句和表达式的可用性和适用性。

此版本的主要变化包括：

- 现在，在枚举类上使用switch表达式或模式会引发MatchException。早些时候，如果在运行时没有应用switch标签，我们常常会收到IncompleteClassChangeError。
- switch标签的语法有所改进。
- 添加了对switch表达式和语句中泛型记录模式参数类型推断的支持，以及支持模式的其他构造。

## 5. 外部函数和内存API([JEP 434](https://openjdk.org/jeps/434))

Java 20合并了对先前Java版本中引入的[外部函数和内存(FFM)API](https://www.baeldung.com/java-17-new-features#12-foreign-function-and-memory-api-incubator-jep-412)的改进和细化，这是第二个[预览版API](https://openjdk.org/jeps/12)。

**外部函数和内存API允许Java开发人员从JVM外部访问代码并管理堆外的内存(即不由JVM管理的内存)**。FFM API旨在为[Java本机接口(JNI)](https://www.baeldung.com/jni)提供安全、改进且纯基于Java的替代品。

它包括以下改进：

- MemorySegment和MemoryAddress抽象是统一的，这意味着我们实际上可以从空间边界确定与段关联的内存地址范围。
- 还通过增强密封的MemoryLayout层次结构，促进了switch表达式和语句中模式匹配的使用。
- 此外，他们将MemorySession分为Arena和SegmentScope，以便于跨维护边界共享段。

## 6. 虚拟线程([JEP 436](https://openjdk.org/jeps/436))

[虚拟线程](https://www.baeldung.com/java-virtual-thread-vs-thread)最初在[JEP 425](https://openjdk.org/jeps/425)中作为预览功能提出，并在[Java 19](https://openjdk.org/projects/jdk/19/)中提供。Java 20提出第二个预览版，旨在收集更多使用后的反馈和改进建议。

虚拟线程是轻量级线程，可以减少编写、维护和观察高吞吐量并发应用程序的工作量。因此，使用现有的JDK工具可以轻松地对服务器应用程序进行调试和故障排除，这对于服务器应用程序的扩展很有用。

但是，我们应该注意到，传统的Thread实现仍然存在，并且它的目的并不是要取代Java的基本并发模型。

以下是自第一次预览以来的一些细微变化：

- 他们使之前在Thread中引入的方法：[join(Duration)](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/lang/Thread.html#join(java.time.Duration))、[sleep(Duration)](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/lang/Thread.html#sleep(java.time.Duration))和[threadId()](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/lang/Thread.html#threadId())在Java 20中永久存在。
- 同样，Future中新引入的用于检查任务状态和结果的方法：[state()](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/concurrent/Future.html#state())和[resultNow()](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/concurrent/Future.html#resultNow())在Java 20中已永久保留。
- 此外，[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)现在扩展了[AutoCloseable](https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html)。
- [ThreadGroup](https://openjdk.org/jeps/425#java-lang-ThreadGroup)是JEP 425中描述的用于分组线程的遗留API，在JDK 19中已永久降级。[ThreadGroup](https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/lang/ThreadGroup.html)不适合对虚拟线程进行分组。

## 7. 结构化并发([JEP 437](https://openjdk.org/jeps/437))

结构化并发由[JEP 428](https://openjdk.org/jeps/428)提出，并在[JDK 19](https://openjdk.org/projects/jdk/19/)中作为[孵化API](https://openjdk.org/jeps/11)提供。该JEP建议在JDK 20中重新孵化API，而无需进行太多更改。因此，它可以有时间获得更多反馈。

目标是通过引入结构化并发API来简化多线程编程，它通过将执行类似任务的多个线程分组到单个工作单元中来提高可靠性。因此，错误处理和线程取消得到了改进。此外，它还促进了一种改进的并发编程方式，旨在防止因线程取消而产生的常见风险。

但是，这个重新孵化的API有一个变化。**我们获得了更新的[StructuredTaskScope](https://docs.oracle.com/en/java/javase/20/docs/api/jdk.incubator.concurrent/jdk/incubator/concurrent/StructuredTaskScope.html)，它支持在任务作用域中创建的线程继承作用域值**。因此，我们现在可以方便地跨多个线程共享不可变数据。

## 8. Vector API([JEP 438](https://openjdk.org/jeps/438))

[Vector API](https://www.baeldung.com/java-16-new-features#vector-api-incubator-jep-338)最初由[JEP 338](https://openjdk.org/jeps/338)提出，并作为孵化API集成到JDK 16中。此版本(第五个孵化器)是多轮孵化和集成到所有先前Java版本的后续版本。

Vector API用于在支持的CPU架构上表达Java中的向量计算。向量计算实际上是向量上的一系列运算，[Vector API](https://www.baeldung.com/java-vector-api)旨在提供一种比传统标量计算更优化的矢量计算方法。

与Java 19相比，此版本没有对API进行任何更改。但是，它包含一些错误修复和性能增强。

最后，**Vector API的开发与[Project Valhalla](https://www.baeldung.com/java-valhalla-project)密切相关，因为它的目标是在未来适应Valhalla项目的增强功能**。

## 9. 总结

Java 20以增量方式构建在过去版本的几个功能之上，包括记录模式、Switch表达式的模式匹配、FFM、虚拟线程等等。它还添加了新的孵化功能(例如作用域值)以及对重新孵化功能(例如结构化并发和Vector API)的一些增强。

在本文中，我们讨论了作为增量Java 20版本的一部分引入的一些功能和更改。Java 20的完整更改列表可在[JDK发行说明](https://www.oracle.com/java/technologies/javase/20-relnote-issues.html)找到。