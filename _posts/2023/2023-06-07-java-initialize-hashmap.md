---
layout: post
title:  在Java中初始化一个HashMap
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将介绍在Java中初始化HashMap的各种方法。

我们将使用Java 8和Java 9。

## 2. 静态HashMap的静态初始化器

我们可以使用静态代码块初始化HashMap：

```java
public static Map<String, String> articleMapOne;
static {
    articleMapOne = new HashMap<>();
    articleMapOne.put("ar01", "Intro to Map");
    articleMapOne.put("ar02", "Some article");
}
```

**这种初始化的优点是Map是可变的，但它只对静态有效。因此，可以根据需要添加和删除条目**。

让我们继续测试它：

```java
@Test
public void givenStaticMap_whenUpdated_thenCorrect() {
    MapInitializer.articleMapOne.put("NewArticle1", "Convert array to List");
    
    assertEquals(MapInitializer.articleMapOne.get("NewArticle1"), "Convert array to List");  
}
```

我们还可以使用双大括号语法初始化Map：

```java
Map<String, String> doubleBraceMap  = new HashMap<String, String>() {{
    put("key1", "value1");
    put("key2", "value2");
}};
```

请注意，**我们必须尽量避免这种初始化技术，因为它会在每次使用时创建一个额外的匿名类，持有对封闭对象的隐藏引用，并可能导致内存泄漏问题**。

## 3. 使用Java Collections

如果我们需要创建一个带有单个条目的单例不可变Map，Collections.singletonMap()会变得非常有用：

```java
public static Map<String, String> createSingletonMap() {
    return Collections.singletonMap("username1", "password1");
}
```

请注意，此处的Map是不可变的，如果我们尝试添加更多条目，它将抛出java.lang.UnsupportedOperationException。

我们还可以使用Collections.emptyMap()创建一个不可变的空Map：

```java
Map<String, String> emptyMap = Collections.emptyMap();
```

## 4. Java 8之道

在本节中，让我们看看使用Java 8 Stream API初始化Map的方法。

### 4.1 使用Collectors.toMap()

让我们使用一个二维字符串数组的Stream并将它们收集到一个Map中：

```java
Map<String, String> map = Stream.of(new String[][] {
    { "Hello", "World" }, 
    { "John", "Doe" }, 
}).collect(Collectors.toMap(data -> data[0], data -> data[1]));
```

请注意，这里Map的键和值的数据类型是相同的。

为了使其更通用，让我们采用Object数组并执行相同的操作：

```java
Map<String, Integer> map = Stream.of(new Object[][] { 
    { "data1", 1 }, 
    { "data2", 2 }, 
}).collect(Collectors.toMap(data -> (String) data[0], data -> (Integer) data[1]));
```

因此，我们创建了一个键为String，值为Integer的Map。

### 4.2 使用Map.Entry流

这里我们将使用Map.Entry的实例，这是我们具有不同键和值类型的另一种方法。

首先，让我们使用Entry接口的SimpleEntry实现：

```java
Map<String, Integer> map = Stream.of(
    new AbstractMap.SimpleEntry<>("idea", 1), 
    new AbstractMap.SimpleEntry<>("mobile", 2))
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

现在让我们使用SimpleImmutableEntry实现来创建Map：

```java
Map<String, Integer> map = Stream.of(
    new AbstractMap.SimpleImmutableEntry<>("idea", 1),    
    new AbstractMap.SimpleImmutableEntry<>("mobile", 2))
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

### 4.3 初始化不可变Map

在某些用例中，我们需要初始化一个不可变Map。这可以通过将Collectors.toMap()包装在Collectors.collectingAndThen()中来完成：

```java
Map<String, String> map = Stream.of(new String[][] { 
    { "Hello", "World" }, 
    { "John", "Doe" },
}).collect(Collectors.collectingAndThen(
    Collectors.toMap(data -> data[0], data -> data[1]), 
    Collections::<String, String> unmodifiableMap));
```

**请注意，我们应该避免使用Stream进行此类初始化，因为它可能会导致巨大的性能开销，并且会创建大量垃圾对象来初始化Map**。

## 5. Java 9之道

Java 9在Map接口中提供了各种工厂方法，可简化不可变Map的创建和初始化。

### 5.1 Map.of()

此工厂方法可以不带参数、单个参数和可变参数：

```java
Map<String, String> emptyMap = Map.of();
Map<String, String> singletonMap = Map.of("key1", "value");
Map<String, String> map = Map.of("key1","value1", "key2", "value2");
```

请注意，此方法最多只支持10个键值对。

### 5.2 Map.ofEntries()

它类似于Map.of()，但对键值对的数量没有限制：

```java
Map<String, String> map = Map.ofEntries(
    new AbstractMap.SimpleEntry<String, String>("name", "John"),
    new AbstractMap.SimpleEntry<String, String>("city", "budapest"),
    new AbstractMap.SimpleEntry<String, String>("zip", "000000"),
    new AbstractMap.SimpleEntry<String, String>("home", "1231231231")
);
```

请注意，工厂方法会生成不可变Map，因此任何更改都将导致UnsupportedOperationException。

此外，它们不允许空键或重复键。

现在，如果我们在初始化后需要一个可变或不断增长的Map，我们可以创建Map接口的任何实现，并在构造函数中传递这些不可变Map：

```java
Map<String, String> map = new HashMap<String, String> (Map.of("key1","value1", "key2", "value2"));
Map<String, String> map2 = new HashMap<String, String> (Map.ofEntries(
    new AbstractMap.SimpleEntry<String, String>("name", "John"),    
    new AbstractMap.SimpleEntry<String, String>("city", "budapest")));
```

## 6. 使用Guava

在我们研究了使用核心Java的方法后，让我们继续使用Guava库初始化Map：

```java
Map<String, String> articles = ImmutableMap.of("Title", "My New Article", "Title2", "Second Article");
```

这将创建一个不可变Map，下面创建一个可变Map：

```java
Map<String, String> articles = Maps.newHashMap(ImmutableMap.of("Title", "My New Article", "Title2", "Second Article"));
```

方法[ImmutableMap.of()](https://guava.dev/releases/23.0/api/docs/com/google/common/collect/ImmutableMap.html#of--)也有重载版本，最多可以接收5对键值参数。下面是带有2对参数的示例：

```java
ImmutableMap.of("key1", "value1", "key2", "value2");
```

## 7. 总结

在本文中，我们探讨了初始化Map的各种方法，特别是创建空的、单例的、不可变的和可变的Map。正如我们所看到的，自Java 9以来，该领域有了巨大的改进。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。