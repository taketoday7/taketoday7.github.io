---
layout: post
title:  OutOfMemoryError：超出GC开销限制
category: java
copyright: java
excerpt: Java Performance
---

## 1. 概述

简而言之，JVM负责在不再使用对象时释放内存，此过程称为垃圾回收([GC](https://www.baeldung.com/jvm-garbage-collectors))。

“GC Overhead Limit Exceeded”错误来自java.lang.OutOfMemoryError系列，它表示资源(内存)耗尽。

在这个简短的教程中，我们将介绍导致“java.lang.OutOfMemoryError: GC Overhead Limit Exceeded”错误的原因及其解决方法。

## 2. GC Overhead Limit Exceeded错误

OutOfMemoryError是java.lang.VirtualMachineError的子类，当遇到与利用资源相关的问题时，它由JVM抛出。更具体地说，**当JVM花费太多时间执行垃圾回收而只能回收很少的堆空间时，就会发生此错误**。

根据Java文档，默认情况下，如果Java进程花费超过98%的时间执行GC并且每次运行中只有不到2%的堆被回收，则JVM配置为抛出此错误。换句话说，这意味着我们的应用程序几乎耗尽了所有可用内存，垃圾回收器花费了太多时间尝试清理它并反复失败。

在这种情况下，用户会体验到应用程序极其缓慢。某些通常在几毫秒内完成的操作需要更多时间才能完成，这是因为CPU将其全部容量用于垃圾回收，因此无法执行任何其他任务。

## 3. 操作失误

让我们来看一段抛出“java.lang.OutOfMemoryError: GC Overhead Limit Exceeded”的代码。

例如，我们可以通过在未终止的循环中添加键值对来实现这一点：

```java
public class OutOfMemoryGCLimitExceed {
    public static void addRandomDataToMap() {
        Map<Integer, String> dataMap = new HashMap<>();
        Random r = new Random();
        while (true) {
            dataMap.put(r.nextInt(), String.valueOf(r.nextInt()));
        }
    }
}
```

调用此方法时，JVM参数为-Xmx100m -XX:+UseParallelGC(Java堆大小设置为100MB，GC算法为ParallelGC)，我们会收到“java.lang.OutOfMemoryError: GC Overhead Limit Exceeded”错误。要更好地了解不同的垃圾回收算法，请查看Oracle的[Java垃圾回收基础](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)教程。

通过从[项目](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules)的根目录运行以下命令，我们将很快得到一个“java.lang.OutOfMemoryError: GC Overhead Limit Exceeded”错误：

```shell
mvn exec:exec
```

还应该注意的是，在某些情况下，我们可能会在遇到“GC Overhead Limit Exceeded”错误之前遇到堆空间错误。

## 4. 解决GC Overhead Limit Exceeded错误

理想的解决方案是通过检查任何内存泄漏的代码来查找应用程序的潜在问题。

这些问题需要解决：

-   应用程序中占用大部分堆的对象是什么？
-   在源代码的哪些部分分配了这些对象？

我们还可以使用[JConsole](https://docs.oracle.com/en/java/javase/11/management/using-jconsole.html#GUID-77416B38-7F15-4E35-B3D1-34BFD88350B5)等自动化图形工具，它有助于检测代码中的性能问题，包括java.lang.OutOfMemoryError。

最后的手段是通过更改JVM启动配置来增加堆大小。

例如，这为Java应用程序提供了1GB的堆空间：

```shell
java -Xmx1024m com.xyz.TheClassName
```

但是，如果实际应用程序代码中存在内存泄漏，这并不能解决问题。相反，我们只会推迟错误。因此，更明智的做法是彻底重新评估应用程序的内存使用情况。

## 5. 总结

在本文中，我们研究了“java.lang.OutOfMemoryError: GC Overhead Limit Exceeded”及其背后的原因。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-perf-1)上获得。