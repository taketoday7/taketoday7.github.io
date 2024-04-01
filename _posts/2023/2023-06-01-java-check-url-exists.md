---
layout: post
title:  在Java中检查URL是否存在
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在本教程中，我们将通过使用GET和HEAD [HTTP方法](https://www.baeldung.com/java-http-request)的Java示例来了解如何检查URL是否存在。

## 2. URL存在性

在编程中，有时我们必须在访问资源之前知道给定URL中是否存在资源，或者我们甚至可能需要检查URL以了解资源的健康状况。

我们通过查看资源的响应代码来确定资源在URL中是否存在。**通常我们寻找200，这意味着“OK”并且请求已成功**。

## 3. 使用GET请求

首先，要发出GET请求，我们可以创建java.net.URL的实例并将我们想要访问的URL作为构造函数参数传递。之后，我们只需打开连接并获取响应码：

```java
URL url = new URL("http://www.example.com");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
 
int responseCode = huc.getResponseCode();
 
Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

当在URL找不到资源时，我们会收到404响应码：

```java
URL url = new URL("http://www.example.com/xyz"); 
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
 
int responseCode = huc.getResponseCode();
 
Assert.assertEquals(HttpURLConnection.HTTP_NOT_FOUND, responseCode);
```

由于**HttpURLConnection中的默认HTTP方法是GET**，因此我们不会在本节的示例中设置请求方法。我们将在下一节中看到如何覆盖默认方法。

## 4. 使用HEAD请求 

**HEAD也是一种HTTP请求方法，与GET相同，只是它不返回响应主体**。 

它获取响应码以及响应标头，如果使用GET方法请求相同的资源，我们将收到这些标头。

要创建HEAD请求，我们可以在获取响应码之前简单地将请求方法设置为HEAD：

```java
URL url = new URL("http://www.example.com");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
huc.setRequestMethod("HEAD");
 
int responseCode = huc.getResponseCode();
 
Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

同样，当在URL找不到资源时：

```java
URL url = new URL("http://www.example.com/xyz");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
huc.setRequestMethod("HEAD");
 
int responseCode = huc.getResponseCode();
 
Assert.assertEquals(HttpURLConnection.HTTP_NOT_FOUND, responseCode);
```

**通过使用HEAD方法而不是下载响应主体，我们减少了响应时间和带宽，并提高了性能**。

尽管大多数现代服务器都支持HEAD方法，**但某些自行开发或遗留服务器可能会拒绝HEAD方法并出现无效的方法类型错误。因此，我们应该谨慎使用HEAD方法**。

## 5. 重定向

最后，在寻找URL是否存在时，最好不要遵循重定向。但这也可能取决于我们查找URL的原因。

移动URL时，服务器可以将请求重定向到具有3xx响应码的新URL。**默认是跟随重定向**，我们可以根据需要选择跟随或忽略重定向。

为此，我们可以为所有HttpURLConnection覆盖followRedirects的默认值：

```java
URL url = new URL("http://www.example.com");
HttpURLConnection.setFollowRedirects(false);
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
 
int responseCode = huc.getResponseCode();
 
Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

或者，我们可以使用setInstanceFollowRedirects()方法为单个连接禁用跟随重定向：

```java
URL url = new URL("http://www.example.com");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
huc.setInstanceFollowRedirects(false);
 
int responseCode = huc.getResponseCode();
 
Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

## 6. 总结

在本文中，我们查看了检查响应码以查找URL的可用性。此外，我们还研究了如何使用HEAD方法来节省带宽并获得更快的响应。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-2)上获得。
