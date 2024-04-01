---
layout: post
title:  如何使用Java高效读取大文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

本教程将展示如何以有效的方式从Java中读取大文件中的所有行。

## 2. 内存中的读取

读取文件行的标准方法是在内存中-Guava 和Apache Commons IO都提供了一种快速的方法来做到这一点：

```java
Files.readLines(new File(path), Charsets.UTF_8);
FileUtils.readLines(new File(path));
```

这种方法的问题是所有文件行都保存在内存中-如果文件足够大，这将很快导致OutOfMemoryError。

例如**读取一个大约1Gb文件**：

```java
@Test
public void givenUsingGuava_whenIteratingAFile_thenWorks() throws IOException {
    String path = ...
    Files.readLines(new File(path), Charsets.UTF_8);
}
```

这从消耗少量内存开始：(消耗约0Mb)

```text
[main] INFO  cn.tuyucheng.taketoday.java.CoreJavaIoUnitTest - Total Memory: 128 Mb
[main] INFO  cn.tuyucheng.taketoday.java.CoreJavaIoUnitTest - Free Memory: 116 Mb
```

但是，**在处理完整个文件后**，我们最后得到了：(消耗了约2Gb)

```text
[main] INFO  cn.tuyucheng.taketoday.java.CoreJavaIoUnitTest - Total Memory: 2666 Mb
[main] INFO  cn.tuyucheng.taketoday.java.CoreJavaIoUnitTest - Free Memory: 490 Mb
```

这意味着该进程消耗了大约2.1Gb的内存，原因很简单-**文件的行现在都存储在内存中**。

很明显，将文件内容保存在内存中会很快耗尽可用内存，不管实际有多少内存。

更重要的是，**我们通常不需要一次将文件中的所有行都存储在内存中**-相反，我们只需要能够遍历每一行，进行一些处理并将其丢弃。所以，这正是我们要做的-遍历这些行，而不是将它们全部保存在内存中。

## 3. 流式传输文件

现在让我们看一个解决方案-我们将使用java.util.Scanner来遍历文件的内容并逐行检索行：

```java
FileInputStream inputStream = null;
Scanner sc = null;
try {
    inputStream = new FileInputStream(path);
    sc = new Scanner(inputStream, "UTF-8");
    while (sc.hasNextLine()) {
        String line = sc.nextLine();
        // System.out.println(line);
    }
    // note that Scanner suppresses exceptions
    if (sc.ioException() != null) {
        throw sc.ioException();
    }
} finally {
    if (inputStream != null) {
        inputStream.close();
    }
    if (sc != null) {
        sc.close();
    }
}
```

该解决方案将遍历文件中的所有行，允许处理每一行-不保留对它们的引用，最后，**不将它们保留在内存中**：(消耗约150Mb)

```bash
[main] INFO  cn.tuyucheng.taketoday.java.CoreJavaIoUnitTest - Total Memory: 763 Mb
[main] INFO  cn.tuyucheng.taketoday.java.CoreJavaIoUnitTest - Free Memory: 605 Mb
```

## 4. 使用 Apache Commons IO进行流式处理

使用Commons IO库也可以通过使用库提供的自定义LineIterator来实现相同的目的：

```java
LineIterator it = FileUtils.lineIterator(theFile, "UTF-8");
try {
    while (it.hasNext()) {
        String line = it.nextLine();
        // do something with line
    }
} finally {
    LineIterator.closeQuietly(it);
}
```

由于整个文件没有完全在内存中，这也会导致相当保守的内存消耗数字：(消耗约150Mb)

```bash
[main] INFO  c.t.t.java.CoreJavaIoIntegrationTest - Total Memory: 752 Mb
[main] INFO  c.t.t.java.CoreJavaIoIntegrationTest - Free Memory: 564 Mb
```

## 5. 总结

这篇简短的文章展示了如何在不反复、不耗尽可用内存的情况下处理大文件中的行-事实证明，这在处理这些大文件时非常有用。
