---
layout: post
title:  JVM中数组长度存储在哪里
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在这个快速教程中，我们将了解HotSpot JVM存储数组长度的方式和位置。

通常，运行时数据区的内存布局不是JVM规范的一部分，[由实现者自行决定](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html)。因此，每个JVM实现可能有不同的策略来在内存中布局对象和数组。

在本教程中，我们将重点介绍一个特定的JVM实现：HotSpot JVM。我们也可以互换使用JVM和HotSpot JVM术语。

## 2. 依赖

为了检查JVM中数组的内存布局，我们将使用Java对象布局([JOL](https://openjdk.java.net/projects/code-tools/jol/))工具。因此，我们需要添加[jol-core](https://search.maven.org/artifact/org.openjdk.jol/jol-core)依赖：

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

## 3. 数组长度

**HotSpot JVM使用一种称为普通对象指针([OOP](https://github.com/openjdk/jdk15/tree/master/src/hotspot/share/oops))的数据结构来表示指向对象的指针**。更具体地说，HotSpot JVM用称为[arrayOop](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/arrayOop.hpp#L35)的特殊OOP表示数组。每个arrayOop都包含一个包含以下详细信息的对象标头：

-   一个标记字用于存储身份哈希码或GC信息
-   一类klass存储常规类元数据
-   4个字节代表数组长度

因此，**JVM将数组长度存储在对象头中**。

让我们通过检查数组的内存布局来验证这一点：

```java
int[] ints = new int[42];
System.out.println(ClassLayout.parseInstance(ints).toPrintable());
```

如上所示，我们从现有数组实例解析内存布局。以下是JVM布局int[]的方式：

```text
[I object internals:
 OFFSET  SIZE   TYPE DESCRIPTION               VALUE
      0     4        (object header)           01 00 00 00 (00000001 00000000 00000000 00000000) (1) # mark
      4     4        (object header)           00 00 00 00 (00000000 00000000 00000000 00000000) (0) # mark
      8     4        (object header)           6d 01 00 f8 (01101101 00000001 00000000 11111000) (-134217363) #klass
     12     4        (object header)           2a 00 00 00 (00101010 00000000 00000000 00000000) (42) # array length
     16   168    int [I.<elements>             N/A
Instance size: 184 bytes
```

如前所述，JVM将数组长度存储在对象头内部的标记和klass字之后。此外，数组长度将以4个字节存储，因此它不能大于32位整数的最大值。

在对象头之后，JVM存储实际的数组元素。因为我们有一个包含42个整数的数组，所以数组的总大小为168字节-42乘以4。

## 4. 总结

在这个简短的教程中，我们了解了JVM如何存储数组长度。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-2)上获得。