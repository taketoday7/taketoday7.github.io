---
layout: post
title:  两个整数数组之间的交集
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在这个快速教程中，我们将了解如何**计算两个整数数组'a'和'b'之间的交集**。

我们还将关注如何处理重复条目。

对于实现，我们将使用Stream。

## 2. 数组的成员谓词

根据定义，两个集合的交集是一个集合，其中包含一个集合的所有值，这些值也是第二个集合的一部分。

因此我们需要一个Function，或者更确切地说是一个Predicate来决定第二个数组中的成员资格。由于List提供了这种开箱即用的方法，我们将把它转换为List：

```java
Predicate isContainedInB = Arrays.asList(b)::contains;
```

## 3. 构建交集

为了构建结果数组，我们将按顺序考虑第一个集合的元素，并验证它们是否也包含在第二个数组中。然后我们将基于此创建一个新数组。

Stream API为我们提供了所需的方法。**首先，我们将创建一个Stream，然后使用membership Predicate进行过滤，最后我们将创建一个新数组**：

```java
public static Integer[] intersectionSimple(Integer[] a, Integer[] b){
    return Stream.of(a)
        .filter(Arrays.asList(b)::contains)
        .toArray(Integer[]::new);
}
```

## 4. 重复条目

由于Java中的数组不是Set实现，因此我们面临着输入和结果中重复条目的问题。请注意，结果中出现的次数取决于第一个参数中出现的次数。

但是对于Set，元素不能出现多次。**我们可以使用distinct()方法将其存档**：

```java
public static Integer[] intersectionSet(Integer[] a, Integer[] b){
    return Stream.of(a)
        .filter(Arrays.asList(b)::contain)
        .distinct()
        .toArray(Integer[]::new);
}
```

因此交集的长度不再依赖于参数顺序。

但是，由于我们删除了双重条目，因此数组与其自身的交集可能不再是数组。

## 5. 多集交集

允许多个相等条目的更一般的概念是多重集。对他们来说，交集由输入出现的最少次数定义。所以我们的membership Predicate必须记录我们向结果添加元素的频率。

remove()方法可用于此，它返回成员资格并使用元素。因此，在'b'中的所有相等元素都被消耗后，不再向结果添加相等元素：

```java
public static Integer[] intersectionSet(Integer[] a, Integer[] b){
    return Stream.of(a)
        .filter(new LinkedList<>(Arrays.asList(b))::remove)
        .toArray(Integer[]::new);
}
```

由于Arrays API只返回一个不可变的列表，我们必须生成一个专用的可变列表。

## 6. 总结

在本文中，我们了解了如何使用contains和remove方法在Java中实现两个数组的交集。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-operations-advanced)上获得。