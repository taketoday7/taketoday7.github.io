---
layout: post
title:  Java - 将数据附加到文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个快速教程中，我们将了解如何使用Java以几种简单的方式将数据附加到文件的内容中。

让我们从如何使用核心Java的FileWriter来做到这一点开始。

## 2. 使用FileWriter

这是一个简单的测试-读取现有文件，附加一些文本，然后确保正确附加：

```java
@Test
public void whenAppendToFileUsingFileWriter_thenCorrect() throws IOException {
 
    FileWriter fw = new FileWriter(fileName, true);
    BufferedWriter bw = new BufferedWriter(fw);
    bw.write("Spain");
    bw.newLine();
    bw.close();
    
    assertThat(getStringFromInputStream(new FileInputStream(fileName))).isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

请注意，如果我们想将数据附加到现有文件，FileWriter的构造函数接收布尔标记。

**如果我们将其设置为false，则现有内容将被替换**。

## 3. 使用FileOutputStream

接下来，让我们看看如何使用FileOutputStream执行相同的操作：

```java
@Test
public void whenAppendToFileUsingFileOutputStream_thenCorrect() throws Exception {
    FileOutputStream fos = new FileOutputStream(fileName, true);
    fos.write("Spain\r\n".getBytes());
    fos.close();
    
    assertThat(StreamUtils.getStringFromInputStream(new FileInputStream(fileName))).isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

同样，FileOutputStream构造函数接收一个布尔值，该值应设置为true以标记我们要将数据附加到现有文件。

## 4. 使用java.nio.file

接下来，我们还可以使用java.nio.file中的功能将内容附加到文件-这是在JDK 7中引入的：

```java
@Test
public void whenAppendToFileUsingFiles_thenCorrect() throws IOException {
    String contentToAppend = "Spain\r\n";
    Files.write(
        Paths.get(fileName), 
        contentToAppend.getBytes(), 
        StandardOpenOption.APPEND);
    
    assertThat(StreamUtils.getStringFromInputStream(new FileInputStream(fileName))).isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

## 5. 使用Guava

要开始使用Guava，我们需要将其依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

现在，让我们看看如何使用Guava将内容附加到现有文件：

```java
@Test
public void whenAppendToFileUsingFileWriter_thenCorrect() throws IOException {
    File file = new File(fileName);
    CharSink chs = Files.asCharSink(file, Charsets.UTF_8, FileWriteMode.APPEND);
    chs.write("Spain\r\n");
	
    assertThat(StreamUtils.getStringFromInputStream(new FileInputStream(fileName))).isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

## 6. 使用Apache Commons IO FileUtils

最后，让我们看看如何使用Apache Commons IO FileUtils将内容附加到现有文件。

首先，让我们将Apache Commons IO依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

现在，让我们看一个快速示例，演示如何使用FileUtils将内容附加到现有文件：

```java
@Test
public void whenAppendToFileUsingFiles_thenCorrect() throws IOException {
    File file = new File(fileName);
    FileUtils.writeStringToFile(file, "Spain\r\n", StandardCharsets.UTF_8, true);
    
    assertThat(StreamUtils.getStringFromInputStream(new FileInputStream(fileName))).isEqualTo("UK\r\n" + "US\r\n" + "Germany\r\n" + "Spain\r\n");
}
```

## 7. 总结

在本文中，我们了解了如何以多种方式追加文件内容。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-2)上获得。
