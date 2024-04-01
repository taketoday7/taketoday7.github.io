---
layout: post
title:  从Java List中删除所有重复元素
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将学习如何清除列表中的重复元素。首先，我们将使用纯Java，然后是Guava，最后是基于Java 8 Lambda的解决方案。

## 2. 使用纯Java从列表中删除重复元素

我们可以使用标准的Java集合框架通过Set轻松地从List中删除重复元素：

```java
public void givenListContainsDuplicates_whenRemovingDuplicatesWithPlainJava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = new ArrayList<>(new HashSet<>(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInAnyOrder(5, 0, 3, 1, 2));
}
```

如我们所见，原始列表保持不变。

在上面的示例中，我们使用了HashSet实现，它是一个无序集合。因此，**清理后的listWithoutDuplicates的顺序可能与原始listWithDuplicates的顺序不同**。

如果我们需要保留顺序，我们可以改用LinkedHashSet：

```java
public void givenListContainsDuplicates_whenRemovingDuplicatesPreservingOrderWithPlainJava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = new ArrayList<>(new LinkedHashSet<>(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInRelativeOrder(5, 0, 3, 1, 2));
}
```

## 3. 使用Guava从列表中删除重复元素

我们也可以使用Guava执行同样的操作：

```java
public void givenListContainsDuplicates_whenRemovingDuplicatesWithGuava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = Lists.newArrayList(Sets.newHashSet(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInAnyOrder(5, 0, 3, 1, 2));
}
```

在这里，原始列表也保持不变。

同样，清理列表中元素的顺序可能是随机的。

如果我们使用LinkedHashSet实现，可以保留初始顺序：

```java
public void givenListContainsDuplicates_whenRemovingDuplicatesPreservingOrderWithGuava_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = Lists.newArrayList(Sets.newLinkedHashSet(listWithDuplicates));

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInRelativeOrder(5, 0, 3, 1, 2));
}
```

## 4. 使用Java 8 Lambda从列表中删除重复项

最后，让我们看一个新的解决方案，在Java 8中使用Lambda。我们将**使用Stream API中的distinct()方法**，该方法根据equals()方法返回的结果返回一个由不同元素组成的流。

此外，**对于有序流，不同元素的选择是稳定的**。这意味着对于重复的元素，将保留在遭遇顺序中首先出现的元素：

```java
public void givenListContainsDuplicates_whenRemovingDuplicatesWithJava8_thenCorrect() {
    List<Integer> listWithDuplicates = Lists.newArrayList(5, 0, 3, 1, 2, 3, 0, 0);
    List<Integer> listWithoutDuplicates = listWithDuplicates.stream()
        .distinct()
        .collect(Collectors.toList());

    assertThat(listWithoutDuplicates, hasSize(5));
    assertThat(listWithoutDuplicates, containsInAnyOrder(5, 0, 3, 1, 2));
}
```

我们有三种快速方法来清理列表中的所有重复元素。

## 5. 总结

在本文中，我们演示了使用纯Java、Google Guava和Java 8从列表中删除重复元素。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-1)上获得。