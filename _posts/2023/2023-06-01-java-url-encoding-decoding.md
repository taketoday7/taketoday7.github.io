---
layout: post
title:  Java URL编码/解码指南
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

简而言之，[URL编码](https://en.wikipedia.org/wiki/Percent-encoding)将URL中的特殊字符转换为符合规范且可以被正确理解和解释的表示形式。

在本教程中，我们将重点介绍**如何对URL或表单数据进行编码/解码**，以使其符合规范并通过网络正确传输。

## 2. 分析URL

让我们先来看一个基本的[URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)语法：

```text
scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]
```

对URI进行编码的第一步是检查其组件，然后仅对相关部分进行编码。

现在让我们看一个URI的例子：

```java
String testUrl = "http://www.tuyucheng.com?key1=value+1&key2=value%40%21%242&key3=value%253";
```

解析URI的一种方法是将String表示形式加载到java.net.URI类：

```java
@Test
public void givenURL_whenAnalyze_thenCorrect() throws Exception {
    URI uri = new URI(testUrl);

    assertThat(uri.getScheme(), is("http"));
    assertThat(uri.getHost(), is("www.tuyucheng.com"));
    assertThat(uri.getRawQuery(),
        .is("key1=value+1&key2=value%40%21%242&key3=value%253"));
}
```

URI类解析字符串表示URL并通过简单的API(例如getXXX)公开其组件。

## 3. 对URL进行编码

编码URI时，常见的陷阱之一是对完整的URI进行编码。通常，我们只需要对URI的查询部分进行编码。

让我们使用URLEncoder类的encode(data, encodingScheme)方法对数据进行编码：

```java
private String encodeValue(String value) {
    return URLEncoder.encode(value, StandardCharsets.UTF_8.toString());
}

@Test
public void givenRequestParam_whenUTF8Scheme_thenEncode() throws Exception {
    Map<String, String> requestParams = new HashMap<>();
    requestParams.put("key1", "value 1");
    requestParams.put("key2", "value@!$2");
    requestParams.put("key3", "value%3");

    String encodedURL = requestParams.keySet().stream()
        .map(key -> key + "=" + encodeValue(requestParams.get(key)))
        .collect(joining("&", "http://www.tuyucheng.com?", ""));

    assertThat(testUrl, is(encodedURL));
```

encode方法接收两个参数：

1.  data：要翻译的字符串
2.  encodingScheme：字符编码的名称

此encode方法将字符串转换为[application/x-www-form-urlencoded](https://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.1)格式。

编码方案将特殊字符转换为八位的两位十六进制表示，将以“%xy”的形式表示。当我们处理路径参数或添加动态参数时，我们会将数据进行编码然后发送到服务器。

> **注意**：**万维网联盟建议指出我们应该使用UTF-8**，不这样做可能会导致不兼容。(参考：[https://docs.oracle.com/javase/7/docs/api/java/net/URLEncoder.html](https://docs.oracle.com/javase/7/docs/api/java/net/URLEncoder.html))

## 4. 解码URL

现在让我们使用URLDecoder的decode方法解码之前的URL：

```java
private String decode(String value) {
    return URLDecoder.decode(value, StandardCharsets.UTF_8.toString());
}

@Test
public void givenRequestParam_whenUTF8Scheme_thenDecodeRequestParams() {
    URI uri = new URI(testUrl);

    String scheme = uri.getScheme();
    String host = uri.getHost();
    String query = uri.getRawQuery();

    String decodedQuery = Arrays.stream(query.split("&"))
        .map(param -> param.split("=")[0] + "=" + decode(param.split("=")[1]))
        .collect(Collectors.joining("&"));

    assertEquals(
        "http://www.tuyucheng.com?key1=value 1&key2=value@!$2&key3=value%3",
        scheme + "://" + host + "?" + decodedQuery);
}
```

这里有两点要记住：

-   解码前解析URL
-   使用相同的编码方案进行编码和解码

如果我们先解码再解析，URL部分可能无法正确解析。如果我们使用另一种编码方案对数据进行解码，则会产生垃圾数据。

## 5. 编码路径段

我们不能使用URLEncoder对URL的路径段进行编码。路径组件是指表示目录路径的层次结构，或者用于定位以“/”分隔的资源。

路径段中的保留字符与查询参数值中的保留字符不同。例如，“+”号是路径段中的有效字符，因此不应进行编码。

为了对路径段进行编码，我们改用Spring Framework的UriUtils类。

UriUtils类提供了encodePath和encodePathSegment方法，分别对路径和路径段进行编码：

```java
private String encodePath(String path) {
    try {
        path = UriUtils.encodePath(path, "UTF-8");
    } catch (UnsupportedEncodingException e) {
        LOGGER.error("Error encoding parameter {}", e.getMessage(), e);
    }
    return path;
}
```

```java
@Test
public void givenPathSegment_thenEncodeDecode() throws UnsupportedEncodingException {
    String pathSegment = "/Path 1/Path+2";
    String encodedPathSegment = encodePath(pathSegment);
    String decodedPathSegment = UriUtils.decode(encodedPathSegment, "UTF-8");
    
    assertEquals("/Path%201/Path+2", encodedPathSegment);
    assertEquals("/Path 1/Path+2", decodedPathSegment);
}
```

在上面的代码片段中，我们可以看到当我们使用encodePathSegment方法时，它返回了编码后的值，而+没有编码，因为它是路径组件中的值字符。

让我们在测试URL中添加一个路径变量：

```java
String testUrl = "/path+1?key1=value+1&key2=value%40%21%242&key3=value%253";
```

为了组装和断言正确编码的URL，我们将更改第2节中的测试：

```java
String path = "path+1";
String encodedURL = requestParams.keySet().stream()
    .map(k -> k + "=" + encodeValue(requestParams.get(k)))
    .collect(joining("&", "/" + encodePath(path) + "?", ""));
assertThat(testUrl, CoreMatchers.is(encodedURL));

```

## 6. 总结

在本文中，我们了解了如何对数据进行编码和解码，以便正确传输和解释数据。

虽然本文着重于编码/解码URI查询参数值，但该方法也适用于HTML表单参数。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-1)上获得。
