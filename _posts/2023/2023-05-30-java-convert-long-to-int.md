---
layout: post
title:  Java将long类型转换为int类型
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将了解如何在Java中将long值转换为int类型。在开始编码之前，我们需要指出有关此数据类型的一些细节。

首先，在Java中，long值由带符号的64位数字表示。另一方面，int值由带符号的32位数字表示。因此，**将较高的数据类型转换为较低的数据类型称为缩小类型转换**。作为这些转换的结果，当long值大于Integer.MAX_VALUE和Integer.MIN_VALUE时，某些位将丢失。

此外，我们将为每个转换变体展示它如何处理等于Integer.MAX_VALUE加1的long值。

## 2. 数据转换

### 2.1 转换值

首先，在Java中强制转换值是最常见的类型转换方式-它很简单：

```java
public int longToIntCast(long number) {
    return (int) number;
}
```

### 2.2 Java 8

从Java 8开始，我们可以使用另外两种方式进行类型转换：使用[Math](https://www.baeldung.com/java-lang-math)包或使用lambda函数。对于Math包，我们可以使用toIntExact方法：

```java
public int longToIntJavaWithMath(long number) {
    return Math.toIntExact(number);
}
```

### 2.3 包装类

另一方面，我们可以使用包装类Long来获取int值：

```java
public int longToIntBoxingValues(long number) {
    return Long.valueOf(number).intValue();
}
```

### 2.4 使用BigDecimal

此外，我们可以使用BigDecimal类完成此转换：

```java
public static int longToIntWithBigDecimal(long number) {
    return new BigDecimal(number).intValueExact();
}
```

### 2.5 使用Guava

接下来，我们将使用[Google Guava](https://www.baeldung.com/guava-guide)的Ints类演示类型转换：

```java
public int longToIntGuava(long number) {
    return Ints.checkedCast(number);
}
```

另外，[Google Guava](https://www.baeldung.com/guava-guide)的Ints类提供了一个saturatedCast方法：

```java
public int longToIntGuavaSaturated(long number) {
    return Ints.saturatedCast(number);
}
```

### 2.6 整数上限和下限

最后，我们需要考虑整数值有上限和下限。这些限制由Integer.MAX_VALUE和Integer.MIN_VALUE定义。对于超出这些限制的值，结果因方法而异。

在下一个代码片段中，我们将测试int值无法容纳long值的情况：

```java
@Test
public void longToIntSafeCast() {
    long max = Integer.MAX_VALUE + 10L;
    int expected = -2147483639;
    assertEquals(expected, longToIntCast(max));
    assertEquals(expected, longToIntJavaWithLambda(max));
    assertEquals(expected, longToIntBoxingValues(max));
}
```

使用直接强制转换、lambda或使用装箱值会产生负值。在这些情况下，long值大于Integer.MAX_VALUE，这就是结果值为负数的原因。如果long值小于Integer.MIN_VALUE，则结果值为正值。

另一方面，本文中描述的三种方法可能会引发不同类型的异常：

```java
@Test
public void longToIntIntegerException() {
    long max = Integer.MAX_VALUE + 10L;
    assertThrows(ArithmeticException.class, () -> ConvertLongToInt.longToIntWithBigDecimal(max));
    assertThrows(ArithmeticException.class, () -> ConvertLongToInt.longToIntJavaWithMath(max));
    assertThrows(IllegalArgumentException.class, () -> ConvertLongToInt.longToIntGuava(max));
}
```

对于第一个和第二个，抛出ArithmeticException。对于后者，抛出IllegalArgumentException。在这种情况下，Ints.checkedCast会检查整数是否超出范围。

最后，来自Guava的saturatedCast方法，首先检查整数限制并返回限制值是传递的数字大于或小于整数上限和下限：

```java
@Test
public void longToIntGuavaSaturated() {
    long max = Integer.MAX_VALUE + 10L;
    int expected = 2147483647;
    assertEquals(expected, ConvertLongToInt.longToIntGuavaSaturated(max));
}
```

## 3. 总结

在本文中，我们通过一些示例介绍了如何在Java中将long类型转换为int类型。使用原生Java强制转换和一些库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-4)上获得。
