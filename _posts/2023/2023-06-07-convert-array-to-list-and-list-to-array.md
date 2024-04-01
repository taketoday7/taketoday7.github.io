---
layout: post
title:  Java中数组和List之间的转换
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将学习如何使用核心Java库、Guava和Apache Commons Collections**在Array和List之间进行转换**。

## 2. 列表转数组

### 2.1 使用纯Java

让我们从使用普通Java从List到Array的转换开始：

```java
@Test
public void givenUsingCoreJava_whenListConvertedToArray_thenCorrect() {
    List<Integer> sourceList = Arrays.asList(0, 1, 2, 3, 4, 5);
    Integer[] targetArray = sourceList.toArray(new Integer[0]);
}
```

请注意，我们使用该方法的首选方式是toArray(new T\[0])与toArray(new T\[size])。正如Aleksey Shipilëv在他的[博客文章](https://shipilev.net/blog/2016/arrays-wisdom-ancients/#_conclusion)中所证明的那样，它似乎更快、更安全、更干净。

### 2.2 使用Guava

现在让我们使用Guava API进行相同的转换：

```java
@Test
public void givenUsingGuava_whenListConvertedToArray_thenCorrect() {
    List<Integer> sourceList = Lists.newArrayList(0, 1, 2, 3, 4, 5);
    int[] targetArray = Ints.toArray(sourceList);
}
```

## 3. 数组转列表

### 3.1 使用纯Java

让我们从将数组转换为List的普通Java解决方案开始：

```java
@Test
public void givenUsingCoreJava_whenArrayConvertedToList_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    List<Integer> targetList = Arrays.asList(sourceArray);
}
```

请注意，这是一个固定大小的列表，仍将由数组支持。如果我们想要一个标准的ArrayList，我们可以简单地实例化一个：

```java
List<Integer> targetList = new ArrayList<Integer>(Arrays.asList(sourceArray));
```

### 3.2 使用Guava

现在让我们使用Guava API进行相同的转换：

```java
@Test
public void givenUsingGuava_whenArrayConvertedToList_thenCorrect() {
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 };
    List<Integer> targetList = Lists.newArrayList(sourceArray);
}
```

### 3.3 使用Commons Collections

最后，让我们使用[Apache Commons Collections](https://commons.apache.org/proper/commons-collections/javadocs/) CollectionUtils.addAll API将数组的元素填充到一个空列表中：

```java
@Test 
public void givenUsingCommonsCollections_whenArrayConvertedToList_thenCorrect() { 
    Integer[] sourceArray = { 0, 1, 2, 3, 4, 5 }; 
    List<Integer> targetList = new ArrayList<>(6); 
    CollectionUtils.addAll(targetList, sourceArray); 
}
```

## 4. 总结

所有这些示例和代码片段的实现都可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-1)上找到。这是一个基于Maven的项目，因此它应该很容易导入并按原样运行。