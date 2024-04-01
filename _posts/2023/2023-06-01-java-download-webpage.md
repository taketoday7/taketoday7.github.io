---
layout: post
title:  用Java下载网页
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在某些应用程序中，我们可能需要从互联网上下载一个网页并将其内容提取为字符串。一个流行的用例是网络抓取或内容解析。

在本教程中，我们将使用[Jsoup](https://www.baeldung.com/java-with-jsoup)和HttpURLConnection下载示例网页。

## 2. 使用HttpURLConnection下载网页

[HttpURLConnection](https://www.baeldung.com/java-http-request)是URLConnection的子类。**它有助于连接到使用HTTP作为其协议的统一资源定位器([URL](https://www.baeldung.com/java-url-vs-uri#uri-and-url))**。该类包含处理[HTTP](https://www.baeldung.com/java-9-http-client)请求的不同方法。

让我们使用HttpURLConnection下载[示例网页](https://example.com/)：

```java
@Test
void givenURLConnection_whenRetrieveWebpage_thenWebpageIsNotNullAndContainsHtmlTag() throws IOException {
    URLurl = new URL("https://example.com");
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setRequestMethod("GET");
    
    try (BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()))) {
        StringBuilder responseBuilder = new StringBuilder();
        String line;
        while ((line = reader.readLine()) != null) {
            responseBuilder.append(line);
        }
    
        assertNotNull(responseBuilder);
        assertTrue(responseBuilder.toString().contains("<html>"));
    }
}
```

在这里，我们创建一个表示网页地址的URL对象。接下来，我们创建一个HttpURLConnection实例并调用URL对象的openConnection()方法。这将打开到网页的连接。此外，我们将请求方法设置为GET以获取网页内容。

然后，我们创建一个新的BufferedReader和InputStreamReader实例来帮助从网页中读取数据。InputStreamReader类有助于将原始字节转换为可由BufferedReader读取的字符。

最后，我们通过从BufferedReader读取并将行拼接在一起，将网页转换为字符串。我们使用StringBuilder对象来有效地拼接这些行。

## 3. 使用Jsoup下载网页

Jsoup是一个流行的开源Java库，用于处理HTML。它有助于获取URL并提取其数据。**它的主要优势之一是使用HTML DOM方法和CSS选择器从URL中抓取HTML**。

要开始使用Jsoup，让我们将[Jsoup](https://mvnrepository.com/artifact/org.jsoup/jsoup)依赖项添加到pom.xml：

```xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.15.4</version>
</dependency>
```

以下是使用Jsoup下载网页的示例：

```java
@Test
void givenJsoup_whenRetrievingWebpage_thenWebpageDocumentIsNotNullAndContainsHtmlTag() throws IOException {
    Document document =Jsoup.connect("https://www.example.com").get();
    String webpage = document.html();
        
    assertNotNull(webpage);
    assertTrue(webpage.contains("<html>"));
}
```

在此示例中，我们创建了一个Document实例，并使用Jsoup.connect()建立了到示例站点的连接。Jsoup.connect()有助于建立到URL的连接并将其内容检索为Document对象。

接下来，我们调用get()方法，该方法向指定的URL发送GET请求。它将响应作为Document返回。

最后，我们将提取出来的内容存入一个String类型的变量webpage中。我们通过在Document对象上调用html()方法来做到这一点。

## 4. 总结

在本文中，我们学习了两种在Java中下载网页的方法。我们使用了HttpURLConnection类和Jsoup来下载网页的内容。这两种方法都可以使用，但Jsoup似乎更容易。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-4)上获得。
