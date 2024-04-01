---
layout: post
title:  在Java中切片数组
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

我们知道Java的List有[subList()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#subList(int,int))方法，可以让我们对源List对象进行切片。但是，数组端没有标准的subArray()方法。

在本教程中，让我们探讨如何在Java中获取给定数组的子数组。

## 2. 问题简介

像往常一样，让我们通过一个例子来理解这个问题。假设我们有一个字符串数组：

```java
String[] LANGUAGES = new String[] { "Python", "Java", "Kotlin", "Scala", "Ruby", "Go", "Rust" };
```

如我们所见，LANGUAGES数组包含一些编程语言名称。此外，由于应用程序是用“Java”、“Kotlin”或“Scala”编写的，因此可以在Java虚拟机上运行，假设我们想要一个包含这三个元素的子数组。换句话说，**我们想要从LANGUAGES数组中获取第二个到第四个元素(索引1、2、3)**：

```java
String[] JVM_LANGUAGES = new String[] { "Java", "Kotlin", "Scala" };
```

在本教程中，我们将介绍解决此问题的不同方法。此外，为简单起见，我们将使用单元测试断言来验证每个解决方案是否按预期工作。

接下来，让我们看看它们的实际效果。

## 3. 使用Stream API

Java 8给我们带来的一个重要的新特性是[Stream API](https://www.baeldung.com/java-8-streams)。因此，如果我们使用的Java版本是8或更高版本，我们可以使用Stream API对给定数组进行切片。

**首先，我们可以使用Arrays.stream()方法[将数组转换为Stream对象](https://www.baeldung.com/java-stream-to-array#array-to-stream)**。我们应该注意，我们应该使用带有三个参数的[Arrays.stream()](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#stream-T:A-int-int-)方法：

-   array：在这个例子中，它是LANGUAGES
-   startInclusive：从上面的数组中提取的起始索引，包含在内
-   endExclusive：要提取的结束索引，顾名思义，排除

因此，为了解决我们的问题，我们可以将LANGUAGES、1和4传递给Arrays.stream()方法。

接下来，让我们创建一个测试，看看它是否可以得到我们想要的子数组：

```java
String[] result = Arrays.stream(LANGUAGES, 1, 4).toArray(String[]::new);
assertArrayEquals(JVM_LANGUAGES, result);
```

如上面的代码所示，在我们将数组转换为Stream之后，我们可以调用[toArray()](https://www.baeldung.com/java-stream-to-array#1-method-reference)方法将其转换回数组。

如果我们运行测试，它就会通过。

## 4. 使用Arrays.copyOfRange()方法

我们已经学会了使用Stream API来解决这个问题。但是，**Stream API仅在Java 8及更高版本中可用**。

如果我们的Java版本是6以上，我们可以使用Arrays.copyOfRange()方法解决问题。该方法的参数类似于Arrays.stream()方法，即array、from-index(包含)和to-index(不包含)。

那么接下来，让我们创建一个测试，看看Arrays.copyOfRange()是否可以解决问题：

```java
String[] result = Arrays.copyOfRange(LANGUAGES, 1, 4);
assertArrayEquals(JVM_LANGUAGES, result);
```

如果我们运行，测试就会通过。

## 5. 使用System.arraycopy()方法

Arrays.copyOfRange()方法通过将给定数组的一部分复制到新数组来解决问题。

当我们想从数组中复制一部分时，除了Arrays.copyOfRange()方法之外，我们还可以使用[System.arraycopy()](https://docs.oracle.com/javase/8/docs/api/java/lang/System.html#arraycopy-java.lang.Object-int-java.lang.Object-int-int-)方法。

我们已经看到Arrays.copyOfRange()返回结果子数组。但是，**System.arraycopy()方法的返回类型是void**。因此，我们必须创建一个新的数组对象并将其传递给arraycopy()方法。该方法填充数组中复制的元素：

```java
String[] result = new String[3];
System.arraycopy(LANGUAGES, 1, result, 0, 3);
assertArrayEquals(JVM_LANGUAGES, result);
```

如果我们运行它，测试就会通过。

正如我们在上面的代码中看到的，arraycopy()方法有5个参数。让我们了解它们的含义：

-   源数组-LANGUAGE
-   要复制的源数组中的from索引–1
-   保存复制结果的目标数组–result
-   目标数组中用于存储复制结果的起始索引–0
-   我们要从源数组中复制的元素数量–3

值得一提的是，**如果结果数组已经包含数据，则arraycopy()方法可能会覆盖数据**：

```java
String[] result2 = new String[] { "value one", "value two", "value three", "value four", "value five", "value six", "value seven" };
System.arraycopy(LANGUAGES, 1, result2, 2, 3);
assertArrayEquals(new String[] { "value one", "value two", "Java", "Kotlin", "Scala", "value six", "value seven" }, result2);
```

这次，result2数组包含7个元素。此外，当我们调用arraycopy()方法时，我们告诉它将从索引2复制的数据填充到result2中。正如我们所看到的，复制的3个元素已经覆盖了原始元素作为result2。

另外，我们应该注意System.arraycopy()是一个本地方法，Arrays.copyOfRange()方法在内部调用System.arraycopy()。

## 6. 使用Apache Commons Lang3库中的ArrayUtils

[Apache Commons Lang3](https://www.baeldung.com/java-commons-lang-3)是一个使用非常广泛的库，它的[ArrayUtils](https://www.baeldung.com/array-processing-commons-lang#arrayutils)提供了很多方便的方法，使我们可以更轻松地使用数组。

最后，让我们使用ArrayUtils类来解决这个问题。

在开始使用ArrayUtils之前，让我们将依赖项添加到我们的Maven配置中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

当然，我们始终可以在Maven中央仓库中找到[最新的版本](https://search.maven.org/search?q=g:org.apache.commonsANDa:commons-lang3&core=gav)。

ArrayUtils类有subarray()方法，它允许我们快速获取子数组：

```java
String[] result = ArrayUtils.subarray(LANGUAGES, 1, 4);
assertArrayEquals(JVM_LANGUAGES, result);
```

正如我们所见，使用subarray()方法解决问题非常简单。

## 7. 总结

在本文中，我们学习了从给定数组中获取子数组的不同方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-operations-advanced)上获得。