---
layout: post
title: 修复IllegalArgumentException:No enum const class
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在这篇简短的文章中，我们将仔细研究异常“IllegalArgumentException: No enum const class”。

首先，我们将了解此异常背后的主要原因。然后，我们将看到如何使用实际示例重现它，最后学习如何修复它。

## 2. 原因

在深入细节之前，让我们先了解异常及其堆栈跟踪的含义。

通常，**当我们将非法或不适当的值传递给方法时，会发生[IllegalArgumentException](https://www.baeldung.com/java-illegalargumentexception-or-nullpointerexception#illegalargumentexception)**。

“No enum const class”告诉我们在指定的枚举类型中没有给定名称的常量。

因此，**此异常的最典型原因通常是使用无效的枚举常量作为方法参数**。

## 3. 重现异常

现在我们知道了异常的含义，让我们看看如何使用实际示例重现它。

例如，让我们考虑Priority枚举：

```java
public enum Priority {

    HIGH("High"), MEDIUM("Medium"), LOW("Low");

    private String name;

    Priority(String name) {
        this.name = name;
    }

    public String getPriorityName() {
        return name;
    }
}
```

如我们所见，我们的枚举有一个私有字段name，表示每个优先级常量的名称。

接下来，让我们创建一个[静态方法](https://www.baeldung.com/java-static-methods-use-cases)来帮助我们通过name获取Priority常量：

```java
public class PriorityUtils {

    public static Priority getByName(String name) {
        return Priority.valueOf(name);
    }

    public static void main(String[] args) {
        System.out.println(getByName("Low"));
    }
}
```

现在，如果我们执行PriorityUtils类，我们会得到一个异常：

```text
Exception in thread "main" java.lang.IllegalArgumentException: No enum constant cn.tuyucheng.taketoday.exception.noenumconst.Priority.Low
    at java.lang.Enum.valueOf(Enum.java:238)
    at cn.tuyucheng.taketoday.exception.noenumconst.Priority.valueOf(Priority.java:1)
....
```

查看堆栈跟踪，getByName(String name)失败并出现异常，因为内置方法[Enum.valueOf()](https://docs.oracle.com/javase/7/docs/api/java/lang/Enum.html#valueOf(java.lang.Class, java.lang.String))无法找到给定名称为“Low”的Priority常量。

**Enum.valueOf()只接收一个字符串，该字符串必须与用于在枚举中声明常量的标识符完全匹配**。换句话说，它只接受HIGH、 MEDIUM和LOW作为参数。**由于它不知道name属性，当我们将值“Low”传递给它时，它会抛出IllegalArgumentException**。

因此，让我们使用测试用例来确认这一点：

```java
@Test
void givenCustomName_whenUsingGetByName_thenThrowIllegalArgumentException() {
    assertThrows(IllegalArgumentException.class, () -> PriorityUtils.getByName("Low"));
}
```

## 4. 解决方案

**最简单的解决方案是在将自定义name传递给Enum.valueOf()方法之前将其转换为大写**。

这样，我们确保传递的字符串与大写的常量名称完全匹配。

现在，让我们看看它的实际效果：

```java
public static Priority getByUpperCaseName(String name) {
    if (name == null || name.isEmpty()) {
        return null;
    }

    return Priority.valueOf(name.toUpperCase());
}
```

为了避免任何[NullPointerException](https://www.baeldung.com/java-illegalargumentexception-or-nullpointerexception#nullpointerexception)或不需要的行为，我们添加了一个检查以确保给定的name不为空且不为空字符串。

最后，让我们添加一些测试用例来确认一切正常：

```java
@Test
void givenCustomName_whenUsingGetByUpperCaseName_thenReturnEnumConstant() {
    assertEquals(Priority.HIGH, PriorityUtils.getByUpperCaseName("High"));
}
```

如我们所见，我们使用自定义名称High成功获得了Priority.HIGH。

现在，让我们检查一下当我们传递空值或空字符串时会发生什么：

```java
@Test
void givenEmptyName_whenUsingGetByUpperCaseName_thenReturnNull() {
    assertNull(PriorityUtils.getByUpperCaseName(""));
}

@Test
void givenNull_whenUsingGetByUpperCaseName_thenReturnNull() {
    assertNull(PriorityUtils.getByUpperCaseName(null));
}
```

正如我们在上面看到的，该方法确实返回null。

## 5. 总结

在这个简短的教程中，我们详细讨论了导致Java抛出异常“IllegalArgumentException: No enum const class”的原因。

在此过程中，我们学习了如何重现异常以及如何使用实际示例修复它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-4)上获得。
