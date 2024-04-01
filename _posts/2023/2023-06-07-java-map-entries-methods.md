---
layout: post
title:  Java Map-keySet() vs entrySet() vs values()方法
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将讨论Java中Map接口的三个方法keySet()、entrySet()和values()。这些方法分别用于检索一组键、一组键值映射和一组值。

## 2. Map初始化

虽然我们可以在任何实现Map接口的类(如[HashMap](https://www.baeldung.com/java-hashmap)、[TreeMap](https://www.baeldung.com/java-treemap)和[LinkedHashMap](https://www.baeldung.com/java-linked-hashmap))上使用这些方法，但我们将在此处使用HashMap。

让我们创建并初始化一个HashMap，它的键是String类型，值是Integer类型：

```java
Map<String, Integer> map = new HashMap<>();
map.put("one", 1);
map.put("two", 2);
```

## 3. keySet()方法

**keySet()方法返回Map中包含的键集**。

让我们将方法keySet()应用于Map并将其存储在Set变量actualValues中：

```java
Set<String> actualValues = map.keySet();
```

现在，让我们确认返回的Set的大小为2：

```java
assertEquals(2, actualValues.size());
```

此外，我们可以看到返回的Set包含Map的键：

```java
assertTrue(actualValues.contains("one"));
assertTrue(actualValues.contains("two"));
```

## 4. entrySet()方法

**entrySet()方法返回一组键值映射**。该方法不接收任何参数，返回类型为Map.Entry。

让我们将方法entrySet()应用于Map：

```java
Set<Map.Entry<String, Integer>> actualValues = map.entrySet();
```

如我们所见，actualValues是一组Map.Entry对象。

Map.Entry是一个包含键和值的静态接口。在内部，它有两个实现-AbstractMap.SimpleEntry和AbstractMap.SimpleImmutableEntry。

和以前一样，让我们确认返回的Set的大小为2：

```java
assertEquals(2, actualValues.size());
```

此外，我们可以看到返回的Set包含Map的键值条目：

```java
assertTrue(actualValues.contains(new SimpleEntry<>("one", 1)));
assertTrue(actualValues.contains(new SimpleEntry<>("two", 2)));
```

在这里，我们选择了接口Map.Entry的AbstractMap.SimpleEntry实现来进行测试。

## 5. values()方法

**values()方法返回Map中包含的值集合**。该方法不接收任何参数，返回类型为Collection。

让我们将方法values()应用于Map并将其存储在Collection变量actualValues中：

```java
Collection<Integer> actualValues = map.values();
```

现在，让我们验证返回的集合的大小：

```java
assertEquals(2, actualValues.size());
```

此外，我们可以看到返回的Set包含Map的值：

```java
assertTrue(actualValues.contains(1));
assertTrue(actualValues.contains(2));
```

## 6. 总结

在本文中，我们讨论了Map接口的keySet()、entrySet()和values()方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-5)上获得。