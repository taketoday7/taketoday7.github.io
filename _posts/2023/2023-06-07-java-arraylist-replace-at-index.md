---
layout: post
title:  替换Java ArrayList中特定索引处的元素
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

通过本教程，我们将了解如何替换Java ArrayList中特定索引处的元素。

## 2. 常见做法

要替换现有元素，首先，我们需要[找到该元素](https://www.baeldung.com/find-list-element-java)在ArrayList中的确切位置。这个位置就是我们所说的索引。然后，我们可以用新元素替换旧元素。

**在Java ArrayList中替换元素的最常见方法是使用set(int index, Object element)方法**。set()方法有两个参数：现有元素的索引和新元素。

ArrayList的索引是从0开始的。因此，要替换第一个元素，0必须是作为参数传递的索引。

**如果提供的索引超出范围，将发生IndexOutOfBoundsException**。

## 3. 实现

让我们通过一个示例来了解如何替换Java ArrayList中特定索引处的元素。

```java
List<Integer> EXPECTED = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

List<Integer> aList = new ArrayList<>(Arrays.asList(1, 2, 7, 4, 5));
aList.set(2, 3);

assertThat(aList).isEqualTo(EXPECTED);
```

首先，我们创建一个包含5个元素的ArrayList。然后，我们用值7替换第三个元素，索引2为3。最后，我们可以看到值7的索引2从列表中删除并更新为新值3。另外，请注意列表大小为不受影响。

## 4. 总结

在这篇简短的文章中，我们学习了如何替换Java ArrayList中特定索引处的元素。此外，你可以将此方法用于任何其他列表类型，如LinkedList。只需确保你使用的列表不是不可变的。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-4)上获得。