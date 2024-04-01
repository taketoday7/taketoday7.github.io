---
layout: post
title:  JVM中的方法内联
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在本教程中，我们将了解Java虚拟机中的方法内联及其工作原理。

我们还将了解如何从JVM获取和读取与内联相关的信息，以及我们可以如何使用这些信息来优化我们的代码。

## 2. 什么是方法内联？

基本上，**内联是一种在运行时优化已编译源代码的方法，方法是用其主体替换最常执行的方法的调用**。

尽管涉及编译，但它不是由传统的javac编译器执行的，而是由JVM本身执行的。更准确地说，**这是JVM的一部分Just-In-Time(JIT)编译器的职责**；javac只生成一个字节码，让JIT施展魔法并优化源代码。

这种方法最重要的结果之一是，如果我们使用旧的Java编译代码，相同的.class文件在较新的JVM上会更快。这样我们就不需要重新编译源代码，而只需要更新Java。

## 3. JIT是如何做到的？

本质上，**JIT编译器试图内联我们经常调用的方法，这样我们就可以避免方法调用的开销**。在决定是否内联方法时，它需要考虑两件事。

首先，它使用计数器来跟踪我们调用该方法的次数。当该方法被调用超过特定次数时，它将变为“热”。默认情况下，此阈值设置为10000，但我们可以在Java启动期间通过JVM标志对其进行配置。我们绝对不能内联所有内容，因为这会很耗时并且会产生巨大的字节码。

我们应该记住，只有当我们达到稳定状态时，内联才会发生。这意味着我们需要多次重复执行，以便为JIT编译器提供足够的分析信息。

此外，“热”并不能保证该方法将被内联。如果它太大，JIT将不会内联它。可接受的大小受-XX:FreqInlineSize=标志限制，该标志指定要为方法内联的字节码指令的最大数量。

尽管如此，强烈建议不要更改此标志的默认值，除非我们完全确定它会产生什么影响。默认值取决于平台-对于64位Linux，它是325。

**JIT通常内联静态、私有或最终方法。虽然公共方法也可以内联，但并不是每个公共方法都必须内联。JVM需要确定这种方法只有一个实现**。任何额外的子类都会阻止内联，性能将不可避免地下降。

## 4. 寻找热点方法

我们当然不想猜测JIT在做什么。因此，我们需要一些方法来查看哪些方法是内联的或未内联的。通过在启动期间设置一些额外的JVM标志，我们可以很容易地实现这一点并将所有这些信息记录到标准输出：

```shell
-XX:+PrintCompilation -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining
```

当JIT编译发生时，将记录第一个标志。第二个标志启用其他标志，包括-XX:+PrintInlining，它将打印哪些方法正在内联以及在哪里内联。

这将以树的形式向我们展示内联方法。叶子用以下选项之一进行注释和标记：

-   inline(hot)：此方法被标记为热并且是内联的
-   too big：该方法不热，但它生成的字节码太大，所以它没有内联
-   hot method too big：这是一个热方法，但由于字节码太大，因此未内联

**我们应该注意第三个值，并尝试优化带有“hot method too big”标签的方法**。

一般情况下，如果我们找到一个具有非常复杂条件语句的热方法，我们应该尽量将if语句的内容分离出来，增加粒度，以便JIT对代码进行优化。switch和for循环语句也是如此。

我们可以得出总结，手动方法内联是我们不需要为了优化代码而做的事情。JVM可以更有效地做到这一点，我们可能会使代码变得冗长且难以理解。

### 4.1 例子

现在让我们看看如何在实践中检查这一点。我们将首先创建一个简单的类来计算前N个连续正整数的总和：

```java
public class ConsecutiveNumbersSum {

    private long totalSum;
    private int totalNumbers;

    public ConsecutiveNumbersSum(int totalNumbers) {
        this.totalNumbers = totalNumbers;
    }

    public long getTotalSum() {
        totalSum = 0;
        for (int i = 0; i < totalNumbers; i++) {
            totalSum += i;
        }
        return totalSum;
    }
}
```

接下来，一个简单的方法将利用该类来执行计算：

```java
private static long calculateSum(int n) {
    return new ConsecutiveNumbersSum(n).getTotalSum();
}
```

最后，我们将多次调用该方法，看看会发生什么：

```java
for (int i = 1; i < NUMBERS_OF_ITERATIONS; i++) {
    calculateSum(i);
}
```

在第一次运行中，我们将运行它1000次(小于上述阈值10000)。如果我们在输出中搜索calculateSum()方法，我们将找不到它。这是意料之中的，因为我们调用它的次数不够多。

如果我们现在将迭代次数更改为15000并再次搜索输出，我们将看到：

```plaintext
664 262 % cn.tuyucheng.taketoday.inlining.InliningExample::main @ 2 (21 bytes)
  @ 10   cn.tuyucheng.taketoday.inlining.InliningExample::calculateSum (12 bytes)   inline (hot)
```

我们可以看到，这次该方法满足了内联的条件，JVM对其进行了内联。

再次值得一提的是，如果方法太大，无论迭代次数如何，JIT都不会内联它。我们可以通过在运行应用程序时添加另一个标志来检查这一点：

```shell
-XX:FreqInlineSize=10
```

正如我们在前面的输出中看到的，我们的方法的大小是12个字节。-XX:FreqInlineSize标志将符合内联条件的方法大小限制为10个字节。因此，这次内联不应该发生。事实上，我们可以通过再次查看输出来确认这一点：

```plaintext
330 266 % cn.tuyucheng.taketoday.inlining.InliningExample::main @ 2 (21 bytes)
  @ 10   cn.tuyucheng.taketoday.inlining.InliningExample::calculateSum (12 bytes)   hot method too big
```

尽管出于说明目的我们在此处更改了标志值，但我们必须强调建议不要更改-XX:FreqInlineSize标志的默认值，除非绝对必要。

## 5. 总结

在本文中，我们了解了JVM中的方法内联是什么以及JIT是如何进行内联的。我们描述了如何检查我们的方法是否符合内联条件，并建议如何通过尝试减少经常调用的长方法的大小来利用此信息，这些方法太大而无法内联。

最后，我们说明了如何在实践中识别热点方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-1)上获得。