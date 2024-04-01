---
layout: post
title:  Character.isAlphabetic与Character.isLetter
category: java
copyright: java
excerpt: Java Char
---

## 1.概述

在本教程中，我们将首先简要介绍每个定义的Unicode代码点或字符范围的一些通用类别类型，以**了解字母和字母字符之间的区别**。

此外，**我们将查看Java中[Character](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html)类的isAlphabetic()和isLetter()方法**。最后，我们将介绍这些方法之间的异同。

## 2. Unicode字符的一般分类类型

[Unicode字符集](https://en.wikipedia.org/wiki/List_of_Unicode_characters)(UCS)包含1114112个代码点：U+0000-U+10FFFF。字符和代码点范围按类别分组。

Character类提供了[getType()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#getType(char))方法的两个重载版本，该方法返回一个指示字符的一般类别类型的值。

让我们看一下第一个方法的签名：

```java
public static int getType(char ch)
```

此方法无法处理增补字符。为了处理所有Unicode字符，包括增补字符，**Java的Character类提供了一个重载的[getType](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#getType(int))方法，它具有以下签名**：

```java
public static int getType(int codePoint)
```

接下来，让我们开始了解一些通用的类别类型。

### 2.1 UPPERCASE_LETTER

UPPERCASE_LETTER通用类别类型表示大写字母。

当我们对一个大写字母(例如'U')调用Character#getType方法时，该方法返回值1，相当于UPPERCASE_LETTER枚举值：

```java
assertEquals(Character.UPPERCASE_LETTER, Character.getType('U'));
```

### 2.2 LOWERCASE_LETTER

LOWERCASE_LETTER通用类别类型与小写字母相关联。

对小写字母(例如'u')调用Character#getType方法时，该方法将返回值2，这与LOWERCASE_LETTER的枚举值相同：

```java
assertEquals(Character.LOWERCASE_LETTER, Character.getType('u'));
```

### 2.3 TITLECASE_LETTER

接下来，TITLECASE_LETTER通用类别表示首字母大写字符。

有些字符看起来像成对的拉丁字母。当我们对此类Unicode字符调用Character#getType方法时，这将返回值3，它等于TITLECASE_LETTER枚举值：

```java
assertEquals(Character.TITLECASE_LETTER, Character.getType('\u01f2'));
```

此处，Unicode字符“\u01f2”代表拉丁大写字母“D”，后跟一个带卡隆的小写字母“Z”。

### 2.4 MODIFIER_LETTER

在Unicode标准中，[修饰符字母](https://en.wikipedia.org/wiki/Modifier_letter)是“一个字母或符号，通常写在它以某种方式修改的另一个字母旁边”。

MODIFIER_LETTER通用类别类型表示此类修饰符字母。

例如，修饰符字母小H('ʰ')，当传递给Character#getType方法时返回值4，这与MODIFIER_LETTER的枚举值相同：

```java
assertEquals(Character.MODIFIER_LETTER, Character.getType('\u02b0'));
```

Unicode字符“\u020b”代表修饰字母小H。

### 2.5 OTHER_LETTER

OTHER_LETTER通用类别类型表示表意文字或单写字母表中的字母。表意文字是代表思想或概念的图形符号，独立于任何特定语言。

单写字母表的字母只有一种情况。例如，希伯来语是一个单写字母系统。

让我们看一个希伯来字母Alef的例子('א')，当我们将它传递给Character#getType方法时，它返回值5，它等于OTHER_LETTER的枚举值：

```java
assertEquals(Character.OTHER_LETTER, Character.getType('\u05d0'));
```

Unicode字符“\u05d0”代表希伯来字母Alef。

### 2.6 LETTER_NUMBER

最后，LETTER_NUMBER类别与由字母或类似字母的符号组成的数字相关联。

例如，罗马数字属于LETTER_NUMBER一般类别。当我们用罗马数字“Ⅴ”调用Character#getType方法时，它返回值10，等于枚举LETTER_NUMBER值：

```java
assertEquals(Character.LETTER_NUMBER, Character.getType('\u2164'));
```

Unicode字符“\u2164”代表罗马数字5。

接下来，让我们看一下Character#isAlphabetic方法。

## 3. Character#isAlphabetic

首先，让我们看一下[isAlphabetic](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#isAlphabetic(int))方法的签名：

```java
public static boolean isAlphabetic(int codePoint)
```

这将Unicode代码点作为输入参数，**如果指定的Unicode代码点是字母，则返回true，否则返回false**。

如果字符的一般类别类型是以下任何一种，则该字符是字母：

-   UPPERCASE_LETTER
-   LOWERCASE_LETTER
-   TITLECASE_LETTER
-   MODIFIER_LETTER
-   OTHER_LETTER
-   LETTER_NUMBER

此外，如果一个字符具有Unicode标准定义的贡献属性Other_Alphabetic，则它是字母字符。

让我们看几个字母字符的例子：

```java
assertTrue(Character.isAlphabetic('A'));
assertTrue(Character.isAlphabetic('\u01f2'));
```

在上面的例子中，我们将UPPERCASE_LETTER 'A'和TITLECASE_LETTER '\u01f2'传递给isAlphabetic方法，它代表拉丁大写字母'D'后跟一个带卡隆的小'Z'，它返回true。

## 4. Character#isLetter

Java的Character类提供了isLetter()方法来判断指定字符是否为字母。让我们看一下方法签名：

```java
public static boolean isLetter(char ch)
```

它接收一个字符作为输入参数，**如果指定的字符是字母则返回true，否则返回false**。

如果字符的一般类别类型(由Character#getType方法提供)是以下任何一种，则该字符被认为是字母：

-   UPPERCASE_LETTER
-   LOWERCASE_LETTER
-   TITLECASE_LETTER
-   MODIFIER_LETTER
-   OTHER_LETTER

但是，此方法无法处理增补字符。为了处理所有Unicode字符，包括增补字符，Java的Character类提供了isLetter()方法的重载版本：

```java
public static boolean isLetter(int codePoint)
```

此方法可以处理所有Unicode字符，因为它采用Unicode代码点作为输入参数。此外，如果指定的Unicode代码点是我们之前定义的字母，它会返回true。

让我们看几个字母字符的例子：

```java
assertTrue(Character.isAlphabetic('a'));
assertTrue(Character.isAlphabetic('\u02b0'));
```

在上面的例子中，我们将LOWERCASE_LETTER 'a'和MODIFIER_LETTER '\u02b0'(代表修饰字母小H)作为isLetter方法的输入，它返回true。

## 5. 比较和对比

最后，**我们可以看到所有字母都是字母字符，但并非所有字母字符都是字母**。

换句话说，如果字符是字母或具有一般类别LETTER_NUMBER，则isAlphabetic方法返回true。此外，如果字符具有Unicode标准定义的Other_Alphabetic属性，它也会返回true。

首先，让我们看一个既是字母又是字母表的字符的例子-字符“a”：

```java
assertTrue(Character.isLetter('a')); 
assertTrue(Character.isAlphabetic('a'));
```

字符“a”在作为输入参数传递给isLetter()和isAlphabetic()方法时返回true。

接下来，让我们看一个字符是字母表而不是字母的例子。在这种情况下，我们将使用Unicode字符“\u2164”，它代表罗马数字5：

```java
assertFalse(Character.isLetter('\u2164'));
assertTrue(Character.isAlphabetic('\u2164'));
```

**传递给isLetter()方法的Unicode字符“\u2164”返回false。另一方面，当传递给isAlphabetic()方法时，它返回tru**e。

当然，对于英语，这种区别没有区别。由于英语的所有字母都属于字母表的范畴。另一方面，其他语言中的某些字符可能会有所不同。

## 6. 总结

在本文中，我们了解了Unicode代码点的不同一般类别。此外，我们介绍了isAlphabetic()和isLetter()方法之间的异同。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-char)上获得。