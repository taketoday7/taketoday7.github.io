---
layout: post
title:  从Java中的List中删除所有空值
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

这个简短的教程将展示如何使用纯Java、Guava、Apache Commons Collections和更新的Java 8 Lambda支持**从列表中删除所有空元素**。

## 2. 使用纯Java从列表中删除空值

Java集合框架提供了一个简单的解决方案来删除List中的所有null元素-一个基本的while循环：

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithPlainJava_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    while (list.remove(null));

    assertThat(list, hasSize(1));
}
```

或者，我们也可以使用以下简单的方法：

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithPlainJavaAlternative_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    list.removeAll(Collections.singleton(null));

    assertThat(list, hasSize(1));
}
```

请注意，这两种解决方案都会修改源列表。

## 3. 使用Google Guava从列表中删除空值

我们还可以使用Guava和更实用的方法通过谓词删除空值：

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithGuavaV1_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null);
    Iterables.removeIf(list, Predicates.isNull());

    assertThat(list, hasSize(1));
}
```

或者，如果我们不想修改源列表，Guava将允许我们创建一个新的过滤器列表：

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithGuavaV2_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, null, 2, 3);
    List<Integer> listWithoutNulls = Lists.newArrayList(Iterables.filter(list, Predicates.notNull()));

    assertThat(listWithoutNulls, hasSize(3));
}
```

## 4. 使用Apache Commons Collections从列表中删除空值

现在让我们看一个简单的解决方案，它使用Apache Commons Collections库，具有类似的函数风格：

```java
@Test
public void givenListContainsNulls_whenRemovingNullsWithCommonsCollections_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    CollectionUtils.filter(list, PredicateUtils.notNullPredicate());

    assertThat(list, hasSize(3));
}
```

请注意，此解决方案也将修改原始列表。

## 5. 使用Lambda从列表中删除空值(Java 8)

最后-让我们看一下**使用Lambda过滤List的Java 8解决方案**；过滤过程可以并行或串行完成：

```java
@Test
public void givenListContainsNulls_whenFilteringParallel_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    List<Integer> listWithoutNulls = list.parallelStream()
        .filter(Objects::nonNull)
        .collect(Collectors.toList());
}

@Test
public void givenListContainsNulls_whenFilteringSerial_thenCorrect() {
    List<Integer> list = Lists.newArrayList(null, 1, 2, null, 3, null);
    List<Integer> listWithoutNulls = list.stream()
        .filter(Objects::nonNull)
        .collect(Collectors.toList());
}

public void givenListContainsNulls_whenRemovingNullsWithRemoveIf_thenCorrect() {
    List<Integer> listWithoutNulls = Lists.newArrayList(null, 1, 2, null, 3, null);
    listWithoutNulls.removeIf(Objects::isNull);

    assertThat(listWithoutNulls, hasSize(3));
}
```

就是这样-一些快速且非常有用的解决方案，用于从List中删除所有空元素。

## 6. 总结

在本文中，我们探讨了使用Java、Guava或Lambda从List中删除空值的不同方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-1)上获得。