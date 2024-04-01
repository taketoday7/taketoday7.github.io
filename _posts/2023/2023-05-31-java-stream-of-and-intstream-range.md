---
layout: post
title:  了解Stream.of()和IntStream.range()之间的区别
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

[Stream API](https://www.baeldung.com/java-8-streams)是Java 8中的主要新特性之一。

在本教程中，我们将讨论一个有趣的话题：Stream.of()和[IntStream.range()](https://www.baeldung.com/java-listing-numbers-within-a-range#intstream)之间的区别。

## 2. 问题简介

**我们可以使用Stream.of()方法初始化一个Stream对象，例如Stream.of(1, 2, 3, 4, 5)**。或者，如果我们想要初始化一个整数流，**[IntStream](https://www.baeldung.com/java-8-primitive-streams)是一种更直接的类型，例如IntStream.range(1, 6)**。但是，这两种方法创建的整数流的行为可能不同。

像往常一样，我们将通过一个例子来理解这个问题。首先，让我们以不同的方式创建两个流：

```java
Stream<Integer> normalStream = Stream.of(1, 2, 3, 4, 5);
IntStream intStreamByRange = IntStream.range(1, 6);
```

接下来，我们将对上面的两个流执行相同的例程：

```java
STREAM.peek(add to a result list)
    .sorted()
    .findFirst();
```

因此，我们在每个流上调用三个方法：

-   首先-调用[peek()](https://www.baeldung.com/java-streams-peek-api)方法将处理过的元素收集到结果列表中
-   然后-对元素进行排序
-   最后-从流中取出第一个元素

由于两个流包含相同的整数元素，我们认为在执行后，两个结果列表也应该包含相同的整数。那么接下来，让我们编写一个测试来检查它是否产生了我们期望的结果：

```java
List<Integer> normalStreamPeekResult = new ArrayList<>();
List<Integer> intStreamPeekResult = new ArrayList<>();

// First, the regular Stream
normalStream.peek(normalStreamPeekResult::add)
    .sorted()
    .findFirst();
assertEquals(Arrays.asList(1, 2, 3, 4, 5), normalStreamPeekResult);

// Then, the IntStream
intStreamByRange.peek(intStreamPeekResult::add)
    .sorted()
    .findFirst();
assertEquals(Arrays.asList(1), intStreamPeekResult);
```

执行后发现，normalStream.peek()填充的结果列表包含所有整数元素。然而，由intStreamByRange.peek()填充的列表只有1个元素。

接下来，让我们弄清楚为什么它会这样。

## 3. 流是惰性的

在我们解释为什么两个流在前面的测试中产生不同的结果列表之前，让我们先了解一下Java Stream在设计上是惰性的。

“惰性”意味着流仅在被告知产生结果时才执行所需的操作。换句话说，**在执行终端操作之前，不会执行对流的中间操作**。这种惰性行为可能是一个优势，因为它允许更高效的处理并防止不必要的计算。

为了快速理解这种惰性行为，让我们暂时摆脱之前测试中的sort()方法调用并重新运行它：

```java
List<Integer> normalStreamPeekResult = new ArrayList<>();
List<Integer> intStreamPeekResult = new ArrayList<>();

// First, the regular Stream
normalStream.peek(normalStreamPeekResult::add)
    .findFirst();
assertEquals(Arrays.asList(1), normalStreamPeekResult);

// Then, the IntStream
intStreamByRange.peek(intStreamPeekResult::add)
    .findFirst();
assertEquals(Arrays.asList(1), intStreamPeekResult);
```

这次，两个流都只填充了相应结果列表中的第一个元素。**这是因为[findFirst()](https://www.baeldung.com/java-stream-findfirst-vs-findany#usingstreamfindfirst)方法是终端操作，只需要一个元素-第一个**。

现在我们了解了Stream是惰性的，接下来，让我们弄清楚为什么当sorted()方法加入调用时两个结果列表不同。

## 4. 调用sorted()可能会将流变成“急切”

首先，让我们看一下由Stream.of()初始化的流。终端操作findFirst()只需要流中的第一个整数，它是**sorted()操作之后的第一个**。

我们知道必须遍历所有整数才能对它们进行排序。因此，调用sorted()已将流变为“急切”。因此，**每个元素都会调用peek()方法**。

另一方面，**IntStream.range()返回一个按顺序排序的IntStream**。也就是说，IntStream对象的输入已经排序了。此外，当它对已经排序的输入进行排序时，**Java会应用优化以[使sorted()操作成为无操作](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/stream/SortedOps.java#L136)**。因此，结果列表中仍然只有一个元素。

接下来，让我们看另一个基于TreeSet的Stream示例：

```java
List<String> peekResult = new ArrayList<>();

TreeSet<String> treeSet = new TreeSet<>(Arrays.asList("CCC", "BBB", "AAA", "DDD", "KKK"));

treeSet.stream()
    .peek(peekResult::add)
    .sorted()
    .findFirst();

assertEquals(Arrays.asList("AAA"), peekResult);
```

我们知道TreeSet是一个有序的集合。因此，我们看到peekResult列表只包含一个字符串，尽管我们调用了sorted()。

## 5. 总结

在本文中，我们以Stream.of()和IntStream.range()为例来理解调用sorted()可能会使Stream从“惰性”变为“急切”。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。
