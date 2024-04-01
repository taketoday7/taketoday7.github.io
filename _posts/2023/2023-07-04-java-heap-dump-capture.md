---
layout: post
title:  捕获Java堆转储的不同方法
category: java
copyright: java
excerpt: Java Performance
---

## 1. 概述

在本教程中，我们将探讨在Java中捕获堆转储的不同方法。

**堆转储是特定时刻JVM内存中所有对象的快照**，它们对于解决内存泄漏问题和优化Java应用程序中的内存使用非常有用。

**堆转储通常存储在二进制格式的hprof文件中**，我们可以使用jhat或JVisualVM等工具打开并分析这些文件。此外，对于Eclipse用户来说，使用[MAT](https://www.eclipse.org/mat/)是很常见的。

在接下来的部分中，我们将通过多种工具和方法来生成堆转储，并且将展示它们之间的主要区别。

## 2. JDK工具

JDK附带了多种工具来以不同的方式捕获堆转储，**所有这些工具都位于JDK主目录中的bin文件夹下**。因此，只要此目录包含在系统路径中，我们就可以从命令行启动它们。

在接下来的部分中，我们将介绍如何使用这些工具来捕获堆转储。

### 2.1 jmap

jmap是一个工具，用于打印正在运行的JVM中有关内存的统计信息。我们可以将它用于本地或远程进程。

**要使用jmap捕获堆转储，我们需要使用dump选项**：

```shell
jmap -dump:[live],format=b,file=<file-path> <pid>
```

除了该选项，我们还应该指定几个参数：

-   live：如果设置，它只打印具有活动引用的对象并丢弃那些准备好被垃圾回收的对象，此参数是可选的
-   format=b：指定转储文件将采用二进制格式，如果不设置，结果是一样的
-   file：将写入转储的文件
-   pid：Java进程的id

示例如下所示：

```shell
jmap -dump:live,format=b,file=/tmp/dump.hprof 12587
```

请记住，我们可以使用jps命令轻松获取Java进程的pid。

**另外，请记住jmap是作为实验性工具引入JDK的，不受支持**。因此，在某些情况下，最好改用其他工具。

### 2.2 jcmd

jcmd是一个非常完整的工具，它的工作原理是向JVM发送命令请求。我们必须在运行Java进程的同一台机器上使用它。

**它的众多命令之一是GC.heap_dump**，我们可以通过指定进程的pid和输出文件路径来使用它来获取堆转储：

```shell
jcmd <pid> GC.heap_dump <file-path>
```

我们可以使用之前使用的相同参数来执行它：

```shell
jcmd 12587 GC.heap_dump /tmp/dump.hprof
```

与jmap一样，生成的转储是二进制格式。

### 2.3 JVisualVM

**JVisualVM是一个带有图形用户界面的工具，可以让我们监控、排除故障和分析Java应用程序**。GUI很简单，但非常直观且易于使用。

它的众多选项之一允许我们捕获堆转储，如果我们右键单击一个Java进程并选择“Heap Dump”选项，该工具将创建一个堆转储并在新选项卡中打开它：

[![转储菜单裁剪1](https://www.baeldung.com/wp-content/uploads/2018/09/dump-menu-cropped-1.png)](https://www.baeldung.com/wp-content/uploads/2018/09/dump-menu-cropped-1.png)

请注意，我们可以在“Basic Info”部分找到创建的文件的路径。

从JDK 9开始，VisualVM不再包含在Oracle JDK和Open JDK发行版中。因此，如果我们使用的是Java 9及更新的JDK版本，我们可以从Visual VM开源[项目站点](https://visualvm.github.io/)获取JVisualVM。

## 3. 自动捕获堆转储

我们在前面部分中展示的所有工具都旨在在特定时间手动捕获堆转储。在某些情况下，我们希望在发生java.lang.OutOfMemoryError时获取堆转储以帮助我们调查错误。

对于这些情况，**Java提供了HeapDumpOnOutOfMemoryError命令行选项，它会在抛出java.lang.OutOfMemoryError时生成堆转储**：

```shell
java -XX:+HeapDumpOnOutOfMemoryError
```

**默认情况下，它将转储存储在我们运行应用程序的目录中的java_pid<pid\>.hprof文件中。如果我们想指定另一个文件或目录，我们可以在HeapDumpPath选项中设置它**：

```shell
java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<file-or-dir-path>
```

当我们的应用程序使用此选项耗尽内存时，我们将能够在日志中找到包含堆转储的已创建文件：

```text
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
Dumping heap to java_pid12587.hprof ...
Exception in thread "main" Heap dump file created [4744371 bytes in 0.029 secs]
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at cn.tuyucheng.taketoday.heapdump.App.main(App.java:7)
```

在上面的示例中，它被写入到java_pid12587.hprof文件中。

如我们所见，此选项非常有用，**使用此选项运行应用程序时不会产生任何开销。因此，强烈建议始终使用此选项，尤其是在生产环境中**。

最后，**还可以使用HotSpotDiagnostic MBean在运行时指定此选项**。为此，我们可以使用JConsole并将HeapDumpOnOutOfMemoryError VM选项设置为true：

[![jconsole设置选项1](https://www.baeldung.com/wp-content/uploads/2018/09/jconsole-setvmoption-1.png)](https://www.baeldung.com/wp-content/uploads/2018/09/jconsole-setvmoption-1.png)

我们可以在[本文](https://www.baeldung.com/java-management-extensions)中找到有关MBean和JMX的更多信息。

## 4. JMX

我们将在本文中介绍的最后一种方法是使用JMX。我们将使用在上一节中简要介绍过的HotSpotDiagnostic MBean，**这个MBean提供了一个接收两个参数的dumpHeap方法**：

-   outputFile：转储文件的路径，此文件应具有hprof扩展名
-   live：如果设置为true，则它只转储内存中的活动对象，就像我们之前在jmap中看到的那样

在接下来的部分中，我们将展示两种不同的方式来调用此方法以捕获堆转储。

### 4.1 JConsole

使用HotSpotDiagnostic MBean的最简单方法是使用JMX客户端，例如JConsole。

如果我们打开JConsole并连接到一个正在运行的Java进程，**我们可以导航到MBeans选项卡并在com.sun.management下找到HotSpotDiagnostic**。在操作中，我们可以找到我们之前描述的dumpHeap方法：

[![jconsole转储1](https://www.baeldung.com/wp-content/uploads/2018/09/jconsole-dump-1.png)](https://www.baeldung.com/wp-content/uploads/2018/09/jconsole-dump-1.png)

如图所示，我们只需要在p0和p1文本块中引入参数outputFile和live，就可以执行dumpHeap操作。

### 4.2 编程化方式

使用HotSpotDiagnostic MBean的另一种方法是从Java代码中以编程方式调用它。

为此，我们首先需要获取一个MBeanServer实例，以便获取在应用程序中注册的MBean。之后，**我们只需要获取HotSpotDiagnosticMXBean的实例，并调用其dumpHeap方法**。

让我们在代码中演示这一点：

```java
public static void dumpHeap(String filePath, boolean live) throws IOException {
    MBeanServer server = ManagementFactory.getPlatformMBeanServer();
    HotSpotDiagnosticMXBean mxBean = ManagementFactory.newPlatformMXBeanProxy(server, "com.sun.management:type=HotSpotDiagnostic", HotSpotDiagnosticMXBean.class);
    mxBean.dumpHeap(filePath, live);
}
```

**请注意，无法覆盖hprof文件**。因此，我们在创建打印堆转储的应用程序时应该考虑到这一点，如果我们不这样做，我们将得到一个异常：

```text
Exception in thread "main" java.io.IOException: File exists
	at sun.management.HotSpotDiagnostic.dumpHeap0(Native Method)
	at sun.management.HotSpotDiagnostic.dumpHeap(HotSpotDiagnostic.java:60)
```

## 5. 总结

在本文中，我们学习了多种在Java中捕获堆转储的方法。

根据经验，我们应该始终记住在运行Java应用程序时使用HeapDumpOnOutOfMemoryError选项。对于不同的目的，可以使用任何其他工具，只要我们记住jmap不受支持的状态。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-perf-1)上获得。