---
layout: post
title:  如何在Java中读取文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将探讨**在Java中读取文件**的不同方法。

首先，我们将学习如何使用标准Java类从类路径、URL或JAR文件加载文件。

其次，我们将了解如何使用BufferedReader、Scanner、StreamTokenizer、DataInputStream、SequenceInputStream和FileChannel读取内容。我们还将讨论如何读取UTF-8编码的文件。

最后，我们将探讨在Java 7和Java 8中加载和读取文件的新技术。

## 2. 设置

### 2.1 输入文件

在本文中的大多数示例中，我们将读取文件名为fileTest.txt的文本文件，其中包含一行：

```text
Hello, world!
```

对于一些示例，我们将使用不同的文件；在这些情况下，我们会明确提及文件及其内容。

### 2.2 辅助方法

我们将使用一组仅包含核心Java类的测试示例，并且在测试中，我们将使用带有[Hamcrest](https://www.baeldung.com/hamcrest-collections-arrays)匹配器的断言。

测试将共享一个通用的readFromInputStream方法，该方法将InputStream转换为String以便于断言结果：

```java
private String readFromInputStream(InputStream inputStream) throws IOException {
    StringBuilder resultStringBuilder = new StringBuilder();
    try (BufferedReader br
      = new BufferedReader(new InputStreamReader(inputStream))) {
        String line;
        while ((line = br.readLine()) != null) {
            resultStringBuilder.append(line).append("\n");
        }
    }
  return resultStringBuilder.toString();
}
```

请注意，还有其他方法可以实现相同的结果。我们可以参考[这篇文章](https://www.baeldung.com/convert-input-stream-to-a-file)了解一些替代方案。

## 3. 从类路径中读取文件

### 3.1 使用标准Java

本节介绍如何读取类路径上可用的文件。我们将读取src/main/resources下可用的“fileTest.txt”：

```java
@Test
public void givenFileNameAsAbsolutePath_whenUsingClasspath_thenFileData() {
    String expectedData = "Hello, world!";
    
    Class clazz = FileOperationsTest.class;
    InputStream inputStream = clazz.getResourceAsStream("/fileTest.txt");
    String data = readFromInputStream(inputStream);

    Assert.assertThat(data, containsString(expectedData));
}
```

**在上面的代码片段中，我们使用当前类通过getResourceAsStream方法加载文件，并传递要加载的文件的绝对路径**。

同样的方法也适用于ClassLoader实例：

```java
ClassLoader classLoader = getClass().getClassLoader();
InputStream inputStream = classLoader.getResourceAsStream("fileTest.txt");
String data = readFromInputStream(inputStream);
```

我们使用getClass().getClassLoader()获取当前类的类加载器。

主要区别在于，在ClassLoader实例上使用getResourceAsStream时，路径被视为从类路径的根开始的绝对路径。

当针对Class实例使用时，路径可以是相对于包的路径，也可以是绝对路径，由前导斜杠暗示。

**当然，请注意，在实践中，打开的流应该始终关闭**，例如我们示例中的InputStream：

```java
InputStream inputStream = null;
try {
    File file = new File(classLoader.getResource("fileTest.txt").getFile());
    inputStream = new FileInputStream(file);
    
    //...
}     
finally {
    if (inputStream != null) {
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 3.2 使用commons-io库

**另一个常见的选择是使用[commons-io](https://www.baeldung.com/apache-commons-io)包的FileUtils类**：

```java
@Test
public void givenFileName_whenUsingFileUtils_thenFileData() {
    String expectedData = "Hello, world!";
        
    ClassLoader classLoader = getClass().getClassLoader();
    File file = new File(classLoader.getResource("fileTest.txt").getFile());
    String data = FileUtils.readFileToString(file, "UTF-8");
        
    assertEquals(expectedData, data.trim());
}
```

这里我们将File对象传递给FileUtils类的readFileToString()方法。这个实用程序类设法加载内容，而无需编写任何样板代码来创建InputStream实例和读取数据。

**同一个库还提供了IOUtils类**：

```java
@Test
public void givenFileName_whenUsingIOUtils_thenFileData() {
    String expectedData = "Hello, world!";
        
    FileInputStream fis = new FileInputStream("src/test/resources/fileTest.txt");
    String data = IOUtils.toString(fis, "UTF-8");
        
    assertEquals(expectedData, data.trim());
}
```

这里我们将FileInputStream对象传递给IOUtils类的toString()方法。这个实用程序类的行为方式与前一个相同，目的是创建一个InputStream实例并读取数据。

## 4. 使用BufferedReader读取

现在让我们关注解析文件内容的不同方法。

**我们将从使用BufferedReader读取文件的简单方法开始**：

```java
@Test
public void whenReadWithBufferedReader_thenCorrect() throws IOException {
     String expected_value = "Hello, world!";
     String file ="src/test/resources/fileTest.txt";
     
     BufferedReader reader = new BufferedReader(new FileReader(file));
     String currentLine = reader.readLine();
     reader.close();

    assertEquals(expected_value, currentLine);
}
```

请注意，当到达文件末尾时，readLine()将返回null。

## 5. 使用Java NIO读取文件

在JDK 7中，NIO包进行了重大更新。

**让我们看一个使用Files类和readAllLines方法的示例**。readAllLines方法接收路径。 

Path类可以被认为是java.io.File的升级，带有一些额外的操作。

### 5.1 读取小文件

下面的代码演示如何使用新的Files类读取小文件：

```java
@Test
public void whenReadSmallFileJava7_thenCorrect() throws IOException {
    String expected_value = "Hello, world!";

    Path path = Paths.get("src/test/resources/fileTest.txt");

    String read = Files.readAllLines(path).get(0);
    assertEquals(expected_value, read);
}
```

请注意，如果我们需要二进制数据，也可以使用readAllBytes()方法。

### 5.2 读取大文件

如果我们想用Files类读取一个大文件，我们可以使用BufferedReader。

以下代码使用新的Files类和BufferedReader读取文件：

```java
@Test
public void whenReadLargeFileJava7_thenCorrect() throws IOException {
    String expected_value = "Hello, world!";

    Path path = Paths.get("src/test/resources/fileTest.txt");

    BufferedReader reader = Files.newBufferedReader(path);
    String line = reader.readLine();
    assertEquals(expected_value, line);
}
```

### 5.3 使用Files.lines()读取文件

**JDK 8在Files类中提供了lines()方法，它返回一个字符串元素流**。

让我们看一个示例，了解如何将数据读入字节并使用UTF-8字符集对其进行解码。

以下代码使用新的Files.lines()读取文件：

```java
@Test
public void givenFilePath_whenUsingFilesLines_thenFileData() {
    String expectedData = "Hello, world!";
         
    Path path = Paths.get(getClass().getClassLoader()
        .getResource("fileTest.txt").toURI());
         
    Stream<String> lines = Files.lines(path);
    String data = lines.collect(Collectors.joining("\n"));
    lines.close();
         
    Assert.assertEquals(expectedData, data.trim());
}
```

将Stream与IO通道(如文件操作)一起使用，我们需要使用close()方法显式关闭流。

正如我们所看到的，Files API提供了另一种将文件内容读入字符串的简单方法。

在接下来的部分中，我们将看看在某些情况下可能适用的其他不太常见的读取文件的方法。

## 6. 使用Scanner读取

接下来让我们使用Scanner从文件中读取。这里我们将使用空格作为分隔符：

```java
@Test
public void whenReadWithScanner_thenCorrect() throws IOException {
    String file = "src/test/resources/fileTest.txt";
    Scanner scanner = new Scanner(new File(file));
    scanner.useDelimiter(" ");

    assertTrue(scanner.hasNext());
    assertEquals("Hello,", scanner.next());
    assertEquals("world!", scanner.next());

    scanner.close();
}
```

请注意，默认分隔符是空格，但Scanner可以使用多个分隔符。

**当从控制台读取内容时，或者当内容包含具有已知分隔符的原始值(例如：以空格分隔的整数列表)时，[Scanner](https://www.baeldung.com/java-scanner)类很有用**。

## 7. 使用StreamTokenizer读取

现在让我们使用[StreamTokenizer](https://www.baeldung.com/java-streamtokenizer)将文本文件读入令牌。

tokenizer的工作原理是首先弄清楚下一个令牌是什么，字符串还是数字。我们通过查看tokenizer.ttype字段来做到这一点。

然后我们将读取基于此类型的实际令牌：

-   tokenizer.nval：如果类型是数字
-   tokenizer.sval：如果类型是字符串

在这个例子中，我们将使用一个不同的输入文件，它只包含：

```text
Hello 1
```

以下代码从文件中读取字符串和数字：

```java
@Test
public void whenReadWithStreamTokenizer_thenCorrectTokens() throws IOException {
    String file = "src/test/resources/fileTestTokenizer.txt";
    FileReader reader = new FileReader(file);
    StreamTokenizer tokenizer = new StreamTokenizer(reader);

    // token 1
    tokenizer.nextToken();
    assertEquals(StreamTokenizer.TT_WORD, tokenizer.ttype);
    assertEquals("Hello", tokenizer.sval);

    // token 2    
    tokenizer.nextToken();
    assertEquals(StreamTokenizer.TT_NUMBER, tokenizer.ttype);
    assertEquals(1, tokenizer.nval, 0.0000001);

    // token 3
    tokenizer.nextToken();
    assertEquals(StreamTokenizer.TT_EOF, tokenizer.ttype);
    reader.close();
}
```

请注意文件结尾令牌是如何在末尾使用的。

**这种方法对于将输入流解析为令牌很有用**。

## 8. 使用DataInputStream读取

**我们可以使用DataInputStream从文件中读取二进制或原始数据类型**。

以下测试使用DataInputStream读取文件：

```java
@Test
public void whenReadWithDataInputStream_thenCorrect() throws IOException {
    String expectedValue = "Hello, world!";
    String file ="src/test/resources/fileTest.txt";
    String result = null;

    DataInputStream reader = new DataInputStream(new FileInputStream(file));
    int nBytesToRead = reader.available();
    if(nBytesToRead > 0) {
        byte[] bytes = new byte[nBytesToRead];
        reader.read(bytes);
        result = new String(bytes);
    }

    assertEquals(expectedValue, result);
}
```

## 9. 使用FileChannel读取

**如果我们正在读取一个大文件，FileChannel可以比标准IO更快**。

以下代码使用FileChannel和RandomAccessFile从文件中读取数据字节：

```java
@Test
public void whenReadWithFileChannel_thenCorrect() throws IOException {
    String expected_value = "Hello, world!";
    String file = "src/test/resources/fileTest.txt";
    RandomAccessFile reader = new RandomAccessFile(file, "r");
    FileChannel channel = reader.getChannel();

    int bufferSize = 1024;
    if (bufferSize > channel.size()) {
        bufferSize = (int) channel.size();
    }
    ByteBuffer buff = ByteBuffer.allocate(bufferSize);
    channel.read(buff);
    buff.flip();
    
    assertEquals(expected_value, new String(buff.array()));
    channel.close();
    reader.close();
}
```

## 10. 读取UTF-8编码文件

现在让我们看看如何使用BufferedReader读取UTF-8编码的文件。在这个例子中，我们将读取一个包含中文字符的文件：

```java
@Test
public void whenReadUTFEncodedFile_thenCorrect() throws IOException {
    String expected_value = "青空";
    String file = "src/test/resources/fileTestUtf8.txt";
    BufferedReader reader = new BufferedReader (new InputStreamReader(new FileInputStream(file), "UTF-8"));
    String currentLine = reader.readLine();
    reader.close();

    assertEquals(expected_value, currentLine);
}
```

## 11. 从URL读取内容

要从URL中读取内容，我们将在示例中使用“/” URL：

```java
@Test
public void givenURLName_whenUsingURL_thenFileData() {
    String expectedData = "Tuyucheng";

    URL urlObject = new URL("/");
    URLConnection urlConnection = urlObject.openConnection();
    InputStream inputStream = urlConnection.getInputStream();
    String data = readFromInputStream(inputStream);

    Assert.assertThat(data, containsString(expectedData));
}
```

还有其他连接到URL的方法。这里我们使用了标准SDK中可用的URL和URLConnection类。

## 12. 从JAR中读取文件

要读取位于JAR文件中的文件，我们需要一个包含文件的JAR。对于我们的示例，我们将从“hamcrest-library-1.3.jar”文件中读取“LICENSE.txt”：

```java
@Test
public void givenFileName_whenUsingJarFile_thenFileData() {
    String expectedData = "BSD License";

    Class clazz = Matchers.class;
    InputStream inputStream = clazz.getResourceAsStream("/LICENSE.txt");
    String data = readFromInputStream(inputStream);

    Assert.assertThat(data, containsString(expectedData));
}
```

这里我们要加载驻留在Hamcrest库中的LICENSE.txt，因此我们将使用有助于获取资源的Matcher类。也可以使用类加载器加载相同的文件。

## 13. 总结

如我们所见，使用纯Java加载文件和从中读取数据有很多可能性。

我们可以从不同的位置加载文件，如类路径、URL或jar文件。

然后我们可以使用BufferedReader逐行读取，Scanner使用不同的分隔符读取，StreamTokenizer将文件读入令牌，DataInputStream读取二进制数据和原始数据类型，SequenceInput Stream将多个文件链接成一个流，FileChannel更快地从大文件中读取等。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-1)上获得。
