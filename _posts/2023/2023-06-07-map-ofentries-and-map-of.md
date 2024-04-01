---
layout: post
title:  Map.ofEntries()和Map.of()之间的区别
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

Java 8引入了Map.of()方法，这使得创建不可变Map变得更加容易。Java 9获得了Map.ofEntries()方法，其功能略有不同。

在本教程中，我们将仔细研究不可变Map的这两个静态工厂方法，并解释哪一个适用于哪种用途。

## 2. Map.of()

**Map.of()方法将指定数量的键值对作为参数，并返回包含每个键值对的不可变Map**。参数中对的顺序对应于将它们添加到Map的顺序。如果我们尝试添加具有重复键的键值对，它会抛出IllegalArgumentException。如果我们尝试添加null键或值，它将抛出NullPointerException。

作为重载的静态工厂方法实现，第一个方法允许我们创建一个空Map：

```java
static <K, V> Map<K, V> of() {
    return (Map<K, V>) ImmutableCollections.EMPTY_MAP;
}
```

让我们看看用法：

```java
Map<Long, String> map = Map.of();
```

在Map<K, V\>接口中还定义了一个方法，它接收单个键和值：

```java
static <K, V> Map<K, V> of(K k1, V v1) {
    return new ImmutableCollections.Map1<>(k1, v1);
}
```

让我们调用它：

```java
Map<Long, String> map = Map.of(1L, "value1");
```

**这些工厂方法被重载了9次，最多接收10个键和10个值**，正如我们在[OpenJDK 17](https://openjdk.org/projects/jdk/17/)中所见：

```java
static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6, K k7, V v7, K k8, V v8, K k9, V v9, K k10, V v10) {
    return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10);
}
```

尽管这些方法非常有用，但创建更多它们将是一团糟。此外，我们不能使用Map.of()方法从现有键和值创建Map，因为此方法只接受未定义的键值对作为参数。这就是Map.ofEntries()方法的用武之地。

## 3. Map.ofEntries()

**Map.ofEntries()方法将未指定数量的Map.Entry<K, V\>对象作为参数，并返回一个不可变Map**。同样，参数中对的顺序与它们添加到Map的顺序相同。如果我们尝试添加具有重复键的键值对，它会抛出IllegalArgumentException。

让我们看看根据[OpenJDK 17](https://openjdk.org/projects/jdk/17/)的静态工厂方法实现：

```java
static <K, V> Map<K, V> ofEntries(Entry<? extends K, ? extends V>... entries) {
    if (entries.length == 0) { // implicit null check of entries array
        var map = (Map<K,V>) ImmutableCollections.EMPTY_MAP;
        return map;
    } else if (entries.length == 1) {
        // implicit null check of the array slot
        return new ImmutableCollections.Map1<>(entries[0].getKey(), entries[0].getValue());
    } else {
        Object[] kva = new Object[entries.length << 1];
        int a = 0;
        for (Entry<? extends K, ? extends V> entry : entries) {
            // implicit null checks of each array slot
            kva[a++] = entry.getKey();
            kva[a++] = entry.getValue();
        }
        return new ImmutableCollections.MapN<>(kva);
     }
}
```

[可变参数](https://www.baeldung.com/java-varargs)实现允许我们传递可变数量的条目。

例如，我们可以创建一个空Map：

```java
Map<Long, String> map = Map.ofEntries();
```

或者我们可以创建并填充Map：

```java
Map<Long, String> longUserMap = Map.ofEntries(Map.entry(1L, "User A"), Map.entry(2L, "User B"));
```

Map.ofEntries()方法的一大优点是我们还**可以使用它从现有的键和值创建Map**。这对于Map.of()方法是不可能的，因为它只接受未定义的键值对作为参数。

## 4. 总结

Map.of()方法只适用于最多10个元素的Map，这是因为它被实现为11个不同的重载方法，这些方法接收0到10个键-值对作为参数。另一方面，Map.ofEntries()方法可用于任何大小的Map，因为它利用了[可变参数](https://www.baeldung.com/java-varargs)功能。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-5)上获得。