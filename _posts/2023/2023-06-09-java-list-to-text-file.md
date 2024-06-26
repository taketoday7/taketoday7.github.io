---
layout: post
title:  将字符串列表写入文本文件
category: java-new
copyright: java-new
excerpt: Java 11
---

## 1. 概述

在本快速教程中，我们将以不同的方式将字符串列表写入Java文本文件。首先，我们将讨论FileWriter，然后是BufferedWriter，最后是Files.writeString。

## 2. 使用FileWriter

java.io包包含一个FileWriter类，我们可以使用它来将字符数据写入文件。如果我们查看层次结构，我们会看到FileWriter类扩展了OutputStreamWriter类，而后者又扩展了Writer类。

让我们看一下可用于初始化FileWriter的构造函数：

```java
FileWriter f = new FileWriter(File file);
FileWriter f = new FileWriter(File file, boolean append);
FileWriter f = new FileWriter(FileDescriptor fd);
FileWriter f = new FileWriter(File file, Charset charset);
FileWriter f = new FileWriter(File file, Charset charset, boolean append);
FileWriter f = new FileWriter(String fileName);
FileWriter f = new FileWriter(String fileName, Boolean append);
FileWriter f = new FileWriter(String fileName, Charset charset);
FileWriter f = new FileWriter(String fileName, Charset charset, boolean append);
```

请注意，FileWriter类的所有构造函数都假定默认字节缓冲区大小和默认字符编码是可接受的。

现在，让我们学习如何使用FileWriter将字符串列表写入文本文件：

```java
FileWriter fileWriter = new FileWriter(TEXT_FILENAME);
for (String str : stringList) {
    fileWriter.write(str + System.lineSeparator());
}
fileWriter.close();
return TEXT_FILENAME;
```

示例中需要注意的一件重要事情是，**如果sampleTextFile.txt不存在，FileWriter会创建它**。如果文件存在，那么我们可以根据构造函数的选择覆盖它或追加它。

## 3. 使用BufferedWriter

java.io包中包含一个BufferedWriter类，可以与其他Writer一起使用，以更好地写入字符数据。但是如何更有效呢？

当我们使用BufferedWriter时，字符被写入缓冲区而不是磁盘。**当缓冲区被填满时，所有数据都会一次性写入磁盘，从而减少进出磁盘的流量**，从而有助于提高性能。

如果我们谈论层次结构，BufferedWriter类扩展了Writer类。

让我们看一下可用于初始化BufferedWriter的构造函数：

```java
BufferedWriter b = new BufferedWriter(Writer w);
BufferedWriter b = new BufferedWriter(Writer w, int size);
```

请注意，如果我们不指定缓冲区大小，它将采用默认值。

现在，让我们探索如何使用BufferedWriter将字符串列表写入文本文件：

```java
BufferedWriter br = new BufferedWriter(new FileWriter(TEXT_FILENAME));
for (String str : stringList) {
    br.write(str + System.lineSeparator());
}
br.close();
return TEXT_FILENAME;
```

在上面的代码中，我们将FileWriter与BufferedWriter包装在一起，从而减少了读取调用，进而提高了性能。

## 4. 使用Files.writeString

java.nio包包含来自Files类的writeString()方法，用于将字符写入文件。此方法是在Java 11中引入的。

让我们看一下java.nio.file.Files中可用的两个重载方法：

```java
public static Path writeString(Path path, CharSequence csq, OpenOption... options) throws IOException
public static Path writeString(Path path, CharSequence csq, Charset cs, OpenOption... options) throws IOException
```

请注意，在第一种方法中，我们不必指定Charset。默认情况下，它采用UTF-8 Charset，而在第二种方法中，我们可以指定Charset。最后，让我们学习如何使用Files.writeString将字符串列表写入文本文件：

```java
Path filePath = Paths.get(TEXT_FILENAME);
Files.deleteIfExists(filePath);
Files.createFile(filePath);
for (String str : stringList) {
    Files.writeString(filePath, str + System.lineSeparator(),
    StandardOpenOption.APPEND);
}
return filePath.toString();
```

在上面的示例中，如果文件已经存在，我们将其删除，然后使用[Files.createFile](https://www.baeldung.com/java-how-to-create-a-file)方法创建一个文件。请注意，我们已将OpenOption字段设置为StandardOperation.APPEND，这意味着文件将以追加模式打开。如果我们不指定OpenOption，那么我们可以预期会发生两件事：

-   如果文件已经存在，它将被覆盖
-   File.writeString将创建一个新文件并在文件不存在时在其上写入

## 5. 测试

对于JUnit测试，让我们定义一个字符串列表：

```java
private static final List<String> stringList = Arrays.asList("Hello", "World");
```

在这里，我们将使用NIO Files.lines和count()找出输出文件的行数。计数应等于List中的字符串数。

现在，让我们开始测试我们的FileWriter实现：

```java
@Test
public void givenUsingFileWriter_whenStringList_thenGetTextFile() throws IOException {
    String fileName = FileWriterExample.generateFileFromStringList(stringList);
    long count = Files.lines(Paths.get(fileName)).count();
    assertTrue("No. of lines in file should be equal to no. of Strings in List", ((int) count) == stringList.size());
}
```

接下来，我们将测试BufferedWriter的实现：

```java
@Test
public void givenUsingBufferedWriter_whenStringList_thenGetTextFile() throws IOException {
    String fileName = BufferedWriterExample.generateFileFromStringList(stringList);
    long count = Files.lines(Paths.get(fileName)).count();
    assertTrue("No. of lines in file should be equal to no. of Strings in List", ((int) count) == stringList.size());
}
```

最后，让我们测试一下我们的Files.writeString实现：

```java
@Test
public void givenUsingFileWriteString_whenStringList_thenGetTextFile() throws IOException {
    String fileName = FileWriteStringExample.generateFileFromStringList(stringList);
    long count = Files.lines(Paths.get(fileName)).count();
    assertTrue("No. of lines in file should be equal to no. of Strings in List", ((int) count) == stringList.size());
}
```

## 6. 总结

在本文中，我们探讨了将字符串列表写入文本文件的三种常用方法。此外，我们通过编写JUnit测试来测试我们的实现。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-11-3)上获得。