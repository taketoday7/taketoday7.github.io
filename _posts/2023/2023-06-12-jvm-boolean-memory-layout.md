---
layout: post
title:  JVM中的boolean和boolean[]内存布局
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在这篇简短的文章中，我们将了解boolean值在不同情况下在JVM中的占用空间是多少。

首先，我们将检查JVM以查看对象大小。然后，我们将了解这些大小背后的基本原理。

## 2. 设置

为了检查JVM中对象的内存布局，我们将广泛使用Java对象布局([JOL](https://openjdk.java.net/projects/code-tools/jol/))。因此，我们需要添加[jol-core](https://search.maven.org/artifact/org.openjdk.jol/jol-core)依赖：

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

## 3. 对象大小

如果我们要求JOL根据对象大小打印VM详细信息：

```java
System.out.println(VM.current().details());
```

启用[压缩引用](https://www.baeldung.com/jvm-compressed-oops)后(默认行为)，我们将看到输出：

```text
# Running 64-bit HotSpot VM.
# Using compressed oop with 3-bit shift.
# Using compressed klass with 3-bit shift.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

在前几行中，我们可以看到有关VM的一些常规信息。之后，我们了解对象大小：

-   Java引用占用4字节，boolean/byte是1字节，char/short是2字节，int/float是4字节，最后，long/double是8字节
-   即使我们将它们用作数组元素，这些类型也会消耗相同数量的内存

**因此，在存在压缩引用的情况下，每个boolean值占用1个字节。同样，boolean[]中的每个boolean值占用1个字节**。但是，对齐填充和对象标头会增加boolean和boolean[]占用的空间，我们将在后面看到。

### 3.1 无压缩引用

即使我们通过-XX:-UseCompressedOops禁用压缩引用，**boolean大小也不会改变**：

```text
# Field sizes by type: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

另一方面，Java引用占用了两倍的内存。

**因此，尽管我们一开始可能会想到，boolean值会消耗1个字节而不仅仅是1位**。

### 3.2 字撕裂

在大多数架构中，无法以原子方式访问单个位。即使我们想这样做，我们也可能会在更新另一个位的同时写入相邻位。

**JVM的设计目标之一就是防止这种现象，称为[字撕裂](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.6)**。也就是说，在JVM中，每个字段和数组元素都应该是不同的；对一个字段或元素的更新不得与任何其他字段或元素的读取或更新交互。

**回顾一下，可寻址性问题和字撕裂是boolean值不仅仅是一个比特的主要原因**。

## 4. 普通对象指针(OOPs)

现在我们知道boolean是1个字节，让我们考虑这个简单的类：

```java
class BooleanWrapper {
    private boolean value;
}
```

如果我们使用JOL检查此类的内存布局：

```java
System.out.println(ClassLayout.parseClass(BooleanWrapper.class).toPrintable());
```

然后JOL将打印内存布局：

```text
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     1   boolean BooleanWrapper.value                      N/A
     13     3           (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 3 bytes external = 3 bytes total
```

BooleanWrapper布局包括：

-   标头为12字节，包括2个[标记字](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/markOop.hpp)和1个[klass字](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/klass.hpp)。HotSpot JVM使用标记字来存储GC元数据、身份哈希码和锁定信息。此外，它使用klass字来存储类元数据，例如运行时类型检查
-   1个字节用于实际boolean值
-   **用于对齐目的的3个字节的填充**

**默认情况下，对象引用应按8字节对齐**。因此，JVM将3个字节添加到13个字节的标头和布尔值，使其成为16个字节。

因此，boolean字段可能会因其字段对齐而消耗更多内存。

### 4.1 自定义对齐

如果我们通过-XX:ObjectAlignmentInBytes=32将对齐值更改为32，则相同的类布局将更改为：

```text
OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     1   boolean BooleanWrapper.value                      N/A
     13    19           (loss due to the next object alignment)
Instance size: 32 bytes
Space losses: 0 bytes internal + 19 bytes external = 19 bytes total
```

如上所示，JVM添加了19个字节的填充，使对象大小成为32的倍数。

## 5. Array OOPs

让我们看看JVM如何在内存中布局一个boolean数组：

```java
boolean[] value = new boolean[3];
System.out.println(ClassLayout.parseInstance(value).toPrintable());
```

这将打印实例布局，如下所示：

```text
OFFSET  SIZE      TYPE DESCRIPTION                              
      0     4           (object header)  # mark word
      4     4           (object header)  # mark word
      8     4           (object header)  # klass word
     12     4           (object header)  # array length
     16     3   boolean [Z.<elements>    # [Z means boolean array                        
     19     5           (loss due to the next object alignment)
```

除了两个标记字和一个klass字外，**[数组指针](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/arrayOop.hpp)还包含额外的4个字节来存储它们的长度**。

由于我们的数组具有3个元素，因此数组元素的大小为3个字节。但是，**这3个字节将由5个字段对齐字节填充以确保正确对齐**。

虽然数组中的每个boolean元素只有1个字节，但整个数组消耗的内存要多得多。换句话说，**我们应该在计算数组大小时考虑标头和填充开销**。

## 6. 总结

在这个快速教程中，我们看到boolean字段占用1个字节。此外，我们了解到我们应该考虑对象大小的标头和填充开销。

对于更详细的讨论，强烈建议查看[JVM源代码的oops部分](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/)。此外，[Aleksey Shipilëv](https://shipilev.net/)在这方面有一篇[更深入的文章](https://shipilev.net/jvm/objects-inside-out/)。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-2)上获得。