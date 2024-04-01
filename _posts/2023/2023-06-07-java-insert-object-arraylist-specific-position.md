---
layout: post
title:  在ArrayList中的特定位置插入对象
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将学习如何在[ArrayList](https://www.baeldung.com/java-arraylist)中的特定位置插入对象。

## 2. 示例

如果我们想将一个元素添加到[ArrayList](https://www.baeldung.com/java-arraylist)的特定位置，**我们可以使用通过实现List<E\>接口提供的add(int index, E element)方法。这个方法允许我们在特定索引处添加一个元素**。

**如果索引超出范围(index < 0或者index > size())，它会抛出IndexOutOfBoundsException**。这意味着如果我们在ArrayList中只有4个元素，我们不能使用它在位置4添加元素，因为我们从0开始计数。我们必须在这里使用标准的add(E e)方法。

首先，我们将创建一个新的ArrayList并向其添加四个元素：

```java
List<Integer> integers = new ArrayList<>();
integers.add(5);
integers.add(6);
integers.add(7);
integers.add(8);
System.out.println(integers);
```

这将导致：

![](/assets/images/2023/javacollection/javainsertobjectarraylistspecificposition01.png)

现在，如果我们在索引1处添加另一个元素：

```java
integers.add(1, 9);
System.out.println(integers);
```

ArrayList内部将首先从给定索引开始移动对象：

![](/assets/images/2023/javacollection/javainsertobjectarraylistspecificposition02.png)

这是可行的，因为ArrayList是一个可增长的数组，可以根据需要自动调整容量：

![](/assets/images/2023/javacollection/javainsertobjectarraylistspecificposition03.png)

然后在给定索引处添加新元素：

![](/assets/images/2023/javacollection/javainsertobjectarraylistspecificposition04.png)

**添加特定索引将导致ArrayList的平均操作性能为O(n/2)**。例如，[LinkedList](https://www.baeldung.com/java-linkedlist)的平均复杂度为O(n/4)，如果索引为0，则复杂度为O(1)。因此，如果我们严重依赖于在特定位置添加元素，则需要仔细研究[LinkedList](https://www.baeldung.com/java-linkedlist)。

我们还可以看到元素的顺序不再正确。当我们在特定位置手动添加元素时，这是我们经常想要实现的。否则，我们可以使用integers.sort(Integer::compareTo)再次对ArrayList进行排序或实现我们自己的比较器。

## 3. 总结

在本文中，我们讨论了add(int index, E element)方法，以便我们可以在特定位置向ArrayList<E\>添加新元素。我们必须注意保持在ArrayList的索引范围内，并确保我们允许正确的对象。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-4)上获得。