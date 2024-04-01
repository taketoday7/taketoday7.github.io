---
layout: post
title:  彩色打印控制台的方法
category: java
copyright: java
excerpt: Java Console
---

## 1. 概述

添加一些颜色可以使日志更易于阅读。

在本文中，我们将了解如何为Visual Studio Code终端、Linux和Windows命令提示符等控制台的日志添加颜色。

在开始之前，让我们注意，不幸的是，Eclipse IDE控制台中只有有限的颜色设置。Eclipse IDE中的控制台不支持由Java代码确定的颜色，**因此本文中介绍的解决方案在Eclipse IDE控制台中不起作用**。

## 2. 如何使用ANSI代码为日志着色

**实现彩色日志记录的最简单方法是使用[ANSI转义序列](https://en.wikipedia.org/wiki/ANSI_escape_code)，通常称为ANSI代码**。

ANSI代码是特殊的字节序列，某些终端将其解释为命令。

让我们使用ANSI代码注销：

```java
System.out.println("Here's some text");
System.out.println("\u001B[31m" + "and now the text is red");
```

在输出中，我们看到未打印ANSI代码，字体颜色已更改为红色：

Here's some text
<font color=red>and now the text is red</font>

**请注意，我们需要确保在完成日志记录后重置字体颜色**。

幸运的是，这很容易。我们可以简单地打印\u001B[31m，这是ANSI重置命令。

重置命令会将控制台重置为其默认颜色。请注意，这不一定是黑色，它可以是白色或控制台配置的任何其他颜色。例如：

```java
System.out.println("Here's some text");
System.out.println("\u001B[31m" + "and now the text is red" + "\u001B[0m");
System.out.println("and now back to the default");
```

这是输出：

Here's some text
<font color=red>and now the text is red</font>
and now back to the default

大多数日志记录库将遵守ANSI代码，这使我们能够构建一些彩色记录器。

例如，我们可以快速构建一个记录器，它为不同的日志级别使用不同的颜色。

```java
public class ColorLogger {

    private static final Logger LOGGER = LoggerFactory.getLogger(ColorLogger.class);

    public void logDebug(String logging) {
        LOGGER.debug("\u001B[34m" + logging + "\u001B[0m");
    }

    public void logInfo(String logging) {
        LOGGER.info("\u001B[32m" + logging + "\u001B[0m");
    }

    public void logError(String logging) {
        LOGGER.error("\u001B[31m" + logging + "\u001B[0m");
    }
}
```

让我们使用它来向控制台打印一些颜色：

```java
ColorLogger colorLogger = new ColorLogger();
colorLogger.logDebug("Some debug logging");
colorLogger.logInfo("Some info logging");
colorLogger.logError("Some error logging");
```

\[main] DEBUG cn.tuyucheng.taketoday.color.ColorLogger - <font color=blue>Some debug logging</font>
\[main] INFO cn.tuyucheng.taketoday.color.ColorLogger - <font color=green>Some info logging</font>
\[main] ERROR cn.tuyucheng.taketoday.color.ColorLogger - <font color=red>Some error logging</font>

我们可以看到每个日志级别都是不同的颜色，使我们的日志更具可读性。

最后，ANSI代码不仅可以用来控制字体颜色，还可以控制背景颜色、字体粗细和样式。示例项目中有这些ANSI代码的选择。

## 3. 如何在Windows命令提示符中为日志着色

遗憾的是，某些终端不支持ANSI代码。一个典型的例子是Windows命令提示符，上面的操作将不起作用。因此，我们需要更复杂的解决方案。

但是，我们可以利用一个名为[JANSI](https://github.com/fusesource/jansi)的已建立库，而不是尝试自己实现它：

```xml
<dependency>
    <groupId>org.fusesource.jansi</groupId>
    <artifactId>jansi</artifactId>
    <version>2.4.0</version>
</dependency>
```

现在要记录颜色，我们可以简单地调用JANSI提供的ANSI API：

```java
private static void logColorUsingJANSI() {
    AnsiConsole.systemInstall();

    System.out.println(ansi()
        .fgRed()
        .a("Some red text")
        .fgBlue()
        .a(" and some blue text")
        .reset());

    AnsiConsole.systemUninstall();
}
```

这将生成文本：

<font color=red>Some red text</font> <font color=blue>and some blue text</font>

**请注意，我们必须先安装AnsiConsole，并在完成后将其卸载**。

与ANSI代码一样，JANSI也提供了大量的日志记录格式。

JANSI通过检测正在使用的终端并调用所需的相应特定于平台的API来实现此功能。这意味着当JANSI检测到Windows命令提示符时，它不会使用不起作用的ANSI代码，而是调用使用[Java本机接口(JNI)](https://baeldung-cn.com/java-native)方法的库。

此外，JANSI不仅仅适用于Windows命令提示符-它能够覆盖大多数终端(不过Eclipse IDE控制台不是其中之一，因为Eclipse中对彩色文本的设置有限)。

最后，JANSI还将确保在环境不需要时删除不需要的ANSI代码，从而帮助保持我们的日志干净整洁。

总的来说，JANSI为我们提供了一种强大而便捷的方式来记录大多数环境和终端的颜色。

## 4. 总结

在本文中，我们学习了如何使用ANSI代码来控制控制台字体的颜色，并看到了如何使用颜色区分日志级别的示例。

最后，我们发现并非所有控制台都支持ANSI代码，并重点介绍了一个JANSI这样的库，它提供了更复杂的支持。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-console)上获得。