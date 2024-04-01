---
layout: post
title:  如何使用Java读取Zip文件条目
category: java-new
copyright: java-new
excerpt: Java 20
---

## 1. 概述

Zip文件广泛用于将多个文件[压缩](https://www.baeldung.com/java-compress-and-uncompress)和归档到一个文件中，以编程方式从zip文件中提取和处理单个条目在各种情况下都非常有用。

在这个简短的教程中，我们将探讨如何使用Java读取zip文件条目。

## 2. 解决方案

我们可以使用java.util.zip包中的ZipFile和ZipEntry类轻松读取zip文件的条目：

```java
String zipFilePath = "path/to/our/zip/file.zip";

try (ZipFile zipFile = new ZipFile(zipFilePath)) {
    Enumeration<? extends ZipEntry> entries = zipFile.entries();
    while (entries.hasMoreElements()) {
        ZipEntry entry = entries.nextElement();
        // Check if entry is a directory
        if (!entry.isDirectory()) {
            try (InputStream inputStream = zipFile.getInputStream(entry)) {
                // Read and process the entry contents using the inputStream
            }
        }
    }
}
```

让我们详细介绍一下这些步骤：

- 首先，我们可以创建一个代表zip文件的ZipFile对象，该对象将提供对文件内条目的访问。
- 一旦我们有了ZipFile对象，我们就可以使用entries()方法迭代它的条目。每个条目代表一个文件或目录。
- 对于每个条目，我们可以访问各种属性，例如名称、大小、修改时间等。让我们使用isDirectory()方法来检查它是否是一个目录。
- 要读取特定条目的内容，我们可以使用getInputStream()方法返回的InputStream，这允许我们访问条目数据的字节。
- 最后，我们使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)，这样我们就不必担心手动关闭ZipFile和InputStream。

## 3. 示例

让我们使用具有以下结构的zip文件来测试我们的解决方案：

```plaintext
fileA.txt
folder1/
folder1/fileB.txt
```

让我们更改上面的代码以读取zip文件中文本文件的内容：

```java
try (InputStream inputStream = zipFile.getInputStream(entry);
     Scanner scanner = new Scanner(inputStream);) {

    while (scanner.hasNextLine()) {
        String line = scanner.nextLine();
        System.out.println(line);
    }
}
```

输出将如下所示：

```text
this is the content in fileA
this is the content in fileB
```

## 4. 总结

在本教程中，我们学习了如何使用Java读取zip文件条目。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-20)上找到。