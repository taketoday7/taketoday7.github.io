---
layout: post
title:  Collection.clear()和Collection.removeAll()之间的区别
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将了解两种看似做同样事情但实际上并非如此的Collection方法：clear()和removeAll()。

我们将首先看到方法定义，然后在简短示例中使用它们。

## 2. Collection.clear()

我们将首先深入了解Collection.clear()方法。让我们检查[该方法的Javadoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#clear())。根据文档，**clear()的目的是从列表中删除每个元素**。

因此，基本上，对任何列表调用clear()都会导致列表变为空。

## 3. Collection.removeAll()

现在我们来看看Collection.removeAll()的[Javadoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#removeAll(java.util.Collection))。我们可以看到该方法将Collection作为参数。**它的目的是删除列表和集合之间的所有公共元素**。

因此，当在集合上调用它时，它将从传递的参数中删除所有元素，这些元素也存在于我们调用removeAll()的集合中。

## 4. 例子

现在让我们看一些代码来了解这些方法的作用。我们将首先创建一个名为ClearVsRemoveAllUnitTest的测试类。

之后，我们将为Collection.clear()创建第一个测试。

我们将用一些数字初始化一个整数集合，并对其调用clear()，这样列表中就没有元素了：

```java
@Test
void whenClear_thenListBecomesEmpty() {
    Collection<Integer> collection = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

    collection.clear();

    assertTrue(collection.isEmpty());
}
```

正如我们所看到的，调用clear()后集合为空。

让我们用两个集合创建第二个测试，一个包含从1到5的数字，另一个包含从3到7的数字。之后，我们将在第一个集合上调用removeAll()并将第二个集合作为参数。

我们预计只有数字1和2保留在第一个集合中(而第二个集合保持不变)：

```java
@Test
void whenRemoveAll_thenFirstListMissElementsFromSecondList() {
    Collection<Integer> firstCollection = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
    Collection<Integer> secondCollection = new ArrayList<>(Arrays.asList(3, 4, 5, 6, 7));

    firstCollection.removeAll(secondCollection);

    assertEquals(Arrays.asList(1, 2), firstCollection);
    assertEquals(Arrays.asList(3, 4, 5, 6, 7), secondCollection);
}
```

我们的期望得到满足。第一个集合中只剩下数字1和2，第二个没有改变。

## 5. 总结

在本文中，我们了解了Collection.clear()和Collection.removeAll()的用途。

尽管我们一开始可能会这么想，但他们并没有做同样的事情。clear()删除集合中的每个元素，而removeAll()只删除与另一个Collection中匹配的元素。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-3)上获得。