---
layout: post
title:  使用Flight Recorder监控Java应用程序
category: java
copyright: java
excerpt: Java Performance
---

## 1. 概述

在本教程中，我们将研究Java Flight Recorder、其概念、基本命令以及如何使用它。

## 2. Java监控实用程序

Java不仅仅是一种编程语言，更是一个拥有大量工具的非常丰富的生态系统。JDK包含的程序允许我们编译自己的程序，以及在程序执行的整个生命周期中监视它们的状态和Java虚拟机的状态。

JDK发行版的bin文件夹包含以下可用于分析和监控的程序：

-   **Java VisualVM**(jvisualvm.exe)
-   **JConsole**(jconsole.exe)
-   **Java Mission Control**(jmc.exe)
-   **诊断命令工具**(jcmd.exe)

我们建议浏览此文件夹的内容，以了解我们拥有哪些工具可供使用。请注意，Java VisualVM过去是Oracle和OpenJDK发行版的一部分。但是，**从Java 9开始，JDK发行版不再附带Java VisualVM**。因此，我们应该从VisualVM[开源项目网站](https://visualvm.github.io/)单独下载。

在本教程中，我们将重点介绍Java Flight Recorder。这在上面提到的工具中不存在，因为它不是一个独立的程序。它的用法与上面的两个工具密切相关-Java Mission Control和诊断命令工具。

## 3. Java Flight Recorder及其基本概念

**Java Flight Recorder(JFR)是一种监视工具，用于收集有关Java应用程序执行期间Java虚拟机(JVM)中事件的信息**。JFR是JDK发行版的一部分，并集成到JVM中。

**JFR旨在尽可能少地影响正在运行的应用程序的性能**。

为了使用JFR，我们应该激活它。我们可以通过两种方式实现这一目标：

1.  启动Java应用程序时
2.  当Java应用程序已在运行时传递jcmd工具的诊断命令

JFR没有独立的工具，我们使用Java Mission Control(JMC)，它包含一个插件，可以让我们可视化JFR收集的数据。

这三个组件-JFR、jcmd和JMC构成了一个完整的套件，用于收集正在运行的Java程序的低级运行时信息。我们可能会发现这些信息在优化我们的程序时非常有用，或者在出现问题时对其进行诊断。

**如果我们的计算机上安装了多个版本的Java，请务必确保Java编译器(javac)、Java启动器(java)和上述工具(JFR、jcmd和JMC)来自同一个Java发行版**。否则，由于不同版本的JFR数据格式可能不兼容，可能存在看不到任何有用数据的风险。

**JFR有两个主要概念：事件和数据流**，让我们简要地讨论一下。

### 3.1 事件

JFR收集Java应用程序运行时JVM中发生的事件，这些事件与JVM本身的状态或程序的状态有关。事件具有名称、时间戳和其他信息(如线程信息、执行堆栈和堆状态)。

**JFR收集三种类型的事件**：

-   **即时事件**一旦发生就会立即记录下来
-   如果持续时间超过指定阈值，则记录**持续时间事件**
-   **示例事件**用于对系统活动进行采样

### 3.2 数据流

JFR收集的事件包含大量数据。出于这个原因，JFR的设计速度足够快，不会妨碍程序。

JFR将有关事件的数据保存在单个输出文件flight.jfr中。

众所周知，磁盘I/O操作非常昂贵。因此，JFR在将数据块刷新到磁盘之前使用各种缓冲区来存储收集的数据。事情可能会变得有点复杂，因为在同一时刻，一个程序可能有多个具有不同选项的注册进程。

正因为如此，**我们可能会在输出文件中发现比请求更多的数据，或者它可能不是按时间顺序排列的**。如果我们使用JMC，我们甚至可能不会注意到这个事实，因为它按时间顺序可视化事件。

**在极少数情况下，JFR可能无法刷新数据**(例如，当事件过多或停电时)。如果发生这种情况，JFR会尝试通知我们输出文件可能丢失了一段数据。

## 4. 如何使用Java Flight Recorder

JFR是一项实验性功能，因此其使用可能会发生变化。事实上，在早期的发行版中，我们必须激活商业功能才能在生产中使用它。但是，从JDK 11开始，我们可以在不激活任何东西的情况下使用它。我们可以随时查阅Java官方发行说明来了解如何使用该工具。

对于JDK 8，为了能够激活JFR，我们应该使用+UnlockCommercialFeatures和+FlightRecorder选项启动JVM。

正如我们上面提到的，有两种激活JFR的方法。当我们在启动应用程序的同时激活它时，我们是从命令行执行的。当应用程序已经运行时，我们使用诊断命令工具。

### 4.1 命令行

首先，我们使用标准的java编译器javac将程序的*.java文件编译成*.class。

编译成功后，我们可以使用以下选项启动程序：

```shell
java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder 
  -XX:StartFlightRecording=duration=200s,filename=flight.jfr path-to-class-file
```

其中path-to-class-file是应用程序的入口点*.class文件。

此命令启动应用程序并激活录制，录制立即开始，持续时间不超过200秒。收集的数据保存在输出文件flight.jfr中，我们将在下一节中更详细地描述其他选项。

### 4.2 诊断命令工具

我们还可以使用jcmd工具开始注册事件，例如：

```shell
jcmd 1234 JFR.start duration=100s filename=flight.jfr
```

在JDK 11之前，为了能够以这种方式激活JFR，我们应该以解锁的商业功能启动应用程序：

```shell
java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -cp ./out/ cn.tuyucheng.taketoday.Main
```

应用程序运行后，我们使用其进程ID来执行各种命令，这些命令采用以下格式：

```shell
jcmd <pid|MainClass> <command> [parameters]
```

以下是诊断命令的完整列表：

-   **JFR.start**：开始一个新的JFR记录
-   **JFR.check**：检查正在运行的JFR记录
-   **JFR.stop**：停止特定的JFR记录
-   **JFR.dump**：将JFR记录的内容复制到文件

每个命令都有一系列参数，例如，JFR.start命令具有以下参数：

-   **name**：记录的名称；它用于稍后使用其他命令引用此记录
-   **delay**：记录开始时间延迟的维度参数，默认值为0s
-   **duration**：记录持续时间的时间间隔的维度参数；默认值为0s，表示无限制
-   **filename**：包含所收集数据的文件的名称
-   **maxage**：所收集数据最长期限的维度参数；默认值为0s，表示无限制
-   **maxsize**：收集数据的最大缓冲区大小(以字节为单位)；默认值为0，表示没有最大大小

我们已经在本节开头看到了这些参数的用法示例，有关参数的完整列表，我们可以随时查阅[Java Flight Recorded官方文档](https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/comline.htm#JFRUH190)。

尽管JFR被设计为尽可能少地占用JVM和应用程序的性能，但最好通过至少设置以下参数之一来限制收集数据的最大量：duration、maxage或maxsize。

## 5. Java Flight Recorder实践

现在让我们使用一个示例程序来演示JFR的实际应用。

### 5.1 示例程序

我们的程序将对象插入到列表中，直到发生OutOfMemoryError，然后程序休眠一秒钟：

```java
public static void main(String[] args) {
    List<Object> items = new ArrayList<>(1);
    try {
        while (true){
            items.add(new Object());
        }
    } catch (OutOfMemoryError e){
        System.out.println(e.getMessage());
    }
    assert items.size() > 0;
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        System.out.println(e.getMessage());
    }
}
```

如果不执行这段代码，我们可以发现一个潜在的缺点：while循环会导致高CPU和内存使用率。让我们使用JFR来了解这些缺点并找到可能的其他缺点。

### 5.2 开始注册

首先，我们通过从命令行执行以下命令来编译我们的程序：

```shell
javac -d out -sourcepath src/main src/main/cn/tuyucheng/taketoday/flightrecorder/FlightRecorder.java
```

此时，我们应该在out/cn/tuyucheng/taketoday/flightrecorder目录下找到一个文件FlightRecorder.class。

现在，我们将使用以下选项启动程序：

```shell
java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder 
  -XX:StartFlightRecording=duration=200s,filename=flight.jfr 
  -cp ./out/ cn.tuyucheng.taketoday.flightrecorder.FlightRecorder
```

### 5.3 可视化数据

现在，我们将文件flight.jfr提供给**Java Mission Control**，它是JDK发行版的一部分，可以帮助我们以一种直观的方式可视化有关事件的数据。

它的主屏幕向我们展示了程序在执行期间如何使用CPU的信息。我们看到CPU负载过重，由于while循环，这是意料之中的：

[![主画面.png](https://www.baeldung.com/wp-content/uploads/2019/01/main-screen.png)](https://www.baeldung.com/wp-content/uploads/2019/01/main-screen.png)

在视图的左侧，我们看到了General、Memory、Code和Threads等部分，每个部分都包含带有详细信息的各种选项卡。例如，Code部分的Hot Methods选项卡包含方法调用的统计信息：

[![码屏热点方法](https://www.baeldung.com/wp-content/uploads/2019/01/code-screen-hot-methods.png)](https://www.baeldung.com/wp-content/uploads/2019/01/code-screen-hot-methods.png)

在此选项卡中，我们可以发现示例程序的另一个缺点：方法java.util.ArrayList.grow(int)已被调用17次，以便在每次没有足够空间添加对象时扩大数组容量。

在更现实的程序中，我们可能会看到很多其他有用的信息：

-   有关已创建对象的统计信息，当它们被垃圾回收器创建和销毁时
-   有关线程时间顺序的详细报告，当它们被锁定或激活时
-   应用程序正在执行哪些I/O操作

## 6. 总结

在本文中，我们介绍了使用Java Flight Recorder监视和分析Java应用程序的主题。该工具仍处于实验阶段，因此我们应该访问其[官方网站](https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/toc.htm)以获取更完整和最新的信息。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-perf-1)上获得。