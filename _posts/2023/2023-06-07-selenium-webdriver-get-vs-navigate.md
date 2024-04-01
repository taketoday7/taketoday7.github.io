---
layout: post
title:  Selenium WebDriver中get()和navigate()的区别
category: selenium
copyright: selenium
excerpt: Selenium
---

## 1. 概述

[Selenium WebDriver](https://www.selenium.dev/documentation/webdriver/)是允许我们[测试网页](https://www.baeldung.com/selenium-webdriver-page-object)的API。在这个简短的教程中，我们将了解WebDriver中get()和navigate()方法之间的区别。

## 2. 关于WebDriver

**Selenium WebDriver API包含与不同Web浏览器交互的高级方法**。使用此API，我们可以调用不同的操作，例如加载网页、[单击链接](https://www.baeldung.com/java-selenium-javascript)、在DOM中搜索特定元素等等。

API中的两个方法get()和navigate()允许我们加载网页。**虽然它们在名称上相似，但正如我们接下来将看到的，在行为上存在一些差异**。

## 3. get()方法

在WebDriver中加载网页的最简单方法是使用get()方法：

```java
WebDriver driver = new ChromeDriver();
driver.get("https://www.taketoday.com/");
```

此代码创建一个新的Chrome WebDriver并加载Taketoday主页。值得注意的是，**get()方法一直等到网页被认为已完全加载并准备好返回控制权**。如果页面包含大量JavaScript或其他资源，则调用可能需要一段时间。

## 4. Navigate API

WebDriver API包括一组独立的导航函数。让我们看第一个：

```java
WebDriver driver = new ChromeDriver();
driver.navigate().to("https://www.taketoday.com/");
```

从功能上讲，**navigate().to()方法的行为与get()方法完全相同**。事实上，它只是get()方法的别名，只是在远程Web浏览器中加载指定的URL。而且因为它只是get()的别名，所以在网页完全加载之前它也不会返回。

但是，navigate API具有get()方法所没有的其他功能。

首先，它跟踪浏览器历史记录并允许一次在页面之间移动：

```java
driver.navigate().forward();
driver.navigate().back();
```

navigate还允许我们刷新当前URL：

```java
driver.navigate().refresh();
```

然而，最重要的是，**每次我们使用navigate API时，它都会维护cookie**。与get()方法在每次调用时丢弃会话状态不同，navigate()方法确实保持状态。

这意味着我们使用navigate API加载的每个页面都包含任何先前的cookie，这对于测试许多场景(例如登录和单页应用程序)是必需的。

## 5. 总结

在这篇快速文章中，我们了解了Selenium WebDriver API中get()和navigate()方法之间的区别。虽然get()更易于使用，但navigate()有两个主要优点。

首先，navigate()提供了用于导航历史页面以及刷新当前页面的附加方法。其次，它会维护它导航到的每个URL之间的状态，这意味着cookie和其他会话数据会在每次页面加载时都会持久保存。

了解这些差异使我们能够根据测试的需要选择最佳方法。