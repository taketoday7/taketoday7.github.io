---
layout: post
title:  将所有键和值从一个HashMap复制到另一个HashMap中而不替换现有键和值
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将了解如何将一个HashMap复制到另一个HashMap而不替换目标HashMap的键和值。Java中的[HashMap](https://www.baeldung.com/java-hashmap)是Map接口的哈希表实现，是一种支持存储键值对的数据结构。

## 2. 问题陈述

考虑我们有两个HashMap，sourceMap和targetMap，包含国家和他们的首都作为键和值。我们希望将sourceMap的内容复制到targetMap中，这样我们只有一个包含所有国家及其首都的Map。复制应遵循以下规则：

-   我们应该保留targetMap的原始内容
-   如果键发生冲突，例如两个Map中都存在的城市，我们应该保留targetMap的条目

让我们使用以下输入：

```java
Map<String, String> sourceMap = new HashMap<>();
sourceMap.put("India", "Delhi");
sourceMap.put("United States", "Washington D.C.");
sourceMap.put("United Kingdom", "London D.C.");

Map<String, String> targetMap = new HashMap<>();
targetMap.put("Zimbabwe", "Harare");
targetMap.put("Norway", "Oslo");
targetMap.put("United Kingdom", "London");
```

修改后的targetMap保留其值并添加sourceMap的所有值：

```text
"India", "Delhi"
"United States", "Washington D.C."
"United Kingdom", "London"
"Zimbabwe", "Harare"
"Norway", "Oslo"
```

## 3. 遍历HashMap

解决我们问题的一种简单方法是[遍历](https://www.baeldung.com/java-iterate-map)sourceMap的每个条目(键值对)并将其与targetMap的条目进行比较，当我们找到只存在于sourceMap中的条目时，我们将它添加到targetMap中。生成的targetMap包含其自身和sourceMap的所有键值。

我们可以遍历sourceMap的entrySet()并检查targetMap中是否存在键，而不是遍历两个Map的entrySets()：

```java
Map<String, String> copyByIteration(Map<String, String> sourceMap, Map<String, String> targetMap) {
    for (Map.Entry<String, String> entry : sourceMap.entrySet()) {
        if (!targetMap.containsKey(entry.getKey())) {
            targetMap.put(entry.getKey(), entry.getValue());
        }
    }
    return targetMap;
}
```

## 4. 使用Map的putIfAbsent()

我们可以重构上面的代码，使用Java 8中新增的putIfAbsent()方法。顾名思义，该方法只有当指定条目中的键不存在时，才会将sourceMap的条目复制到targetMap中：

```java
Map<String, String> copyUsingPutIfAbsent(Map<String, String> sourceMap, Map<String, String> targetMap) {
    for (Map.Entry<String, String> entry : sourceMap.entrySet()) {
        targetMap.putIfAbsent(entry.getKey(), entry.getValue());
    }
    return targetMap;
}
```

使用循环的另一种方法是使用Java 8中添加的[forEach](https://www.baeldung.com/foreach-java)结构。我们提供了一个操作，在我们的例子中是在targetMap输入上调用putIfAbsent()方法，它对给定HashMap的每个条目执行，直到所有元素都已处理或引发异常：

```java
Map<String, String> copyUsingPutIfAbsentForEach(Map<String, String> sourceMap, Map<String, String> targetMap) {
    sourceMap.forEach(targetMap::putIfAbsent);
    return targetMap;
}
```

## 5. 使用Map的putAll()

Map接口提供了一个方法putAll()，我们可以使用它来实现我们想要的结果。该方法将输入Map的所有键和值复制到当前Map中。**在这里我们应该注意，如果源HashMap和目标HashMap之间发生键冲突，则来自源的条目将替换targetMap的条目**。

我们可以通过从sourceMap中显式删除公共键来解决这个问题：

```java
Map<String, String> copyUsingPutAll(Map<String, String> sourceMap, Map<String, String> targetMap) {
    sourceMap.keySet().removeAll(targetMap.keySet());
    targetMap.putAll(sourceMap);
    return targetMap;
}
```

## 6. 在Map上使用merge()

Java 8在Map接口中引入了merge()方法。**它需要一个键、值和重映射函数作为方法参数**。

假设我们在输入中指定的键尚未与当前Map中的值关联(或与null关联)。在这种情况下，该方法将它与提供的非空值相关联。

**如果键在两个Map中都存在，则关联的值将替换为给定重映射函数的结果**。如果重映射函数的结果为null，它会删除键值对。

我们可以使用merge()方法将条目从sourceMap复制到targetMap：

```java
Map<String, String> copyUsingMapMerge(Map<String, String> sourceMap, Map<String, String> targetMap) {
    sourceMap.forEach((key, value) -> targetMap.merge(key, value, (oldVal, newVal) -> oldVal));
    return targetMap;
}
```

我们的重映射函数确保它在发生碰撞时保留targetMap中的值。

## 7. 使用Guava的Maps.difference()

[Guava](https://www.baeldung.com/guava-guide)库在其Maps类中使用difference()方法。要使用[Guava](https://search.maven.org/classic/#search|gav|1|g%3A"com.google.guava"ANDa%3A"guava")库，我们应该在我们的pom.xml中添加相应的依赖项：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

**difference()方法将两个Map作为输入并计算这两个Map之间的差集，提供的Map的键应该遵守[equals()和hashCode()契约](https://www.baeldung.com/java-equals-hashcode-contracts)**。

为了解决我们的问题，我们首先评估Map之间的差集。一旦我们知道仅存在于sourceMap(左侧Map)中的条目，我们将它们放入我们的targetMap中：

```java
Map<String, String> copyUsingGuavaMapDifference(Map<String, String> sourceMap, Map<String, String> targetMap) {
    MapDifference<String, String> differenceMap = Maps.difference(sourceMap, targetMap);
    targetMap.putAll(differenceMap.entriesOnlyOnLeft());
    return targetMap;
}
```

## 8. 总结

在本文中，我们研究了将条目从一个HashMap复制到另一个HashMap的不同方法，同时保留目标HashMap的现有条目。我们实现了一种基于迭代的方法，并使用不同的Java库函数解决了这个问题。我们还研究了如何使用Guava库解决问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-6)上获得。