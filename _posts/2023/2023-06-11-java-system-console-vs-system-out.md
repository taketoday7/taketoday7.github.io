---
layout: post
title:  System.console()与System.out
category: java
copyright: java
excerpt: Java Console
---

## 1. 概述

在本教程中，我们将探讨System.console()和System.out之间的区别。

## 2. System.console()

让我们首先创建一个程序来检索Console对象：

```java
void printConsoleObject() {
    Console console = System.console();
    console.writer().print(console);
}
```

从交互式终端运行此程序将输出类似java.io.Console@546a03af的内容。

但是，从其他媒体运行它会抛出NullPointerException，因为Console对象将为null。

或者，如果我们按如下方式运行程序：

```shell
$ java ConsoleAndOut > test.txt
```

然后程序也会在我们重定向流时抛出NullPointerException。

Console类还提供了在不回显字符的情况下读取密码的方法。

让我们看看实际效果：

```java
void readPasswordFromConsole() {
    Console console = System.console();
    char[] password = console.readPassword("Enter password: ");
    console.printf(String.valueOf(password));
}
```

这将提示输入密码，并且在我们键入时不会回显字符。

## 3. System.out

现在让我们打印System.out的对象：

```java
System.out.println(System.out);
```

这将返回类似java.io.PrintStream的内容。

任何地方的输出都是一样的。

System.out用于将数据打印到输出流，并且没有读取数据的方法。输出流可以重定向到任何目标，例如文件，输出将保持不变。

我们可以按以下方式运行程序：

```shell
$ java ConsoleAndOut > test.txt
```

这会将输出打印到test.txt文件。

## 4. 差异

根据示例，我们可以确定一些差异：

-   System.console()从交互式终端运行时返回一个java.io.Console实例，另一方面，System.out将返回java.io.PrintStream对象，而不管调用介质如何
-   如果我们没有重定向任何流，System.out和System.console()的行为是相似的；否则，System.console()返回null
-   当多个线程提示输入时，Console会很好地排列这些提示-而在System.out的情况下，所有提示同时出现

## 5. 总结

我们在本文中了解了System.console()和System.out之间的区别。当应用程序应该从交互式控制台运行时，Console很有用，但它有一些应该注意和照顾的怪癖。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-console)上获得。