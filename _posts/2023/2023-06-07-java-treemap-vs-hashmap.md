---
layout: post
title:  Java TreeMap与HashMap
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将比较两个Map实现：TreeMap和HashMap。

这两种实现构成了Java集合框架的组成部分，并将数据存储为键值对。

## 2. 差异

### 2.1 实现

我们将首先讨论HashMap，这是一个基于哈希表的实现。它扩展了AbstractMap类并实现了Map接口。HashMap的工作原理是[哈希](https://www.baeldung.com/java-hashcode)。

此Map实现通常用作存储桶哈希表，**但当桶变得太大时，它们会转换为TreeNodes的节点**，每个节点的结构与java.util.TreeMap中的节点类似。

另一方面，TreeMap扩展了AbstractMap类并实现了NavigableMap接口。TreeMap将Map元素存储在红黑树中，**这是一种自平衡二叉搜索树**。

### 2.2 顺序

**HashMap不对元素在Map中的排列方式提供任何保证**。

这意味着，**我们不能在遍历HashMap的键和值时假定任何顺序**：

```java
@Test
public void whenInsertObjectsHashMap_thenRandomOrder() {
    Map<Integer, String> hashmap = new HashMap<>();
    hashmap.put(3, "TreeMap");
    hashmap.put(2, "vs");
    hashmap.put(1, "HashMap");
    
    assertThat(hashmap.keySet(), containsInAnyOrder(1, 2, 3));
}
```

但是，TreeMap中的元素是**根据它们的自然顺序排序的**。

如果TreeMap对象不能按照自然顺序排序，那么我们可以使用Comparator或Comparable来定义元素在Map中的排列顺序：

```java
@Test
public void whenInsertObjectsTreeMap_thenNaturalOrder() {
    Map<Integer, String> treemap = new TreeMap<>();
    treemap.put(3, "TreeMap");
    treemap.put(2, "vs");
    treemap.put(1, "HashMap");
    
    assertThat(treemap.keySet(), contains(1, 2, 3));
}
```

### 2.3 空值

HashMap允许存储最多一个空键和多个空值。

让我们看一个例子：

```java
@Test
public void whenInsertNullInHashMap_thenInsertsNull() {
    Map<Integer, String> hashmap = new HashMap<>();
    hashmap.put(null, null);
    
    assertNull(hashmap.get(null));
}
```

但是，TreeMap不允许空键，但可能包含许多空值。

不允许使用null键，因为compareTo()或compare()方法会抛出NullPointerException：

```java
@Test(expected = NullPointerException.class)
public void whenInsertNullInTreeMap_thenException() {
    Map<Integer, String> treemap = new TreeMap<>();
    treemap.put(null, "NullPointerException");
}
```

**如果我们将TreeMap与用户定义的Comparator一起使用，那么它取决于compare()方法的实现如何处理空值**。

## 3. 性能分析

性能是最关键的指标，可帮助我们了解给定用例的数据结构的适用性。

在本节中，我们将对HashMap和TreeMap的性能进行全面分析。

### 3.1 HashMap

**HashMap是基于哈希表的实现，内部使用基于数组的数据结构根据hash函数组织其元素**。

HashMap为大多数操作(如add()、remove()和contains())提供预期的恒定时间性能O(1)。因此，它比TreeMap快得多。

在合理假设下，在哈希表中搜索元素的平均时间为O(1)。但是，hash函数的不正确实现可能会导致桶中值的分布不均，从而导致：

-   内存开销：许多桶未被使用
-   性能下降：碰撞次数越多，性能越低

**在Java 8之前，分离链接是处理冲突的唯一首选方法**。它通常使用链表实现，即，如果存在任何冲突或两个不同的元素具有相同的哈希值，则将这两个元素存储在同一个链表中。

因此，在最坏的情况下，在HashMap中搜索一个元素可能会花费与在链表中搜索元素一样长的时间，即O(n)。

**然而，随着[JEP 180](https://openjdk.java.net/jeps/180)的出现，元素在HashMap中的排列方式的实现发生了微妙的变化**。

根据规范，当桶变得太大并且包含足够多的节点时，它们会转换为TreeNodes模式，每个模式的结构都与TreeMap中的模式相似。

**因此，在发生高哈希冲突的情况下，最坏情况下的性能将从O(n)提高到O(logn)**。

执行此转换的代码如下所示：

```java
if(binCount >= TREEIFY_THRESHOLD - 1) {
    treeifyBin(tab, hash);
}
```

TREEIFY_THRESHOLD的值为8，它有效地表示使用树而不是链表作为存储桶的阈值计数。

很明显：

-   HashMap需要比保存其数据所需更多的内存
-   HashMap不应超过70%–75%满。如果它接近，它会调整大小并重新哈希条目
-   重哈希需要n个操作，这是昂贵的，其中我们的常数时间插入变成O(n)阶
-   哈希算法决定了在HashMap中插入对象的顺序

**HashMap的性能可以通过在创建HashMap对象时设置自定义初始容量和负载因子来调整**。

但是，如果出现以下情况，我们应该选择HashMap：

-   我们大约知道我们的集合中要保留多少元素
-   我们不想以自然顺序提取元素

在上述情况下，HashMap是我们的最佳选择，因为它提供了恒定时间的插入、查找和删除。

### 3.2 TreeMap

TreeMap将其数据存储在分层树中，能够在自定义比较器的帮助下对元素进行排序。

其性能总结：

-   TreeMap为大多数操作提供O(log(n))的性能，如add()、remove()和contains()
-   TreeMap可以节省内存(与HashMap相比)，因为它只使用保存其元素所需的内存量，这与使用连续内存区域的HashMap不同
-   一棵树应该保持其平衡以保持其预期性能，这需要大量的努力，因此使实现复杂化

我们应该在以下情况下使用TreeMap：

-   必须考虑内存限制
-   我们不知道有多少元素必须存储在内存中
-   我们想以自然顺序提取对象
-   元素是否会持续添加和删除
-   我们愿意接受O(logn)搜索时间

## 4. 相似之处

### 4.1 唯一元素

TreeMap和HashMap都不支持重复键。如果添加重复键，它将覆盖前一个元素(没有错误或异常)：

```java
@Test
public void givenHashMapAndTreeMap_whenputDuplicates_thenOnlyUnique() {
    Map<Integer, String> treeMap = new HashMap<>();
    treeMap.put(1, "Tuyucheng");
    treeMap.put(1, "Tuyucheng");

    assertTrue(treeMap.size() == 1);

    Map<Integer, String> treeMap2 = new TreeMap<>();
    treeMap2.put(1, "Tuyucheng");
    treeMap2.put(1, "Tuyucheng");

    assertTrue(treeMap2.size() == 1);
}
```

### 4.2 并发访问

**两个Map实现不同步**，我们需要自己管理并发访问。

每当多个线程同时访问它们并且至少有一个线程修改它们时，两者都必须在外部同步。

我们必须显式使用Collections.synchronizedMap(mapName)来获取所提供Map的同步视图。

### 4.3 快速失败迭代器

如果Map在创建迭代器后的任何时间以任何方式被修改，迭代器将抛出ConcurrentModificationException。

此外，我们可以使用迭代器的remove方法在迭代期间更改Map。

让我们看一个例子：

```java
@Test
public void whenModifyMapDuringIteration_thenThrowExecption() {
    Map<Integer, String> hashmap = new HashMap<>();
    hashmap.put(1, "One");
    hashmap.put(2, "Two");
    
    Executable executable = () -> hashmap
        .forEach((key,value) -> hashmap.remove(1));
 
    assertThrows(ConcurrentModificationException.class, executable);
}
```

## 5. 使用哪个实现？

总的来说，这两种实现都有各自的优缺点，但是，**这是关于理解潜在的期望和要求，这些期望和要求必须支配我们的选择**。

总结：

-   如果我们想保持条目排序，我们应该使用TreeMap
-   如果我们优先考虑性能而不是内存消耗，我们应该使用HashMap
-   由于TreeMap具有更显著的局部性，因此如果我们想根据自然顺序访问彼此相对较近的对象，我们可能会考虑它
-   HashMap可以使用initialCapacity和loadFactor进行调整，这对于TreeMap是不可能的
-   如果我们想保留插入顺序同时受益于恒定时间访问，我们可以使用LinkedHashMap

## 6.  总结

在本文中，我们展示了TreeMap和HashMap之间的异同。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-3)上获得。