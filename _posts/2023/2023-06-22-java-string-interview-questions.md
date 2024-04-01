---
layout: post
title:  Java字符串面试题及答案
category: interview
copyright: interview
excerpt: Java String
---

## 1. 概述

[String](https://www.tuyucheng.com/java-string)类是Java中使用最广泛的类之一，这促使语言设计者对其进行特殊对待。这种特殊的行为使其成为Java面试中最热门的话题之一。

在本教程中，我们将介绍有关String的一些最常见的面试问题。

## 2. String基础

本节包含有关String内部结构和内存的问题。

### Q1. Java中的字符串

在Java中，String在内部由字节值数组(或JDK 9之前的char值)表示。

在Java 8及之前的版本中，String由不可变的Unicode字符数组组成。但是，大多数字符只需要8位(1字节)来表示它们而不是16位(字符大小)。

为了改善内存消耗和性能，Java 9引入了[紧凑的字符串](https://www.tuyucheng.com/java-9-compact-string)。这意味着如果String仅包含1个字节的字符，它将使用Latin-1编码表示。如果一个字符串至少包含1个多字节字符，它将使用UTF-16编码表示为每个字符2个字节。

在C和C++中，String也是一个字符数组，但在Java中，它是一个具有自己API的独立对象。

### Q2. 我们如何在Java中创建字符串对象？

[java.lang.String](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html)定义了[13种不同的方法](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#constructor.summary)来创建String，不过一般来说最主要使用两种：

-   通过String文本：

    ```java
    String s = "abc";
    ```

-   通过new关键字：

    ```java
    String s = new String("abc");
    ```

Java中的所有字符串文本都是String类的实例。

### Q3. String是原始类型还是派生类型？

String是派生类型，因为它具有状态和行为。例如，它有像substring()、indexOf()和equals()这样的方法，而这些方法是原始类型所没有的。

但是，由于我们都经常使用它，因此它具有一些特殊特性，使它感觉像一个原始类型：

-   虽然字符串不像原始类型那样存储在调用堆栈中，但它们**存储在称为[字符串池](https://www.tuyucheng.com/java-string-pool)的特殊内存区域中**
-   像原始类型一样，我们可以在字符串上使用+运算符
-   同样，像原始类型一样，我们可以不需要new关键字创建一个String实例

### Q4. 字符串不可变有什么好处？

根据[James Gosling的采访](https://www.artima.com/intv/gosling313.html)，字符串是不可变的以提高性能和安全性。

实际上，我们可以总结[不可变字符串的几个好处](https://www.tuyucheng.com/java-string-immutable)：

-   **字符串池只有在字符串一旦创建就永远不会更改的情况下才有可能**，因为它们应该被重用
-   代码**可以安全地将一个字符串传递给另一个方法**，知道它不能被那个方法改变
-   **不可变地自动使此类成为线程安全的**
-   由于这个类是线程安全的，因此**不需要同步公共数据**，从而提高了性能
-   由于保证它们不会更改，因此**可以轻松缓存它们的哈希码**

### Q5. 字符串如何存储在内存中？

根据JVM规范，String字面量存储在[运行时常量池](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.5)中，该常量池是从JVM的[方法区](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.4)分配的。

**尽管方法区在逻辑上是堆内存的一部分，但规范并未规定位置、内存大小或垃圾回收策略。它可以是特定于实现的**。

类或接口的运行时常量池是在JVM创建类或接口时构建的。

### Q6. 在Java中，Interned字符串是否符合垃圾回收的条件？

是的，如果没有来自程序的引用，字符串池中的所有String都可以进行垃圾回收。

### Q7. 什么是字符串常量池？

[字符串池](https://www.tuyucheng.com/java-string-pool)，也称为String常量池或String intern池，是JVM存放String实例的特殊内存区域。

**它通过减少分配字符串的频率和数量来优化应用程序性能**：

-   JVM在池中只存储特定字符串的一个副本
-   创建新的String时，JVM在池中搜索具有相同值的String
-   如果找到，JVM返回对该String的引用而不分配任何额外的内存
-   如果未找到，则JVM将其添加到池中(interns它)并返回其引用

### Q8. 字符串是线程安全的吗？

字符串确实是完全线程安全的，因为它们是不可变的。任何不可变的类都自动符合线程安全的条件，因为它的不可变性保证了它的实例不会在多个线程中被更改。

**例如，如果一个线程更改了一个字符串的值，则会创建一个新的字符串而不是修改现有的字符串**。

### Q9. 为哪些字符串操作提供语言环境很重要？

Locale类允许我们区分不同的文化区域并适当地格式化我们的内容。

当涉及到String类时，我们在以格式呈现字符串或小写或大写字符串时需要它。

事实上，如果我们忘记这样做，我们可能会遇到可移植性、安全性和可用性方面的问题。

### Q10. 字符串的底层字符编码是什么？

根据Java 8及以下版本的String的[Javadocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html)，字符串在内部以UTF-16格式存储。

char数据类型和java.lang.Character对象[也基于原始的Unicode规范](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#unicode)，该规范将字符定义为固定宽度的16位实体。

从JDK 9开始，仅包含1个字节字符的字符串使用Latin-1编码，而至少包含1个多字节字符的字符串使用UTF-16编码。

## 3. String API

在本节中，我们将讨论与String API相关的一些问题。

### Q11. 我们如何在Java中比较两个字符串？str1==str2和str1.equals(str2)有什么区别？

我们可以通过两种不同的方式[比较字符串](https://www.tuyucheng.com/java-compare-strings)：使用等于运算符(==)和使用equals()方法。

两者彼此完全不同：

-   **运算符(str1==str2)**检查引用相等性
-   **方法(str1.equals(str2))**检查词法相等性

但是，如果两个字符串在词法上相等，那么str1.intern() == str2.intern()也是true。

**通常，为了比较两个字符串的内容，我们应该始终使用String.equals。**

### Q12. 我们如何在Java中拆分字符串？

String类本身为我们提供了[String#split](https://www.tuyucheng.com/string/split)方法，该方法接收正则表达式分隔符，返回给我们一个String[]数组：

```java
String[] parts = "john,peter,mary".split(",");
assertEquals(new String[] { "john", "peter", "mary" }, parts);
```

**关于split的一个棘手问题是，当拆分一个空字符串时，我们可能会得到一个非空数组**：

```java
assertEquals(new String[] { "" }, "".split(","));
```

当然，split只是[拆分Java字符串](https://www.tuyucheng.com/java-split-string)的众多方法之一。

### Q13. 什么是StringJoiner？

[StringJoiner](https://www.tuyucheng.com/java-string-joiner)是Java 8中引入的一个类，用于**将单独的字符串拼接成一个**，例如获取颜色列表并将它们作为逗号分隔的字符串返回。我们可以提供分隔符以及前缀和后缀：

```java
StringJoiner joiner = new StringJoiner(",", "[", "]");
joiner.add("Red")
    .add("Green")
    .add("Blue");

assertEquals("[Red,Green,Blue]", joiner.toString());
```

### Q14. String、StringBuffer和StringBuilder的区别？

字符串是不可变的，这意味着**如果我们尝试更改或改变它的值，那么Java会创建一个全新的字符串**。

例如，如果我们在创建字符串str1之后添加内容：

```java
String str1 = "abc";
str1 = str1 + "def";
```

那么JVM不会修改str1，而是创建一个全新的字符串。

但是，对于大多数简单的情况，编译器在内部使用StringBuilder并优化上述代码。

但是，**对于像循环这样更复杂的代码，它将创建一个全新的String**，从而降低性能。这是StringBuilder和StringBuffer有用的地方。

[Java中的StringBuilder和StringBuffer](https://www.tuyucheng.com/java-string-builder-string-buffer)都创建包含可变字符序列的对象。**StringBuffer是同步的，因此是线程安全的，而StringBuilder不是**。

由于StringBuffer中的额外同步通常是不必要的，因此我们通常可以通过选择StringBuilder来提高性能。

### Q15. 为什么将密码存储在Char[]数组中比存储在字符串中更安全？

由于字符串是不可变的，因此它们不允许修改。**这种行为使我们无法覆盖、修改或清除其内容，从而使字符串不适合存储敏感信息**。

我们必须依靠垃圾回收器来删除字符串的内容。此外，在Java 6及以下版本中，字符串存储在PermGen中，这意味着一旦创建了一个字符串，它就永远不会被垃圾回收。

**通过使用char[]数组，我们可以完全控制该信息。我们甚至可以在不依赖垃圾回收器的情况下对其进行修改或彻底清除**。

在String上使用char[]并不能完全保护信息；这只是一种额外的措施，可以减少恶意用户访问敏感信息的机会。

### Q16. String的intern()方法有什么作用？

**[intern()](https://www.tuyucheng.com/string/intern)方法在堆中创建String对象的精确副本，并将其存储在JVM维护的字符串常量池中**。

Java会自动intern所有使用字符串文本创建的字符串，但是如果我们使用new运算符创建一个字符串，例如String str = new String("abc")，那么Java会将它添加到堆中，就像任何其他对象一样。

我们可以调用intern()方法告诉JVM将它添加到字符串池中(如果它尚不存在)，并返回该interned字符串的引用：

```java
String s1 = "Tuyucheng";
String s2 = new String("Tuyucheng");
String s3 = new String("Tuyucheng").intern();

assertThat(s1 == s2).isFalse();
assertThat(s1 == s3).isTrue();
```

### Q17. 我们如何在Java中将字符串转换为整数以及将整数转换为字符串？

[将String转换为Integer](https://www.tuyucheng.com/java-convert-string-to-int-or-integer)最直接的方法是使用Integer#parseInt：

```java
int num = Integer.parseInt("22");
```

要执行相反的转换，我们可以使用Integer#toString：

```java
String s = Integer.toString(num);
```

### Q18. 什么是String.format()以及我们如何使用它？

[String#format](https://www.tuyucheng.com/string/format)使用指定的格式字符串和参数返回格式化字符串。

```java
String title = "Tuyucheng"; 
String formatted = String.format("Title is %s", title);
assertEquals("Title is Tuyucheng", formatted);
```

我们还需要记住指定用户的区域设置，除非我们可以简单地接受操作系统默认设置：

```java
Locale usersLocale = Locale.ITALY;
assertEquals("1.024", String.format(usersLocale, "There are %,d shirts to choose from. Good luck.", 1024))
```

### Q19. 我们如何将字符串转换为大写和小写？

String隐式提供[String#toUpperCase](https://www.tuyucheng.com/string/to-upper-case)以将大小写更改为大写。

但是，Javadocs提醒我们需要指定用户的Locale以确保正确性：

```java
String s = "Welcome to Tuyucheng!";
assertEquals("WELCOME TO TUYUCHENG!", s.toUpperCase(Locale.US));
```

同样，要转换为小写，我们有[String#toLowerCase](https://www.tuyucheng.com/string/to-lower-case)：

```java
String s = "Welcome to Tuyucheng!";
assertEquals("welcome to tuyucheng!", s.toLowerCase(Locale.UK));
```

### Q20. 我们如何从字符串中获取字符数组？

String提供toCharArray，它返回JDK 9之前的内部char数组的副本(并在JDK 9+中将String转换为新的char数组)：

```java
char[] hello = "hello".toCharArray();
assertArrayEquals(new String[] { 'h', 'e', 'l', 'l', 'o' }, hello);
```

### Q21. 我们如何将Java字符串转换为字节数组？

默认情况下，方法[String#getBytes()](https://www.tuyucheng.com/string/get-bytes)使用平台的默认字符集将字符串编码为字节数组。

虽然API不要求我们指定字符集，但[为了确保安全性和可移植性](https://www.tuyucheng.com/java-char-encoding)，我们应该这样做：

```java
byte[] byteArray2 = "efgh".getBytes(StandardCharsets.US_ASCII);
byte[] byteArray3 = "ijkl".getBytes("UTF-8");
```

## 4. 基于字符串的算法

在本节中，我们将讨论一些与字符串相关的编程问题。

### Q22. 我们如何在Java中检查两个字符串是否是Anagrams？

字谜是通过重新排列另一个给定单词的字母而形成的单词，例如“car”和“arc”。

首先，我们检查两个字符串的长度是否相等。

然后我们[将它们转换为char[]数组，对它们进行排序，然后检查是否相等](https://www.tuyucheng.com/java-sort-string-alphabetically)。

### Q23. 我们如何计算给定字符在字符串中出现的次数？

Java 8真正简化了如下聚合任务：

```java
long count = "hello".chars().filter(ch -> (char)ch == 'l').count();
assertEquals(2, count);
```

而且，还有其他几种[统计l](https://www.tuyucheng.com/java-count-chars)的好方法，包括循环、递归、正则表达式和外部库。

### Q24. 我们如何在Java中反转字符串？

有很多方法可以做到这一点，最直接的方法是使用StringBuilder(或StringBuffer)的reverse方法：

```java
String reversed = new StringBuilder("tuyucheng").reverse().toString();
assertEquals("gnehcuyut", reversed);
```

### Q25. 我们如何检查一个字符串是否是回文？

[回文](https://www.tuyucheng.com/java-palindrome-substrings)是向后和向前读相同的任何字符序列，例如“madam”、“radar”或“level”。

要[检查一个字符串是否为回文](https://www.tuyucheng.com/java-palindrome)，我们可以开始在单个循环中向前和向后迭代给定的字符串，一次一个字符。循环在第一次不匹配时退出。

## 5. 总结

在本文中，我们介绍了一些最常见的字符串面试问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-string-algorithms-1)上获得。