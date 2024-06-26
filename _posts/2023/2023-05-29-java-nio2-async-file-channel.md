---
layout: post
title:  NIO 2 AsynchronousFileChannel指南
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在本文中，我们将探讨Java 7中新I/O(NIO2)的关键附加API之一，即异步文件通道API。

你还可以阅读更多关于NIO.2[文件操作](https://www.baeldung.com/java-nio-2-file-api)和[路径操作](https://www.baeldung.com/java-nio-2-path)的信息-理解这些将使本文更容易理解。

要在我们的项目中使用NIO2异步文件通道，我们必须导入java.nio.channels包，因为它捆绑了所有必需的类：

```java
import java.nio.channels.*;
```

## 2. AsynchronousFileChannel

在本节中，我们将探讨如何使用使我们能够对文件执行异步操作的主类，即AsynchronousFileChannel类。要创建它的实例，我们调用静态open方法：

```java
Path filePath = Paths.get("/path/to/file");

AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(filePath, READ, WRITE, CREATE, DELETE_ON_CLOSE);
```

所有枚举值都来自StandardOpenOption。

open API的第一个参数是表示文件位置的Path对象。要了解有关NIO2中路径操作的更多信息，请访问[此链接](https://www.baeldung.com/java-nio-2-path)。其他参数组成一组指定选项，这些选项应该可用于返回的文件通道。

我们创建的异步文件通道可用于对文件执行所有已知操作。要仅执行操作的一个子集，我们将仅为那些操作指定选项。例如，只读：

```java
Path filePath = Paths.get("/path/to/file");

AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(filePath, StandardOpenOption.READ);
```

## 3. 从文件中读取

就像NIO2中的所有异步操作一样，可以通过两种方式读取文件内容。使用Future和使用CompletionHandler。在每种情况下，我们都使用返回通道的read API。

在Maven的测试资源文件夹或源目录中(如果不使用Maven)，让我们创建一个名为file.txt的文件，其开头只有文本tuyucheng.com。我们现在将演示如何读取此内容。

### 3.1 Future方法

首先，我们将看到如何使用Future类异步读取文件：

```java
@Test
public void givenFilePath_whenReadsContentWithFuture_thenCorrect() {
    Path path = Paths.get(URI.create(this.getClass().getResource("/file.txt").toString()));
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, StandardOpenOption.READ);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    Future<Integer> operation = fileChannel.read(buffer, 0);

    // run other code as operation continues in background
    operation.get();
      
    String fileContent = new String(buffer.array()).trim();
    buffer.clear();

    assertEquals(fileContent, "tuyucheng.com");
}
```

在上面的代码中，在创建文件通道后，我们使用了read API-它需要一个ByteBuffer来存储从通道读取的内容作为它的第一个参数。

第二个参数是long值，表示从文件中哪个位置开始读取。

无论文件是否已被读取，该方法都会立即返回。

接下来，我们可以在后台继续操作时执行任何其他代码。当我们执行完其他代码时，我们可以调用get() API，如果我们正在执行其他代码时操作已经完成，它会立即返回，否则它会阻塞直到操作完成。

我们的断言确实证明文件中的内容已被读取。

如果我们将read API调用中的位置参数从0更改为其他值，我们也会看到效果。例如，字符串tuyucheng.com中的第7个字符是n。因此，将位置参数更改为7会导致缓冲区包含字符串ng.com。

### 3.2 CompletionHandler方法

接下来，我们将看到如何使用CompletionHandler实例读取文件的内容：

```java
@Test
public void givenPath_whenReadsContentWithCompletionHandler_thenCorrect() {
    Path path = Paths.get(URI.create( this.getClass().getResource("/file.txt").toString()));
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, StandardOpenOption.READ);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    fileChannel.read(buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {

        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            // result is number of bytes read
            // attachment is the buffer containing content
        }
        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {

        }
    });
}
```

在上面的代码中，我们使用了read API的第二个变体。它仍然以一个ByteBuffer和读取操作的起始位置分别作为第一个和第二个参数。第三个参数是CompletionHandler实例。

CompletionHandler的第一个泛型类型是操作的返回类型，在本例中是一个整数，表示读取的字节数。

第二个是附件的类型。我们选择附加缓冲区，以便在read完成时，我们可以在completed的回调API中使用文件的内容。

从语义上讲，这不是一个真正的有效单元测试，因为我们不能在completed的回调方法中进行断言。但是，我们这样做是为了保持一致性，因为我们希望我们的代码尽可能复制粘贴运行。

## 4. 写入文件

Java NIO2还允许我们对文件执行写操作。正如我们对其他操作所做的那样，我们可以通过两种方式写入文件。使用Future和使用CompletionHandler。在每种情况下，我们都使用返回通道的write API。

创建用于写入文件的AsynchronousFileChannel可以像这样完成：

```java
AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);
```

### 4.1 特别注意事项

请注意传递给open API的选项。我们还可以添加另一个选项StandardOpenOption.CREATE如果我们希望创建由路径表示的文件以防它不存在。另一个常见选项是StandardOpenOption.APPEND，它不会覆盖文件中的现有内容。

我们将使用以下行来创建用于测试目的的文件通道：

```java
AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, WRITE, CREATE, DELETE_ON_CLOSE);
```

这样，我们将提供任意路径并确保将创建该文件。测试退出后，创建的文件将被删除。为确保创建的文件在测试退出后不被删除，你可以删除最后一个选项。

要运行断言，我们需要在写入文件后尽可能读取文件内容。让我们将读取逻辑隐藏在一个单独的方法中以避免冗余：

```java
public static String readContent(Path file) {
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(file, StandardOpenOption.READ);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    Future<Integer> operation = fileChannel.read(buffer, 0);

    // run other code as operation continues in background
    operation.get();     

    String fileContent = new String(buffer.array()).trim();
    buffer.clear();
    return fileContent;
}
```

### 4.2 Future方法

使用Future类异步写入文件：

```java
@Test
public void givenPathAndContent_whenWritesToFileWithFuture_thenCorrect() {
    String fileName = UUID.randomUUID().toString();
    Path path = Paths.get(fileName);
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, WRITE, CREATE, DELETE_ON_CLOSE);

    ByteBuffer buffer = ByteBuffer.allocate(1024);

    buffer.put("hello world".getBytes());
    buffer.flip();

    Future<Integer> operation = fileChannel.write(buffer, 0);
    buffer.clear();

    //run other code as operation continues in background
    operation.get();

    String content = readContent(path);
    assertEquals("hello world", content);
}
```

让我们检查一下上面的代码中发生了什么。我们创建一个随机文件名并使用它来获取Path对象。我们使用此路径通过前面提到的选项打开异步文件通道。

然后我们将要写入文件的内容放在一个缓冲区中，然后执行write。我们使用我们的辅助方法来读取文件的内容，并确认它是我们所期望的。

### 4.3 CompletionHandler方法

我们还可以使用CompletionHandler，这样我们就不必在while循环中等待操作完成：

```java
@Test
public void givenPathAndContent_whenWritesToFileWithHandler_thenCorrect() {
    String fileName = UUID.randomUUID().toString();
    Path path = Paths.get(fileName);
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, WRITE, CREATE, DELETE_ON_CLOSE);

    ByteBuffer buffer = ByteBuffer.allocate(1024);
    buffer.put("hello world".getBytes());
    buffer.flip();

    fileChannel.write(buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {

        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            // result is number of bytes written
            // attachment is the buffer
        }
        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {

        }
    });
}
```

这次我们调用write API时，唯一的新事物是第三个参数，我们在其中传递类型为CompletionHandler的匿名内部类。

当操作完成时，该类调用它的completed方法，我们可以在其中定义应该发生的事情。

## 5. 总结

在本文中，我们探讨了Java NIO2的异步文件通道API的一些最重要的功能。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-1)上获得。
