---
layout: post
title:  使用Stream API求和BigDecimal
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

我们通常使用Java [Stream API](https://www.baeldung.com/java-8-streams-introduction)来处理数据集合。

一个不错的功能是支持对数字流的操作，比如求和操作。但是，**我们不能以这种方式处理所有数字类型**。

在本教程中，我们将了解如何对BigDecimal等数字流执行求和运算。

## 2. 我们通常如何对流求和

Stream API提供数字流，包括IntStream、DoubleStream和LongStream。

让我们创建一个数字流，然后，我们将使用[IntStream#sum](https://www.baeldung.com/java-intstream-convert)计算总和：

```java
IntStream intNumbers = IntStream.range(0, 3);
assertEquals(3, intNumbers.sum());
```

我们可以对Double列表执行同样的操作。通过使用流，我们可以使用mapToDouble从对象流转换为DoubleStream：

```java
List<Double> doubleNumbers = Arrays.asList(23.48, 52.26, 13.5);
double result = doubleNumbers.stream()
    .mapToDouble(Double::doubleValue)
    .sum();
assertEquals(89.24, result, .1);
```

因此，如果我们能够以相同的方式对BigDecimal数字的集合求和，将会很有用。

**不幸的是，没有BigDecimalStream**。因此，我们需要另一种解决方案。

## 3. 使用Reduce将BigDecimal数相加

我们可以使用[Stream.reduce](https://www.baeldung.com/java-stream-reduce)而不是依赖sum：

```java
Stream<Integer> intNumbers = Stream.of(5, 1, 100);
int result = intNumbers.reduce(0, Integer::sum);
assertEquals(106, result);
```

**这适用于任何可以逻辑相加的东西，包括BigDecimal**：

```java
Stream<BigDecimal> bigDecimalNumber = Stream.of(BigDecimal.ZERO, BigDecimal.ONE, BigDecimal.TEN);
BigDecimal result = bigDecimalNumber.reduce(BigDecimal.ZERO, BigDecimal::add);
assertEquals(11, result);
```

reduce方法有两个参数：

-   Identity：相当于0-它是归约的起始值
-   Accumulator函数：接收两个参数，到目前为止的结果和流的下一个元素

## 4. 总结

在本文中，我们研究了如何对数字流进行求和，然后我们介绍了如何使用reduce作为替代方法。

使用reduce允许我们对BigDecimal数字的集合求和，它可以应用于任何其他类型。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-3)上获得。
