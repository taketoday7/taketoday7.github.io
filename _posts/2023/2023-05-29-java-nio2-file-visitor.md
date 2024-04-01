---
layout: post
title:  NIO 2 FileVisitor指南
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在这篇文章中，我们将探讨NIO2的一个有趣功能-FileVisitor接口。

所有操作系统和一些第三方应用程序都具有文件搜索功能，用户可以在其中定义搜索条件。

这个接口是我们在Java应用程序中实现这样的功能所需要的。如果你需要搜索所有.mp3文件、查找并删除.class文件或查找上个月未访问过的所有文件，那么这个接口就是你所需要的。

实现此功能所需的所有类都捆绑在一个包中：

```java
import java.nio.file.*;
```

## 2. FileVisitor的工作原理

使用FileVisitor接口，你可以遍历文件树到任意深度，并对在任何分支上找到的文件或目录执行任何操作。

FileVisitor接口的典型实现如下所示：

```java
public class FileVisitorImpl implements FileVisitor<Path> {

    @Override
    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
        return null;
    }

    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
        return null;
    }

    @Override
    public FileVisitResult visitFileFailed(Path file, IOException exc) {
        return null;
    }

    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) {
        return null;
    }
}
```

这四个接口方法允许我们在遍历过程中的关键点指定所需的行为：分别在访问目录之前、访问文件时、发生故障时和访问目录之后。

每个阶段的返回值都是FileVisitResult类型，控制遍历的流程。也许你想遍历文件树以查找特定目录并在找到该目录时终止进程，或者你想跳过特定目录或文件。

FileVisitResult是FileVisitor接口方法的四个可能返回值的枚举：

-   **FileVisitResult.CONTINUE**：表示文件树遍历应该在返回它的方法退出后继续
-   **FileVisitResult.TERMINATE**：停止文件树遍历并且不再访问目录或文件
-   **FileVisitResult.SKIP_SUBTREE**：这个结果只有在从preVisitDirectory API返回时才有意义，在其他地方，它的工作方式类似于CONTINUE。它表示应该跳过当前目录及其所有子目录
-   **FileVisitResult.SKIP_SIBLINGS**：指示遍历应继续而不访问当前文件或目录的同级文件。如果在preVisitDirectory阶段调用，那么即使是当前目录也会被跳过并且不会调用postVisitDirectory

最后，必须有一种方法来触发遍历过程，也许是当用户在定义搜索条件后从图形用户界面单击搜索按钮时。这是最简单的部分。

我们只需要调用Files类的静态walkFileTree API，并向其传递一个表示遍历起点的Path类实例，然后传递一个FileVisitor实例：

```java
Path startingDir = Paths.get("pathToDir");
FileVisitorImpl visitor = new FileVisitorImpl();
Files.walkFileTree(startingDir, visitor);
```

## 3. 文件搜索示例

在本节中，我们将使用FileVisitor接口实现一个文件搜索应用程序。我们希望让用户指定完整的文件名、扩展名和要查找的起始目录。

当我们找到文件时，我们会在屏幕上打印一条成功消息，当搜索整个文件树而没有找到文件时，我们也会打印一条适当的失败消息。

### 3.1 主类

我们将这个类称为FileSearchExample.java：

```java
public class FileSearchExample implements FileVisitor<Path> {
    private String fileName;
    private Path startDir;

    // standard constructors
}
```

我们还没有实现接口方法。请注意，我们已经创建了一个构造函数，它接收要搜索的文件名和开始搜索的路径。我们将仅使用起始路径作为基本情况来得出文件未找到的总结。

在下面的小节中，我们将实现每个接口方法并讨论它在这个特定示例应用程序中的作用。

### 3.2 preVisitDirectory API

让我们从实现preVisitDirectory API开始：

```java
@Override
public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
    return CONTINUE;
}
```

正如我们之前所说，每次进程在树中遇到新目录时都会调用此API。它的返回值决定了接下来会发生什么，这取决于我们的决定。这是我们将跳过特定目录并将它们从搜索样本空间中删除的点。

让我们选择不区分任何目录，只搜索所有目录。

### 3.3 visitFile API

接下来，我们将实现visitFile API。这是主要操作发生的地方。每次遇到文件时都会调用此API。我们利用它来检查文件属性并与我们的标准进行比较并返回适当的结果：

```java
@Override
public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
    String fileName = file.getFileName().toString();
    if (FILE_NAME.equals(fileName)) {
        System.out.println("File found: " + file.toString());
        return TERMINATE;
    }
    return CONTINUE;
}
```

在我们的例子中，我们只检查正在访问的文件的名称，以了解它是否是用户正在搜索的文件。如果名称匹配，我们将打印一条成功消息并终止该过程。

但是，这里可以做很多事情，尤其是在读取了文件属性部分之后。你可以检查创建时间、上次修改时间或上次访问时间或attrs参数中可用的几个属性并做出相应决定。

### 3.4 visitFileFailed API

接下来，我们将实现visitFileFailed API。当JVM无法访问特定文件时调用此API。也许它已被另一个应用程序锁定或者可能只是一个权限问题：

```java
@Override
public FileVisitResult visitFileFailed(Path file, IOException exc) {
    System.out.println("Failed to access file: " + file.toString());
    return CONTINUE;
}
```

我们只需记录一条失败消息并继续遍历目录树的其余部分。在图形应用程序中，你可以选择使用对话框询问用户是否继续，或者只是将消息记录在某处并编译报告以供以后使用。

### 3.5 postVisitDirectory API

最后，我们将实现postVisitDirectory API。每次完全遍历目录时都会调用此API：

```java
@Override
public FileVisitResult postVisitDirectory(Path dir, IOException exc){
    boolean finishedSearch = Files.isSameFile(dir, START_DIR);
    if (finishedSearch) {
        System.out.println("File:" + FILE_NAME + " not found");
        return TERMINATE;
    }
    return CONTINUE;
}
```

我们使用Files.isSameFile API来检查刚刚遍历的目录是否是我们开始遍历的目录。如果返回值为true，则表示搜索已完成，尚未找到该文件。。因此，我们终止该过程并显示失败消息。

但是，如果返回值为false，这意味着我们刚刚完成了一个子目录的遍历，并且仍有可能在其他子目录中找到该文件。所以我们继续遍历。

现在，我们可以添加main方法来执行FileSearchExample应用程序：

```java
public static void main(String[] args) {
    Path startingDir = Paths.get("C:/Users/user/Desktop");
    String fileToSearch = "hibernate-guide.txt"
    FileSearchExample crawler = new FileSearchExample(fileToSearch, startingDir);
    Files.walkFileTree(startingDir, crawler);
}
```

你可以通过更改startingDir和fileToSearch变量的值来尝试这个示例。当fileToSearch存在于startingDir或其任何子目录中时，你将收到一条成功消息，否则会收到一条失败消息。

## 4. 总结

在本文中，我们探讨了Java 7 NIO.2文件系统API中一些不太常用的功能，尤其是FileVisitor接口。我们还成功完成了构建文件搜索应用程序的步骤，以演示其功能。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-1)上获得。
