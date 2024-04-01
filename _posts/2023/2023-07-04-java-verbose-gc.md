---
layout: post
title:  Java中的详细垃圾回收
category: java
copyright: java
excerpt: Java Performance
---

## 1. 概述

在本教程中，**我们将介绍如何在Java应用程序中启用详细垃圾回收**。我们将首先介绍什么是详细垃圾回收以及为什么它很有用。

接下来，我们将查看几个不同的示例，并了解可用的不同配置选项。此外，**我们还将重点介绍如何解释详细日志的输出**。

要了解有关垃圾回收(GC)和可用的不同实现的更多信息，请查看我们关于[Java垃圾回收器](https://www.baeldung.com/jvm-garbage-collectors)的文章。

## 2. 详细垃圾回收简介

**在调整和调试许多问题(尤其是内存问题)时，通常需要打开详细的垃圾回收日志记录**。事实上，有些人会争辩说，为了严格监控我们的应用程序运行状况，我们应该始终监控JVM的垃圾回收性能。

正如我们将看到的，GC日志是一个非常重要的工具，用于揭示应用程序的堆和GC配置的潜在改进。**对于每次发生的GC，GC日志都会提供有关其结果和持续时间的准确数据**。

随着时间的推移，对这些信息的分析可以帮助我们更好地了解应用程序的行为，并帮助我们调整应用程序的性能。此外，**它可以通过指定最佳堆大小、其他JVM选项和备用GC算法来帮助优化GC频率和收集时间**。

### 2.1 一个简单的Java程序

我们将使用一个简单的Java程序来演示如何启用和解释我们的GC日志：

```java
public class Application {

    private static Map<String, String> stringContainer = new HashMap<>();

    public static void main(String[] args) {
        System.out.println("Start of program!");
        String stringWithPrefix = "stringWithPrefix";

        // Load Java Heap with 3 M java.lang.String instances
        for (int i = 0; i < 3000000; i++) {
            String newString = stringWithPrefix + i;
            stringContainer.put(newString, newString);
        }
        System.out.println("MAP size: " + stringContainer.size());

        // Explicit GC!
        System.gc();

        // Remove 2 M out of 3 M
        for (int i = 0; i < 2000000; i++) {
            String newString = stringWithPrefix + i;
            stringContainer.remove(newString);
        }

        System.out.println("MAP size: " + stringContainer.size());
        System.out.println("End of program!");
    }
}
```

正如我们在上面的例子中看到的，这个简单的程序将300万个String实例加载到一个Map对象中，然后我们使用System.gc()显式调用垃圾回收器。

最后，我们从Map中删除了200万个String实例。我们还明确使用System.out.println来简化输出的解释。

在下一节中，我们将了解如何激活GC日志记录。

## 3. 激活“简单”的GC日志记录

让我们首先运行我们的程序并通过我们的JVM启动参数启用详细GC：

```shell
-XX:+UseSerialGC -Xms1024m -Xmx1024m -verbose:gc
```

**这里的重要参数是-verbose:gc，它以最简单的形式激活垃圾回收信息的日志记录**。默认情况下，GC日志被写入标准输出，并且应该为每个新生代GC和每个full GC输出一行。

出于我们示例的目的，我们通过参数-XX:+UseSerialGC指定了串行垃圾回收器，这是最简单的GC实现。

我们还设置了1024mb的最小和最大堆大小，但当然，我们可以调整更多的[JVM参数](https://www.baeldung.com/jvm-parameters)。

### 3.1 对详细输出的基本理解

现在让我们看一下这个简单程序的输出：

```text
Start of program!
[GC (Allocation Failure)  279616K->146232K(1013632K), 0.3318607 secs]
[GC (Allocation Failure)  425848K->295442K(1013632K), 0.4266943 secs]
MAP size: 3000000
[Full GC (System.gc())  434341K->368279K(1013632K), 0.5420611 secs]
[GC (Allocation Failure)  647895K->368280K(1013632K), 0.0075449 secs]
MAP size: 1000000
End of program!
```

在上面的输出中，我们已经可以看到很多关于JVM内部发生的事情的有用信息。

乍一看，这个输出看起来相当令人生畏，但现在让我们一步步来了解它。

首先，**我们可以看到发生了4次回收，1次Full GC和3次清理年轻代**。

### 3.2 更详细的输出

让我们更详细地分解输出行，以准确了解发生了什么：

1.  GC或Full GC：**垃圾回收的类型，GC或Full GC以区分次要垃圾回收或完整垃圾回收**
2.  (Allocation Failure) or (System.gc())：**回收的原因–Allocation Failure表示Eden中没有更多的空间来分配我们的对象**
3.  279616K->146232K：分别为GC前后占用的堆内存(用箭头分隔)
4.  (1013632K)：堆的当前容量
5.  0.3318607 secs：GC事件的持续时间(以秒为单位)

因此，如果我们取第一行，279616K->146232K(1013632K)意味着GC将占用的堆内存从279616K减少到146232K。GC时的堆容量为1013632K，GC耗时0.3318607秒。

但是，虽然简单的GC日志记录格式很有用，但它提供的详细信息有限。**例如，我们无法判断GC是否将任何对象从年轻代移动到老年代，或者每次回收前后年轻代的总大小是多少**。

出于这个原因，详细的GC日志记录比简单的更有用。

## 4. 激活“详细”GC日志记录

**要激活详细的GC日志记录，我们使用参数-XX:+PrintGCDetails**。这将为我们提供有关每个GC的更多详细信息，例如：

-   每次GC前后新生代和老年代的大小
-   GC在年轻代和老年代发生的时间
-   每次GC提升的对象的大小
-   总堆大小的摘要

在下一个示例中，我们将看到如何将-verbose:gc与这个额外参数结合起来，在我们的日志中捕获更详细的信息。

请注意-XX:+PrintGCDetails标志在Java 9中已被弃用，取而代之的是新的统一日志记录机制(稍后会详细介绍)。不管怎样，**-XX:+PrintGCDetails的新等价物是-Xlog:gc*选项**。 

## 5. 解释“详细”的详细输出

让我们再次运行示例程序：

```bash
-XX:+UseSerialGC -Xms1024m -Xmx1024m -verbose:gc -XX:+PrintGCDetails
```

这次输出更详细：

```text
Start of program!
[GC (Allocation Failure) [DefNew: 279616K->34944K(314560K), 0.3626923 secs] 279616K->146232K(1013632K), 0.3627492 secs] [Times: user=0.33 sys=0.03, real=0.36 secs] 
[GC (Allocation Failure) [DefNew: 314560K->34943K(314560K), 0.4589079 secs] 425848K->295442K(1013632K), 0.4589526 secs] [Times: user=0.41 sys=0.05, real=0.46 secs] 
MAP size: 3000000
[Full GC (System.gc()) [Tenured: 260498K->368281K(699072K), 0.5580183 secs] 434341K->368281K(1013632K), [Metaspace: 2624K->2624K(1056768K)], 0.5580738 secs] [Times: user=0.50 sys=0.06, real=0.56 secs] 
[GC (Allocation Failure) [DefNew: 279616K->0K(314560K), 0.0076722 secs] 647897K->368281K(1013632K), 0.0077169 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
MAP size: 1000000
End of program!
Heap
 def new generation   total 314560K, used 100261K [0x00000000c0000000, 0x00000000d5550000, 0x00000000d5550000)
  eden space 279616K,  35% used [0x00000000c0000000, 0x00000000c61e9370, 0x00000000d1110000)
  from space 34944K,   0% used [0x00000000d3330000, 0x00000000d3330188, 0x00000000d5550000)
  to   space 34944K,   0% used [0x00000000d1110000, 0x00000000d1110000, 0x00000000d3330000)
 tenured generation   total 699072K, used 368281K [0x00000000d5550000, 0x0000000100000000, 0x0000000100000000)
   the space 699072K,  52% used [0x00000000d5550000, 0x00000000ebcf65e0, 0x00000000ebcf6600, 0x0000000100000000)
 Metaspace       used 2637K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 283K, capacity 386K, committed 512K, reserved 1048576K
```

我们应该能够从简单的GC日志中识别出所有元素，但是有很多新元素。

现在让我们考虑输出中的新元素，它们在下一节中以蓝色突出显示。

### 5.1 解释年轻代中的Minor GC

我们将从分析Minor GC中的新部分开始：

-   \[GC (Allocation Failure) <font color=Blue>\[DefNew: 279616K->34944K(314560K), 0.3626923 secs]</font> 279616K->146232K(1013632K), 0.3627492 secs] <font color=Blue>\[Times: user=0.33 sys=0.03, real=0.36 secs]</font>

和以前一样，我们将这些行分解成几个部分：

1.  DefNew：使用的垃圾回收器的名称，**这个不太明显的名字代表单线程标记复制停止世界垃圾(single-threaded mark-copy stop-the-world)回收器，用于清理年轻代**
2.  279616K->34944K：回收前后年轻代的使用情况
3.  (314560K)：年轻代的总大小
4.  0.3626923 secs：以秒为单位的持续时间
5.  \[Times: user=0.33 sys=0.03, real=0.36 secs ]：GC事件的持续时间，在不同类别中测量

现在让我们解释一下不同的类别：

-   user：垃圾回收器消耗的总CPU时间
-   sys：在操作系统调用或等待系统事件中花费的时间
-   real：这是所有经过的时间，包括其他进程使用的时间片

**由于我们使用串行垃圾回收器运行示例，该回收器始终仅使用单个线程，因此real等于user和sys时间的总和**。

### 5.2 解释Full GC

在倒数第二个示例中，我们看到对于由我们的系统调用触发的主要回收(Full GC)，使用的回收器是Tenured。

我们看到的最后一条附加信息是元空间遵循相同模式的细分：

```text
[Metaspace: 2624K->2624K(1056768K)], 0.5580738 secs]
```

**元空间是Java 8中引入的一种新的[内存空间](https://www.baeldung.com/java-permgen-metaspace)，是本机内存的一块区域**。 

### 5.3 Java堆分解分析

**输出的最后部分包括堆的细分，包括内存每个部分的内存占用摘要**。

我们可以看到Eden空间占35%，Tenured占52%，还包括元数据空间和类空间的摘要。

从上面的示例中，**我们现在可以准确了解在GC事件期间JVM内部的内存消耗情况**。

## 6. 添加日期和时间信息

没有日期和时间信息的好日志是不完整的。

**当我们需要将GC日志数据与来自其他来源的数据相关联时，这些额外信息可能非常有用，或者它可以简单地帮助促进搜索**。

我们可以在运行应用程序时添加以下两个参数以获取要在日志中显示的日期和时间信息：

```shell
-XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps
```

现在每一行都以写入时的绝对日期和时间开头，后跟一个时间戳，反映自JVM启动以来经过的实时时间(以秒为单位)：

```text
2018-12-11T02:55:23.518+0100: 2.601: [GC (Allocation ...
```

请注意，这些调整标志已在Java 9中删除，新的替代方案是：

```shell
-Xlog:gc*::time
```

## 7. 记录到文件

正如我们已经看到的，默认情况下GC日志写入stdout(标准输出)，一个更实用的解决方案是指定一个输出文件。

**我们可以通过使用参数-Xloggc:<file\>来做到这一点，其中file是输出文件的绝对路径**：

```shell
-Xloggc:/path/to/file/gc.log
```

与其他调优标志类似，Java 9弃用了-Xloggc标志以支持新的统一日志记录。更具体地说，现在记录到文件的替代方法是：

```shell
-Xlog:gc:/path/to/file/gc.log
```

## 8. Java 9：统一的JVM日志记录

从Java 9开始，大多数与GC相关的调优标志已被弃用，取而代之的是统一日志记录选项-Xlog:gc。但是，-verbose:gc选项在Java 9和更新版本中仍然有效。

例如，从Java 9开始，新的统一日志系统中-verbose:gc标志的等价物是：

```shell
-Xlog:gc
```

这会将所有info级别的GC日志记录到标准输出，也可以使用-Xlog:gc=<level\>语法来更改日志级别。例如，要查看所有debug级别的日志：

```shell
-Xlog:gc=debug
```

正如我们之前看到的，我们可以通过-Xlog:gc=<level>:<output\>语法更改输出目标。默认情况下，输出是stdout，但我们可以将其更改为stderr甚至文件：

```shell
-Xlog:gc=debug:file=gc.txt
```

此外，还可以使用装饰器向输出添加更多字段。例如：

```shell
-Xlog:gc=debug::pid,time,uptime
```

在这里，我们在每个日志语句中打印进程ID、正常运行时间和当前时间戳。

要查看统一JVM日志记录的更多示例，请参阅[JEP 158标准](https://openjdk.java.net/jeps/158)。

## 9. 分析GC日志的工具

使用文本编辑器分析GC日志可能既费时又乏味，根据JVM版本和使用的GC算法，GC日志格式可能会有所不同。

有一个非常好的免费图形分析工具可以分析垃圾回收日志，提供许多关于潜在垃圾回收问题的指标，甚至提供这些问题的潜在解决方案。

一定要看看[通用GC日志分析器](http://gceasy.io/)！

## 10. 总结

总而言之，在本教程中，我们详细探讨了Java中的详细垃圾回收。

首先，我们介绍了详细垃圾回收是什么以及我们可能想要使用它的原因。然后，我们使用一个简单的Java应用程序查看了几个示例。在探索几个更详细的示例以及如何解释输出之前，我们首先以最简单的形式启用GC日志记录。

最后，我们研究了几个用于记录时间和日期信息以及如何将信息写入日志文件的额外选项。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-perf-1)上获得。