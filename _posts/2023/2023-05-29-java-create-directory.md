---
layout: post
title:  在Java中创建目录
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

使用Java创建目录非常简单。该语言为我们提供了两种方法，允许我们创建单个目录或多个嵌套目录-mkdir()和mkdirs()。

在本教程中，我们将了解它们的行为方式。

## 2. 创建单个目录

让我们从创建单个目录开始。

出于我们的目的，我们将使用用户临时目录。我们可以使用System.getProperty(“java.io.tmpdir”)查找它。

我们会将此路径传递给Java File对象，该对象将代表我们的临时目录：

```java
private static final File TEMP_DIRECTORY = new File(System.getProperty("java.io.tmpdir"));
```

现在让我们在其中创建一个新目录。**我们将通过在表示要创建的目录的新File对象上调用File::mkdir方法来实现此目的**：

```java
File newDirectory = new File(TEMP_DIRECTORY, "new_directory");
assertFalse(newDirectory.exists());
assertTrue(newDirectory.mkdir());
```

**为了确保我们的目录还不存在，我们首先使用了exists()方法**。

然后我们调用mkdir()方法来告诉我们目录创建是否成功。如果该目录已经存在，该方法将返回false。

如果我们再次进行相同的调用：

```java
assertTrue(newDirectory.exists());
assertFalse(newDirectory.mkdir());
```

然后，正如我们所料，该方法在第二次调用时返回false。

**而且，mkdir()方法不仅在目录已经存在时返回false**，在[其他一些情况](https://twitter.com/steveloughran/status/1087427627869261825)下也会返回false。例如，可能存在一个文件名为我们要创建的目录的文件。或者我们可能缺少创建此目录的权限。

考虑到这一点，我们必须找到一种方法来确保我们的目录最终存在，无论是我们创建的还是已经存在的。为此，我们可以使用[isDirectory()方法](https://twitter.com/steveloughran/status/1087428893882175490)：

```java
newDirectory.mkdir() || newDirectory.isDirectory()
```

这样，我们就可以确保我们需要的目录在那里。

## 3. 创建多个嵌套目录

到目前为止，我们所看到的在单个目录上运行良好，但是如果我们想创建多个嵌套目录会发生什么？

在下面的示例中，我们将看到File::mkdir对此不起作用：

```java
File newDirectory = new File(TEMP_DIRECTORY, "new_directory");
File nestedDirectory = new File(newDirectory, "nested_directory");
assertFalse(newDirectory.exists());
assertFalse(nestedDirectory.exists());
assertFalse(nestedDirectory.mkdir());
```

由于new_directory不存在，mkdir不会创建底层的nested_directory。

**但是，File类为我们提供了另一种方法来实现这一点-mkdirs()**。此方法的行为类似于mkdir()，但也会创建所有不存在的父目录。

在我们前面的示例中，这意味着不仅要创建nested_directory，还要创建new_directory。

请注意，到目前为止我们使用的是File(File, String)构造函数，但**我们也可以使用File(String)构造函数并使用File.separator传递文件的完整路径以分隔路径的不同部分**：

```java
File newDirectory = new File(System.getProperty("java.io.tmpdir") + File.separator + "new_directory");
File nestedDirectory = new File(newDirectory, "nested_directory");
assertFalse(newDirectory.exists());
assertFalse(nestedDirectory.exists());
assertTrue(nestedDirectories.mkdirs());
```

如我们所见，目录是按预期创建的。**此外，该方法仅在至少创建了一个目录时才返回true。至于mkdir()方法，在其他情况下它会返回false**。

因此，这意味着在父目录存在的目录上使用的mkdirs()方法与mkdir()方法的工作方式相同。

## 4. 总结

在本文中，我们看到了两种允许我们在Java中创建目录的方法。第一个mkdir()，目标是创建一个目录，前提是它的父目录已经存在。第二个mkdirs()能够创建目录及其不存在的父目录。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-2)上获得。
