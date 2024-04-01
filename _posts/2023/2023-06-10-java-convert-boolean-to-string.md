---
layout: post
title:  在Java中将布尔值转换为字符串
category: java
copyright: java
excerpt: Java Boolean
---

## 1. 概述

在Java中，我们经常需要将布尔值转换为字符串表示。例如，这对于在用户界面中显示值或将值写入文件或数据库非常有用。

在这个快速教程中，我们将探讨将布尔值转换为字符串的各种方法。

## 2. 问题简介

将布尔值转换为字符串在Java中是一项简单的任务。但是，**正如我们所知，Java中有两种布尔类型：[原始](https://www.baeldung.com/java-primitives)布尔值和[对象Boolean](https://www.baeldung.com/java-primitives-vs-objects)**。

从原始boolean值和Boolean对象到字符串的转换非常相似。但是，有几点我们应该考虑。

那么接下来，让我们从原始布尔值开始，看看如何将它们转换为字符串。

为简单起见，我们将使用单元测试断言来验证转换结果是否符合预期。

## 3. 将原始布尔值转换为字符串

**原始布尔变量可以赋值true或false**。因此，我们可以使用if-else语句将其转换为字符串。此外，在Java中，[三元运算符](https://www.baeldung.com/java-ternary-operator)(也称为条件运算符)是编写if-else语句的一种简写方式。

因此，让我们使用三元运算符使转换代码紧凑且可读：

```java
boolean primitiveBoolean = true;
assertEquals("true", primitiveBoolean ? "true" : "false");
                                                           
primitiveBoolean = false;
assertEquals("false", primitiveBoolean ? "true" : "false");
```

上面的代码非常简单。如我们所见，我们将true值转换为字符串“true”，将false值转换为“false”。这是一种标准的转换方式。但是，有时候，我们可能希望重新定义转换后的字符串，例如true为“YES”，false为“NO”。然后，**我们可以简单地更改三元表达式中的字符串**。

当然，如果我们需要多次调用转换，我们可以将其包装在一个方法中。接下来让我们看一个将布尔值转换为自定义字符串的例子：

```java
String optionToString(String optionName, boolean optionValue) {
    return String.format("The option '%s' is %s.", optionName, optionValue ? "Enabled" : "Disabled");
}
```

optionToString()方法接收布尔选项的名称及其值来构建选项状态的描述：

```java
assertEquals("The option 'IgnoreWarnings' is Enabled.", optionToString("IgnoreWarnings", true));
```

## 4. 使用Boolean.toString()方法将Boolean对象转换为字符串 

现在，让我们看看如何将Boolean变量转换为字符串。**Boolean类提供了Boolean.toString()方法来将Boolean转换为String**：

```java
Boolean myBoolean = Boolean.TRUE;
assertEquals("true", myBoolean.toString());
                                            
myBoolean = Boolean.FALSE;
assertEquals("false", myBoolean.toString());
```

如果我们仔细看看Boolean.toString()方法，我们会发现它的实现与我们的三元运算符解决方案完全相同：

```java
public String toString() {
    return this.value ? "true" : "false";
}
```

Boolean对象类似于原始对象。但是，**除了true和false之外，它还可以是null。因此，在调用Boolean.toString()方法之前，我们需要确保布尔变量不为空**。否则，将引发NullPointerException：

```java
Boolean nullBoolean = null;
assertThrows(NullPointerException.class, () -> nullBoolean.toString());
```

## 5. 使用String.valueOf()方法将Boolean对象转换为字符串 

我们已经看到标准库中的Boolean.toString()可以将Boolean变量转换为字符串。或者，**我们可以使用String类中的[valueOf()](https://www.baeldung.com/string/value-of)方法来解决问题**：

```java
Boolean myBoolean = Boolean.TRUE;
assertEquals("true", String.valueOf(myBoolean));
                                                 
myBoolean = Boolean.FALSE;
assertEquals("false", String.valueOf(myBoolean));
```

值得一提的是，String.valueOf()方法是空安全的。换句话说，**如果我们的布尔变量是null，String.valueOf()生成“null”而不是抛出NullPointerException**：

```java
Boolean nullBoolean = null;
assertEquals("null", String.valueOf(nullBoolean));
```

这是因为String.valueOf(Object obj)方法执行空值检查：

```java
public static String valueOf(Object obj) {
    return obj == null ? "null" : obj.toString();
}
```

## 6. 总结

在本文中，我们探讨了在Java中将布尔值转换为字符串的各种方法。

我们讨论了原始布尔值和对象布尔值的情况：

-   boolean：使用三元运算符
-   Boolean：我们可以使用Boolean.toString()方法(需要空值检查)或String.valueOf()方法(空值安全)

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-booleans)上获得。