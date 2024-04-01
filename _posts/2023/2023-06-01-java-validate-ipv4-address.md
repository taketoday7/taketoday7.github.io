---
layout: post
title:  在Java中验证IPv4地址
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在这个简短的教程中，我们将了解如何**在Java中验证IPv4地址**。

## 2. IPv4验证规则

我们的有效IPv4地址采用“x.x.x.x”形式，其中每个x是0 <= x <= 255范围内的数字，没有前导0，并由点分隔。

以下是一些有效的IPv4地址：

-   192.168.0.1
-   10.0.0.255
-   255.255.255.255

这是无效的：

-   192.168.0.256(值高于255)
-   192.168.0(只有3个八位字节)
-   .192.168.0.1(以“.”开头)
-   192.168.0.01(有前导0)

## 3. 使用Apache Commons Validator

我们可以使用[Apache Commons Validator](https://mvnrepository.com/artifact/commons-validator/commons-validator)库中的InetAddressValidator类来验证我们的IPv4或IPv6地址。

让我们将依赖项添加到pom.xml文件中：

```xml
<dependency>
    <groupId>commons-validator</groupId>
    <artifactId>commons-validator</artifactId>
    <version>1.7</version>
</dependency>
```

然后，我们只需要使用InetAddressValidator对象的isValid()方法：

```java
InetAddressValidator validator = InetAddressValidator.getInstance();
validator.isValid("127.0.0.1");
```

## 4. 使用Guava

或者，我们可以使用[Guava](https://www.baeldung.com/guava-guide)库中的InetAddresses类来实现相同的目标：

```java
InetAddresses.isInetAddress("127.0.0.1");
```

## 5. 使用正则表达式

最后，我们还可以使用[正则表达式](https://www.baeldung.com/regular-expressions-java)：

```java
String regex = "^((25[0-5]|(2[0-4]|1\\d|[1-9]|)\\d)\\.?\\b){4}$";
Pattern pattern = Pattern.compile(regex);
Matcher matcher = pattern.matcher(ip);
matcher.matches();
```

在这个正则表达式中，((25\[0-5]|(2\[0-4]|1\\\d|\[1-9]|)\\\d)\\\\.?\\\b)是一个重复四次以匹配IPv4地址中的四个八位字节的组。以下匹配每个八位字节：

-   25\[0-5]：这匹配250到255之间的数字
-   (2\[0-4]|1\\\d|\[1-9])：这匹配200 – 249、100 – 199和1 – 9之间的数字
-   \\\d：这匹配任何数字(0-9)
-   \\\.?：这匹配可选的点(.)字符
-   \\\b：这是一个单词边界

因此，此正则表达式匹配IPv4地址。

## 6. 总结

总之，我们学习了在Java中验证IPv4地址的不同方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-4)上获得。
