---
layout: post
title:  Java数组的最大大小
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将了解Java中数组的最大大小。

## 2. 最大大小

Java程序最多只能分配一个特定大小的数组，这通常取决于我们使用的JVM和平台。**由于数组的索引是int，因此索引的近似值可以是2^31–1。基于这个近似值，我们可以说数组理论上可以容纳2147483647个元素**。

对于我们的示例，**我们在Linux和Windows机器上使用Java 8和Java 15的[OpenJDK](https://openjdk.java.net/)和[Oracle](https://www.oracle.com/in/java/technologies/javase-downloads.html)实现**。在我们的整个测试过程中，结果都是一样的。

这可以使用一个简单的例子来验证：

```java
for (int i = 2; i >= 0; i--) {
    try {
        int[] arr = new int[Integer.MAX_VALUE - i];
        System.out.println("Max-Size : " + arr.length);
    } catch (Throwable t) {
        t.printStackTrace();
    }
}
```

在上述程序的执行过程中，使用Linux和Windows机器，观察到类似的行为。在使用**VM参数-Xms2G -Xmx2G**执行时，我们将收到以下错误：

```text
java.lang.OutOfMemoryError: Java heap space
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
```

请注意，第一个错误与后两个不同。**最后两个错误提到了VM限制，而第一个错误是关于堆内存限制**。

现在让我们尝试使用**VM参数-Xms9G -Xmx9G**来接收确切的最大大小：

```text
Max-Size: 2147483645
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at com.example.demo.ArraySizeCheck.main(ArraySizeCheck.java:8)
```

结果显示**最大大小为2147483645**。

对于byte、boolean、long和数组中的其他数据类型，可以观察到相同的行为，并且结果是相同的。

## 3. ArraySupport

[ArraysSupport](https://github.com/openjdk/jdk14u/blob/84917a040a81af2863fddc6eace3dda3e31bf4b5/src/java.base/share/classes/jdk/internal/util/ArraysSupport.java#L577)是OpenJDK中的实用程序类，它**建议将最大大小设置为Integer.MAX_VALUE – 8以使其适用于所有JDK版本和实**现。

## 4. 总结

在本文中，我们研究了Java中数组的最大大小。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-guides)上获得。