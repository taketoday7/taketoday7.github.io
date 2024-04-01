---
layout: post
title:  在Java数组中查找总和和平均值
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在这个快速教程中，我们将介绍如何使用Java标准循环和Stream API计算数组中的总和和平均值。

## 2. 求数组元素之和

### 2.1 使用For循环求和

为了找到数组中所有元素的总和，**我们可以简单地迭代数组并将每个元素添加到一个累加变量sum中**。

这非常简单地从0的总和开始，并在此过程中将数组中的每个元素相加：

```java
public static int findSumWithoutUsingStream(int[] array) {
    int sum = 0;
    for (int value : array) {
        sum += value;
    }
    return sum;
}
```

### 2.2 使用Java Stream API求和

**我们可以使用Stream API来实现相同的结果**：

```java
public static int findSumUsingStream(int[] array) {
    return Arrays.stream(array).sum();
}
```

重要的是要知道sum()方法只支持[原始类型流](https://www.baeldung.com/java-8-primitive-streams)。

如果我们想在装箱的Integer值上使用流，我们必须首先使用mapToInt方法将流转换为IntStream。

之后，我们可以将sum()方法应用于我们新转换的IntStream：

```java
public static int findSumUsingStream(Integer[] array) {
    return Arrays.stream(array)
        .mapToInt(Integer::intValue)
        .sum();
}
```

你可以在[此处](https://www.baeldung.com/java-8-streams)阅读更多关于Stream API的信息。

## 3. 在Java数组中求平均值

### 3.1 不使用Stream API的平均值

一旦我们知道如何计算数组元素的总和，求平均值就很容易了-因为平均值 = 元素总和 / 元素数量：

```java
public static double findAverageWithoutUsingStream(int[] array) {
    int sum = findSumWithoutUsingStream(array);
    return (double) sum / array.length;
}
```

注意事项：

1.  将一个int除以另一个int返回一个int结果。**为了获得准确的平均值，我们首先将sum转换为double**。
2.  Java数组有一个length字段，用于存储数组中元素的数量。

### 3.2 使用Java Stream API的平均值

```java
public static double findAverageUsingStream(int[] array) {
    return Arrays.stream(array).average().orElse(Double.NaN);
}
```

IntStream.average()返回一个OptionalDouble，它可能不包含值并且需要特殊处理。

阅读[本文](https://www.baeldung.com/java-optional)中有关Optionals的更多信息以及[Java 8文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/OptionalDouble.html#orElse(double))中有关OptionalDouble类的信息。

## 4。总结

在本文中，我们探讨了如何求出int数组元素的总和/平均值。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-operations-advanced)上获得。