---
layout: post
title:  使用HttpURLConnection发出JSON POST请求
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在本教程中，我们将演示如何使用[HttpURLConnection](https://www.baeldung.com/java-http-request)发出[JSON](https://www.baeldung.com/category/json/) POST请求。

## 2. 使用HttpURLConnection构建JSON POST请求

### 2.1 创建URL对象

让我们创建一个带有目标URI字符串的URL对象，该对象通过HTTP POST方法接收JSON数据：

```java
URL url = new URL("https://reqres.in/api/users");
```

### 2.2 打开连接

从上面的URL对象，我们可以调用openConnection方法来获取HttpURLConnection对象。

我们不能直接实例化HttpURLConnection，因为它是一个抽象类：

```java
HttpURLConnection con = (HttpURLConnection)url.openConnection();
```

### 2.3 设置请求方法

要发送POST请求，我们必须将请求方法属性设置为POST：

```java
con.setRequestMethod("POST");
```

### 2.4 设置请求Content-Type标头参数

**将“content-type”请求头设置为“application/json”**，以JSON形式发送请求内容。必须设置此参数才能以JSON格式发送请求正文。

如果不这样做，服务器将返回HTTP状态代码“400-bad request”：

```java
con.setRequestProperty("Content-Type", "application/json");
```

### 2.5 设置响应格式类型

**将“Accept”请求标头设置为“application/json”以所需的格式读取响应**：

```java
con.setRequestProperty("Accept", "application/json");
```

### 2.6 确保连接将用于发送内容

要发送请求内容，让我们将URLConnection对象的doOutput属性设置为true。

否则，我们将无法将内容写入连接输出流：

```java
con.setDoOutput(true);
```

### 2.7 创建请求主体

创建自定义JSON字符串后：

```java
String jsonInputString = "{"name": "Upendra", "job": "Programmer"}";
```

我们需要写入它：

```java
try(OutputStream os = con.getOutputStream()) {
    byte[] input = jsonInputString.getBytes("utf-8");
    os.write(input, 0, input.length);			
}
```

### 2.8 从输入流读取响应

获取输入流读取响应内容。请记住使用try-with-resources自动关闭响应流。

通读整个响应内容，并打印最终的响应字符串：

```java
try(BufferedReader br = new BufferedReader(new InputStreamReader(con.getInputStream(), "utf-8"))) {
    StringBuilder response = new StringBuilder();
    String responseLine = null;
    while ((responseLine = br.readLine()) != null) {
        response.append(responseLine.trim());
    }
    System.out.println(response.toString());
}
```

如果响应是JSON格式，请使用任何第三方JSON解析器(例如[Jackson](https://www.baeldung.com/jackson)库、[Gson](https://www.baeldung.com/gson-string-to-jsonobject)或[org.json)](https://www.baeldung.com/java-org-json)来解析响应。

## 3. 总结

在本文中，我们学习了如何使用HttpURLConnection使用JSON内容正文发出POST请求。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-2)上获得。
