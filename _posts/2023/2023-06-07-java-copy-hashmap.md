---
layout: post
title:  在Java中复制HashMap
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将探讨HashMap的浅拷贝和[深拷贝](https://www.baeldung.com/java-deep-copy)的概念，以及在Java中拷贝[HashMap](https://www.baeldung.com/java-hashmap)的几种技术。

我们还将考虑一些可以在特定情况下帮助我们的外部库。

## 2. 浅拷贝与深拷贝

首先，让我们了解一下HashMaps中浅拷贝和深拷贝的概念。

### 2.1 浅拷贝

**HashMap的浅表副本是一个新的HashMap，它映射到与原始HashMap相同的键和值对象**。

例如，我们将创建一个Employee类，然后创建一个以Employee实例作为值的Map：

```java
public class Employee {
    private String name;

    // constructor, getters and setters
}
```

```java
HashMap<String, Employee> map = new HashMap<>();
Employee emp1 = new Employee("John");
Employee emp2 = new Employee("Norman");
map.put("emp1", emp1);
map.put("emp2", emp2);
```

现在，我们将验证原始Map及其浅表副本是不同的对象：

```java
HashMap<String, Employee> shallowCopy = // shallow copy implementation
assertThat(shallowCopy).isNotSameAs(map);
```

由于这是一个浅拷贝，因此如果我们改变一个Employee实例的属性，它会影响原始Map和它的浅拷贝：

```java
emp1.setFirstName("Johny");
assertThat(shallowCopy.get("emp1")).isEqualTo(map.get("emp1"));
```

### 2.2 深拷贝

**HashMap的深拷贝是一个新的HashMap，可以深度复制所有映射**。因此，它为所有键、值和映射创建新对象。

在这里，显式修改映射(键值)不会影响深拷贝：

```java
HashMap<String, Employee> deepCopy = // deep copy implementation

emp1.setFirstName("Johny");

assertThat(deepCopy.get("emp1")).isNotEqualTo(map.get("emp1"));
```

## 3. HashMap接口

### 3.1 使用HashMap构造函数

**HashMap的参数化构造函数HashMap(Map<? extends K, ? extends V\> m)提供了一种浅拷贝整个Map的快速方法**：

```java
HashMap<String, Employee> shallowCopy = new HashMap<String, Employee>(originalMap);
```

### 3.2 使用Map.clone()

与构造函数类似，HashMap#clone方法也创建了一个快速的浅拷贝：

```java
HashMap<String, Employee> shallowCopy = originalMap.clone();
```

### 3.3 使用Map.put()

通过遍历每个条目并在另一个Map上调用put()方法，可以轻松地浅拷贝HashMap：

```java
HashMap<String, Employee> shallowCopy = new HashMap<String, Employee>();
Set<Entry<String, Employee>> entries = originalMap.entrySet();
for (Map.Entry<String, Employee> mapEntry : entries) {
    shallowCopy.put(mapEntry.getKey(), mapEntry.getValue());
}
```

### 3.4 使用Map.putAll()

我们可以使用putAll()方法，而不是遍历所有条目，该方法一步浅拷贝所有映射：

```java
HashMap<String, Employee> shallowCopy = new HashMap<>();
shallowCopy.putAll(originalMap);
```

我们应该注意，**如果有匹配的键，put()和putAll()会替换值**。

有趣的是，如果我们查看HashMap的构造函数clone()和putAll()实现，我们会发现它们都使用相同的内部方法来复制条目—putMapEntries()。

## 4. 使用Java 8 Stream API复制HashMap

我们可以使用[Java 8 Stream API](https://www.baeldung.com/java-8-streams)来创建HashMap的浅表副本：

```java
Set<Entry<String, Employee>> entries = originalMap.entrySet();
HashMap<String, Employee> shallowCopy = (HashMap<String, Employee>) entries.stream()
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

## 5. Google Guava

使用[Guava Maps](https://www.baeldung.com/guava-maps)，我们可以轻松创建不可变Map，以及排序和双向Map。要对这些Map中的任何一个进行不可变的浅拷贝，我们可以使用copyOf方法：

```java
Map<String, Employee> map = ImmutableMap.<String, Employee>builder()
    .put("emp1",emp1)
    .put("emp2",emp2)
    .build();
Map<String, Employee> shallowCopy = ImmutableMap.copyOf(map);
    
assertThat(shallowCopy).isSameAs(map);
```

## 6. Apache Commons Lang

**现在，Java没有任何内置的深度复制实现**。因此，要进行深拷贝，我们可以覆盖clone()方法或使用序列化-反序列化技术。

Apache Commons有SerializationUtils和clone()方法来创建深层副本。为此，任何要包含在深拷贝中的类都必须实现Serializable接口：

```java
public class Employee implements Serializable {
    // implementation details
}

HashMap<String, Employee> deepCopy = SerializationUtils.clone(originalMap);
```

## 7. 总结

在这个快速教程中，我们了解了在Java中复制HashMap的各种技术，以及HashMap的浅拷贝和深拷贝的概念。

此外，我们探讨了一些非常适合创建浅拷贝和深拷贝的外部库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。