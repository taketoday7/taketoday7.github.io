---
layout: post
title:  Java中的LinkedHashMap指南
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将探讨LinkedHashMap类的内部实现。LinkedHashMap是Map接口的常见实现。

这个特定的实现是HashMap的子类，因此共享[HashMap实现](https://www.baeldung.com/java-hashmap)的核心构建块。因此，强烈建议在继续阅读本文之前复习一下。

## 2. LinkedHashMap与HashMap

LinkedHashMap类在大多数方面与HashMap非常相似。但是LinkedHashMap是基于哈希表和链表的，增强了哈希表的功能。

除了默认大小为16的底层数组之外，它还维护一个贯穿其所有条目的双向链表。

为了保持元素的顺序，LinkedHashMap通过添加指向下一个和上一个条目的指针来修改HashMap的Map.Entry类：

```java
static class Entry<K, V> extends HashMap.Node<K, V> {
    Entry<K, V> before, after;

    Entry(int hash, K key, V value, Node<K, V> next) {
        super(hash, key, value, next);
    }
}
```

请注意，Entry类只是添加了两个指针before和after，使它能够将自身挂接到链表。除此之外，它还使用HashMap的Entry类实现。

最后，请记住这个链表定义了迭代的顺序，默认情况下是元素的插入顺序(insertion-order)。

## 3. 插入顺序LinkedHashMap

让我们看一下LinkedHashMap实例，它根据条目插入Map的方式对其条目进行排序。它还保证将在Map的整个生命周期中维护此顺序：

```java
@Test
public void givenLinkedHashMap_whenGetsOrderedKeyset_thenCorrect() {
    LinkedHashMap<Integer, String> map = new LinkedHashMap<>();
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);

    Set<Integer> keys = map.keySet();
    Integer[] arr = keys.toArray(new Integer[0]);

    for (int i = 0; i < arr.length; i++) {
        assertEquals(new Integer(i + 1), arr[i]);
    }
}
```

在这里，我们只是对LinkedHashMap中的条目顺序进行基本的、非结论性的测试。

我们可以保证此测试将始终通过，因为插入顺序将始终保持不变。**我们不能对HashMap做出同样的保证**。

此属性在接收任何Map、创建副本以进行操作并将其返回给调用代码的API中具有很大优势。如果客户端需要在调用API之前以相同的方式对返回的Map进行排序，那么LinkedHashMap就是可行的方法。

如果将键重新插入到Map中，插入顺序不会受到影响。

## 4. 访问顺序LinkedHashMap

LinkedHashMap提供了一个特殊的构造函数，使我们能够在自定义负载因子(LF)和初始容量中**指定一种称为访问顺序的不同排序机制/策略**：

```java
LinkedHashMap<Integer, String> map = new LinkedHashMap<>(16, .75f, true);
```

第一个参数是初始容量，其次是负载因子，**最后一个参数是排序模式**。因此，通过传入true，我们开启了访问顺序，而默认设置是插入顺序。

这种机制确保元素的迭代顺序是元素最后被访问的顺序，从最近最少访问到最近最多访问。

因此，使用这种Map构建最近最少使用(LRU)缓存非常简单实用。成功的put或get操作会导致对条目的访问：

```java
@Test
public void givenLinkedHashMap_whenAccessOrderWorks_thenCorrect() {
    LinkedHashMap<Integer, String> map = new LinkedHashMap<>(16, .75f, true);
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);

    Set<Integer> keys = map.keySet();
    assertEquals("[1, 2, 3, 4, 5]", keys.toString());
 
    map.get(4);
    assertEquals("[1, 2, 3, 5, 4]", keys.toString());
 
    map.get(1);
    assertEquals("[2, 3, 5, 4, 1]", keys.toString());
 
    map.get(3);
    assertEquals("[2, 5, 4, 1, 3]", keys.toString());
}
```

请注意，**当我们在Map上执行访问操作时，键集中元素的顺序是如何转换的**。

简单地说，Map上的任何访问操作都会产生一个顺序，如果要立即执行迭代，被访问的元素将出现在最后。

在上面的示例之后，很明显，putAll操作会为指定Map中的每个映射生成一个条目访问。

自然地，对Map视图的迭代不会影响后备Map的迭代顺序；**只有对Map的显式访问操作才会影响顺序**。

LinkedHashMap还提供了一种机制，用于维护固定数量的映射，并在需要添加新条目时不断丢弃最旧的条目。

可以覆盖removeEldestEntry方法以强制执行此策略以自动删除过期的映射。

为了在实践中看到这一点，让我们创建自己的LinkedHashMap类，其唯一目的是通过扩展LinkedHashMap强制删除过期的映射：

```java
public class MyLinkedHashMap<K, V> extends LinkedHashMap<K, V> {

    private static final int MAX_ENTRIES = 5;

    public MyLinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
        super(initialCapacity, loadFactor, accessOrder);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_ENTRIES;
    }
}
```

我们上面的重写将允许Map增长到最大5个条目。当大小超过该大小时，将以丢失Map中最旧的条目为代价插入每个新条目，即最后访问时间在所有其他条目之前的条目：

```java
@Test
public void givenLinkedHashMap_whenRemovesEldestEntry_thenCorrect() {
    LinkedHashMap<Integer, String> map = new MyLinkedHashMap<>(16, .75f, true);
    map.put(1, null);
    map.put(2, null);
    map.put(3, null);
    map.put(4, null);
    map.put(5, null);
    Set<Integer> keys = map.keySet();
    assertEquals("[1, 2, 3, 4, 5]", keys.toString());
 
    map.put(6, null);
    assertEquals("[2, 3, 4, 5, 6]", keys.toString());
 
    map.put(7, null);
    assertEquals("[3, 4, 5, 6, 7]", keys.toString());
 
    map.put(8, null);
    assertEquals("[4, 5, 6, 7, 8]", keys.toString());
}
```

请注意，当我们向Map中添加新条目时，键集开头的最旧条目是如何不断减少的。

## 5. 性能考虑

就像HashMap一样，LinkedHashMap在恒定时间内执行add、remove和contains基本Map操作，只要哈希函数的维度合适即可。它还接受空键和空值。

但是，**由于增加了维护双向链表的开销，LinkedHashMap的这种恒定时间性能可能会比HashMap的恒定时间差一点**。

迭代LinkedHashMap的集合视图也需要类似于HashMap的线性时间O(n)。**另一方面，LinkedHashMap在迭代过程中的线性时间性能优于HashMap的线性时间**。

这是因为，对于LinkedHashMap，O(n)中的n只是Map中的条目数，与容量无关。然而，对于HashMap，n是容量和总和的大小，O(size + capacity)。

加载因子和初始容量的定义与HashMap一样。但是请注意，**对于LinkedHashMap来说，为初始容量选择过高值的惩罚不如HashMap的那么严重**，因为此类的迭代时间不受容量影响。

## 6. 并发

就像HashMap一样，LinkedHashMap实现是不同步的。因此如果你打算从多个线程访问它并且这些线程中至少有一个可能会在结构上改变它，那么它必须进行外部同步。

最好在创建时这样做：

```java
Map m = Collections.synchronizedMap(new LinkedHashMap());
```

与HashMap的区别在于需要进行结构修改。**在按访问顺序LinkedHashMap中，仅调用get API会导致结构修改**。除此之外，还有像put和remove这样的操作。

## 7. 总结

在本文中，我们探讨了Java LinkedHashMap类作为Map接口在使用方面最重要的实现之一。我们还根据与其超类HashMap的区别探索了它的内部工作原理。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-1)上获得。