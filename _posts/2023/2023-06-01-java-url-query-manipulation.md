---
layout: post
title:  Java中的URL查询操作
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在Java中，我们可以使用几个库来动态地向URL添加查询，同时保持URL的有效性。

在本文中，我们将学习如何使用其中的三个。这三者中的每一个都执行完全相同的任务。因此，我们将看到生成的URL是相同的。

## 2. Java EE 7 UriBuilder

**最接近内置Java解决方案的是javax.ws.rs-api中的UriBuilder**，我们需要将其导入到我们的pom.xml中：

```xml
<dependency>
    <groupId>javax.ws.rs</groupId>
    <artifactId>javax.ws.rs-api</artifactId>
    <version>2.1.1</version>
</dependency>
```

最新版本可以在[Maven仓库](https://mvnrepository.com/artifact/javax.ws.rs/javax.ws.rs-api)中找到。我们可能还需要导入[jersey-commons](https://mvnrepository.com/artifact/org.glassfish.jersey.core/jersey-common)才能运行我们的应用程序。

UriBuilder对象为我们提供了fromUri()方法来创建基本URI，并使用queryParam()来添加我们的查询。然后我们可以调用build()返回一个URI：

```java
@Test
void whenUsingJavaUriBuilder_thenParametersAreCorrectlyAdded() {
    String url = "tuyucheng.com";
    String key = "article";
    String value = "beta";
    URI uri = UriBuilder.fromUri(url)
        .queryParam(key, value)
        .build();

    assertEquals("tuyucheng.com?article=beta", uri.toString());
}
```

如上所示，带有附加查询的URL看起来符合预期。

## 3. Apache UriBuilder

**Apache在[HttpClient](https://www.baeldung.com/httpclient-guide)包中提供了自己的解决方案UriBuilder**。要使用它，我们需要将它添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
```

最新版本可以在[Maven仓库](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient)中找到。

要使用它，我们首先使用我们的基本URL字符串调用URIBuilder构造函数。然后使用它的构建器方法addParameter()附加我们的参数，最后调用build()：

```java
@Test
void whenUsingApacheUriBuilder_thenParametersAreCorrectlyAdded() {
    String url = "tuyucheng.com";
    String key = "article";
    String value = "alpha";
    URI uri = new URIBuilder(url).addParameter(key, value)
        .build();

    assertEquals("tuyucheng.com?article=alpha", uri.toString());
}
```

## 4. Spring UriComponentsBuilder

如果我们有一个[Spring](https://www.baeldung.com/spring-tutorial)应用程序，那么使用Spring提供的[UriComponentsBuilder](https://www.baeldung.com/spring-uricomponentsbuilder)可能是有意义的。要使用它，我们需要在pom.xml中添加spring-web依赖项：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>6.0.6</version>
</dependency>
```

我们可以在[Maven仓库](https://mvnrepository.com/artifact/org.springframework/spring-web)中找到最新版本。

**我们可以使用UriComponentsBuilder通过fromUriString()创建一个URI**，然后使用queryParam()附加我们的查询：

```java
@Test
void whenUsingSpringUriComponentsBuilder_thenParametersAreCorrectlyAdded() {
    String url = "tuyucheng.com";
    String key = "article";
    String value = "charlie";
    URI uri = UriComponentsBuilder.fromUriString(url)
        .queryParam(key, value)
        .build()
        .toUri();

    assertEquals("tuyucheng.com?article=charlie", uri.toString());
}
```

与其他方法不同，build()方法返回一个UriComponents对象，因此要以URI结束，我们还需要调用toURI()。

## 5. 总结

在本文中，我们了解了在Java中操作URL的三种方法。我们可以使用Java扩展包、Apaches UriBuilder或spring-web解决方案添加查询。

出于这个原因，决定将哪个用于我们的应用程序取决于我们有哪些包和导入以及我们已经使用了哪些。每个库都带来了一系列有用的功能，因此我们还应该考虑是否可以同时满足项目中的其他需求。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-4)上获得。
