---
layout: post
title:  如何在Java中将List转换为Map
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

将List转换为Map是一项常见任务。在本教程中，我们将介绍执行此操作的几种方法。

我们假设List的每个元素都有一个标识符，该标识符将用作生成的Map中的键。

## 2. 示例数据结构

首先，我们将对元素建模：

```java
public class Animal {
    private int id;
    private String name;

    //  constructor/getters/setters
}
```

id字段是唯一的，因此我们可以将其作为键。

让我们开始使用传统方式进行转换。

## 3. Java 8之前

显然，我们可以使用核心Java方法将List转换为Map：

```java
public Map<Integer, Animal> convertListBeforeJava8(List<Animal> list) {
    Map<Integer, Animal> map = new HashMap<>();
    for (Animal animal : list) {
        map.put(animal.getId(), animal);
    }
    return map;
}
```

现在我们测试转换：

```java
@Test
public void givenAList_whenConvertBeforeJava8_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService
        .convertListBeforeJava8(list);
    
    assertThat(map.values(), containsInAnyOrder(list.toArray()));
}
```

## 4. 使用Java 8

从Java 8开始，我们可以使用流和收集器将List转换为Map：

```java
public Map<Integer, Animal> convertListAfterJava8(List<Animal> list) {
    Map<Integer, Animal> map = list.stream()
        .collect(Collectors.toMap(Animal::getId, Function.identity()));
    return map;
}
```

同样，让我们确保转换正确完成：

```java
@Test
public void givenAList_whenConvertAfterJava8_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService.convertListAfterJava8(list);
    
    assertThat(map.values(), containsInAnyOrder(list.toArray()));
}
```

## 5. 使用Guava库

除了核心Java，我们还可以使用第三方库进行转换。

### 5.1 Maven配置

首先，我们需要将以下依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

此库的最新版本始终可以在[这里](https://search.maven.org/classic/#search|gav|1|g%3A"com.google.guava"ANDa%3A"guava")找到。

### 5.2 使用Maps.uniqueIndex()进行转换

其次，让我们使用Maps.uniqueIndex()方法将List转换为Map：

```java
public Map<Integer, Animal> convertListWithGuava(List<Animal> list) {
    Map<Integer, Animal> map = Maps
        .uniqueIndex(list, Animal::getId);
    return map;
}
```

最后，我们测试转换：

```java
@Test
public void givenAList_whenConvertWithGuava_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService
        .convertListWithGuava(list);
    
    assertThat(map.values(), containsInAnyOrder(list.toArray()));
}
```

## 6. 使用Apache Commons库

我们也可以使用Apache Commons库的方法进行转换。

### 6.1 Maven配置

首先，让我们包含Maven依赖项：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

此依赖项的最新版本可在[此处](https://search.maven.org/classic/#search|gav|1|g%3A"org.apache.commons"ANDa%3A"commons-collections4")获得。

### 6.2 MapUtils

其次，我们将使用MapUtils.populateMap()进行转换：

```java
public Map<Integer, Animal> convertListWithApacheCommons(List<Animal> list) {
    Map<Integer, Animal> map = new HashMap<>();
    MapUtils.populateMap(map, list, Animal::getId);
    return map;
}
```

最后，我们可以确保它按预期工作：

```java
@Test
public void givenAList_whenConvertWithApacheCommons_thenReturnMapWithTheSameElements() {
    Map<Integer, Animal> map = convertListService
        .convertListWithApacheCommons(list);
    
    assertThat(map.values(), containsInAnyOrder(list.toArray()));
}
```

## 7. 值冲突

让我们看看如果id字段不唯一会发生什么。

### 7.1 具有重复ID的Animal列表

首先，我们创建一个具有非唯一id的Animal列表：

```java
@Before
public void init() {
    this.duplicatedIdList = new ArrayList<>();

    Animal cat = new Animal(1, "Cat");
    duplicatedIdList.add(cat);
    Animal dog = new Animal(2, "Dog");
    duplicatedIdList.add(dog);
    Animal pig = new Animal(3, "Pig");
    duplicatedIdList.add(pig);
    Animal cow = new Animal(4, "Cow");
    duplicatedIdList.add(cow);
    Animal goat= new Animal(4, "Goat");
    duplicatedIdList.add(goat);
}
```

如上所示，cow和goat具有相同的id。

### 7.2 检查行为

**Java Map的put()方法的实现使得最新添加的值覆盖具有相同键的前一个值**。

因此，传统的转换和Apache Commons MapUtils.populateMap()的行为方式相同：

```java
@Test
public void givenADupIdList_whenConvertBeforeJava8_thenReturnMapWithRewrittenElement() {
    Map<Integer, Animal> map = convertListService
        .convertListBeforeJava8(duplicatedIdList);

    assertThat(map.values(), hasSize(4));
    assertThat(map.values(), hasItem(duplicatedIdList.get(4)));
}

@Test
public void givenADupIdList_whenConvertWithApacheCommons_thenReturnMapWithRewrittenElement() {
    Map<Integer, Animal> map = convertListService
        .convertListWithApacheCommons(duplicatedIdList);

    assertThat(map.values(), hasSize(4));
    assertThat(map.values(), hasItem(duplicatedIdList.get(4)));
}
```

我们可以看到goat用相同的id覆盖了cow。

但是，**Collectors.toMap()和MapUtils.populateMap()分别抛出IllegalStateException和IllegalArgumentException**：

```java
@Test(expected = IllegalStateException.class)
public void givenADupIdList_whenConvertAfterJava8_thenException() {
    convertListService.convertListAfterJava8(duplicatedIdList);
}

@Test(expected = IllegalArgumentException.class)
public void givenADupIdList_whenConvertWithGuava_thenException() {
    convertListService.convertListWithGuava(duplicatedIdList);
}
```

## 8. 总结

在这篇简短的文章中，我们介绍了将List转换为Map的各种方法，给出了核心Java示例以及一些流行的第三方库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-1)上获得。