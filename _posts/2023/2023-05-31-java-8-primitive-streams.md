---
layout: post
title:  Java中的原始类型流
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

Stream API是Java 8中添加的关键特性之一。

简而言之，API允许我们通过提供声明性API来[方便且更有效](https://www.baeldung.com/java-8-streams)地处理集合和其他元素序列。

## 2. 原始流

Stream主要处理对象集合而不是原始类型。

幸运的是，为了提供一种方法来处理三种最常用的原始类型-int 、long和double，标准库包括三个原始类型专用的实现：IntStream、LongStream和DoubleStream。

原始类型流受到限制的主要原因是装箱开销，并且因为在许多情况下为其他原始类型流创建专用流并不是那么有用。

## 3. 算术运算

让我们从一些用于频繁使用的算术运算(如min、max、sum和average )的有趣方法开始：

```java
int[] integers = new int[] {20, 98, 12, 7, 35};
int min = Arrays.stream(integers)
    .min()
    .getAsInt(); // returns 7
```

现在让我们逐步浏览上面的代码片段以了解发生了什么。

我们使用java.util.Arrays.stream(int[])创建了IntStream，然后使用min()方法获取最小整数作为java.util.OptionalInt，最后调用getAsInt()获取int值。

创建IntStream的另一种方法是使用IntStream.of(int ...)。max()方法将返回最大的整数：

```java
int max = IntStream.of(20, 98, 12, 7, 35)
    .max()
    .getAsInt(); // returns 98
```

接下来，要获得整数的总和，我们只需调用sum()方法，我们不需要使用getAsInt()，因为它已经将结果作为int值返回：

```java
int sum = IntStream.of(20, 98, 12, 7, 35).sum(); // returns 172
```

我们调用average()方法来获取整数值的平均值，正如我们所看到的，我们应该使用getAsDouble()，因为它返回一个double类型的值。

```java
double avg = IntStream.of(20, 98, 12, 7, 35)
    .average()
    .getAsDouble(); // returns 34.4
```

## 4. 范围

我们还可以创建一个基于范围的IntStream：

```java
int sum = IntStream.range(1, 10)
    .sum(); // returns 45
int sum = IntStream.rangeClosed(1, 10)
    .sum(); // returns 55
```

如上面的代码片段所示，有两种方法range()和rangeClosed()可以创建整数值范围。

不同之处在于range()的末尾是排除的，而在rangeClosed()中是包含的。

范围方法仅适用于IntStream和LongStream。

我们可以使用range作为for-each循环的奇特形式：

```java
IntStream.rangeClosed(1, 5)
    .forEach(System.out::println);
```

将它们用作for-each循环替代品的好处在于，我们还可以利用并行执行：

```java
IntStream.rangeClosed(1, 5)
    .parallel()
    .forEach(System.out::println);
```

尽管这些花哨的循环很有用，但在某些情况下，由于简单性、可读性和性能，使用传统的for循环而不是函数式循环进行简单迭代仍然更好。

## 5. 装箱和拆箱

有时我们需要将原始值转换为它们的包装等价物。

在这些情况下，我们可以使用boxed()方法：

```java
List<Integer> evenInts = IntStream.rangeClosed(1, 10)
    .filter(i -> i % 2 == 0)
    .boxed()
    .collect(Collectors.toList());
```

我们还可以将包装类流转换为原始类型流：

```java
// returns 78
int sum = Arrays.asList(33,45)
    .stream()
    .mapToInt(i -> i)
    .sum();
```

我们总是可以使用mapToXxx和flatMapToXxx方法来创建原始类型流。

## 6. 总结

Java Stream是对该语言的一个非常强大的补充。我们在这里几乎没有触及原始类型流的表面，但是，因为你已经可以使用它们来提高工作效率。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-3)上获得。
