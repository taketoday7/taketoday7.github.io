---
layout: post
title:  在Java中确定文件创建日期
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

JDK 7引入了获取文件创建日期的功能。

在本教程中，我们将学习如何通过java.nio访问它。

## 2. Files.getAttribute

获取文件创建日期的一种方法是**使用方法Files.getAttribute**和给定的Path：

```java
try {
    FileTime creationTime = (FileTime) Files.getAttribute(path, "creationTime");
} catch (IOException ex) {
    // handle exception
}
```

creationTime的类型是FileTime，**但是由于该方法返回的是Object，我们必须强制转换它**。

FileTime将日期值保存为时间戳属性。例如，可以使用toInstant()方法将其转换为Instant。

如果文件系统不存储文件的创建日期，则该方法将返回null。

## 3. Files.readAttributes

另一种获取创建日期的方法是**使用Files.readAttributes，对于给定的Path，它会一次返回文件的所有基本属性**：

```java
try {
    BasicFileAttributes attr = Files.readAttributes(path, BasicFileAttributes.class);
    FileTime fileTime = attr.creationTime();
} catch (IOException ex) {
    // handle exception
}
```

该方法返回一个BasicFileAttributes，我们可以使用它来获取文件的基本属性。creationTime()方法返回文件的创建日期为FileTime。

这一次，**如果文件系统不存储创建文件的日期，那么该方法将返回上次修改日期**。如果上次修改日期也未存储，则将返回纪元(01.01.1970)。

## 4. 总结

在本教程中，我们学习了如何在Java中确定文件创建日期。具体来说，我们了解到可以使用Files.getAttribute和Files.readAttributes来做到这一点。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-1)上获得。
