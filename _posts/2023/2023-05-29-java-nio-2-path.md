---
layout: post
title:  Java NIO 2- Paths API
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在本文中，我们将学习如何在Java中使用新的I/O(NIO2) Path API。

NIO2中的Path API构成了Java 7附带的主要新功能区域之一，特别是新文件系统API和File API的子集。

## 2. 设置

NIO2支持捆绑在java.nio.file包中。因此，设置你的项目以使用Path API只需导入此包中的所有内容：

```java
import java.nio.file.*;
```

由于本文中的代码示例可能会在不同的环境中运行，因此让我们获取用户的主目录：

```java
private static String HOME = System.getProperty("user.home");
```

此变量将指向任何环境中的有效位置。

Paths类是涉及文件系统路径的所有操作的主要入口点。它允许我们创建和操作文件和目录的路径。

值得注意的是，路径操作本质上主要是句法操作；它们对底层文件系统没有影响，文件系统对它们的成功或失败也没有任何影响。这意味着将不存在的路径作为路径操作的参数传递与它的成功或失败无关。

## 3. 路径操作

在本节中，我们将介绍路径操作中使用的主要语法。顾名思义，Path类是文件系统中路径的编程表示形式。

Path对象包含用于构造路径的文件名和目录列表，并用于检查、定位和操作文件。

辅助类java.nio.file.Paths(复数形式)是创建Path对象的正式方式，它有两个用于从路径字符串创建Path的静态方法：

```java
Path path = Paths.get("path string");
```

无论我们在路径字符串中使用正斜杠还是反斜杠，都无关紧要，API会根据底层文件系统的要求解析此参数。

从java.net.URI对象创建：

```java
Path path = Paths.get(URI object);
```

## 4. 创建Path

从路径字符串创建Path对象：

```java
@Test
public void givenPathString_whenCreatesPathObject_thenCorrect() {
    Path p = Paths.get("/articles/tuyucheng");
 
    assertEquals("\\articles\\tuyucheng", p.toString());
}
```

除了第一部分(在本例中为articles)之外，get API还可以接收路径字符串部分(在本例中为articles和tuyucheng)的可变参数。

如果我们提供这些部分而不是完整的路径字符串，它们将用于构造Path对象，我们不需要在变量参数部分包含名称分隔符(斜杠)：

```java
@Test
public void givenPathParts_whenCreatesPathObject_thenCorrect() {
    Path p = Paths.get("/articles", "tuyucheng");
    
    assertEquals("\\articles\\tuyucheng", p.toString());
}
```

## 5. 获取路径信息

你可以将Path对象视为序列的名称元素。诸如E:\tuyucheng\articles\java的路径字符串由三个名称元素组成，即tuyucheng、articles和java。目录结构中的最高元素将位于索引0处，在本例中为tuyucheng。

目录结构中的最低元素将位于索引[n-1\]处，其中n是路径中名称元素的数量。这个最低的元素被称为文件名，不管它是否是一个实际的文件：

```java
@Test
public void givenPath_whenRetrievesFileName_thenCorrect() {
    Path p = Paths.get("/articles/tuyucheng/logs");

    Path fileName = p.getFileName();
 
    assertEquals("logs", fileName.toString());
}
```

方法可用于按索引检索单个元素：

```java
@Test
public void givenPath_whenRetrievesNameByIndex_thenCorrect() {
    Path p = Paths.get("/articles/tuyucheng/logs");
    Path name0 = getName(0);
    Path name1 = getName(1);
    Path name2 = getName(2);
    assertEquals("articles", name0.toString());
    assertEquals("tuyucheng", name1.toString());
    assertEquals("logs", name2.toString());
}
```

或使用这些索引范围的路径的子序列：

```java
@Test
public void givenPath_whenCanRetrieveSubsequenceByIndex_thenCorrect() {
    Path p = Paths.get("/articles/tuyucheng/logs");

    Path subPath1 = p.subpath(0,1);
    Path subPath2 = p.subpath(0,2);
 
    assertEquals("articles", subPath1.toString());
    assertEquals("articles\\tuyucheng", subPath2.toString());
    assertEquals("articles\\tuyucheng\\logs", p.subpath(0, 3).toString());
    assertEquals("tuyucheng", p.subpath(1, 2).toString());
    assertEquals("tuyucheng\\logs", p.subpath(1, 3).toString());
    assertEquals("logs", p.subpath(2, 3).toString());
}
```

每个路径都与父路径关联，如果路径没有父路径，则为null。Path对象的父级由路径的根组件(如果有)和路径中除文件名外的每个元素组成。例如，/a/b/c的父路径为/a/b，而/a的父路径为空：

```java
@Test
public void givenPath_whenRetrievesParent_thenCorrect() {
    Path p1 = Paths.get("/articles/tuyucheng/logs");
    Path p2 = Paths.get("/articles/tuyucheng");
    Path p3 = Paths.get("/articles");
    Path p4 = Paths.get("/");

    Path parent1 = p1.getParent();
    Path parent2 = p2.getParent();
    Path parent3 = p3.getParent();
    Path parent4 = p4.getParent();

    assertEquals("\\articles\\tuyucheng", parent1.toString());
    assertEquals("\\articles", parent2.toString());
    assertEquals("\\", parent3.toString());
    assertEquals(null, parent4);
}
```

我们还可以获得路径的根元素：

```java
@Test
public void givenPath_whenRetrievesRoot_thenCorrect() {
    Path p1 = Paths.get("/articles/tuyucheng/logs");
    Path p2 = Paths.get("c:/articles/tuyucheng/logs");

    Path root1 = p1.getRoot();
    Path root2 = p2.getRoot();

    assertEquals("\\", root1.toString());
    assertEquals("c:\\", root2.toString());
}
```

## 6. 路径规范化

许多文件系统使用“.”符号表示当前目录，使用“..”表示父目录。你可能遇到路径包含冗余目录信息的情况。

例如，考虑以下路径字符串：

```plaintext
/tuyucheng/./articles
/tuyucheng/authors/../articles
/tuyucheng/articles
```

它们都解析到相同的位置/tuyucheng/articles。前两个有冗余，而最后一个没有。

规范化路径涉及删除其中的冗余。Path.normalize()操作就是为此目的而提供的。

这个例子现在应该是不言自明的：

```java
@Test
public void givenPath_whenRemovesRedundancies_thenCorrect1() {
    Path p = Paths.get("/home/./tuyucheng/articles");

    Path cleanPath = p.normalize();
 
    assertEquals("\\home\\tuyucheng\\articles", cleanPath.toString());
}
```

这个也是：

```java
@Test
public void givenPath_whenRemovesRedundancies_thenCorrect2() {
    Path p = Paths.get("/home/tuyucheng/../articles");

    Path cleanPath = p.normalize();
 
    assertEquals("\\home\\articles", cleanPath.toString());
}
```

## 7. 路径转换

有一些操作可以将路径转换为选定的表示格式。要将任何路径转换为可以从浏览器打开的字符串，我们使用toUri方法：

```java
@Test
public void givenPath_whenConvertsToBrowseablePath_thenCorrect() {
    Path p = Paths.get("/home/tuyucheng/articles.html");

    URI uri = p.toUri();
    assertEquals("file:///E:/home/tuyucheng/articles.html", uri.toString());
}
```

我们还可以将路径转换为其绝对表示。toAbsolutePath方法根据文件系统默认目录解析路径：

```java
@Test
public void givenPath_whenConvertsToAbsolutePath_thenCorrect() {
    Path p = Paths.get("/home/tuyucheng/articles.html");

    Path absPath = p.toAbsolutePath();
 
    assertEquals("E:\\home\\tuyucheng\\articles.html", absPath.toString());
}
```

但是，当检测到要解析的路径已经是绝对路径时，该方法按原样返回：

```java
@Test
public void givenAbsolutePath_whenRetainsAsAbsolute_thenCorrect() {
    Path p = Paths.get("E:\\home\\tuyucheng\\articles.html");

    Path absPath = p.toAbsolutePath();
 
    assertEquals("E:\\home\\tuyucheng\\articles.html", absPath.toString());
}
```

我们还可以通过调用toRealPath方法将任何路径转换为其实际等效路径。此方法尝试通过将路径的元素映射到文件系统中的实际目录和文件来解析路径。

是时候使用我们在设置部分创建的变量了，该变量指向登录用户在文件系统中的家位置：

```java
@Test
public void givenExistingPath_whenGetsRealPathToFile_thenCorrect() {
    Path p = Paths.get(HOME);

    Path realPath = p.toRealPath();
 
    assertEquals(HOME, realPath.toString());
}
```

上面的测试并没有真正告诉我们很多关于这个操作的行为。最明显的结果是，如果路径在文件系统中不存在，那么该操作将抛出IOException。

由于缺乏更好的方法来说明这一点，请看下一个测试，它试图将不存在的路径转换为真实路径：

```java
@Test(expected = NoSuchFileException.class)
public void givenInExistentPath_whenFailsToConvert_thenCorrect() {
    Path p = Paths.get("E:\\home\\tuyucheng\\articles.html");
    
    p.toRealPath();
}
```

当我们捕获IOException时，测试成功。此操作抛出的IOException的实际子类是NoSuchFileException。

## 8. 拼接路径

使用resolve方法可以实现拼接任意两个路径。

简单地说，我们可以在任何Path上调用resolve方法，并传入一个部分路径作为参数。该部分路径附加到原始路径：

```java
@Test
public void givenTwoPaths_whenJoinsAndResolves_thenCorrect() {
    Path p = Paths.get("/tuyucheng/articles");

    Path p2 = p.resolve("java");
 
    assertEquals("\\tuyucheng\\articles\\java", p2.toString());
}
```

但是，当传递给resolve方法的路径字符串不是部分路径时；最值得注意的是绝对路径，然后返回传入的路径：

```java
@Test
public void givenAbsolutePath_whenResolutionRetainsIt_thenCorrect() {
    Path p = Paths.get("/tuyucheng/articles");

    Path p2 = p.resolve("C:\\tuyucheng\\articles\java");
 
    assertEquals("C:\\tuyucheng\\articles\\java", p2.toString());
}
```

任何具有根元素的路径都会发生同样的事情。路径字符串“java”没有根元素，而路径字符串“/java”有根元素。因此，当你传入具有根元素的路径时，它会按原样返回：

```java
@Test
public void givenPathWithRoot_whenResolutionRetainsIt_thenCorrect2() {
    Path p = Paths.get("/tuyucheng/articles");

    Path p2 = p.resolve("/java");
 
    assertEquals("\\java", p2.toString());
}
```

## 9. 相对化路径

术语相对化只是意味着在两个已知路径之间创建直接路径。例如，如果我们有一个目录/tuyucheng并且在其中，我们还有另外两个目录，例如/tuyucheng/authors和/tuyucheng/articles是有效路径。

相对于authors的articles路径将被描述为“在目录层次结构中向上移动一个级别然后进入articles目录”或..\articles：

```java
@Test
public void givenSiblingPaths_whenCreatesPathToOther_thenCorrect() {
    Path p1 = Paths.get("articles");
    Path p2 = Paths.get("authors");

    Path p1_rel_p2 = p1.relativize(p2);
    Path p2_rel_p1 = p2.relativize(p1);
 
    assertEquals("..\\authors", p1_rel_p2.toString());
    assertEquals("..\\articles", p2_rel_p1.toString());
}
```

假设我们将articles目录移动到authors文件夹，这样他们就不再是同级。以下相对化操作涉及在tuyucheng和articles之间创建路径，反之亦然：

```java
@Test
public void givenNonSiblingPaths_whenCreatesPathToOther_thenCorrect() {
    Path p1 = Paths.get("/tuyucheng");
    Path p2 = Paths.get("/tuyucheng/authors/articles");

    Path p1_rel_p2 = p1.relativize(p2);
    Path p2_rel_p1 = p2.relativize(p1);
 
    assertEquals("authors\\articles", p1_rel_p2.toString());
    assertEquals("..\\..", p2_rel_p1.toString());
}
```

## 10. 比较路径

Path类具有equals方法的直观实现，使我们能够比较两个路径是否相等：

```java
@Test
public void givenTwoPaths_whenTestsEquality_thenCorrect() {
    Path p1 = Paths.get("/tuyucheng/articles");
    Path p2 = Paths.get("/tuyucheng/articles");
    Path p3 = Paths.get("/tuyucheng/authors");

    assertTrue(p1.equals(p2));
    assertFalse(p1.equals(p3));
}
```

你还可以检查路径是否以给定字符串开头：

```java
@Test
public void givenPath_whenInspectsStart_thenCorrect() {
    Path p1 = Paths.get("/tuyucheng/articles");
 
    assertTrue(p1.startsWith("/tuyucheng"));
}
```

或是否以其他字符串结尾：

```java
@Test
public void givenPath_whenInspectsEnd_thenCorrect() {
    Path p1 = Paths.get("/tuyucheng/articles");
  
    assertTrue(p1.endsWith("articles"));
}
```

## 11. 总结

在本文中，我们展示了作为Java 7的一部分发布的新文件系统API(NIO2)中的Path操作，并看到了其中的大部分操作。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-1)上获得。
