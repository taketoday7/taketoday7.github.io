---
layout: post
title:  了解Java中的内存泄漏
category: java
copyright: java
excerpt: Java Performance
---

## 1. 概述

Java的核心优势之一是在内置垃圾回收器(或简称GC)的帮助下进行自动内存管理。GC隐式地负责分配和释放内存，因此能够处理大多数内存泄漏问题。

虽然GC可以有效地处理大部分内存，但它并不能保证万无一失的内存泄漏解决方案。GC非常聪明，但并非完美无缺。内存泄漏仍然可能悄悄发生，即使在尽职尽责的开发人员的应用程序中也是如此。

仍然可能存在应用程序生成大量多余对象的情况，从而耗尽关键内存资源，有时会导致整个应用程序失败。

内存泄漏是Java中的一个真正问题。**在本教程中，我们将了解内存泄漏的潜在原因是什么、如何在运行时识别它们以及如何在我们的应用程序中处理它们**。

## 2. 什么是内存泄漏

**内存泄漏是指堆中存在不再使用的对象，但垃圾回收器无法将它们从内存中移除，因此不必要地维护它们的情况**。

内存泄漏是不好的，因为**它会阻塞内存资源并随着时间的推移降低系统性能**。如果不加以处理，应用程序最终将耗尽其资源，最终以致命的java.lang.OutOfMemoryError终止。

有两种不同类型的对象驻留在堆内存中，引用的和未引用的。引用的对象是那些在应用程序中仍然具有活动引用的对象，而未引用对象没有任何活动引用。

**垃圾回收器会定期删除未引用的对象，但它从不回收仍在引用的对象**，这是可能发生内存泄漏的地方：

[![Java中的内存泄漏](https://www.baeldung.com/wp-content/uploads/2018/11/Memory-_Leak-_In-_Java.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Memory-_Leak-_In-_Java.png)

**内存泄漏的症状**：

-   应用程序长时间连续运行时性能严重下降
-   应用程序中的OutOfMemoryError堆错误
-   自发和奇怪的应用程序崩溃
-   应用程序偶尔会用完连接对象

让我们仔细看看其中的一些场景以及如何处理它们。

## 3. Java内存泄漏的类型

在任何应用程序中，内存泄漏的发生可能有多种原因。在本节中，我们将讨论最常见的。

### 3.1 静态字段的内存泄漏

可能导致潜在内存泄漏的第一种情况是大量使用静态变量。

在Java中，**静态字段的生命周期通常与正在运行的应用程序的整个生命周期相匹配(除非ClassLoader符合垃圾回收条件)**。

让我们创建一个简单的Java程序来填充静态列表：

```java
public class StaticTest {
    public static List<Double> list = new ArrayList<>();

    public void populateList() {
        for (int i = 0; i < 10000000; i++) {
            list.add(Math.random());
        }
        Log.info("Debug Point 2");
    }

    public static void main(String[] args) {
        Log.info("Debug Point 1");
        new StaticTest().populateList();
        Log.info("Debug Point 3");
    }
}
```

如果我们在程序执行期间分析堆内存，那么我们将看到在调试点1和2之间，堆内存按预期增加。

但是当我们在调试点3处保留populateList()方法时，**堆内存还没有被垃圾回收**，正如我们在这个VisualVM响应中看到的那样：

[![静态内存](https://www.baeldung.com/wp-content/uploads/2018/11/memory-with-static.png)](https://www.baeldung.com/wp-content/uploads/2018/11/memory-with-static.png)

但是，如果我们只是在上面程序的第2行中删除关键字static，那么它会给内存使用带来巨大的变化，如这个VisualVM响应所示：

[![无静电记忆](https://www.baeldung.com/wp-content/uploads/2018/11/memory-without-static.png)](https://www.baeldung.com/wp-content/uploads/2018/11/memory-without-static.png)

直到调试点的第一部分与我们在静态情况下获得的几乎相同。但是这一次，在我们离开populateList()方法之后，**列表的所有内存都被垃圾回收了，因为我们没有对它的任何引用**。

所以我们需要非常注意我们对静态变量的使用，如果集合或大型对象被声明为static，那么它们将在应用程序的整个生命周期内保留在内存中，从而阻塞原本可以在其他地方使用的重要内存。

**如何预防**？

-   尽量减少静态变量的使用
-   使用单例时，依赖于延迟加载对象的实现，而不是急切加载

### 3.2 未关闭的资源

每当我们建立新连接或打开流时，JVM都会为这些资源分配内存。这方面的一些示例包括数据库连接、输入流和会话对象。

忘记关闭这些资源可能会阻塞内存，从而使它们远离GC。如果出现阻止程序执行到达处理代码以关闭这些资源的语句的异常，甚至会发生这种情况。

在任何一种情况下，**资源留下的打开连接都会消耗内存**，如果我们不处理它们，它们会降低性能，甚至导致OutOfMemoryError。

**如何预防**？

-   始终使用finally块来关闭资源
-   关闭资源的代码(即使在finally块中)本身不应有任何异常
-   使用Java 7+时，我们可以使用try-with-resources块

### 3.3 不正确的equals()和hashCode()实现

在定义新类时，一个非常常见的疏忽是没有为equals()和hashCode()方法编写正确的重写方法。

HashSet和HashMap在许多操作中使用这些方法，如果未正确重写它们，它们可能成为潜在内存泄漏问题的来源。

让我们以一个简单的Person类为例，并将其用作HashMap中的键：

```java
public class Person {
    public String name;

    public Person(String name) {
        this.name = name;
    }
}
```

现在我们将重复的Person对象插入到使用此键的Map中。

请记住Map不能包含重复键：

```java
@Test
public void givenMap_whenEqualsAndHashCodeNotOverridden_thenMemoryLeak() {
    Map<Person, Integer> map = new HashMap<>();
    for(int i=0; i<100; i++) {
        map.put(new Person("jon"), 1);
    }
    Assert.assertFalse(map.size() == 1);
}
```

这里我们使用Person作为键，由于Map不允许重复的键，我们作为键插入的大量重复的Person对象不应该增加内存。

但是**由于我们没有定义合适的equals()方法，重复的对象会堆积起来并增加内存**，这就是我们在内存中看到多个对象的原因。VisualVM中的堆内存如下所示：

[![在实施equals和hashcode之前](https://www.baeldung.com/wp-content/uploads/2018/11/Before_implementing_equals_and_hashcode.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Before_implementing_equals_and_hashcode.png)

但是，**如果我们正确地重写了equals()和hashCode()方法，那么这个Map中将只存在一个Person对象**。

让我们看一下Person类的equals()和hashCode()的正确实现：

```java
public class Person {
    public String name;

    public Person(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Person)) {
            return false;
        }
        Person person = (Person) o;
        return person.name.equals(name);
    }

    @Override
    public int hashCode() {
        int result = 17;
        result = 31 * result + name.hashCode();
        return result;
    }
}
```

在这种情况下，以下断言为true：

```java
@Test
public void givenMap_whenEqualsAndHashCodeNotOverridden_thenMemoryLeak() {
    Map<Person, Integer> map = new HashMap<>();
    for(int i=0; i<2; i++) {
        map.put(new Person("jon"), 1);
    }
    Assert.assertTrue(map.size() == 1);
}
```

在正确重写equals()和hashCode()之后，同一程序的堆内存如下所示：

[![执行equals和hashcode之后](https://www.baeldung.com/wp-content/uploads/2018/11/Afterimplementing_equals_and_hashcode.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Afterimplementing_equals_and_hashcode.png)

**另一种选择是使用像Hibernate这样的ORM工具**，它使用equals()和hashCode()方法来分析对象并将它们保存在缓存中。

如果不重写这些方法，则内存泄漏的可能性非常高，因为Hibernate将无法比较对象，并且会用重复的对象填充其缓存。

**如何预防**？

-   根据经验，在定义新实体时，始终重写equals()和hashCode()方法
-   仅仅重写是不够的，还必须以最佳方式重写这些方法

有关更多信息，请访问我们的教程[使用Eclipse生成equals()和hashCode()](https://www.baeldung.com/java-eclipse-equals-and-hashcode)和[Java中的hashCode()指南](https://www.baeldung.com/java-hashcode)。

### 3.4 引用外部类的内部类

这发生在非静态内部类(匿名类)的情况下。对于初始化，这些内部类始终需要一个封闭类的实例。

默认情况下，每个非静态内部类都有一个对其包含类的隐式引用。如果我们在应用程序中使用这个内部类的对象，那么**即使我们的包含类的对象超出范围，它也不会被垃圾回收**。

考虑一个类，该类包含对大量庞大对象的引用并具有非静态内部类。当我们只创建内部类的对象时，内存模型如下所示：

[![引用外部类的内部类](https://www.baeldung.com/wp-content/uploads/2018/11/Inner_Classes_That_Reference_Outer_Classes.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Inner_Classes_That_Reference_Outer_Classes.png)

但是，如果我们只是将内部类声明为静态的，那么同样的内存模型看起来是这样的：

[![引用外部类的静态类](https://www.baeldung.com/wp-content/uploads/2018/11/Static_Classes_That_Reference_Outer_Classes.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Static_Classes_That_Reference_Outer_Classes.png)

发生这种情况是因为内部类对象隐式持有对外部类对象的引用，从而使其成为垃圾回收的无效候选对象。在匿名类的情况下也会发生同样的情况。

**如何预防**？

-   如果内部类不需要访问包含类成员，请考虑将其变成静态类。

### 3.5 finalize()方法

使用终结器是潜在内存泄漏问题的另一个来源，**每当重写类的finalize()方法时，该类的对象都不会立即被垃圾回收**。取而代之的是，GC将它们排队等待最终确定，这发生在稍后的时间点。

此外，如果在finalize()方法中编写的代码不是最优的，并且如果终结器队列跟不上Java垃圾回收器，那么我们的应用程序迟早会遇到OutOfMemoryError。

为了演示这一点，让我们假设我们有一个类，我们已经为其重写了finalize()方法，并且该方法需要一点时间来执行。当这个类的大量对象被垃圾回收时，在VisualVM中看起来像这样：

[![Finalize方法被重写](https://www.baeldung.com/wp-content/uploads/2018/11/Finalize_method_overridden.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Finalize_method_overridden.png)

但是，如果我们只是删除重写的finalize()方法，那么同一个程序会给出以下响应：

[![Finalize方法未被重写](https://www.baeldung.com/wp-content/uploads/2018/11/Finalize_method_not_overridden.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Finalize_method_not_overridden.png)

**如何预防**？

-   我们应该始终避免终结器。

有关finalize()的更多详细信息，请参阅我们的[Java finalize方法指南](https://www.baeldung.com/java-finalize)中的第3节(避免终结器)。

### 3.6 池化字符串

当Java字符串池从PermGen转移到HeapSpace时，它在Java 7中经历了重大变化。但是，对于在版本6及以下版本上运行的应用程序，我们在处理大型字符串时需要更加注意。

**如果我们读取一个巨大的String对象，并在该对象上调用intern()，它会进入位于PermGen(永久内存)中的字符串池，并且只要我们的应用程序运行就会一直留在那里**。这会阻塞内存并在我们的应用程序中造成严重的内存泄漏。

JVM 1.6中这种情况的PermGen在VisualVM中看起来像这样：

[![实习字符串](https://www.baeldung.com/wp-content/uploads/2018/11/Interned_Strings.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Interned_Strings.png)

相反，如果我们只是在一个方法中从文件中读取一个字符串，并且不池化它，那么PermGen看起来像：

[![普通字符串](https://www.baeldung.com/wp-content/uploads/2018/11/Normal_Strings.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Normal_Strings.png)

**如何预防**？

-   解决此问题的最简单方法是升级到最新的Java版本，因为从Java版本7开始，字符串池已移至HeapSpace。

-   如果我们处理大字符串，我们可以增加PermGen空间的大小以避免任何潜在的OutOfMemoryErrors：

    ```shell
    -XX:MaxPermSize=512m
    ```

### 3.7 使用ThreadLocal

[ThreadLocal](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ThreadLocal.html)(在Java中的[ThreadLocal简介](https://www.baeldung.com/java-threadlocal)教程中详细讨论)是一种构造，它使我们能够将状态隔离到特定线程，从而使我们能够实现线程安全。

使用此构造时，**只要线程处于活动状态，每个线程都将持有对其ThreadLocal变量副本的隐式引用，并将维护自己的副本，而不是在多个线程之间共享资源**。

尽管有很多优点，但使用ThreadLocal变量是有争议的，因为如果使用不当，它们会因引入内存泄漏而臭名昭著。Joshua Bloch曾经评论过ThreadLocal的使用：

>   “线程池的草率使用与线程局部变量的草率使用相结合可能会导致意外的对象保留，正如在许多地方所指出的那样，但将责任归咎于ThreadLocal是没有根据的。”

**ThreadLocal的内存泄漏**：

一旦持有的线程不再存在，ThreadLocal就应该被垃圾回收。但是当我们将ThreadLocal与现代应用程序服务器一起使用时，问题就出现了。

现代应用程序服务器使用线程池来处理请求，而不是创建新线程(例如，Apache Tomcat中的[Executor](https://tomcat.apache.org/tomcat-7.0-doc/config/executor.html))。此外，它们还使用单独的类加载器。

由于应用程序服务器中的[线程池](https://www.baeldung.com/thread-pool-java-and-guava)基于线程重用的概念，因此它们永远不会被垃圾回收；相反，它们被重新用于处理另一个请求。

如果任何类创建了一个ThreadLocal变量，但没有显式删除它，那么即使在Web应用程序停止后，该对象的副本仍将保留在工作线程中，从而防止该对象被垃圾回收。

**如何预防**？

-   当我们不再使用ThreadLocal时，清理它们是一种很好的做法。ThreadLocal提供了[remove()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ThreadLocal.html#remove())方法，该方法删除此变量的当前线程值。
-   **不要使用ThreadLocal.set(null)来清除值**，它实际上并没有清除该值，而是会查找与当前线程关联的Map，并将键值对分别设置为当前线程和null。
-   最好将ThreadLocal视为我们需要在finally块中关闭的资源，即使在出现异常的情况下也是如此：

```java
try {
    threadLocal.set(System.nanoTime());
    // ... further processing
}
finally {
    threadLocal.remove();
}
```

## 4. 处理内存泄漏的其他策略

虽然在处理内存泄漏时没有一刀切的解决方案，但我们可以通过一些方法来最大限度地减少这些泄漏。

### 4.1 启用分析

Java分析器是监视和诊断应用程序内存泄漏的工具，它们分析我们应用程序内部发生的事情，比如我们如何分配内存。

**使用分析器，我们可以比较不同的方法并找到我们可以最佳利用资源的领域**。

在本教程的第3节中，我们使用了[Java VisualVM](https://visualvm.github.io/)。请查看我们的[Java分析器指南](https://www.baeldung.com/java-profilers)，了解不同类型的分析器，例如Mission Control、JProfiler、YourKit、Java VisualVM和Netbeans Profiler。

### 4.2 详细垃圾回收

通过启用详细垃圾回收，我们可以跟踪GC的详细踪迹。要启用此功能，我们需要将以下内容添加到我们的JVM配置中：

```shell
-verbose:gc
```

通过添加这个参数，我们可以看到GC内部发生的事情的详细信息：

[![详细垃圾收集](https://www.baeldung.com/wp-content/uploads/2018/11/verbose-garbage-collection.jpg)](https://www.baeldung.com/wp-content/uploads/2018/11/verbose-garbage-collection.jpg)

### 4.3 使用引用对象避免内存泄漏

我们还可以使用Java中内置的java.lang.ref包中的引用对象来处理内存泄漏。使用java.lang.ref包，我们不是直接引用对象，而是使用对对象的特殊引用，使它们很容易被垃圾回收。

引用队列使我们知道垃圾回收器执行的操作。有关更多信息，我们可以阅读Java教程中的[软引用](https://www.baeldung.com/java-soft-references)，特别是第4节。

### 4.4 Eclipse内存泄漏警告

对于JDK 1.5及更高版本的项目，Eclipse会在遇到明显的内存泄漏情况时显示警告和错误。因此，在Eclipse中开发时，我们可以定期访问“Problems”选项卡，并提高警惕内存泄漏警告(如果有)：

[![Eclipse内存泄漏警告](https://www.baeldung.com/wp-content/uploads/2018/11/Eclipse-_Memor-_Leak-_Warnings.png)](https://www.baeldung.com/wp-content/uploads/2018/11/Eclipse-_Memor-_Leak-_Warnings.png)

### 4.5 基准测试

我们可以通过执行基准测试来测量和分析Java代码的性能，这样，我们就可以比较执行相同任务的替代方法的性能。这可以帮助我们选择最佳方法，并可以帮助我们节省内存。

有关基准测试的更多信息，请访问我们的[Java微基准测试](https://www.baeldung.com/java-microbenchmark-harness)教程。

### 4.6 代码审查

最后，我们始终可以采用经典的老式方法来进行简单的代码审查(Code Review)。

在某些情况下，即使是这种看起来微不足道的方法也可以帮助消除一些常见的内存泄漏问题。

## 5. 总结

通俗地说，我们可以将内存泄漏视为一种疾病，它通过阻塞重要的内存资源来降低应用程序的性能。和所有其他疾病一样，如果不治愈，随着时间的推移，它可能会导致致命的应用程序崩溃。

内存泄漏很难解决，找到它们需要对Java语言进行复杂的掌握和理解。**在处理内存泄漏时，没有一种万能的解决方案，因为泄漏可能通过各种不同的事件发生**。

但是，如果我们采用最佳实践并定期执行严格的代码审查和分析，则可以最大程度地降低应用程序中内存泄漏的风险。

与往常一样，用于生成本文中描述的VisualVM响应的代码片段可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-perf-1)上找到。