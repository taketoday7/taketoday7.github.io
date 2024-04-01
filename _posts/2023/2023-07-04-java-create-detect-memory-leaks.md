---
layout: post
title:  在Java中创建和检测内存泄漏
category: java
copyright: java
excerpt: Java Performance
---

## 1. 概述

在Java应用程序中，[内存泄漏](https://www.baeldung.com/java-memory-leaks#what-is-a-memory-leak)可能导致严重的性能下降和系统故障，开发人员必须了解内存泄漏是如何发生的以及如何识别和解决它们。

在本教程中，我们将使用失效监听器问题作为示例，提供有关在Java中创建内存泄漏的指南。我们还将讨论检测内存泄漏的各种方法，包括日志记录、分析、详细垃圾回收和堆转储。

## 2. 造成内存泄漏

我们将考虑[失效监听器问题](https://en.wikipedia.org/wiki/Lapsed_listener_problem)作为内存泄漏的示例，这是了解Java内存分配和垃圾回收的绝佳方法。

让我们创建一个应用程序，向登录并订阅我们服务的用户发送随机电影报价。该应用程序非常简单，一次只能为一个用户提供服务：

```java
public static void main(String[] args) {
    while (true) {
        User user = generateUser();
        logger.debug("{} logged in", user.getName());
        user.subscribe(movieQuoteService);
        userUsingService();
        logger.debug("{} logged out", user.getName());
    }
}
```

UserGenerator是一个简单的类，提供无限量的随机用户。我们将使用[Datafaker](https://www.baeldung.com/java-datafaker)进行随机化：

```java
public class UserGenerator {

    private final static Faker faker = new Faker();

    public static User generateUser() {
        System.out.println("Generating user");
        String name = faker.name().fullName();
        String email = faker.internet().emailAddress();
        String phone = faker.phoneNumber().cellPhone();
        String street = faker.address().streetAddress();
        String city = faker.address().city();
        String state = faker.address().state();
        String zipCode = faker.address().zipCode();
        return new User(name, email, phone, street, city, state, zipCode);
    }
}
```

用户和我们的服务之间的关系将基于[观察者模式](https://www.baeldung.com/java-observer-pattern)，因此，用户可以订阅该服务，我们的MovieQuoteService将为用户更新新的电影报价。

**此示例的主要问题是用户永远不会取消订阅该服务**，这会造成内存泄漏，因为即使用户超出范围，垃圾回收器也无法将其删除，因为服务保存了他们的引用。

我们可以显示取消订阅用户来缓解这个问题，这会起作用。**但是，最好的解决方案是使用[WeakReferences](https://www.baeldung.com/java-weak-reference)来自动化此过程**。

## 3. 检测内存泄漏

在上一节中，我们创建了一个存在严重问题的应用程序-内存泄漏。尽管这个问题可能是灾难性的，但通常很难被发现。让我们回顾一下可以找到此问题的一些方法。

### 3.1 日志记录

让我们从最直接的方法开始，使用日志记录来查找系统的问题。这不是检测内存泄漏的最先进方法，但它很容易使用，并且可能有助于发现异常情况。

运行我们的服务时，日志输出将向我们显示用户活动：

```text
21:58:24.280 [pool-1-thread-1] DEBUG c.b.lapsedlistener.MovieQuoteService - New quote: Go ahead, make my day.
21:58:24.358 [main] DEBUG c.t.t.l.LapsedListenerRunner - Earl Runolfsdottir logged in
21:58:24.358 [main] DEBUG c.t.t.lapsedlistener.MovieQuoteService - Current number of subscribed users: 0
21:58:24.371 [main] DEBUG c.t.t.l.LapsedListenerRunner - Earl Runolfsdottir logged out
21:58:24.372 [main] DEBUG c.t.t.l.LapsedListenerRunner - Barbra Rosenbaum logged in
21:58:24.372 [main] DEBUG c.t.t.lapsedlistener.MovieQuoteService - Current number of subscribed users: 1
21:58:24.383 [main] DEBUG c.t.t.l.LapsedListenerRunner - Barbra Rosenbaum logged out
21:58:24.383 [main] DEBUG c.t.t.l.LapsedListenerRunner - Leighann McCullough logged in
21:58:24.383 [main] DEBUG c.t.t.lapsedlistener.MovieQuoteService - Current number of subscribed users: 2
21:58:24.396 [main] DEBUG c.t.t.l.LapsedListenerRunner - Leighann McCullough logged out
21:58:24.397 [main] DEBUG c.t.t.l.LapsedListenerRunner - Mr. Charlie Keeling logged in
21:58:24.397 [main] DEBUG c.t.t.lapsedlistener.MovieQuoteService - Current number of subscribed users: 3
21:58:24.409 [main] DEBUG c.t.t.l.LapsedListenerRunner - Mr. Charlie Keeling logged out
21:58:24.410 [main] DEBUG c.t.t.l.LapsedListenerRunner - Alvin O'Connell logged in
21:58:24.410 [main] DEBUG c.t.t.lapsedlistener.MovieQuoteService - Current number of subscribed users: 4
21:58:24.423 [main] DEBUG c.t.t.l.LapsedListenerRunner - Alvin O'Connell logged out
21:58:24.423 [main] DEBUG c.t.t.l.LapsedListenerRunner - Tracey Stoltenberg logged in
21:58:24.423 [main] DEBUG c.t.t.lapsedlistener.MovieQuoteService - Current number of subscribed users: 5
```

我们可以在前面的片段中注意到一件有趣的事情，如前所述，我们的应用程序一次只能处理一个用户。

因此，只能有一个用户订阅我们的服务。**同时，日志显示订阅者数量超过了该值，进一步阅读可以提供更多证据证明我们的系统存在问题**。

尽管日志没有显示问题发生的位置，但这是防止我们的系统出现问题的第一步。

### 3.2 分析

与上一步一样，此步骤旨在查找工作应用程序中的异常情况。但是，[分析器](https://www.baeldung.com/java-profilers)可以极大地简化对工作应用程序的内存占用的监控。

**第一个泄露是使用的内存随着时间的推移单调递增**，这并不总是内存泄漏的迹象。然而，在像我们这样的应用程序上，内存使用量的增加可能是一个好兆头，表明我们遇到了问题。

我们将使用JConsole分析器，这是一个基本的分析器，但它提供了所有需要的功能，并且包含在每个JDK发行版中。此外，在任何系统上启动它都很容易：

```shell
$ jconsole
```

让我们启动我们的应用程序，看看JConsole会告诉我们什么。启动应用程序后，其内存消耗增加：

[![内存消耗增加](https://www.baeldung.com/wp-content/uploads/2023/04/2_Screen-Shot-2023-04-25-at-19.51.52.png)](https://www.baeldung.com/wp-content/uploads/2023/04/2_Screen-Shot-2023-04-25-at-19.51.52.png)

但是，内存使用情况并不总是内存泄漏的迹象。让我们尝试提示垃圾回收器清理一些死亡对象：

[![垃圾回收器清理](https://www.baeldung.com/wp-content/uploads/2023/04/2_Screen-Shot-2023-04-25-at-19.52.29.png)](https://www.baeldung.com/wp-content/uploads/2023/04/2_Screen-Shot-2023-04-25-at-19.52.29.png)

正如我们所看到的，垃圾回收器工作得很好并且清理了一些空间。因此，我们可以假设我们根本没有任何问题。但是，让我们看看老年代，这是在我们的应用程序中经过多次垃圾回收一段时间后仍然存在的对象的[空间](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)。我们可以看到它的大小不断增加：

[![老一辈](https://www.baeldung.com/wp-content/uploads/2023/04/2_Screen-Shot-2023-04-25-at-19.52.52.png)](https://www.baeldung.com/wp-content/uploads/2023/04/2_Screen-Shot-2023-04-25-at-19.52.52.png)

一种解释是，除了用户之外，我们还有报价。我们不会在应用程序中存储对报价的引用，因此垃圾回收器在清理它们时没有问题。**同时，我们的服务保留每个用户的引用，防止他们被垃圾回收，并且他们被提升到老年代**：

[![参考](https://www.baeldung.com/wp-content/uploads/2023/04/2_Heap-Over-Time.png)](https://www.baeldung.com/wp-content/uploads/2023/04/2_Heap-Over-Time.png)

尽管垃圾回收器会定期进行清理，但很明显，总体内存消耗会随着时间的推移而增加。**几分钟内我们的大小就从大约10MB增加到了30MB**，这可能在数小时甚至数天内不会对服务器造成任何问题。如果服务器定期重新启动，我们可能永远不会看到OutOfMemoryError：

[![内存不足错误](https://www.baeldung.com/wp-content/uploads/2023/04/2_Old-Gen-Over-Time.png)](https://www.baeldung.com/wp-content/uploads/2023/04/2_Old-Gen-Over-Time.png)

我们在老年代中也有同样的情况：内存消耗只会增长。对于我们的应用程序来说，一次只能为一个用户提供服务，这是问题的征兆。

### 3.3 详细垃圾回收

这是检查堆状态和垃圾回收过程的另一种方法。根据Java版本，我们可以使用几个标志来打开[详细垃圾回收](https://www.baeldung.com/java-verbose-gc)。日志上的输出将反映我们之前在JConsole中获得的信息：

```shell
[0.004s][info][gc] Using G1
[0.210s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 23M->6M(392M) 1.693ms
[33.169s][info][gc] GC(1) Pause Young (Normal) (G1 Evacuation Pause) 38M->7M(392M) 1.994ms
[250.890s][info][gc] GC(2) Pause Young (Normal) (G1 Evacuation Pause) 203M->16M(392M) 11.420ms
[507.259s][info][gc] GC(3) Pause Young (Normal) (G1 Evacuation Pause) 228M->25M(392M) 14.321ms
[786.181s][info][gc] GC(4) Pause Young (Normal) (G1 Evacuation Pause) 229M->33M(392M) 17.410ms
[1073.277s][info][gc] GC(5) Pause Young (Normal) (G1 Evacuation Pause) 241M->41M(392M) 11.251ms
[1341.717s][info][gc] GC(6) Pause Young (Normal) (G1 Evacuation Pause) 241M->48M(392M) 17.132ms
```

这些日志使用[特定格式](https://www.baeldung.com/java-verbose-gc#2-the-verbose-output-in-more-detail)来显示总体内存消耗随着时间的推移而增加，这是检查应用程序内存占用并查找问题的一种非常快速且直接的方法。

但是，在此步骤之后，我们需要找到这个问题的原因。在我们的应用程序中，这个任务可能很简单，只有几个类，我们可以通过检查代码来解决它。但是，对大型应用程序中的代码通过观察可能无法检测到问题。

### 3.4 堆转储

有多种方法可以捕获堆转储，并且JDK包含多种[控制台工具](https://www.baeldung.com/java-heap-dump-capture#jdk-tools)，我们将使用[VisualVM](https://www.baeldung.com/java-heap-dump-capture#3jvisualvm)捕获和读取堆转储：

[![VisualVM 堆转储](https://www.baeldung.com/wp-content/uploads/2023/04/2_Heap-Dump-VisualVM-e1682520077803.png)](https://www.baeldung.com/wp-content/uploads/2023/04/2_Heap-Dump-VisualVM-e1682520077803.png)

这是一个捕获堆转储的便捷工具，包含JConsole的所有功能，使该过程变得简单。

捕获堆转储后，我们可以对其进行查看和分析。在我们的例子中，我们将尝试找到不应该存在的活动对象。幸运的是，VisualVM会生成堆转储的摘要，其中显示了重要信息：

[![VisualVM总结](https://www.baeldung.com/wp-content/uploads/2023/04/2_VisualVM-Summary.png)](https://www.baeldung.com/wp-content/uploads/2023/04/2_VisualVM-Summary.png)

我们系统中的用户在实例数量和整体大小方面排名第3，我们已经知道存在内存消耗问题，现在我们找到了罪魁祸首。

此外，VisualVM允许我们更彻底地分析堆转储并检查堆中的所有实例：

[![堆](https://www.baeldung.com/wp-content/uploads/2023/04/1_Screen-Shot-2023-04-25-at-21.08.34-e1682520157113.png)](https://www.baeldung.com/wp-content/uploads/2023/04/1_Screen-Shot-2023-04-25-at-21.08.34.png)

这对于具有复杂对象交互的大型应用程序可能会有所帮助。此外，这对于调整应用程序和查找有问题的地方可能很有用。

找到有问题的实例后，我们仍然需要检查代码以查看内存泄漏何时出现，但现在我们可以缩小排查范围。

## 4. 总结

内存泄漏会严重影响Java应用程序，导致内存逐渐耗尽和潜在的系统故障。在本教程中，我们出于演示目的创建了内存泄漏，并讨论了各种检测技术，包括日志记录、分析、详细垃圾回收和堆转储。

每种方法都提供了有关应用程序运行时行为和内存消耗的宝贵见解。日志记录有助于识别异常情况，同时分析和详细垃圾回收日志可监视内存使用情况和垃圾回收过程。堆转储可识别有问题的对象及其引用，从而缩小内存泄漏的来源范围。

**了解Java中的内存分配和垃圾回收有助于开发人员防止内存泄漏并构建更高效、更健壮的应用程序**。与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-perf-2)上获得。