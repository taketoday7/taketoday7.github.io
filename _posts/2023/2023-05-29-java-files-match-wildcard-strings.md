---
layout: post
title:  在Java中查找与通配符字符串匹配的文件
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在本教程中，我们将学习如何在Java中使用通配符字符串查找文件。

## 2. 简介

在编程领域，**[glob](https://www.baeldung.com/linux/bash-globbing)是一种使用通配符来匹配文件名的模式**。我们将使用glob模式来为我们的示例过滤文件名列表。我们将使用流行的通配符“*”和“?”。Java从Java SE 7开始支持这个功能。

**Java在其FileSystem类中提供了getPathMatcher()方法。它可以采用正则表达式(regex)或glob模式**。我们将在此示例中使用glob模式，因为与正则表达式相比，应用通配符更简单。

让我们看一个将此方法与glob模式结合使用的示例：

```java
String pattern = "myCustomPattern";
PathMatcher matcher = FileSystems.getDefault().getPathMatcher("glob:" + pattern);
```

以下是Java中glob模式的一些示例：

| Glob           | 描述                                 |
|----------------|------------------------------------|
| *.java         | 匹配所有扩展名为“java”的文件                  |
| *.{java,class} | 匹配所有扩展名为“java”或“class”的文件          |
| \*.*           | 匹配名称中带有“.”的所有文件                  |
| ????           | 匹配名称中包含四个字符的所有文件                   |
| \[test].docx   | 匹配文件名为“t”、“e”、“s”或“t”且扩展名为“docx”的所有文件 |
| \[0-4].csv     | 匹配文件名为“0”、“1”、“2”、“3”或“4”且扩展名为“csv”的所有文件 |
| C:\\\temp\\\\* | 匹配Windows系统上“C:\temp”目录中的所有文件      |
| src/test/*     | 匹配基于Unix的系统上“src/test/”目录中的所有文件    |

## 3. 实现

让我们深入了解实现此解决方案的细节。完成此任务有两个步骤。

**首先，我们创建一个带有两个参数的方法-一个要在其中搜索的根目录和一个要查找的通配符模式**。此方法将包含用于访问每个文件和目录、利用glob模式并最终返回匹配文件名列表的编程逻辑。

**其次，我们使用Java提供的Files类中的walkFileTree方法来调用我们的搜索过程**。

首先，让我们使用searchWithWc()方法创建SearchFileByWildcard类，该方法接收Path和String模式作为参数：

```java
class SearchFileByWildcard {
    static List<String> matchesList = new ArrayList<String>();
    List<String> searchWithWc(Path rootDir, String pattern) throws IOException {
        matchesList.clear();
        FileVisitor<Path> matcherVisitor = new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attribs) throws IOException {
                FileSystem fs = FileSystems.getDefault();
                PathMatcher matcher = fs.getPathMatcher(pattern);
                Path name = file.getFileName();
                if (matcher.matches(name)) {
                    matchesList.add(name.toString);
                }
                return FileVisitResult.CONTINUE;
            }
        };
        Files.walkFileTree(rootDir, matcherVisitor);
        return matchesList;
    }
}
```

要访问rootDir中的文件，我们使用FileVisitor接口。一旦我们通过调用getDefault()方法获得文件系统的接口，我们就可以使用FileSystem类中的getPathMatcher()方法。这是我们在rootDir中的各个文件路径上应用glob模式的地方。

在我们的例子中，我们可以使用生成的PathMatcher来获取匹配文件名的[ArrayList](https://www.baeldung.com/java-arraylist)。

最后，我们从NIO Files类中调用walkFileTree方法。文件遍历从rootDir开始，并以深度优先的方式递归访问树中的每个节点。matcherVisitor包含SimpleFileVisitor类的visitFile方法的实现。

现在我们已经讨论了实现基于通配符的文件搜索，让我们看一些示例输出。我们将在示例中使用以下文件结构：

![](/assets/images/2023/javanio/javafilesmatchwildcardstrings01.png)

如果我们传递一个带有“glob:*.{txt,docx}”模式的字符串，我们的代码会输出扩展名为“txt”的三个文件名和一个扩展名为“docx”的文件名：

```java
SearchFileByWildcard sfbw = new SearchFileByWildcard();
List<String> actual = sfbw.searchWithWc(Paths.get("src/test/resources/sfbw"), "glob:*.{txt,docx}");

assertEquals(new HashSet<>(Arrays.asList("six.txt", "three.txt", "two.docx", "one.txt")), new HashSet<>(actual));
```

如果我们传递一个带有“glob:????.{csv}”模式的字符串，我们的代码会输出一个包含四个字符后跟“.”的文件名。扩展名为“csv”：

```java
SearchFileByWildcard sfbw = new SearchFileByWildcard();
List<String> actual = sfbw.searchWithWc(Paths.get("src/test/resources/sfbw"), "glob:????.{csv}");

assertEquals(new HashSet<>(Arrays.asList("five.csv")), new HashSet<>(actual));
```

## 4. 总结

在本教程中，我们学习了如何在Java中使用通配符模式搜索文件。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-2)上获得。
