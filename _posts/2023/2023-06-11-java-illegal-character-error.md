---
layout: post
title:  非法字符编译错误
category: java
copyright: java
excerpt: Java
---

## 1. 概述

非法字符编译错误是文件类型编码错误。如果我们在创建文件时使用了不正确的编码，就会产生它。因此，在像Java这样的语言中，当我们尝试编译项目时，我们可能会遇到这种类型的错误。在本教程中，我们将详细描述该问题以及我们可能会遇到的一些场景，然后我们将提供一些解决该问题的示例。

## 2. 非法字符编译错误

### 2.1 字节顺序标记(BOM)

在进入字节顺序标记之前，我们需要快速浏览一下UCS(Unicode)转换格式(UTF)。**UTF是一种字符编码格式，可以对Unicode中所有可能的字符代码点进行编码**。有几种UTF编码，其中，UTF-8使用最多。

UTF-8使用8位可变宽度编码来最大化与ASCII的兼容性。当我们在我们的文件中使用这种编码时，我们可能会发现一些表示Unicode代码点的字节。因此，我们的文件以U+FEFF字节顺序标记(BOM)开头。如果使用得当，这个标记是不可见的。但是，在某些情况下，它可能会导致数据错误。

**在UTF-8编码中，BOM的存在并不重要**。尽管这不是必需的，但BOM仍可能以UTF-8编码的文本形式出现。BOM添加可以通过编码转换或通过将内容标记为UTF-8的文本编辑器进行。

Windows上的记事本等文本编辑器可以产生这种添加。因此，当我们使用类似记事本的文本编辑器创建代码示例并尝试运行它时，我们可能会遇到编译错误。相比之下，现代IDE将创建的文件编码为没有BOM的UTF-8。接下来的部分将展示此问题的一些示例。

### 2.2 存在非法字符编译错误的类

通常，我们使用高级IDE，但有时，我们使用文本编辑器。不幸的是，正如我们所知，一些文本编辑器可能会产生比解决方案更多的问题，因为保存带有BOM的文件可能会导致Java中的编译错误。**“非法字符”错误发生在编译阶段，因此很容易检测到**。下一个示例向我们展示了它是如何工作的。

首先，让我们在文本编辑器(例如记事本)中编写一个简单的类。这个类只是一个表示-我们可以编写任何代码来测试。接下来，我们将我们的文件与BOM一起保存以进行测试：

```java
public class TestBOM {
    public static void main(String... args) {
        System.out.println("BOM Test");
    }
}
```

现在，当我们尝试使用javac命令编译这个文件时：

```shell
$ javac ./TestBOM.java
```

因此，我们收到错误消息：

```shell
∩╗┐public class TestBOM {
 ^
.\TestBOM.java:1: error: illegal character: '\u00bf'
∩╗┐public class TestBOM {
  ^
2 errors
```

理想情况下，要解决此问题，唯一要做的就是将文件另存为不带BOM编码的UTF-8。之后，问题就解决了。**我们应该始终检查我们的文件是否在没有BOM的情况下保存**。

**解决此问题的另一种方法是使用类似[dos2unix](https://linux.die.net/man/1/dos2unix)的工具**，此工具将删除BOM并处理Windows文本文件的其他特性。

## 3. 读取文件

此外，让我们分析一些读取使用BOM编码的文件的示例。

最初，我们需要创建一个包含BOM的文件以用于我们的测试。该文件包含我们的示例文本“Hello world with BOM”，这将是我们预期的字符串。接下来，让我们开始测试。

### 3.1 使用BufferedReader读取文件

首先，我们将使用BufferedReader类测试文件：

```java
@Test
public void whenInputFileHasBOM_thenUseInputStream() throws IOException {
    String line;
    String actual = "";
    try (BufferedReader br = new BufferedReader(new InputStreamReader(file))) {
        while ((line = br.readLine()) != null) {
            actual += line;
        }
    }
    assertEquals(expected, actual);
}
```

在这种情况下，当我们尝试断言字符串相等时，我们会得到一个错误：

```text
org.opentest4j.AssertionFailedError: expected: <Hello world with BOM.> but was: <Hello world with BOM.>
Expected :Hello world with BOM.
Actual   :Hello world with BOM.
```

实际上，如果我们浏览测试响应，两个字符串看起来显然是相等的。即便如此，字符串的实际值仍包含BOM。因此，字符串不相等。

此外，**一个快速解决方法是替换BOM字符**：

```java
@Test
public void whenInputFileHasBOM_thenUseInputStreamWithReplace() throws IOException {
    String line;
    String actual = "";
    try (BufferedReader br = new BufferedReader(new InputStreamReader(file))) {
        while ((line = br.readLine()) != null) {
            actual += line.replace("\uFEFF", "");
        }
    }
    assertEquals(expected, actual);
}
```

replace方法从字符串中清除BOM，因此我们的测试通过了。我们需要小心使用replace方法，要处理的大量文件可能会导致性能问题。

### 3.2 使用Apache Commons IO读取文件

**此外，[Apache Commons IO](https://www.baeldung.com/apache-commons-io)库提供了BOMInputStream类**。这个类是一个包装器，它包含一个编码的ByteOrderMark作为它的第一个字节，让我们看看它是如何工作的：

```java
@Test
public void whenInputFileHasBOM_thenUseBOMInputStream() throws IOException {
    String line;
    String actual = "";
    ByteOrderMark[] byteOrderMarks = new ByteOrderMark[] { 
        ByteOrderMark.UTF_8, ByteOrderMark.UTF_16BE, ByteOrderMark.UTF_16LE, ByteOrderMark.UTF_32BE, ByteOrderMark.UTF_32LE
    };
    InputStream inputStream = new BOMInputStream(ioStream, false, byteOrderMarks);
    Reader reader = new InputStreamReader(inputStream);
    BufferedReader br = new BufferedReader(reader);
    while ((line = br.readLine()) != null) {
        actual += line;
    }
    assertEquals(expected, actual);
}
```

代码与前面的示例类似，但我们将BOMInputStream作为参数传递给InputStreamReader。

### 3.3 使用Google Data(GData)读取文件

另一方面，**另一个处理BOM的有用库是Google Data(GData)**。这是一个较旧的库，但它有助于管理文件中的BOM。它使用XML作为其基础格式。让我们看看它的实际效果：

```java
@Test
public void whenInputFileHasBOM_thenUseGoogleGdata() throws IOException {
    char[] actual = new char[21];
    try (Reader r = new UnicodeReader(ioStream, null)) {
        r.read(actual);
    }
    assertEquals(expected, String.valueOf(actual));
}
```

最后，正如我们在前面的示例中观察到的，从文件中删除BOM很重要。如果我们在文件中处理不当，读取数据时就会出现意想不到的结果。这就是为什么我们需要知道我们的文件中存在这个标记。

## 4. 总结

在本文中，我们介绍了几个有关Java中非法字符编译错误的主题。首先，我们了解了UTF是什么以及BOM是如何集成到其中的。其次，我们展示了一个使用文本编辑器(在本例中为Windows记事本)创建的示例类。生成的类抛出非法字符的编译错误。最后，我们提供了一些有关如何使用BOM读取文件的代码示例。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-compiler)上获得。