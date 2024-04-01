---
layout: post
title:  数组到字符串的转换
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个简短的教程中，我们将介绍如何将字符串或整数数组转换为字符串，然后再转换回来。

**我们可以使用常用库中的原生Java和Java实用程序类来实现这一点**。

## 2. 数组转换为字符串

有时我们需要将字符串或整数数组转换为字符串，但不幸的是，没有直接的方法来执行这种转换。

数组上的toString()方法的默认实现返回类似于Ljava.lang.String;@74a10858的内容，它仅告知我们对象的类型和哈希码。

但是，[java.util.Arrays](https://www.baeldung.com/java-util-arrays)实用程序类支持数组和字符串操作，包括用于数组的toString()方法。

**Arrays.toString()返回一个包含输入数组内容的字符串**。创建的新字符串是以逗号分隔的数组元素列表，用方括号("[]")括起来：

```java
String[] strArray = { "one", "two", "three" };
String joinedString = Arrays.toString(strArray);
assertEquals("[one, two, three]", joinedString);
```

```java
int[] intArray = { 1,2,3,4,5 }; 
joinedString = Arrays.toString(intArray);
assertEquals("[1, 2, 3, 4, 5]", joinedString);
```

而且，虽然Arrays.toString(int[])方法非常好地为我们完成了这项任务，但让我们将它与我们可以自己实现的不同方法进行比较。

### 2.1 StringBuilder.append()

首先，让我们看看如何使用StringBuilder.append()进行这种转换：

```java
String[] strArray = { "Convert", "Array", "With", "Java" };
StringBuilder stringBuilder = new StringBuilder();
for (int i = 0; i < strArray.length; i++) {
    stringBuilder.append(strArray[i]);
}
String joinedString = stringBuilder.toString();
assertEquals("ConvertArrayWithJava", joinedString);
```

此外，要转换整数数组，我们可以使用相同的方法，但在追加到StringBuilder时调用Integer.valueOf(int Array\[i])。

### 2.2 Java Stream API

Java 8及更高版本提供了String.join()方法，该方法通过拼接元素并使用指定的分隔符分隔它们来生成新字符串，在我们的例子中只是空字符串：

```java
String joinedString = String.join("", new String[]{ "Convert", "With", "Java", "Streams" });
assertEquals("ConvertWithJavaStreams", joinedString);
```

此外，我们可以使用Java Stream API中的Collectors.joining()方法，该方法按照与其源数组相同的顺序拼接来自Stream的字符串：

```java
String joinedString = Arrays
    .stream(new String[]{ "Convert", "With", "Java", "Streams" })
    .collect(Collectors.joining());
assertEquals("ConvertWithJavaStreams", joinedString);
```

### 2.3 StringUtils.join()

[Apache Commons Lang](https://www.baeldung.com/java-commons-lang-3)永远不会被排除在这样的任务之外。

StringUtils类有几个StringUtils.join()方法，可用于将字符串数组更改为单个字符串：

```java
String joinedString = StringUtils.join(new String[]{ "Convert", "With", "Apache", "Commons" });
assertEquals("ConvertWithApacheCommons", joinedString);
```

### 2.4 Joiner.join()

Guava也不甘示弱，它的[Joiner](https://www.baeldung.com/guava-joiner-and-splitter-tutorial)类也是如此。Joiner类提供了一个流式的API，并提供了一些辅助方法来拼接数据。

例如，我们可以添加分隔符或跳过空值：

```java
String joinedString = Joiner.on("")
    .skipNulls()
    .join(new String[]{ "Convert", "With", "Guava", null });
assertEquals("ConvertWithGuava", joinedString);
```

## 3. 将字符串转换为字符串数组

类似地，我们有时需要将一个字符串拆分为一个数组，该数组包含由指定分隔符拆分的输入字符串的某个子集，让我们也看看如何做到这一点。

### 3.1 String.split()

首先，让我们首先使用String.split()方法拆分空格，不带分隔符：

```java
String[] strArray = "loremipsum".split("");
```

这生成：

```text
["l", "o", "r", "e", "m", "i", "p", "s", "u", "m"]
```

### 3.2 StringUtils.split()函数

其次，让我们再看看Apache的Commons Lang库中的StringUtils类。

在字符串对象的许多空安全方法中，我们可以找到StringUtils.split()。默认情况下，它采用空格分隔符：

```java
String[] splitted = StringUtils.split("lorem ipsum dolor sit amet");
```

结果是：

```text
["lorem", "ipsum", "dolor", "sit", "amet"]
```

但是，如果需要，我们也可以提供分隔符。

### 3.3 Splitter.split()

最后，我们还可以使用Guava及其Splitter流式API：

```java
List<String> resultList = Splitter.on(' ')
    .trimResults()
    .omitEmptyStrings()
    .splitToList("lorem ipsum dolor sit amet");   
String[] strArray = resultList.toArray(new String[0]);
```

这生成：

```text
["lorem", "ipsum", "dolor", "sit", "amet"]
```

## 4. 总结

在本文中，我们说明了如何使用核心Java和流行的实用程序库将数组转换为字符串，然后再转换回来。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-2)上获得。