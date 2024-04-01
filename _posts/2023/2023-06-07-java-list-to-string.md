---
layout: post
title:  在Java中将List转换为字符串
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将解释如何将元素列表转换为字符串。这在某些情况下很有用，例如以人类可读的形式将内容打印到控制台以供检查/调试。

## 2. List中的标准toString()

最简单的方法之一是调用List上的toString()方法：

```java
@Test
public void whenListToString_thenPrintDefault() {
    List<Integer> intLIst = Arrays.asList(1, 2, 3);
 
    System.out.println(intLIst);
}
```

输出：

```text
[1, 2, 3]
```

该技术在内部使用List中元素类型的toString()方法。在我们的例子中，我们使用Integer类型，它具有toString()方法的正确实现。

如果我们使用我们的自定义类型，例如Person，那么我们需要确保Person类重写toString()方法并且不依赖于默认实现。如果我们没有正确实现toString()方法，我们可能会得到意想不到的结果：

```plaintext
[cn.tuyucheng.taketoday.java.lists.ListToSTring$Person@1edf1c96,
  cn.tuyucheng.taketoday.java.lists.ListToSTring$Person@368102c8,
  cn.tuyucheng.taketoday.java.lists.ListToSTring$Person@6996db8]
```

## 3. 使用Collectors自定义实现

通常，我们可能需要以不同的格式显示输出。

与前面的示例相比，我们将逗号(,)替换为连字符(-)，将方括号(\[,])替换为一组大括号({,})：

```java
@Test
public void whenCollectorsJoining_thenPrintCustom() {
    List<Integer> intList = Arrays.asList(1, 2, 3);
    String result = intList.stream()
        .map(n -> String.valueOf(n))
        .collect(Collectors.joining("-", "{", "}"));
 
    System.out.println(result);
}
```

输出：

```plaintext
{1-2-3}
```

Collectors.joining()方法需要一个CharSequence，所以我们需要将Integer映射到String。我们可以将同样的想法用于其他类，即使我们无法访问该类的代码。

## 4. 使用外部库

现在我们将使用Apache Commons的StringUtils类来获得类似的结果。

### 4.1 Maven依赖

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.11</version>
</dependency>
```

可以在[此处](https://search.maven.org/classic/#search|gav|1|g%3A"org.apache.commons"ANDa%3A"commons-lang3")找到最新版本的依赖项。

### 4.2 实现

该实现实际上是一个单一的方法调用：

```java
@Test
public void whenStringUtilsJoin_thenPrintCustom() {
    List<Integer> intList = Arrays.asList(1, 2, 3);
 
    System.out.println(StringUtils.join(intList, "|"));
}
```

输出：

```plaintext
1|2|3
```

同样，此实现在内部依赖于我们正在考虑的类型的toString()实现。

## 5. 总结

在本文中，我们介绍了使用不同的技术将List转换为String。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-1)上获得。