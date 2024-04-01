---
layout: post
title:  在Java中查找两个List之间的差集
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

查找相同数据类型的对象集合之间的差集是一项常见的编程任务。例如，假设我们有一个申请考试的学生列表，以及另一个通过考试的学生列表。这两个列表之间的差集可以告诉我们没有通过考试的学生。

在Java中，没有明确的方法可以找到List API中两个列表之间的差集，尽管有一些辅助方法很接近。

在这个快速教程中，**我们将学习如何找出两个列表之间的差集**。我们将尝试几种不同的方法，包括普通Java(使用和不使用Stream)和第三方库，例如Guava和Apache Commons Collections。

## 2. 测试设置

让我们首先定义两个列表，我们将使用它们来测试我们的示例：

```java
public class FindDifferencesBetweenListsUnitTest {
    private static final List listOne = Arrays.asList("Jack", "Tom", "Sam", "John", "James", "Jack");
    private static final List listTwo = Arrays.asList("Jack", "Daniel", "Sam", "Alan", "James", "George");
}
```

## 3. 使用Java List API

**我们可以创建一个列表的副本，然后使用List方法removeAll()删除与另一个列表共有的所有元素**：

```java
List<String> differences = new ArrayList<>(listOne);
differences.removeAll(listTwo);
assertEquals(2, differences.size());
assertThat(differences).containsExactly("Tom", "John");
```

让我们反过来找出差集：

```java
List<String> differences = new ArrayList<>(listTwo);
differences.removeAll(listOne);
assertEquals(3, differences.size());
assertThat(differences).containsExactly("Daniel", "Alan", "George");
```

我们还应该注意，如果我们想找到两个列表之间的共同元素，List还包含一个retainAll方法。

## 4. 使用Stream API

Java [Stream](https://www.baeldung.com/java-streams)可用于对集合中的数据执行顺序操作，**其中包括过滤列表之间的差集**：

```java
List<String> differences = listOne.stream()
    .filter(element -> !listTwo.contains(element))
    .collect(Collectors.toList());
assertEquals(2, differences.size());
assertThat(differences).containsExactly("Tom", "John");
```

与我们的第一个示例一样，我们可以切换列表的顺序以从第二个列表中找到不同的元素：

```java
List<String> differences = listTwo.stream()
    .filter(element -> !listOne.contains(element))
    .collect(Collectors.toList());
assertEquals(3, differences.size());
assertThat(differences).containsExactly("Daniel", "Alan", "George");
```

我们应该注意，对于较大的列表，重复调用List.contains()可能是一个代价高昂的操作。

## 5. 使用第三方库

### 5.1 使用Google Guava

**Guava包含一个方便的Sets.difference方法**，但要使用它，我们需要先将我们的List转换为Set：

```java
List<String> differences = new ArrayList<>(Sets.difference(Sets.newHashSet(listOne), Sets.newHashSet(listTwo)));
assertEquals(2, differences.size());
assertThat(differences).containsExactlyInAnyOrder("Tom", "John");
```

我们应该注意到，将List转换为Set会产生复制和重新排序的效果。

### 5.2 使用Apache Commons Collections

Apache Commons Collections中的CollectionUtils类包含一个removeAll方法。

**此方法执行与List.removeAll相同的操作，同时还为结果创建一个新集合**：

```java
List<String> differences = new ArrayList<>((CollectionUtils.removeAll(listOne, listTwo)));
assertEquals(2, differences.size());
assertThat(differences).containsExactly("Tom", "John");
```

## 6. 处理重复值

现在让我们看一下当两个列表包含重复值时获取差集。

为此，**我们需要从第一个列表中删除重复元素，精确地删除它们在第二个列表中包含的次数**。

在我们的示例中，值“Jack”在第一个列表中出现了两次，而在第二个列表中只出现了一次：

```java
List<String> differences = new ArrayList<>(listOne);
listTwo.forEach(differences::remove);
assertThat(differences).containsExactly("Tom", "John", "Jack");
```

我们也可以使用Apache Commons Collections的subtract方法来实现这一点：

```java
List<String> differences = new ArrayList<>(CollectionUtils.subtract(listOne, listTwo));
assertEquals(3, differences.size());
assertThat(differences).containsExactly("Tom", "John", "Jack");
```

## 7. 总结

在本文中，我们探讨了几种查找列表之间差集的方法。我们介绍了一个基本的Java解决方案、一个使用Stream API的解决方案，以及使用第三方库(如Google Guava和Apache Commons Collections)的解决方案。

我们还讨论了如何处理重复值。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-3)上获得。