---
layout: post
title:  在Java中将Map转换为数组、列表或集合
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

这篇简短的文章将展示如何使用纯Java以及基于[Guava](https://code.google.com/p/guava-libraries/)的快速示例将Map的值转换为Array、List或Set。

## 2. Map值转换为数组

首先，让我们看看使用普通Java将Map的值转换为数组：

```java
@Test
public void givenUsingCoreJava_whenMapValuesConvertedToArray_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    Collection<String> values = sourceMap.values();
    String[] targetArray = values.toArray(new String[0]);
}
```

请注意，与toArray(new T\[size])相比，toArray(new T\[0])是使用该方法的首选方式。正如Aleksey Shipilëv在他的[博客文章](https://shipilev.net/blog/2016/arrays-wisdom-ancients/#_conclusion)中所证明的那样，它似乎更快、更安全、更干净。

## 3. Map值转换为列表

接下来，让我们使用纯Java将Map的值转换为List：

```java
@Test
public void givenUsingCoreJava_whenMapValuesConvertedToList_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    List<String> targetList = new ArrayList<>(sourceMap.values());
}
```

以及使用Guava：

```java
@Test
public void givenUsingGuava_whenMapValuesConvertedToList_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    List<String> targetList = Lists.newArrayList(sourceMap.values());
}
```

## 4. Map值转换为集合

最后，让我们使用纯java将Map的值转换为Set：

```java
@Test
public void givenUsingCoreJava_whenMapValuesConvertedToS_thenCorrect() {
    Map<Integer, String> sourceMap = createMap();

    Set<String> targetSet = new HashSet<>(sourceMap.values());
}
```

## 5. 总结

如你所见，所有转换都可以使用一行代码完成，仅使用Java标准集合库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-1)上获得。