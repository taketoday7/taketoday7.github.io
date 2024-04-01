---
layout: post
title: Java 21中的有序集合
category: java-new
copyright: java-new
excerpt: Java 21
---

## 1. 概述

**Java 21**预计将于2023年9月发布，成为继Java 17之后的下一个长期支持版本。在新功能中，我们可以看到Java[集合框架](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/doc-files/coll-index.html)的更新，称为Sequenced Collections。

**序列集合**提案作为一项改变游戏规则的增强功能脱颖而出，有望重新定义开发人员与集合交互的方式。**此功能将新接口注入现有层次结构，提供无缝机制来使用内置默认方法访问集合的第一个和最后一个元素。此外，它还支持获取集合的反向视图**。

在本文中，我们将探讨这一新的增强功能、其潜在风险以及它带来的优势。

## 2. 动机

缺乏具有定义的遭遇顺序的集合的通用超类型一直是问题和抱怨的根源。此外，**缺乏访问第一个和最后一个元素以及按相反顺序迭代的统一方法一直是Java集合框架的一个长期限制**。

我们可以以List和Deque为例：它们都定义了遭遇顺序，但它们的公共超类型Collection没有定义。同样，Set不定义遭遇顺序，但某些子类型(例如SortedSet和LinkedHashSet)定义了遭遇顺序。因此，对遭遇顺序的支持分布在类型层次结构中，与遭遇顺序相关的操作要么不一致，要么缺失。

为了演示这种不一致，让我们来比较一下访问不同集合类型的第一个和最后一个元素：

|                   | 访问第一个元素                  | 访问最后一个元素          |
| ----------------- | ------------------------------- | ------------------------- |
| **List**          | list.get(0)                     | list.get(list.size() – 1) |
| **Deque**         | deque.getFirst()                | deque.getLast()           |
| **SortedSet**     | sortedSet.first()               | sortedSet.last()          |
| **LinkedHashSet** | linkedHashSet.iterator().next() | // 缺失                   |

当尝试获取集合的反向视图时，也会发生同样的情况。虽然从第一个元素到最后一个元素迭代集合的元素遵循清晰一致的模式，但以相反的方向这样做会带来挑战。

举例来说，在处理NavigableSet时，我们可以使用DescendingSet()方法。对于Deque，descendingIterator()方法被证明是有用的。类似地，在处理List时，listIterator()方法效果很好。但是，LinkedHashSet的情况并非如此，因为它不提供任何反向迭代支持。

**所有这些差异导致了代码库的碎片化和复杂性，使得在API中表达某些有用的概念变得具有挑战性**。

## 3. 新的Java集合层次结构

**此新功能引入了用于排序List、排序Set和排序Map的三个新接口，这些接口已添加到现有的集合层次结构中**：

![](/assets/images/2023/javanew/java21sequencedcollections.png)

该图是JEP 431官方文档的一部分：[序列化集合](https://openjdk.org/jeps/431)([来源](https://cr.openjdk.org/~smarks/collections/SequencedCollectionDiagram20220216.png))。

### 3.1 SequencedCollection

**有序集合是其元素具有定义的遭遇顺序的集合**，新的SequencedCollection接口提供了在集合两端添加、检索或删除元素的方法，以及获取集合的反向视图的方法。

```java
interface SequencedCollection<E> extends Collection<E> {

    // new method
    SequencedCollection<E> reversed();

    // methods promoted from Deque
    void addFirst(E);
    void addLast(E);

    E getFirst();
    E getLast();

    E removeFirst();
    E removeLast();
}
```

除reverse()之外的所有方法都是默认方法，提供默认实现，并且从Deque提升。reversed()方法提供原始集合的逆序视图。此外，对原始集合的任何修改都对反向视图可见。

add*()和remove*()方法是可选的，并在其默认实现中抛出UnsupportedOperationException，主要是为了支持不可修改的集合和具有已定义排序顺序的集合的情况。如果集合为空，get*()和remove*()方法将抛出NoSuchElementException。

### 3.2 SequencedSet

**序列集可以定义为专门的Set，其功能类似于SequencedCollection，确保不存在重复元素**。SequencedSet接口扩展了SequencedCollection并重写了其returned()方法。唯一的区别是SequencedSet.reversed()的返回类型是SequencedSet。

```java
interface SequencedSet<E> extends Set<E>, SequencedCollection<E> {

    // covariant override
    SequencedSet<E> reversed();
}
```

### 3.3 SequencedMap

**排序Map是其条目具有定义的遭遇顺序的Map**。SequencedMap不扩展SequencedCollection，并提供自己的方法来操作集合两端的元素。

```java
interface SequencedMap<K, V> extends Map<K, V> {
    
    // new methods
    SequencedMap<K, V> reversed();
    SequencedSet<K> sequencedKeySet();
    SequencedCollection<V> sequencedValues();
    SequencedSet<Entry<K, V>> sequencedEntrySet();

    V putFirst(K, V);
    V putLast(K, V);

    // methods promoted from NavigableMap
    Entry<K, V> firstEntry();
    Entry<K, V> lastEntry();
    Entry<K, V> pollFirstEntry();
    Entry<K, V> pollLastEntry();
}
```

与SequencedCollection类似，put*()方法对于不可修改的Map或具有已定义排序顺序的Map抛出UnsupportedOperationException。此外，在空Map上调用从NavigableMap提升的方法之一会导致抛出NoSuchElementException。

## 4. 风险

新接口的引入应该不会影响仅使用集合实现的代码。但是，如果在我们的代码库中定义自定义集合类型，可能会出现几种冲突：

- **方法命名**：引入的新方法可能会与现有类上的方法发生冲突。例如，如果我们有一个List接口的自定义实现，该实现已经定义了getFirst()方法，但返回类型与SequencedCollection中定义的getFirst()不同，那么在升级到Java 21时，它将导致源不兼容。
- **协变重写**：List和Deque都提供了reversed()方法的协变重写，一个返回List，另一个返回Deque。因此，任何实现这两个接口的自定义集合在升级到Java 21时都会导致编译时错误，因为编译器无法选择其中之一。

报告[JDK-8266572](https://bugs.openjdk.org/browse/JDK-8266572)包含对不兼容风险的完整分析。

## 5. 总结

总之，序列集合标志着[Java集合](https://www.baeldung.com/java-collections)的重大飞跃。通过满足长期以来对以统一方式处理具有定义的遭遇顺序的集合的需求，Java使开发人员能够更高效、更直观地工作。新接口建立了更清晰的结构和一致的行为，从而产生更健壮和可读的代码。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-21)上获得。