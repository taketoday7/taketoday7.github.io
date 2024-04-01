---
layout: post
title:  Java中数组和Set之间的转换
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这篇简短的文章中，我们将研究数组和Set之间的转换-首先使用纯java，然后使用Guava和Apache的Commons Collections库。

## 2. 将数组转换为Set

### 2.1 使用纯Java

让我们首先看看如何使用普通Java将数组转换为Set：

```java
@Test
public void givenUsingCoreJavaV1_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = new HashSet<Integer>(Arrays.asList(sourceArray));
}
```

或者，可以先创建Set，然后用数组元素填充：

```java
@Test
public void givenUsingCoreJavaV2_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = new HashSet<Integer>();
    Collections.addAll(targetSet, sourceArray);
}
```

### 2.2 使用Google Guava

接下来，让我们看看Guava从数组到Set的转换：

```java
@Test
public void givenUsingGuava_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = Sets.newHashSet(sourceArray);
}
```

### 2.3 使用Apache Commons Collections

最后，让我们使用Apache的Commons Collection库进行转换：

```java
@Test
public void givenUsingCommonsCollections_whenArrayConvertedToSet_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    Set<Integer> targetSet = new HashSet<>(6);
    CollectionUtils.addAll(targetSet, sourceArray);
}
```

## 3. Set转数组

### 3.1 使用纯Java

现在让我们看看相反的情况-将现有的Set转换为数组：

```java
@Test
public void givenUsingCoreJava_whenSetConvertedToArray_thenCorrect() {
    Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
    Integer[] targetArray = sourceSet.toArray(new Integer[0]);
}
```

请注意，与toArray(new T\[size])相比，toArray(new T\[0])是使用该方法的首选方式。正如Aleksey Shipilëv在他的[博客文章](https://shipilev.net/blog/2016/arrays-wisdom-ancients/#_conclusion)中所证明的那样，它似乎更快、更安全、更干净。

### 3.2 使用Guava

接下来-Guava解决方案：

```java
@Test
public void givenUsingGuava_whenSetConvertedToArray_thenCorrect() {
    Set<Integer> sourceSet = Sets.newHashSet(0, 1, 2, 3, 4, 5);
    int[] targetArray = Ints.toArray(sourceSet);
}
```

请注意，我们使用的是Guava的Ints API，因此此解决方案特定于我们正在使用的数据类型。

## 4. 总结

所有这些示例和代码片段的实现都可以在[Github](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-1)上找到-这是一个基于Maven的项目，因此应该很容易导入和运行。