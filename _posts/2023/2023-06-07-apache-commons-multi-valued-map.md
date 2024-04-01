---
layout: post
title:  Apache Commons MultiValuedMap指南
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将了解Apache Commons Collections库中提供的[MultiValuedMap](https://commons.apache.org/proper/commons-collections/apidocs/index.html?org/apache/commons/collections4/MultiValuedMap.html)接口。

**MultiValuedMap提供了一个简单的API，用于将每个键映射到Java中的值集合**。它是[org.apache.commons.collections4.MultiMap](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/MultiMap.html)的继承者，后者在Commons Collection 4.1中被弃用。

## 2. Maven依赖

对于Maven项目，我们需要添加[commons-collections4](https://search.maven.org/classic/#search|ga|1|g%3A"org.apache.commons"ANDa%3A"commons-collections4")依赖：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.2</version>
</dependency>
```

## 3. 将元素添加到MultiValuedMap中

我们可以使用put和putAll方法添加元素。

让我们从创建MultiValuedMap的实例开始：

```java
MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
```

接下来，让我们看看如何使用put方法一次添加一个元素：

```java
map.put("fruits", "apple");
map.put("fruits", "orange");
```

此外，让我们使用putAll方法添加一些元素，该方法在一次调用中将一个键映射到多个元素：

```java
map.putAll("vehicles", Arrays.asList("car", "bike"));
assertThat((Collection<String>) map.get("vehicles"))
    .containsExactly("car", "bike");
```

## 4. 从MultiValuedMap中检索元素

MultiValuedMap提供检索键、值和键值映射的方法。让我们来看看其中的每一个。

### 4.1 获取键的所有值

要获取与键关联的所有值，我们可以使用get方法，它返回一个[Collection](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)：

```java
assertThat((Collection<String>) map.get("fruits"))
    .containsExactly("apple", "orange");
```

### 4.2 获取所有键值映射

或者，我们可以使用entries方法获取Map中包含的所有键值映射的[Collection](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)：

```java
Collection<Map.Entry<String, String>> entries = map.entries();
```

### 4.3 获取所有键

有两种方法可以检索MultiValuedMap中包含的所有键。

让我们使用keys方法来获取键的[MultiSet](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/MultiSet.html)视图：

```java
MultiSet<String> keys = map.keys();
assertThat(keys).contains("fruits", "vehicles");
```

或者，我们可以使用keySet方法获取键的[Set](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Set.html)视图：

```java
Set<String> keys = map.keySet();
assertThat(keys).contains("fruits", "vehicles");
```

### 4.4 获取Map的所有值

最后，如果我们想要获取Map中包含的所有值的[Collection](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)视图，我们可以使用values方法：

```java
Collection<String> values = map.values();
assertThat(values).contains("apple", "orange", "car", "bike");
```

## 5. 从MultiValuedMap中移除元素

现在，让我们看看删除元素和键值映射的所有方法。

### 5.1 删除映射到键的所有元素

首先，让我们看看如何使用remove方法删除与指定键关联的所有值：

```java
Collection<String> removedValues = map.remove("fruits");
assertThat(map.containsKey("fruits")).isFalse();
assertThat(removedValues).contains("apple", "orange");
```

此方法返回已删除值的[Collection](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)视图。

### 5.2 删除单个键值映射

现在，假设我们有一个映射到多个值的键，但我们只想删除一个映射值，而保留其他值。我们可以使用removeMapping方法轻松做到这一点：

```java
boolean isRemoved = map.removeMapping("fruits","apple");
assertThat(map.containsMapping("fruits","apple")).isFalse();
```

### 5.3 删除所有键值映射

最后，我们可以使用clear方法从Map中删除所有映射：

```java
map.clear();
assertThat(map.isEmpty()).isTrue();
```

## 6. 检查MultiValuedMap中的元素

接下来，让我们看一下检查Map中是否存在指定键或值的各种方法。

### 6.1 检查键是否存在

要确定我们的Map是否包含指定键的映射，我们可以使用containsKey方法：

```java
assertThat(map.containsKey("vehicles")).isTrue();
```

### 6.2 检查值是否存在

接下来，假设我们要检查Map中是否至少有一个键包含特定值的映射。我们可以使用containsValue方法来做到这一点：

```java
assertThat(map.containsValue("orange")).isTrue();
```

### 6.3 检查键值映射是否存在

同样，如果我们想检查一个Map是否包含特定键值对的映射，我们可以使用containsMapping方法：

```java
assertThat(map.containsMapping("fruits","orange")).isTrue();
```

### 6.4 检查Map是否为空

要检查Map是否根本不包含任何键值映射，我们可以使用isEmpty方法：

```java
assertThat(map.isEmpty()).isFalse;
```

### 6.5 检查Map的大小

最后，我们可以使用size方法来获取Map的总大小。当一个Map具有多个值的键时，Map的总大小是所有键的所有值的计数：

```java
assertEquals(4, map.size());
```

## 7. 实现

Apache Commons Collections库还提供了此接口的多种实现。让我们来看看它们。

### 7.1 ArrayListValuedHashMap

[ArrayListValuedHashMap](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/multimap/ArrayListValuedHashMap.html)在内部使用ArrayList来存储与每个键关联的值，**因此它允许重复的键值对**：

```java
MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
map.put("fruits", "apple");
map.put("fruits", "orange");
map.put("fruits", "orange");
assertThat((Collection<String>) map.get("fruits"))
    .containsExactly("apple", "orange", "orange");
```

现在，值得注意的是**这个类不是线程安全的**。因此，如果我们想从多个线程使用这个Map，我们必须确保使用适当的同步。

### 7.2 HashSetValuedHashMap

[HashSetValuedHashMap](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/multimap/HashSetValuedHashMap.html)使用HashSet来存储每个给定键的值。因此，**它不允许重复的键值对**。

让我们看一个简单的例子，其中我们添加相同的键值映射两次：

```java
MultiValuedMap<String, String> map = new HashSetValuedHashMap<>();
map.put("fruits", "apple");
map.put("fruits", "apple");
assertThat((Collection<String>) map.get("fruits"))
    .containsExactly("apple");
```

请注意，与前面使用ArrayListValuedHashMap的示例不同，HashSetValuedHashMap实现忽略了重复映射。

**HashSetValuedHashMap类也不是线程安全的**。

### 7.3 UnmodifiableMultiValuedMap

UnmodifiableMultiValuedMap是一个装饰器类，当我们需要一个[MultiValuedMap](https://commons.apache.org/proper/commons-collections/apidocs/index.html?org/apache/commons/collections4/MultiValuedMap.html)的不可变实例时它很有用-也就是说，它不允许进一步修改：

```java
@Test(expected = UnsupportedOperationException.class)
public void givenUnmodifiableMultiValuedMap_whenInserting_thenThrowingException() {
    MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
    map.put("fruits", "apple");
    map.put("fruits", "orange");
    MultiValuedMap<String, String> immutableMap = MultiMapUtils.unmodifiableMultiValuedMap(map);
    immutableMap.put("fruits", "banana"); // throws exception
}
```

同样，值得注意的是，**最终的put修改将导致UnsupportedOperationException**。

## 8. 总结

我们已经看到Apache Commons Collections库中MultiValuedMap接口的各种方法。此外，我们还探索了一些流行的实现。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-1)上获得。