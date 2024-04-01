---
layout: post
title:  在Java中使用Selenium WebDriver打开一个新选项卡
category: selenium
copyright: selenium
excerpt: Selenium
---

## 1. 简介

Selenium WebDriver是一种流行的自动化Web测试工具，具有许多功能。自动化Web时的常见操作之一是在浏览器窗口中打开一个新选项卡。

打开新选项卡在各种情况下都很有用，包括测试多页面工作流、验证在新选项卡中打开的外部链接、与弹出窗口交互以及在并行运行测试时模拟多个用户同时与应用程序的不同部分交互。

早期的解决方案是自定义脚本，例如发送组合键，例如“Ctrl”+“T”，这通常会根据浏览器和操作系统产生不同的结果。

在本教程中，我们将探索**打开新选项卡的稳定方法-Selenium 4中引入的[newWindow()](https://w3c.github.io/webdriver/#new-window) API和JavaScript代码执行**。

## 2. 使用newWindow() API

Selenium 4引入了一个强大而灵活的API方法newWindow()用于在当前浏览器会话中创建一个新窗口。**它允许我们打开一个新的浏览器选项卡并自动切换到它**。此方法接收WindowType参数WINDOW或TAB并创建它。语法非常简单：

```java
driver.switchTo().newWindow(WindowType.TAB);
```

## 3. 使用JavaScript

使用Selenium WebDriver打开新选项卡的另一种方法是执行JavaScript代码。它涉及使用JavascriptExecutor接口的executeScript()方法，它允许我们直接在浏览器中运行JavaScript代码。**window.open()脚本在我们想要对新选项卡进行更多控制时很有用，例如指定要打开的URL**。

以下是使用此方法打开新选项卡的方法：

```java
((JavascriptExecutor) driver).executeScript("window.open()");
```

以及如何使用URL打开新标签页：

```java
((JavascriptExecutor) driver).executeScript("window.open('https://google.com')");
```

**请务必牢记，在执行window.open()方法后，驱动程序仍将聚焦于原始选项卡**。为了与新选项卡上的元素交互，我们需要使用driver.switchTo().window()方法将驱动程序的焦点切换到该选项卡。

下面是使用JavaScript打开后切换到新选项卡的示例：

```java
String newTab = driver.getWindowHandles()
    .stream()
    .filter(handle -> !handle.equals(originalTab))
    .findFirst()
    .get();
driver.switchTo().window(newTab);
```

## 4. 总结

在本文中，我们探讨了两种使用Selenium打开新选项卡的方法：Selenium 4中引入的newWindow()方法和使用JavaScript执行的window.open()方法。

newWindow()方法是Selenium 4中引入的新API，它使创建新选项卡或窗口变得简单直观。另一方面，使用JavaScript执行window.open()可以更好地控制新选项卡的打开方式，并且可以与早期版本的Selenium一起使用。但是，它可能需要更多代码并且更难使用，尤其是对于初学者而言。

与往常一样，代码示例可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/selenium-junit-testng)上找到。