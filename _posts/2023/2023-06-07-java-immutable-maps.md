---
layout: post
title:  Java中的不可变Map实现
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

有时最好禁止修改java.util.Map，例如跨线程共享只读数据。为此，我们可以使用UnmodifiableMap或ImmutableMap。

在这个快速教程中，我们将了解它们之间的区别。然后，我们将介绍创建不可变Map的各种方法。

## 2. 不可修改与不可变

**UnmodifiableMap只是对可修改Map的包装，它不允许直接对其进行修改**：

```java
Map<String, String> mutableMap = new HashMap<>();
mutableMap.put("USA", "North America");

Map<String, String> unmodifiableMap = Collections.unmodifiableMap(mutableMap);
assertThrows(UnsupportedOperationException.class,
    () -> unmodifiableMap.put("Canada", "North America"));
```

但是底层的可变Map仍然可以更改，并且修改也会反映在不可修改的Map中：

```java
mutableMap.remove("USA");
assertFalse(unmodifiableMap.containsKey("USA"));
		
mutableMap.put("Mexico", "North America");
assertTrue(unmodifiableMap.containsKey("Mexico"));
```

**另一方面，不可变Map包含其自己的私有数据并且不允许对其进行修改**。因此，一旦创建了ImmutableMap的实例，数据就不能以任何方式更改。

## 3. Guava的ImmutableMap

[Guava](https://github.com/google/guava)使用ImmutableMap提供了每个java.util.Map的不可变版本。每当我们尝试修改它时，它都会抛出UnsupportedOperationException。

由于它包含自己的私有数据，因此当原始Map发生变化时，这些数据不会改变。

现在我们将讨论创建ImmutableMap实例的各种方法。

### 3.1 使用copyOf()方法

首先，让我们使用ImmutableMap.copyOf()方法，该方法返回原始Map中所有条目的副本：

```java
ImmutableMap<String, String> immutableMap = ImmutableMap.copyOf(mutableMap);
assertTrue(immutableMap.containsKey("USA"));
```

它不能直接或间接修改：

```java
assertThrows(UnsupportedOperationException.class,
    () -> immutableMap.put("Canada", "North America"));
		
mutableMap.remove("USA");
assertTrue(immutableMap.containsKey("USA"));
		
mutableMap.put("Mexico", "North America");
assertFalse(immutableMap.containsKey("Mexico"));
```

### 3.2 使用builder()方法

我们还可以使用ImmutableMap.builder()方法创建原始Map中所有条目的副本。

此外，我们可以使用此方法添加原始Map中不存在的其他条目：

```java
ImmutableMap<String, String> immutableMap = ImmutableMap.<String, String>builder()
    .putAll(mutableMap)
    .put("Costa Rica", "North America")
    .build();
assertTrue(immutableMap.containsKey("USA"));
assertTrue(immutableMap.containsKey("Costa Rica"));
```

与前面的示例相同，我们不能直接或间接修改它：

```java
assertThrows(UnsupportedOperationException.class,
    () -> immutableMap.put("Canada", "North America"));
		
mutableMap.remove("USA");
assertTrue(immutableMap.containsKey("USA"));
		
mutableMap.put("Mexico", "North America");
assertFalse(immutableMap.containsKey("Mexico"));
```

### 3.3 使用of()方法

**最后，我们可以使用ImmutableMap.of()方法创建一个不可变Map，其中包含一组动态提供的条目。它最多支持5个键/值对**：

```java
ImmutableMap<String, String> immutableMap = ImmutableMap.of("USA", "North America", "Costa Rica", "North America");
assertTrue(immutableMap.containsKey("USA"));
assertTrue(immutableMap.containsKey("Costa Rica"));
```

我们也不能修改它：

```java
assertThrows(UnsupportedOperationException.class,
    () -> immutableMap.put("Canada", "North America"));
```

### 3.4 使用ofEntries()方法

最后，我们可以使用ImmutableMap.ofEntries()方法创建一个不可修改的Map，其中包含从给定条目中提取的键和值。

与ImmutableMap.of()不同，**我们可以将任意数量的条目作为参数传递给此方法**。

现在，让我们举例说明ImmutableMap.ofEntries()方法的使用：

```java
ImmutableMap<Integer, String> immutableMap = ImmutableMap.ofEntries(new AbstractMap.SimpleEntry<>(1, "USA"));
assertEquals(1, immutableMap.size());
assertThat(immutableMap, IsMapContaining.hasEntry(1, "USA"));
```

如我们所见，我们使用了[AbstractMap](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/AbstractMap.html)类来创建Map条目。

同样，尝试修改返回的Map将导致抛出UnsupportedOperationException：

```java
ImmutableMap<Integer, String> immutableMap = ImmutableMap.ofEntries(new AbstractMap.SimpleEntry<>(1, "USA"), new AbstractMap.SimpleEntry<>(2, "Canada"));
assertThrows(UnsupportedOperationException.class, () -> immutableMap.put(2, "Mexico"));
```

通常，如果我们添加具有重复键的条目，该方法会抛出IllegalArgumentException：

```java
assertThrows(IllegalArgumentException.class,
    () -> ImmutableMap.ofEntries(new AbstractMap.SimpleEntry<>(1, "USA"), new AbstractMap.SimpleEntry<>(1, "Canada")));
```

请注意，ImmutableMap.ofEntries()不接受null作为键或值。

因此，尝试使用null键指定条目将导致NullPointerException：

```java
assertThrows(NullPointerException.class,
    () -> ImmutableMap.ofEntries(new AbstractMap.SimpleEntry<>(null, "USA")));
```

并且，将null指定为值也会导致NullPointerException：

```java
assertThrows(NullPointerException.class,
    () -> ImmutableMap.ofEntries(new AbstractMap.SimpleEntry<>(1, null)));
```

## 4. 总结

在这篇简短的文章中，我们讨论了UnmodifiableMap和ImmutableMap之间的区别。

我们还了解了创建Guava的ImmutableMap的不同方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-1)上获得。