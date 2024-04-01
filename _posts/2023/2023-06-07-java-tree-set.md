---
layout: post
title:  Java TreeSet指南
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将介绍Java集合框架的组成部分和最流行的Set实现之一–TreeSet。

## 2. TreeSet介绍

简单地说，TreeSet是一个扩展了AbstractSet类并实现了NavigableSet接口的排序集合。

以下是此实现最重要方面的快速总结：

-   它存储唯一的元素
-   它不保留元素的插入顺序
-   它按升序对元素进行排序
-   它不是线程安全的

**在此实现中，对象根据其自然顺序按升序排序和存储**。TreeSet使用自平衡二叉搜索树，更具体地说是[红黑树](https://www.baeldung.com/cs/red-black-trees)。

简单来说，作为一个自平衡二叉搜索树，二叉树的每个节点都包含一个额外的位，用于标识该节点的颜色是红色还是黑色。在随后的插入和删除过程中，这些“颜色”位有助于确保树或多或少保持平衡。

让我们创建一个TreeSet的实例：

```java
Set<String> treeSet = new TreeSet<>();
```

### 2.1 使用构造函数比较器参数的TreeSet

或者，我们可以使用构造函数构造一个TreeSet，该构造函数允许我们使用Comparable或Comparator定义元素排序的顺序：

```java
Set<String> treeSet = new TreeSet<>(Comparator.comparing(String::length));
```

**虽然TreeSet不是线程安全的，但可以使用Collections.synchronizedSet()包装器在外部同步它**：

```java
Set<String> syncTreeSet = Collections.synchronizedSet(treeSet);
```

好了，现在我们对如何创建TreeSet实例有了清晰的认识，让我们来看看我们可用的常用操作。

## 3. add()

正如预期的那样，add()方法可用于将元素添加到TreeSet。如果添加了元素，该方法返回true，否则返回false。

**该方法的约定规定，只有当Set中不存在相同元素时，才会添加该元素**。

让我们向TreeSet添加一个元素：

```java
@Test
public void whenAddingElement_shouldAddElement() {
    Set<String> treeSet = new TreeSet<>();

    assertTrue(treeSet.add("String Added"));
 }
```

**add方法非常重要，因为该方法的实现细节说明了TreeSet的内部工作方式**，以及它如何利用TreeMap的put方法来存储元素：

```java
public boolean add(E e) {
    return m.put(e, PRESENT) == null;
}
```

变量m指的是内部支持TreeMap(请注意TreeMap实现了NavigableMap)：

```java
private transient NavigableMap<E, Object> m;
```

因此，TreeSet在内部依赖于后备NavigableMap，该NavigableMap在创建TreeSet的实例时使用TreeMap的实例进行初始化：

```java
public TreeSet() {
    this(new TreeMap<E,Object>());
}
```

有关更多信息，请参阅[本文](https://www.baeldung.com/java-treemap)。

## 4. contains()

**contains()方法用于检查给定元素是否存在于给定的TreeSet中**。如果找到该元素，则返回true，否则返回false。

让我们看看contains()的实际应用：

```java
@Test
public void whenCheckingForElement_shouldSearchForElement() {
    Set<String> treeSetContains = new TreeSet<>();
    treeSetContains.add("String Added");

    assertTrue(treeSetContains.contains("String Added"));
}
```

## 5. remove()

**remove()方法用于从集合中删除指定的元素(如果存在)**。

如果集合包含指定元素，则此方法返回true。

让我们看看它的实际效果：

```java
@Test
public void whenRemovingElement_shouldRemoveElement() {
    Set<String> removeFromTreeSet = new TreeSet<>();
    removeFromTreeSet.add("String Added");

    assertTrue(removeFromTreeSet.remove("String Added"));
}
```

## 6. clear()

如果我们想从集合中删除所有元素，我们可以使用clear()方法：

```java
@Test
public void whenClearingTreeSet_shouldClearTreeSet() {
    Set<String> clearTreeSet = new TreeSet<>();
    clearTreeSet.add("String Added");
    clearTreeSet.clear();
 
    assertTrue(clearTreeSet.isEmpty());
}
```

## 7. size()

size()方法用于识别TreeSet中存在的元素数量，它是API中的基本方法之一：

```java
@Test
public void whenCheckingTheSizeOfTreeSet_shouldReturnThesize() {
    Set<String> treeSetSize = new TreeSet<>();
    treeSetSize.add("String Added");
 
    assertEquals(1, treeSetSize.size());
}
```

## 8. isEmpty()

isEmpty()方法可用于确定给定的TreeSet实例是否为空：

```java
@Test
public void whenCheckingForEmptyTreeSet_shouldCheckForEmpty() {
    Set<String> emptyTreeSet = new TreeSet<>();
    
    assertTrue(emptyTreeSet.isEmpty());
}
```

## 9. iterator()

iterator()方法返回一个迭代器，该迭代器按升序迭代Set中的元素。**这些迭代器是快速失败的**。

我们可以在这里观察递增的迭代顺序：

```java
@Test
public void whenIteratingTreeSet_shouldIterateTreeSetInAscendingOrder() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        System.out.println(itr.next());
    }
}
```

此外，TreeSet使我们能够按降序遍历Set。

让我们看看实际效果：

```java
@Test
public void whenIteratingTreeSet_shouldIterateTreeSetInDescendingOrder() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.descendingIterator();
    while (itr.hasNext()) {
        System.out.println(itr.next());
    }
}
```

**如果在创建迭代器之后以任何方式(通过迭代器的remove()方法除外)修改集合，迭代器将抛出ConcurrentModificationException**。

让我们为此创建一个测试：

```java
@Test(expected = ConcurrentModificationException.class)
public void whenModifyingTreeSetWhileIterating_shouldThrowException() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        itr.next();
        treeSet.remove("Second");
    }
}
```

或者，如果我们使用迭代器的remove方法，那么我们就不会遇到异常：

```java
@Test
public void whenRemovingElementUsingIterator_shouldRemoveElement() {
 
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Second");
    treeSet.add("Third");
    Iterator<String> itr = treeSet.iterator();
    while (itr.hasNext()) {
        String element = itr.next();
        if (element.equals("Second"))
            itr.remove();
    }
 
    assertEquals(2, treeSet.size());
}
```

**无法保证迭代器的快速失败行为，因为在存在非同步并发修改的情况下不可能做出任何硬性保证**。

可以在[此处](https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator)找到有关此的更多信息。

## 10. first()

此方法返回TreeSet中的第一个元素(如果它不为空)。否则，它会抛出一个NoSuchElementException。

让我们看一个例子：

```java
@Test
public void whenCheckingFirstElement_shouldReturnFirstElement() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");
   
    assertEquals("First", treeSet.first());
}
```

## 11. last()

与上面的例子类似，如果集合不为空，此方法将返回最后一个元素：

```java
@Test
public void whenCheckingLastElement_shouldReturnLastElement() {
    TreeSet<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add("Last");
    
    assertEquals("Last", treeSet.last());
}
```

## 12. subSet()

此方法将返回从fromElement到toElement范围内的元素。请注意，fromElement是包含的，而toElement是排除的：

```java
@Test
public void whenUsingSubSet_shouldReturnSubSetElements() {
    SortedSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);
    
    Set<Integer> expectedSet = new TreeSet<>();
    expectedSet.add(2);
    expectedSet.add(3);
    expectedSet.add(4);
    expectedSet.add(5);

    Set<Integer> subSet = treeSet.subSet(2, 6);
 
    assertEquals(expectedSet, subSet);
}
```

## 13. headSet()

此方法将返回小于指定元素的TreeSet元素：

```java
@Test
public void whenUsingHeadSet_shouldReturnHeadSetElements() {
    SortedSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> subSet = treeSet.headSet(6);
 
    assertEquals(subSet, treeSet.subSet(1, 6));
}
```

## 14. tailSet()

此方法将返回大于或等于指定元素的TreeSet元素：

```java
@Test
public void whenUsingTailSet_shouldReturnTailSetElements() {
    NavigableSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(2);
    treeSet.add(3);
    treeSet.add(4);
    treeSet.add(5);
    treeSet.add(6);

    Set<Integer> subSet = treeSet.tailSet(3);
 
    assertEquals(subSet, treeSet.subSet(3, true, 6, true));
}
```

## 15. 存储空元素

**在Java 7之前，可以将null元素添加到空的TreeSet中**。

但是，这被认为是一个错误。**因此TreeSet不再支持添加null值**。

当我们向TreeSet添加元素时，元素会根据它们的自然顺序或比较器指定的顺序进行排序。因此，与现有元素相比，添加null会导致NullPointerException，因为null无法与任何值进行比较：

```java
@Test(expected = NullPointerException.class)
public void whenAddingNullToNonEmptyTreeSet_shouldThrowException() {
    Set<String> treeSet = new TreeSet<>();
    treeSet.add("First");
    treeSet.add(null);
}
```

插入到TreeSet中的元素必须实现Comparable接口或至少被指定的比较器接受。**所有这些元素必须是相互可比的，即e1.compareTo(e2)或comparator.compare(e1, e2)不得抛出ClassCastException**。

让我们看一个例子：

```java
class Element {
    private Integer id;

    // Other methods...
}

Comparator<Element> comparator = (ele1, ele2) -> {
    return ele1.getId().compareTo(ele2.getId());
};

@Test
public void whenUsingComparator_shouldSortAndInsertElements() {
    Set<Element> treeSet = new TreeSet<>(comparator);
    Element ele1 = new Element();
    ele1.setId(100);
    Element ele2 = new Element();
    ele2.setId(200);
    
    treeSet.add(ele1);
    treeSet.add(ele2);
    
    System.out.println(treeSet);
}
```

## 16. TreeSet的性能

与HashSet相比，TreeSet的性能较低。添加、删除和搜索等操作需要O(logn)时间，而按排序顺序打印n个元素等操作需要O(n)时间。

如果我们希望保持元素排序，那么TreeSet应该是我们的主要选择，因为TreeSet可以按升序或降序访问和遍历，并且升序操作和视图的性能可能比降序操作和视图的性能更快。

局部性原则-是指根据内存访问模式，频繁访问相同值或相关存储位置的现象的术语。

当我们说局部性时：

-   相似数据经常被相似频率的应用程序访问
-   如果两个元素在给定的顺序附近，则TreeSet将它们放在数据结构中彼此靠近的位置，因此在内存中

TreeSet是一种具有更大局部性的数据结构，因此我们可以根据局部性原则得出总结，如果内存不足并且我们想要按照自然顺序访问相对接近的元素，我们应该优先考虑TreeSet。

如果需要从硬盘读取数据(这比从缓存或内存读取数据的延迟更大)，那么首选TreeSet，因为它具有更大的局部性

## 17. 总结

在本文中，我们重点介绍如何使用Java中的标准TreeSet实现。我们看到了它的目的以及它在可用性方面的效率，因为它能够避免重复和对元素进行排序。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-set-1)上获得。