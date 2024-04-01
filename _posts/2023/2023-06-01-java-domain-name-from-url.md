---
layout: post
title:  在Java中从给定的URL获取域名
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在这篇简短的文章中，我们将介绍在Java中从给定URL获取域名的不同方法。

## 2. 什么是域名？

简单地说，域名表示一个指向IP地址的字符串。它是统一资源定位符(URL)的一部分。使用域名，用户可以通过客户端软件访问特定的网站。

域名通常由两到三部分组成，每个部分由一个点分隔。

从末尾开始，域名可能包括：

-   顶级域(例如tuyucheng.com中的com)，
-   二级域(例如，google.co.uk 中的co或tuyucheng.com中的tuyucheng)，
-   三级域(例如，google.co.uk中的google)

域名需要遵循[域名系统](https://www.baeldung.com/cs/dns-intro)(DNS)指定的规则和程序。

## 3. 使用URI类

让我们看看如何使用[java.net.URI](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URI.html)类从URL中提取域名。URI类提供了getHost()方法，该方法返回URL的主机部分：

```java
URI uri = new URI("https://www.tuyucheng.com/domain");
String host = uri.getHost();
assertEquals("www.tuyucheng.com", host);
```

**主机包含子域以及第三级、二级和顶级域**。

此外，要获得域名，我们需要从给定主机中删除子域：

```java
String domainName = host.startsWith("www.") ? host.substring(4) : host;
assertEquals("tuyucheng.com", domainName);
```

但是，在某些情况下，我们无法使用URI类获取域名。例如，如果我们不知道它的确切值，就不可能从URL中取出子域。

## 4. 使用Guava库中的InternetDomainName类

现在我们将了解如何使用Guava库和InternetDomainName类获取域名。

InternetDomainName类提供了topPrivateDomain()方法，该方法返回给定域名中公共后缀下一级的部分。**换句话说，该方法将返回顶级、二级和三级域**。

首先，我们需要从给定的URL值中提取主机。我们可以使用URI类：

```java
String urlString = "https://www.tuyucheng.com/java-tutorial";
URI uri = new URI(urlString);
String host = uri.getHost();
```

接下来，让我们使用InternetDomainName类及其topPrivateDomain()方法获取域名：

```java
InternetDomainName internetDomainName = InternetDomainName.from(host).topPrivateDomain(); 
String domainName = internetDomainName.toString(); 
assertEquals("tuyucheng.com", domainName);
```

与URI类相比，InternetDomainName将从返回值中省略子域。

最后，我们也可以从给定的URL中删除顶级域：

```java
String publicSuffix = internetDomainName.publicSuffix().toString();
String name = domainName.substring(0, domainName.lastIndexOf("." + publicSuffix));
```

此外，让我们创建一个测试来检查功能：

```java
assertEquals("tuyucheng", domainNameClient.getName("jira.tuyucheng.com"));
assertEquals("google", domainNameClient.getName("www.google.co.uk"));
```

我们可以看到子域和顶级域都从结果中删除了。

## 5. 使用正则表达式

使用正则表达式获取域名可能具有挑战性。例如，如果我们不知道确切的子域值，我们就无法确定应该从给定的URL中提取哪个词(如果有的话)。

另一方面，如果我们知道子域值，我们可以使用正则表达式将其从URL中删除：

```java
String url = "https://www.tuyucheng.com/domain";
String domainName =  url.replaceAll("http(s)?://|www\\.|/.*", "");
assertEquals("tuyucheng.com", domainName);
```

## 6. 总结

在本文中，我们研究了如何从给定的URL中提取域名。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-3)上获得。
