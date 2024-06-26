---
layout: post
title:  Spring MVC中的HttpMediaTypeNotAcceptableException
category: springweb
copyright: springweb
excerpt: Spring Web
---

## 一、概述

在这篇简短的文章中，我们将了解HttpMediaTypeNotAcceptableException异常，并了解我们可能遇到它的情况。

## 2.问题

在使用 Spring 实现 API 端点时，我们通常需要指定消费/生产的媒体类型(通过consumes和produces参数)。这缩小了 API 将针对该特定操作返回给客户端的可能格式。

HTTP 还有专用的“接受”标头——用于指定客户端识别和可以接受的媒体类型。简而言之，服务器将使用客户端请求的一种媒体类型发回资源表示。

但是，如果没有双方都可以使用的通用类型，Spring 将抛出HttpMediaTypeNotAcceptableException异常。

## 3. 实例

让我们创建一个简单示例来演示此场景。

我们将使用一个 POST 端点——它只能与“application/ json ”一起工作，并返回 JSON 数据：

```java
@PostMapping(
  value = "/test", 
  consumes = MediaType.APPLICATION_JSON_VALUE, 
  produces = MediaType.APPLICATION_JSON_VALUE)
public Map<String, String> example() {
    return Collections.singletonMap("key", "value");
}
```

然后，让我们使用 CURL 发送一个内容类型无法识别的请求：

```bash
curl -X POST --header "Accept: application/pdf" http://localhost:8080/test -v

> POST /test HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.51.0
> Accept: application/pdf
```

我们得到的回应是：

```bash
< HTTP/1.1 406 
< Content-Length: 0
```

## 4.解决方案

只有一种方法可以解决这个问题——发送/接收其中一种受支持的类型。

我们所能做的就是提供一个更具描述性的消息(默认情况下 Spring 返回一个空主体)和一个自定义的ExceptionHandler通知客户端所有可接受的媒体类型。

在我们的例子中，它只是“application/json”：

```java
@ResponseBody
@ExceptionHandler(HttpMediaTypeNotAcceptableException.class)
public String handleHttpMediaTypeNotAcceptableException() {
    return "acceptable MIME type:" + MediaType.APPLICATION_JSON_VALUE;
}
```

## 5.总结

在本教程中，我们考虑了当客户端请求的内容与服务器实际生成的内容不匹配时 Spring MVC 抛出的HttpMediaTypeNotAcceptableException异常。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-web-modules)上获得。