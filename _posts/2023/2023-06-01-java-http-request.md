---
layout: post
title:  使用Java执行一个简单的HTTP请求
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在这个快速教程中，我们将介绍一种**在Java中执行HTTP请求的方法**-使用内置的Java类HttpUrlConnection。

请注意，从JDK 11开始，Java提供了一个用于执行HTTP请求的新API，这意味着HttpUrlConnection的替代品，即[HttpClient API](https://www.baeldung.com/java-9-http-client)。

## 2. HttpUrlConnection

**HttpUrlConnection类允许我们在不使用任何其他库的情况下执行基本的HTTP请求**。我们需要的所有类都是java.net包的一部分。

使用此方法的**缺点是代码可能比其他HTTP库更繁琐，并且它不提供更高级的功能，例如用于添加标头或身份验证的专用方法**。

## 3. 创建请求

**我们可以使用URL类的openConnection()方法创建一个HttpUrlConnection实例**。请注意，此方法仅创建一个连接对象，但尚未建立连接。

HttpUrlConnection类用于所有类型的请求，方法是将requestMethod属性设置为以下值之一：GET、POST、HEAD、OPTIONS、PUT、DELETE、TRACE。

让我们使用GET方法创建到给定URL的连接：

```java
URL url = new URL("http://example.com");
HttpURLConnection con = (HttpURLConnection) url.openConnection();
con.setRequestMethod("GET");
```

## 4. 添加请求参数

如果我们想向请求添加参数，**我们必须将doOutput属性设置为true，然后将param1=value&m2=value形式的字符串写入HttpUrlConnection实例的OutputStream**：

```java
Map<String, String> parameters = new HashMap<>();
parameters.put("param1", "val");

con.setDoOutput(true);
DataOutputStream out = new DataOutputStream(con.getOutputStream());
out.writeBytes(ParameterStringBuilder.getParamsString(parameters));
out.flush();
out.close();
```

为了方便参数Map的转换，我们编写了一个名为ParameterStringBuilder的实用程序类，其中包含一个静态方法getParamsString()，它将Map转换为所需格式的String：

```java
public class ParameterStringBuilder {
    public static String getParamsString(Map<String, String> params) throws UnsupportedEncodingException{
        StringBuilder result = new StringBuilder();

        for (Map.Entry<String, String> entry : params.entrySet()) {
            result.append(URLEncoder.encode(entry.getKey(), "UTF-8"));
            result.append("=");
            result.append(URLEncoder.encode(entry.getValue(), "UTF-8"));
            result.append("&");
        }

        String resultString = result.toString();
        return resultString.length() > 0
                ? resultString.substring(0, resultString.length() - 1)
                : resultString;
    }
}
```

## 5. 设置请求头

可以使用setRequestProperty()方法向请求添加标头：

```java
con.setRequestProperty("Content-Type", "application/json");
```

要从连接中读取标头的值，我们可以使用getHeaderField()方法：

```java
String contentType = con.getHeaderField("Content-Type");
```

## 6. 配置超时

HttpUrlConnection类**允许设置连接和读取超时**。这些值定义了等待与服务器建立连接或等待数据可供读取的时间间隔。

要设置超时值，我们可以使用setConnectTimeout()和setReadTimeout()方法：

```java
con.setConnectTimeout(5000);
con.setReadTimeout(5000);
```

在此示例中，我们将两个超时值都设置为5秒。

## 7. 处理Cookie

java.net包包含简化使用cookie的类，例如CookieManager和HttpCookie。

首先，要**从响应中读取cookie**，我们可以检索Set-Cookie标头的值并将其解析为HttpCookie对象列表：

```java
String cookiesHeader = con.getHeaderField("Set-Cookie");
List<HttpCookie> cookies = HttpCookie.parse(cookiesHeader);
```

接下来，我们**将把cookie添加到cookie存储中**：

```java
cookies.forEach(cookie -> cookieManager.getCookieStore().add(null, cookie));
```

让我们检查一个名为username的cookie是否存在，如果不存在，我们将把它添加到cookie存储中，值为“john”：

```java
Optional<HttpCookie> usernameCookie = cookies.stream()
    .findAny().filter(cookie -> cookie.getName().equals("username"));
if (usernameCookie == null) {
    cookieManager.getCookieStore().add(null, new HttpCookie("username", "john"));
}
```

最后，要**将cookie添加到请求中**，我们需要在关闭并重新打开连接后设置Cookie标头：

```java
con.disconnect();
con = (HttpURLConnection) url.openConnection();

con.setRequestProperty("Cookie", StringUtils.join(cookieManager.getCookieStore().getCookies(), ";"));
```

## 8. 处理重定向

我们可以**使用带有true或false参数的setInstanceFollowRedirects()方法为特定连接自动启用或禁用跟随重定向**：

```java
con.setInstanceFollowRedirects(false);
```

也可以**为所有连接启用或禁用自动重定向**：

```java
HttpUrlConnection.setFollowRedirects(false);
```

默认情况下，该行为处于启用状态。

当请求返回状态码301或302时，表示重定向，我们可以检索Location标头并创建对新URL的新请求：

```java
if (status == HttpURLConnection.HTTP_MOVED_TEMP || status == HttpURLConnection.HTTP_MOVED_PERM) {
    String location = con.getHeaderField("Location");
    URL newUrl = new URL(location);
    con = (HttpURLConnection) newUrl.openConnection();
}
```

## 9. 读取响应

**读取请求的响应可以通过解析HttpUrlConnection实例的InputStream来完成**。

**要执行请求，我们可以使用getResponseCode()、connect()、getInputStream()或getOutputStream()方法**：

```java
int status = con.getResponseCode();
```

最后，让我们读取请求的响应并将其放入content字符串中：

```java
BufferedReader in = new BufferedReader(new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer content = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    content.append(inputLine);
}
in.close();
```

要**关闭连接**，我们可以使用disconnect()方法：

```java
con.disconnect();
```

## 10. 读取失败请求的响应

如果请求失败，尝试读取HttpUrlConnection实例的InputStream将不起作用。相反，**我们可以使用HttpUrlConnection.getErrorStream()提供的流**。

我们可以通过比较[HTTP状态码](https://www.baeldung.com/cs/http-status-codes)来决定使用哪个InputStream：

```java
int status = con.getResponseCode();

Reader streamReader = null;

if (status > 299) {
    streamReader = new InputStreamReader(con.getErrorStream());
} else {
    streamReader = new InputStreamReader(con.getInputStream());
}
```

最后，我们可以像上一节一样读取streamReader。

## 11. 构建完整的响应

使用HttpUrlConnection实例无法获得完整的响应表示。

但是，**我们可以使用HttpUrlConnection实例提供的一些方法来构建它**：

```java
public class FullResponseBuilder {
    public static String getFullResponse(HttpURLConnection con) throws IOException {
        StringBuilder fullResponseBuilder = new StringBuilder();

        // read status and message

        // read headers

        // read response content

        return fullResponseBuilder.toString();
    }
}
```

在这里，我们读取响应的各个部分，包括状态码、状态消息和标头，并将它们添加到StringBuilder实例中。

**首先，让我们添加响应状态信息**：

```java
fullResponseBuilder.append(con.getResponseCode())
    .append(" ")
    .append(con.getResponseMessage())
    .append("\n");
```

**接下来，我们将使用getHeaderFields()获取标头**，并将它们以HeaderName:HeaderValues格式添加到我们的StringBuilder中：

```java
con.getHeaderFields().entrySet().stream()
    .filter(entry -> entry.getKey() != null)
    .forEach(entry -> {
        fullResponseBuilder.append(entry.getKey()).append(": ");
        List headerValues = entry.getValue();
        Iterator it = headerValues.iterator();
        if (it.hasNext()) {
            fullResponseBuilder.append(it.next());
            while (it.hasNext()) {
                fullResponseBuilder.append(", ").append(it.next());
            }
        }
        fullResponseBuilder.append("\n");
});
```

**最后，我们将像之前一样读取响应内容并将其附加**。

请注意，getFullResponse方法将验证请求是否成功，以决定是否需要使用con.getInputStream()或con.getErrorStream()来检索请求的内容。

## 12. 总结

在本文中，我们展示了如何使用HttpUrlConnection类执行HTTP请求。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-2)上获得。
