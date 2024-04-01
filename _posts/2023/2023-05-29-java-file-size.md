---
layout: post
title:  Java中的文件大小
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个快速教程中，我们将学习如何**使用Java获取文件的大小**-使用Java 7、新的Java 8和Apache Common IO。

最后，我们还将获得文件大小的人类可读表示。

## 2. Java IO标准

让我们从一个计算文件大小的简单示例开始-使用[File.length()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#length())方法：

```java
private long getFileSize(File file) {
    long length = file.length();
    return length;
}
```

我们可以相对简单地测试我们的实现：

```java
@Test
public void whenGetFileSize_thenCorrect() {
    long expectedSize = 12607;
 
    File imageFile = new File("src/test/resources/image.jpg");
    long size = getFileSize(imageFile);
 
    assertEquals(expectedSize, size);
}
```

请注意，默认情况下，文件大小以字节为单位计算。

## 3. Java NIO

接下来，让我们看看如何使用NIO库来获取文件的大小。

在下面的示例中，我们将使用[FileChannel.size()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/FileChannel.html#size()) API来获取文件的大小(以字节为单位)：

```java
@Test
public void whenGetFileSizeUsingNioApi_thenCorrect() throws IOException {
    long expectedSize = 12607;
 
    Path imageFilePath = Paths.get("src/test/resources/image.jpg");
    FileChannel imageFileChannel = FileChannel.open(imageFilePath);

    long imageFileSize = imageFileChannel.size();
    assertEquals(expectedSize, imageFileSize);
}
```

## 4. 使用Apache Commons IO

接下来，让我们看看如何使用Apache Commons IO获取文件大小。在下面的例子中，我们简单地使用[FileUtils.sizeOf()](https://commons.apache.org/proper/commons-io/javadocs/api-2.5/org/apache/commons/io/FileUtils.html#sizeOf(java.io.File))来获取文件大小：

```java
@Test
public void whenGetFileSizeUsingApacheCommonsIO_thenCorrect() {
    long expectedSize = 12607;
 
    File imageFile = new File("src/test/resources/image.jpg");
    long size = FileUtils.sizeOf(imageFile);
 
    assertEquals(expectedSize, size);
}
```

请注意，对于受安全限制的文件，FileUtils.sizeOf()会将大小报告为0。

## 5. 人类可读大小

最后，让我们看看如何使用Apache Commons IO获得更可读的文件大小表示形式-而不仅仅是字节大小：

```java
@Test
public void whenGetReadableFileSize_thenCorrect() {
    File imageFile = new File("src/test/resources/image.jpg");
    long size = getFileSize(imageFile);
 
    assertEquals("12 KB", FileUtils.byteCountToDisplaySize(size));
}
```

## 6. 总结

在本教程中，我们举例说明了使用Java和Apache Commons IO来计算文件系统中文件的大小。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-1)上获得。
