---
layout: post
title:  在Java中遍历Map
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将介绍在Java中循环访问Map条目的不同方法。

简单地说，我们可以使用entrySet()、keySet()或values()提取Map的内容。由于这些都是集合，因此类似的迭代原则适用于所有这些集合。

让我们仔细看看其中的一些。

## 2. Map的entrySet()、keySet()和values()方法的简短介绍

在使用这三种方法遍历Map之前，让我们先了解一下这些方法的作用：

-   [entrySet()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#entrySet())：返回Map的集合视图，其元素来自Map.Entry类。entry.getKey()方法返回key，entry.getValue()返回对应的value
-   [keySet()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#keySet())：返回此Map中包含的所有键作为集合
-   [values()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#values())：返回此Map中包含的所有值作为集合

接下来，让我们看看这些方法的实际应用。

## 3. 使用for循环

### 3.1 使用entrySet()

首先，让我们看看**如何使用EntrySet遍历Map**：

```java
public void iterateUsingEntrySet(Map<String, Integer> map) {
    for (Map.Entry<String, Integer> entry : map.entrySet()) {
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```

在这里，我们从Map中提取条目集，然后使用经典的for-each方法遍历它们。

### 3.2 使用keySet()

或者，我们可以先使用keySet方法获取Map中的所有键，然后按每个键遍历Map：

```java
public void iterateUsingKeySetAndForeach(Map<String, Integer> map) {
    for (String key : map.keySet()) {
        System.out.println(key + ":" + map.get(key));
    }
}
```

### 3.3 使用values()迭代值

有时，**我们只对Map中的值感兴趣，而不管与它们相关联的键是什么**。在这种情况下，values()是我们的最佳选择：

```java
public void iterateValues(Map<String, Integer> map) {
    for (Integer value : map.values()) {
        System.out.println(value);
    }
}
```

## 4. Iterator

执行迭代的另一种方法是使用[Iterator](https://www.baeldung.com/java-iterator)。接下来，让我们看看这些方法如何与Iterator对象一起使用。

### 4.1 Iterator 和entrySet()

首先，让我们使用Iterator和entrySet()遍历Map：

```java
public void iterateUsingIteratorAndEntry(Map<String, Integer> map) {
    Iterator<Map.Entry<String, Integer>> iterator = map.entrySet().iterator();
    while (iterator.hasNext()) {
        Map.Entry<String, Integer> entry = iterator.next();
        System.out.println(entry.getKey() + ":" + entry.getValue());
    }
}
```

请注意我们如何使用entrySet()返回的Set的iterator() API获取Iterator实例。然后，像往常一样，我们使用iterator.next()遍历迭代器。

### 4.2 Iterator和keySet()

同样，我们可以使用Iterator和keySet()遍历Map：

```java
public void iterateUsingIteratorAndKeySet(Map<String, Integer> map) {
    Iterator<String> iterator = map.keySet().iterator();
    while (iterator.hasNext()) {
        String key = iterator.next();
        System.out.println(key + ":" + map.get(key));
    }
}
```

### 4.3 Iterator和values()

我们还可以使用Iterator和values()方法遍历Map的值：

```java
public void iterateUsingIteratorAndValues(Map<String, Integer> map) {
    Iterator<Integer> iterator = map.values().iterator();
    while (iterator.hasNext()) {
        Integer value = iterator.next();
        System.out.println("value :" + value);
    }
}
```

## 5. 使用Lambda和Stream API

从版本8开始，Java引入了[Stream API](https://www.baeldung.com/java-8-streams)和Lambda。接下来，让我们看看如何使用这些技术迭代Map。

### 5.1 使用forEach()和Lambda

与Java 8中的大多数其他内容一样，事实证明这比替代方案简单得多。我们只需使用forEach()方法：

```java
public void iterateUsingLambda(Map<String, Integer> map) {
    map.forEach((k, v) -> System.out.println((k + ":" + v)));
}
```

在这种情况下，我们不需要将Map转换为条目集合。要了解有关Lambda表达式的更多信息，我们可以从[这里](https://www.baeldung.com/java-8-lambda-expressions-tips)开始。

当然，我们可以从键开始遍历Map：

```java
public void iterateByKeysUsingLambda(Map<String, Integer> map) {
    map.keySet().foreach(k -> System.out.println((k + ":" + map.get(k))));
}
```

同样，我们可以对values()方法使用相同的技术：

```java
public void iterateValuesUsingLambda(Map<String, Integer> map) {
    map.values().forEach(v -> System.out.println(("value: " + v)));
}
```

### 5.2 使用Stream API

Stream API是Java 8的一个重要特性，我们也可以使用此功能来循环遍历Map。

**当我们计划进行一些额外的Stream处理时，应该使用Stream API；否则，它只是一个简单的forEach()，如前所述**。

下面以entrySet()为例，看看Stream API是如何工作的：

```java
public void iterateUsingStreamAPI(Map<String, Integer> map) {
    map.entrySet().stream()
        // ... some other Stream processings
        .forEach(e -> System.out.println(e.getKey() + ":" + e.getValue()));
}
```

[Stream API](https://www.baeldung.com/java-8-streams)与keySet()和values()方法的用法与上面的示例非常相似。

## 6.  总结

在本文中，我们重点介绍了一个关键但简单的操作：遍历Map的条目。

我们探讨了几种只能用于Java 8+的方法，即Lambda表达式和Stream API。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。