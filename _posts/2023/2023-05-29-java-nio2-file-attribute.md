---
layout: post
title:  NIO 2 FileAttribute API指南
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在本文中，我们将探讨Java 7 NIO.2文件系统API的一项高级功能-特别是文件属性API。

如果你想先深入了解这些基础部分，我们之前已经介绍了[File](https://www.baeldung.com/java-nio-2-file-api)和[Path](https://www.baeldung.com/java-nio-2-path) API。

处理文件系统操作所需的所有文件都捆绑在java.nio.file包中：

```java
import java.nio.file.*;
```

## 2. 文件基本属性

让我们从所有文件系统共有的基本属性的高级视图开始-由BasicFileAttributeView提供，它存储所有必需和可选的可见文件属性。

我们可以通过创建到HOME的路径并获取它的基本属性视图来探索当前机器上用户家位置的基本属性：

```java
String HOME = System.getProperty("user.home");
Path home = Paths.get(HOME);
BasicFileAttributeView basicView = Files.getFileAttributeView(home, BasicFileAttributeView.class);
```

经过上述步骤，我们现在可以一次批量读取所指向路径的所有属性：

```java
BasicFileAttributes basicAttribs = basicView.readAttributes();
```

我们现在可以探索不同的通用属性，并且我们可以在应用程序中实际使用这些属性，尤其是在条件语句中。

我们可以从其基本属性容器中查询文件的大小：

```java
@Test
public void givenPath_whenGetsFileSize_thenCorrect() {
    long size = basicAttribs.size();
    assertTrue(size > 0);
}
```

我们还可以检查它是否是一个目录：

```java
@Test
public void givenPath_whenChecksIfDirectory_thenCorrect() {
    boolean isDir = basicAttribs.isDirectory();
    assertTrue(isDir);
}
```

或常规文件：

```java
@Test
public void givenPath_whenChecksIfFile_thenCorrect() {
    boolean isFile = basicAttribs.isRegularFile();
    assertFalse(isFile);
}
```

使用Java NIO.2，我们现在能够处理文件系统中的符号链接或软链接。这些是我们通常称为快捷方式的文件或目录。

检查文件是否为符号链接：

```java
@Test
public void givenPath_whenChecksIfSymLink_thenCorrect() {
    boolean isSymLink = basicAttribs.isSymbolicLink();
    assertFalse(isSymLink);
}
```

在极少数情况下，我们可以调用isOther API来检查文件是否不属于常规文件、目录或符号链接等常见类别：

```java
@Test
public void givenPath_whenChecksIfOther_thenCorrect() {
    boolean isOther = basicAttribs.isOther();
    assertFalse(isOther);
}
```

要获取文件的创建时间，执行以下操作：

```java
FileTime created = basicAttribs.creationTime();
```

获取最后修改时间：

```java
FileTime modified = basicAttribs.lastModifiedTime();
```

获取最后访问时间：

```java
FileTime accessed = basicAttribs.lastAccessTime();
```

以上所有示例都返回一个FileTime对象。这是一个比单纯的时间戳更有用的抽象。

例如，我们可以很容易地比较两个FileTime来知道哪个事件发生在另一个事件之前或之后：

```java
@Test
public void givenFileTimes_whenComparesThem_ThenCorrect() {
    FileTime created = basicAttribs.creationTime();
    FileTime modified = basicAttribs.lastModifiedTime();
    FileTime accessed = basicAttribs.lastAccessTime();

    assertTrue(0 >= created.compareTo(accessed));
    assertTrue(0 <= modified.compareTo(created));
    assertTrue(0 == created.compareTo(created));
}
```

compareTo API的工作方式与Java中的其他Comparable对象相同。如果调用的对象小于参数，它返回一个负值；在我们的例子中，创建时间肯定在访问时间之前，就像第一个断言一样。

在第二个断言中，我们得到一个正整数值，因为修改只能在创建事件之后进行。最后，当比较的时间相等时，它返回0。

当我们有一个FileTime对象时，我们可以根据需要将它转换为大多数其他单位：天、小时、分钟、秒、毫秒等。我们通过调用适当的API来做到这一点：

```java
accessed.to(TimeUnit.SECONDS);
accessed.to(TimeUnit.HOURS);
accessed.toMillis();
```

我们还可以通过调用其toString API来打印FileTime的人类可读形式：

```java
accessed.toString();
```

它以ISO时间格式打印一些有用的东西：

```plaintext
2016-11-24T07:52:53.376Z
```

我们还可以通过调用其setTimes(modified, accessed, created) API来更改视图的时间属性。我们在要更改的位置传入新的FileTime对象，在不想更改的地方传入null。

要将上次访问时间更改为未来一分钟，我们将遵循以下步骤：

```java
FileTime newAccessTime = FileTime.fromMillis(basicAttribs.lastAccessTime().toMillis() + 60000);
basicView.setTimes(null, newAccessTime , null);
```

从机器上运行并使用文件系统的任何其他应用程序可以看出，此更改将保留在实际文件中。

## 3. 文件空间属性

当你在Windows、Linux或Mac上打开我的电脑时，你通常可以看到有关你的存储驱动器的空间信息的图形分析。

Java NIO.2使这种高级功能变得非常容易。它与底层文件系统交互以检索此信息，而我们只需调用简单的API。

我们可以使用FileStore类来检查存储驱动器并获取重要信息，例如它的大小、已使用的空间量以及尚未使用的空间量。

要获取文件系统中任意文件位置的FileStore实例，我们使用Files类的getFileStore API：

```java
Path file = Paths.get("file");
FileStore store = Files.getFileStore(file);
```

这个FileStore实例具体代表指定文件所在的文件存储，而不是文件本身。要获得总空间：

```java
long total = store.getTotalSpace();
```

获取已用空间：

```java
long used = store.getTotalSpace() - store.getUnallocatedSpace();
```

与下一种相比，我们不太可能遵循这种方法。

更常见的是，我们更可能会获取所有文件存储的存储信息。要在程序中模拟我的电脑的图形驱动器空间信息，我们可以使用FileSystem类来枚举文件存储：

```java
Iterable<FileStore> fileStores = FileSystems.getDefault().getFileStores();
```

然后我们可以遍历返回值并对信息做任何我们需要做的事情，例如更新图形用户界面：

```java
for (FileStore fileStore : fileStores) {
    long totalSpace = fileStore.getTotalSpace();
    long unAllocated = fileStore.getUnallocatedSpace();
    long usable = fileStore.getUsableSpace();
}
```

请注意，所有返回值均以字节为单位。我们可以转换为合适的单位以及使用基本算法计算其他信息，例如已用空间。

**未分配空间和可用空间之间的区别在于JVM的可访问性**。

可用空间是JVM可用的空间，而未分配空间是底层文件系统所见的可用空间。因此，可用空间有时可能小于未分配空间。

## 4. 文件所有者属性

要检查文件所有权信息，我们使用FileOwnerAttributeView接口。它为我们提供了所有权信息的高级视图。

我们可以像这样创建一个FileOwnerAttributeView对象：

```java
Path path = Paths.get(HOME);
FileOwnerAttributeView ownerView = Files.getFileAttributeView(attribPath, FileOwnerAttributeView.class);
```

从上面的视图中获取文件的所有者：

```java
UserPrincipal owner = ownerView.getOwner();
```

除了出于其他任意目的获取所有者的名称之外，我们实际上无法通过编程方式对上述对象执行任何操作：

```java
String ownerName = owner.toString();
```

## 5. 用户自定义文件属性

在某些情况下，文件系统中定义的文件属性不足以满足你的需求。如果你遇到这种情况并需要在文件上设置自己的属性，那么UserDefinedFileAttributeView接口将派上用场：

```java
Path path = Paths.get("somefile");
UserDefinedFileAttributeView userDefView = Files.getFileAttributeView(attribPath, UserDefinedFileAttributeView.class);
```

检索已为上述视图表示的文件定义的用户定义属性列表：

```java
List<String> attribList = userDefView.list();
```

要在文件上设置用户定义的属性，我们使用以下习惯用法：

```java
String name = "attrName";
String value = "attrValue";
userDefView.write(name, Charset.defaultCharset().encode(value));
```

当你需要访问用户定义的属性时，你可以遍历视图返回的属性列表并使用以下习惯用法检查它们：

```java
ByteBuffer attrValue = ByteBuffer.allocate(userView.size(attrName));
userDefView.read(attribName, attribValue);
attrValue.flip();
String attrValue = Charset.defaultCharset().decode(attrValue).toString();
```

要从文件中删除用户定义的属性，我们只需调用视图的delete API：

```java
userDefView.delete(attrName);
```

## 6. 总结

在本文中，我们探讨了Java 7 NIO.2文件系统API中一些不太常用的功能，特别是文件属性API。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-1)上获得。
