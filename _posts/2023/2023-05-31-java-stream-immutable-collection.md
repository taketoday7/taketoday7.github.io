---
layout: post
title:  将Java流收集到不可变集合
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

我们经常希望将Java Stream转换为集合。这通常会产生一个可变集合，但我们可以自定义它。

在这个简短的教程中，我们将仔细研究**如何将Java Stream收集到不可变集合中**-首先使用纯Java，然后使用Guava库。

## 2. 使用标准Java

### 2.1 使用Java的toUnmodifiableList

从Java 10开始，我们可以使用Java的Collectors类中的toUnmodifiableList方法：

```java
List<String> givenList = Arrays.asList("a", "b", "c");
List<String> result = givenList.stream()
    .collect(toUnmodifiableList());
```

通过使用此方法，我们得到了一个来自Java ImmutableCollections的不支持空值的List实现：

```java
class java.util.ImmutableCollections$ListN
```

### 2.2 使用Java的collectingAndThen

Java的Collectors类中的collectingAndThen方法接收一个Collector和一个finisher函数。此finisher函数应用于从收集器返回的结果：

```java
List<String> givenList = Arrays.asList("a", "b", "c");
List<String> result = givenList.stream()
    .collect(collectingAndThen(toList(), ImmutableList::copyOf));

System.out.println(result.getClass());
```

使用这种方法，**由于我们不能直接使用toCollection收集器，我们需要将元素收集到一个临时列表中**。然后，我们从中构造一个不可变列表。

### 2.3 使用Stream.toList()方法

Java 16在[Stream API](https://www.baeldung.com/java-8-streams)上引入了一个名为toList()的新方法。这个方便的方法**返回一个包含流元素的不可修改的List**：

```java
@Test
public void whenUsingStreamToList_thenReturnImmutableList() {
    List<String> immutableList = Stream.of("a", "b", "c", "d").toList();
	
    Assertions.assertThrows(UnsupportedOperationException.class, () -> {
        immutableList.add("e");
    });
}
```

正如我们在单元测试中看到的，Stream.toList()返回一个不可变列表。因此，尝试向列表中添加新元素只会导致[UnsupportedOperationException](https://www.baeldung.com/java-list-unsupported-operation-exception)。

请记住，新的Stream.toList()方法与现有的[Collectors.toList()](https://www.baeldung.com/java-8-collectors#1-collectorstolist)略有不同，因为它返回一个不可修改的列表。

## 3. 构建自定义收集器

我们还可以选择[实现自定义Collector](https://www.baeldung.com/java-8-collectors#Custom)。

### 3.1 基本的不可变收集器

为此，我们可以使用静态Collector.of方法：

```java
public static <T> Collector<T, List<T>, List<T>> toImmutableList() {
    return Collector.of(ArrayList::new, List::add,
        (left, right) -> {
            left.addAll(right);
            return left;
        }, Collections::unmodifiableList);
}
```

我们可以像使用任何内置收集器一样使用此函数：

```java
List<String> givenList = Arrays.asList("a", "b", "c", "d");
List<String> result = givenList.stream()
    .collect(MyImmutableListCollector.toImmutableList());
```

最后，让我们检查输出类型：

```java
class java.util.Collections$UnmodifiableRandomAccessList
```

### 3.2 使MyImmutableListCollector泛型

我们的实现有一个限制-它总是返回一个由ArrayList支持的不可变实例。然而，稍微改进一下，我们就可以让这个收集器返回一个用户指定的类型：

```java
public static <T, A extends List<T>> Collector<T, A, List<T>> toImmutableList(Supplier<A> supplier) {
    return Collector.of(
        supplier,
        List::add, (left, right) -> {
            left.addAll(right);
            return left;
        }, Collections::unmodifiableList);
}
```

所以现在，我们不再在方法实现中确定Supplier，而是向用户请求Supplier：

```java
List<String> givenList = Arrays.asList("a", "b", "c", "d");
List<String> result = givenList.stream()
    .collect(MyImmutableListCollector.toImmutableList(LinkedList::new));
```

此外，我们使用LinkedList而不是ArrayList。

```java
class java.util.Collections$UnmodifiableList
```

这一次，我们得到了UnmodifiableList而不是UnmodifiableRandomAccessList。

## 4. 使用Guava的Collector

在本节中，我们将使用[Google Guava](https://mvnrepository.com/artifact/com.google.guava/guava)库来驱动我们的一些示例：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

从Guava 21开始，每个不可变类都附带一个Collector，它与Java的标准Collector一样易于使用：

```java
List<Integer> list = IntStream.range(0, 9)
    .boxed()
    .collect(ImmutableList.toImmutableList());
```

生成的实例是RegularImmutableList：

```java
class com.google.common.collect.RegularImmutableList
```

## 5. 总结

在这篇简短的文章中，我们了解了将Stream收集到不可变集合中的各种方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-2)上获得。
