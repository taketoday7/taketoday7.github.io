---
layout: post
title:  java.net.UnknownHostException：服务器的主机名无效
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将通过示例了解UnknownHostException的原因。我们还将讨论防止和处理异常的可能方法。

## 2. 什么时候抛出异常？

**UnknownHostException表示无法确定主机名的IP地址，这可能是由于主机名中的拼写错误而发生的**：

```java
String hostname = "http://locaihost";
URL url = new URL(hostname);
HttpURLConnection con = (HttpURLConnection) url.openConnection();
con.getResponseCode();
```

上面的代码抛出UnknownHostException，因为拼写错误的locaihost没有指向任何IP地址。

**UnknownHostException的另一个可能原因是DNS传播延迟或DNS配置错误**。

新的DNS条目最多可能需要48小时才能传播到整个互联网。

## 3. 如何预防？

首先防止异常发生比事后处理它更好。一些防止异常的技巧是：

1.  **仔细检查主机名**：确保没有拼写错误，并删除所有空格。
2.  **检查系统的DNS设置**：确保DNS服务器已启动且可访问，如果主机名是新的，请等待DNS服务器赶上。

## 4. 如何处理？

UnknownHostException扩展了IOException，这是一个[受检的异常](https://www.baeldung.com/java-checked-unchecked-exceptions#checked)。与任何其他受检的异常类似，我们必须抛出它或用try-catch块包围它。

让我们处理示例中的异常：

```java
try {
    con.getResponseCode();
} catch (UnknownHostException e) {
    con.disconnect();
}
```

**发生UnknownHostException时关闭连接是一个好习惯**。大量浪费性的打开连接会导致应用程序内存不足。

## 5. 总结

在本文中，我们了解了导致UnknownHostException的原因、如何防止它以及如何处理它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-2)上获得。
