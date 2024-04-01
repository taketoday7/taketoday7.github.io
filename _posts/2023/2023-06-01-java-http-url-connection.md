---
layout: post
title:  使用HttpUrlConnection进行身份验证
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在本教程中，我们将探讨如何使用HttpUrlConnection类对[HTTP请求](https://www.baeldung.com/java-http-request)进行身份验证。

## 2. HTTP身份验证

在Web应用程序中，服务器可能需要客户端对自己进行身份验证。不遵守通常会导致服务器返回HTTP 401(Unauthorized)状态码。

有多种[身份验证方案](https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml)，它们提供的安全强度各不相同。但是，实施工作也各不相同。

让我们看看其中的三个：

-   basic是我们将在下一节中详细介绍的方案
-   digest在用户凭证和服务器指定的随机数上应用哈希算法
-   bearer使用访问令牌作为[OAuth 2.0](https://www.baeldung.com/spring-security-5-oauth2-login)的一部分

## 3. 基本认证

基本身份验证允许客户端通过Authorization标头使用**编码的用户名和密码**对自己进行身份验证：

```text
GET / HTTP/1.1
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

要创建编码后的用户名和密码字符串，我们只需对用户名进行Base64编码，后跟冒号，然后是密码：

```text
basic(user, pass) = base64-encode(user + ":" + pass)
```

不过，请记住[RFC 7617](https://tools.ietf.org/html/rfc7617)中的一些警告：

>   该方案不被认为是一种安全的用户身份验证方法，除非与某些外部安全系统(如TLS)结合使用

当然，这是因为用户名和密码在每个请求中以纯文本形式在网络上传输。

## 4. 验证连接

好的，有了这个作为背景，让我们开始配置HttpUrlConnection以使用HTTP Basic。

HttpUrlConnection类可以发送请求，但首先，我们必须从URL对象中获取它的一个实例：

```java
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
```

连接提供了许多方法来配置它，例如setRequestMethod和setRequestProperty。

尽管setRequestProperty听起来很奇怪，但这正是我们想要的。

使用“:”拼接用户名和密码后，我们可以使用java.util.Base64类对凭据进行编码：

```java
String auth = user + ":" + password;
byte[] encodedAuth = Base64.encodeBase64(auth.getBytes(StandardCharsets.UTF_8));
```

然后，我们从文字“Basic”创建标头值，后跟编码的凭据：

```java
String authHeaderValue = "Basic " + new String(encodedAuth);
```

接下来，我们调用方法setRequestProperty(key, value)来验证请求。如前所述，**我们必须使用“Authorization”作为标头，使用“Basic”+编码凭据作为我们的值**：

```java
connection.setRequestProperty("Authorization", authHeaderValue);
```

最后，我们需要实际发送HTTP请求，例如通过调用getResponseCode()。结果，我们从服务器得到一个HTTP响应码：

```java
int responseCode = connection.getResponseCode();
```

2xx系列中的任何内容都意味着我们的请求(包括身份验证部分)没问题！

## 5. Java Authenticator

上面提到的基本身份验证实现需要为每个请求设置授权标头。相比之下，**抽象类java.net.Authenticator允许为所有连接设置全局身份验证**。

我们需要先扩展类。然后，我们调用静态方法Authenticator.setDefault()来注册我们的身份验证器的实例：

```java
Authenticator.setDefault(new BasicAuthenticator());
```

我们的基本身份验证类只是覆盖了基类的getPasswordAuthentication()非抽象方法：

```java
private final class BasicAuthenticator extends Authenticator {
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication(user, password.toCharArray());
    }
}
```

Authenticator类利用我们的身份验证器的凭据自动完成服务器所需的身份验证方案。

## 6. 总结

在这个简短的教程中，我们了解了如何将基本身份验证应用于通过HttpUrlConnection发送的请求。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-2)上获得。
