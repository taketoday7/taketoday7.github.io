---
layout: post
title:  使用Java查找文件中的行数
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在本教程中，我们将学习如何在标准Java IO API、[Google Guava](https://www.baeldung.com/category/guava/)和[Apache Commons IO](https://www.baeldung.com/apache-commons-io)库的帮助下使用Java查找文件中的行数。

## 2. NIO2 Files

请注意，在本教程中，我们将使用以下示例值作为输入文件名和总行数：

```java
static final String INPUT_FILE_NAME = "src/main/resources/input.txt";
static final int NO_OF_LINES = 45;
```

Java 7对现有的IO库进行了许多改进，并将其封装在[NIO2下](https://www.baeldung.com/java-nio-2-file-api)：

让我们从Files开始，看看我们如何使用它的API来计算行数：

```java
@Test
public void whenUsingNIOFiles_thenReturnTotalNumberOfLines() throws IOException {
    try (Stream<String> fileStream = Files.lines(Paths.get(INPUT_FILE_NAME))) {
        int noOfLines = (int) fileStream.count();
        assertEquals(NO_OF_LINES, noOfLines);
    }
}
```

或者简单地使用Files#readAllLines方法：

```java
@Test
public void whenUsingNIOFilesReadAllLines_thenReturnTotalNumberOfLines() throws IOException {
    List<String> fileStream = Files.readAllLines(Paths.get(INPUT_FILE_NAME));
    int noOfLines = fileStream.size();
    assertEquals(NO_OF_LINES, noOfLines);
}
```

## 3. NIO FileChannel

现在让我们看看FileChannel，这是一种用于读取行数的高性能Java NIO替代方案：

```java
@Test
public void whenUsingNIOFileChannel_thenReturnTotalNumberOfLines() throws IOException {
    int noOfLines = 1;
    try (FileChannel channel = FileChannel.open(Paths.get(INPUT_FILE_NAME), StandardOpenOption.READ)) {
        ByteBuffer byteBuffer = channel.map(MapMode.READ_ONLY, 0, channel.size());
        while (byteBuffer.hasRemaining()) {
            byte currentByte = byteBuffer.get();
            if (currentByte == '\n')
                noOfLines++;
       }
    }
    assertEquals(NO_OF_LINES, noOfLines);
}
```

虽然FileChannel是在JDK 4中引入的，但**上述解决方案仅适用于JDK 7或更高版本**。

## 4. Google Guava Files

另一种第三方库是Google Guava Files类。这个类也可以用来计算总行数，类似于我们在Files#readAllLines中看到的。

让我们首先在pom.xml中添加[guava](https://mvnrepository.com/artifact/com.google.guava/guava)依赖项：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

然后我们可以使用readLines获取文件行列表：

```java
@Test
public void whenUsingGoogleGuava_thenReturnTotalNumberOfLines() throws IOException {
    List<String> lineItems = Files.readLines(Paths.get(INPUT_FILE_NAME)
        .toFile(), Charset.defaultCharset());
    int noOfLines = lineItems.size();
    assertEquals(NO_OF_LINES, noOfLines);
}
```

## 5. Apache Commons IO FileUtils

现在，让我们看看[Apache Commons IO](https://www.baeldung.com/apache-commons-io)FileUtils API，它是Guava的并行解决方案。

要使用该库，我们必须在pom.xml中包含[commons-io](https://mvnrepository.com/artifact/commons-io/commons-io)依赖项：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

现在，我们可以使用Apache Commons IO的FileUtils#lineIterator，它会为我们清理一些文件处理：

```java
@Test
public void whenUsingApacheCommonsIO_thenReturnTotalNumberOfLines() throws IOException {
    int noOfLines = 0;
    LineIterator lineIterator = FileUtils.lineIterator(new File(INPUT_FILE_NAME));
    while (lineIterator.hasNext()) {
        lineIterator.nextLine();
        noOfLines++;
    }
    assertEquals(NO_OF_LINES, noOfLines);
}
```

正如我们所看到的，这比Google Guava解决方案更冗长。

## 6. BufferedReader

那么，老派的方法呢？如果我们不在JDK 7上并且我们不能使用第三方库，我们有[BufferedReader](https://www.baeldung.com/java-buffered-reader)：

```java
@Test
public void whenUsingBufferedReader_thenReturnTotalNumberOfLines() throws IOException {
    int noOfLines = 0;
    try (BufferedReader reader = new BufferedReader(new FileReader(INPUT_FILE_NAME))) {
        while (reader.readLine() != null) {
            noOfLines++;
        }
    }
    assertEquals(NO_OF_LINES, noOfLines);
}
```

## 7. LineNumberReader

或者，我们可以使用LineNumberReader，它是[BufferedReader](https://www.baeldung.com/java-buffered-reader)的直接子类，只是稍微简洁一点：

```java
@Test
public void whenUsingLineNumberReader_thenReturnTotalNumberOfLines() throws IOException {
    try (LineNumberReader reader = new LineNumberReader(new FileReader(INPUT_FILE_NAME))) {
        reader.skip(Integer.MAX_VALUE);
        int noOfLines = reader.getLineNumber() + 1;
        assertEquals(NO_OF_LINES, noOfLines);
    }
}
```

在这里，我们**调用skip方法转到文件末尾**，并且我们将自行编号从0开始以来计数的总行数加1。

## 8. Scanner

最后，如果我们已经将[Scanner](https://www.baeldung.com/java-scanner)作为更大解决方案的一部分使用，它也可以为我们解决问题：

```java
@Test
public void whenUsingScanner_thenReturnTotalNumberOfLines() throws IOException {
    try (Scanner scanner = new Scanner(new FileReader(INPUT_FILE_NAME))) {
        int noOfLines = 0;
        while (scanner.hasNextLine()) {
            scanner.nextLine();
            noOfLines++;
        }
        assertEquals(NO_OF_LINES, noOfLines);
    }
}
```

## 9. 总结

在本教程中，我们探索了使用Java获取文件行数的不同方法。由于所有这些API的主要目的不是计算文件中的行数，因此建议根据我们的需要选择正确的解决方案。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-1)上获得。
