---
layout: post
title:  在Java中对HashSet进行排序
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

[HashSet](https://www.baeldung.com/java-hashset)是java.util包中的集合类。该类继承于AbstractSet类，实现了[Set](https://www.baeldung.com/java-set-operations)接口。此外，HashSet不保留元素的顺序，因此需要找到对这些元素进行排序的方法。

在这个快速教程中，**我们将学习多种技术来对HashSet的元素进行排序**。

## 2. 使用Collections.sort()方法

Collections.sort()方法对实现java.util.List接口的对象集合进行排序。因此，我们可以将HashSet转换为List，然后使用Collections.sort()对其进行排序：

```java
HashSet<Integer> numberHashSet = new HashSet<>();
numberHashSet.add(2);
numberHashSet.add(1);
numberHashSet.add(4);
numberHashSet.add(3);

// converting HashSet to arraylist
ArrayList arrayList = new ArrayList(numberHashSet);

// sorting the list
Collections.sort(arrayList);

assertThat(arrayList).containsExactly(1, 2, 3, 4);
```

在上面的示例中，我们首先将HashSet的元素复制到[ArrayList](https://www.baeldung.com/java-arraylist)中。然后，我们使用ArrayList作为Collections.sort()方法的参数。除了ArrayList，我们还可以使用[LinkedList](https://www.baeldung.com/java-linkedlist)或[Vector](https://www.baeldung.com/java-arraylist-vs-vector#vector)。

## 3. 使用TreeSet

使用这种方法，我们将HashSet转换为[TreeSet](https://www.baeldung.com/java-tree-set)，它与HashSet类似，只是它按升序存储元素。因此，**当HashSet转换为TreeSet时，HashSet元素会按顺序排列**：

```java
HashSet<Integer> numberHashSet = new HashSet<>();
numberHashSet.add(2);
numberHashSet.add(1);
numberHashSet.add(4);
numberHashSet.add(3);

TreeSet<Integer> treeSet = new TreeSet<>();
treeSet.addAll(numberHashSet);

assertThat(treeSet).containsExactly(1, 2, 3, 4);
```

我们可以看到，使用TreeSet对HashSet进行排序非常简单。我们只需要使用HashSet列表作为参数创建TreeSet的实例。

## 4. 使用stream().sorted()方法

有一种简洁的方法可以使用[Stream API](https://www.baeldung.com/java-8-streams)的stream().sorted()方法对HashSet进行排序。这个在Java 8中引入的API允许我们对一组元素执行函数操作。此外，它可以从不同的集合中获取对象并以所需的方式显示它们，具体取决于我们使用的管道方法。

在我们的示例中，**我们将使用stream().sorted()方法，该方法返回一个Stream，其元素按特定顺序排序**。需要注意的是，由于原来的HashSet没有被修改，所以我们需要将排序的结果保存在一个新的Collection中。我们将使用collect()方法将数据存储回新的HashSet中：

```java
HashSet<Integer> numberHashSet = new HashSet<>();
numberHashSet.add(200);
numberHashSet.add(100);
numberHashSet.add(400);
numberHashSet.add(300);

HashSet<Integer> sortedHashSet = numberHashSet.stream()
    .sorted()
    .collect(Collectors.toCollection(LinkedHashSet::new));

assertThat(sortedHashSet).containsExactly(100, 200, 300, 400);
```

我们应该注意，当我们使用不带参数的stream().sorted()方法时，它按自然顺序对HashSet进行排序。我们还可以使用比较器重载它来定义自定义排序顺序。

## 5. 总结

在本文中，我们讨论了如何使用三种方式在Java中对HashSet进行排序：使用Collections.sort()方法、使用TreeSet和使用stream().sorted()方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-4)上获得。