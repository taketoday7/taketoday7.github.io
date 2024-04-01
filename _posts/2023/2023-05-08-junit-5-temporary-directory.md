---
layout: post
title:  JUnit 5临时目录支持
category: unittest
copyright: unittest
excerpt: JUnit 5 @TempDir
---

## 1. 概述

在测试时，我们经常需要访问临时文件。但是，自己管理这些文件的创建和删除可能很麻烦。

在这个快速教程中，**我们将了解JUnit 5如何通过提供TempDirectory扩展来缓解这种情况**。

有关使用JUnit进行测试的深入介绍，请阅读[JUnit 5指南](https://www.baeldung.com/junit-5)。

## 2. TempDirectory扩展

从5.4.2版本开始，JUnit 5提供了TempDirectory扩展。但是，重要的是要注意，官方上这仍然是一个[实验性](https://apiguardian-team.github.io/apiguardian/docs/1.0.0/api/org/apiguardian/api/API.Status.html?is-external=true#EXPERIMENTAL)功能，因此鼓励我们向JUnit团队提供反馈。

正如我们稍后将看到的，**我们可以使用此扩展为单个测试或测试类中的所有测试创建和清理临时目录**。

通常在使用[扩展](https://www.baeldung.com/junit-5-extensions)时，我们需要使用@ExtendWith注解从JUnit 5测试中注册它。但是，对于默认情况下内置并注册的TempDirectory扩展，这不是必需的。

## 3. Maven依赖

首先，让我们添加示例所需的项目依赖项。

除了主要的JUnit 5依赖junit-jupiter-engine，我们还需要junit-jupiter-api库：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

与往常一样，我们可以从[Maven Central](https://central.sonatype.com/artifact/org.junit.jupiter/junit-jupiter-api/5.9.3)获取最新版本。

除此之外，我们还需要添加junit-jupiter-params依赖项：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

同样，我们可以在[Maven Central](https://central.sonatype.com/artifact/org.junit.jupiter/junit-jupiter-params/5.9.3)中找到最新版本。

## 4. 使用@TempDir注解

为了使用TempDirectory扩展，**我们需要使用@TempDir注解**。我们只能将此注解与以下两种类型一起使用：

+ java.io.file.Path
+ java.io.File

事实上，如果我们尝试将它与不同的类型一起使用，则会抛出org.junit.jupiter.api.extension.ParameterResolutionException。

接下来，让我们探讨使用此注解的几种不同方法。

### 4.1 @TempDir作为方法参数

让我们首先看看如何**将带有@TempDir注解的参数注入到单个测试方法中**：

```java
@Test
void givenTestMethodWithTempDirectoryPath_whenWriteToFile_thenContentIsCorrect(@TempDir Path tempDir) throws IOException {
    Path numbers = tempDir.resolve("numbers.txt");

    List<String> lines = Arrays.asList("1", "2", "3");
    Files.write(numbers, lines);

    assertAll(
        () -> assertTrue(Files.exists(numbers), "File should exist"),
        () -> assertLinesMatch(lines, Files.readAllLines(numbers)));
}
```

如我们所见，我们的测试方法在临时目录tempDir中创建了一个名为numbers.txt的文件，并向其写入一些内容。

然后我们检查文件是否存在以及内容是否与最初写入的内容相匹配。

### 4.2 @TempDir标注实例变量

在下一个示例中，我们将使用@TempDir注解在测试类中标注一个字段：

```java
@TempDir
File anotherTempDir;

@Test
void givenFieldWithTempDirectoryFile_whenWriteToFile_thenContentIsCorrect() throws IOException {
    assertTrue(this.anotherTempDir.isDirectory(), "Should be a directory ");

    File letters = new File(anotherTempDir, "letters.txt");
    List<String> lines = Arrays.asList("x", "y", "z");

    Files.write(letters.toPath(), lines);

    assertAll(
        () -> assertTrue(Files.exists(letters.toPath()), "File should exist"),
        () -> assertLinesMatch(lines, Files.readAllLines(letters.toPath())));
}
```

这一次，我们使用java.io.File作为临时目录。同样，我们写入一些内容并检查它们是否成功写入。

**如果我们在其他测试方法中再次使用这个anotherTempDir变量，那么每个测试都将使用自己的临时目录**。

### 4.3 共享临时目录

有时，我们可能希望**在测试方法之间共享一个临时目录**。

我们可以通过将字段声明为static来做到这一点：

```java
@TestMethodOrder(OrderAnnotation.class)
class SharedTemporaryDirectoryUnitTest {

    @TempDir
    static Path sharedTempDir;

    @Test
    @Order(1)
    void givenFieldWithSharedTempDirectoryPath_whenWriteToFile_thenContentIsCorrect() throws IOException {
        Path numbers = sharedTempDir.resolve("numbers.txt");

        List<String> lines = Arrays.asList("1", "2", "3");
        Files.write(numbers, lines);

        assertAll(
            () -> assertTrue(Files.exists(numbers), "File should exist"),
            () -> assertLinesMatch(lines, Files.readAllLines(numbers)));

        Files.createTempDirectory("bpb");
    }

    @Test
    @Order(2)
    void givenAlreadyWrittenToSharedFile_whenCheckContents_thenContentIsCorrect() throws IOException {
        Path numbers = sharedTempDir.resolve("numbers.txt");

        assertLinesMatch(Arrays.asList("1", "2", "3"), Files.readAllLines(numbers));
    }
}
```

**这里的关键点是我们使用一个静态字段sharedTempDir在两个测试方法之间共享它**。

在第一个测试中，我们将一些内容写入名为numbers.txt的文件中。然后我们在下一个测试中检查文件和内容是否已经存在。

我们还通过@Order注解强制[测试执行的顺序](https://www.baeldung.com/junit-5-test-order)，以确保行为始终一致。

### 4.4 清理选项

通常，@TempDir创建的临时目录会在测试执行后自动删除。但是，在某些情况下，我们可能希望保留临时目录以用于调试目的。

例如，如果测试失败，我们可能想检查临时目录的内容，看看是否有任何关于失败原因的线索。在这种情况下，JUnit 5自动删除临时目录可能是不可取的。

为了解决这个问题，Junit5为@TempDir注解提供了一个cleanup选项。**cleanup选项可用于指定是否应在测试方法结束时自动删除临时目录**。

cleanup选项可以设置为以下CleanupMode值之一：

-   ALWAYS：该选项指定无论测试是成功还是失败，都应始终在测试方法结束时自动删除临时目录。这是默认模式。
-   ON_SUCCESS：只有在测试成功时，才会在测试方法执行后删除临时目录。
-   NEVER：执行测试方法后不会自动删除临时目录。

接下来，让我们创建一个测试类来验证我们是否将NEVER设置为cleanup选项的值，测试执行后临时目录不会被删除：

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class TemporaryDirectoryWithCleanupUnitTest {

    private Path theTempDirToBeChecked = null;

    @Test
    @Order(1)
    void whenTestMethodWithTempDirNeverCleanup_thenSetInstanceVariable(@TempDir(cleanup = NEVER) Path tempDir) {
        theTempDirToBeChecked = tempDir;
        System.out.println(tempDir.toFile().getAbsolutePath());
    }

    @Test
    @Order(2)
    void whenTestMethodWithTempDirNeverCleanup_thenTempDirShouldNotBeRemoved() {
        assertNotNull(theTempDirToBeChecked);
        assertTrue(theTempDirToBeChecked.toFile().isDirectory());
    }
}
```

如上面的类所示，我们将[@TestInstance(TestInstance.Lifecycle.PER_CLASS)](https://www.baeldung.com/junit-testinstance-annotation#test-instance)注解添加到测试类中，以便JUnit仅创建测试类的一个实例并将其用于所有测试。这是因为我们将在第一个测试中设置theTempDirToBeChecked实例变量，并在第二个测试中验证临时目录是否仍然存在。

如果我们运行测试，它就会通过。因此它表明在第二个测试运行期间，第一个测试中创建的临时目录没有被删除。

此外，我们可以看到第一个测试的输出：

```text
/tmp/junit16624071649911791120
```

而JUnit 5的INFO日志提醒我们，第一次测试后临时目录不会被删除：

```shell
INFO: Skipping cleanup of temp dir /tmp/junit16624071649911791120 due to cleanup mode configuration.
```

在所有测试执行之后，如果我们检查文件系统上的临时目录，该目录仍然存在：

```shell
$ ls -ld /tmp/junit16624071649911791120
drwx------ 2 kent kent 40 Apr  1 18:23 /tmp/junit16624071649911791120/
```

## 5. 陷阱

现在让我们回顾一下使用TempDirectory扩展时应该注意的一些细微之处。

### 5.1 创建

好奇的读者可能想知道这些临时文件实际上是在哪里创建的？

实际上，JUnit TemporaryDirectory类在内部使用了Files.createTempDirectory(String prefix)方法。**同样，此方法使用默认的系统临时文件目录**。

这通常在环境变量TEMP中指定：

```text
TEMP=C:\Users\27597\AppData\Local\Temp\
```

例如，在我的Windows电脑上临时文件的位置为：

```text
C:\Users\27597\AppData\Local\Temp\junit5311844216996678616\numbers.txt
```

同时，如果无法创建临时目录，则会根据需要抛出ExtensionConfigurationException，或者之前所说的ParameterResolutionException。

### 5.2 删除

cleanup属性的默认值是ALWAYS。也就是说，当测试方法或类执行完毕且临时目录超出作用域时，**JUnit框架将尝试递归删除该目录中的所有文件和目录，最后删除临时目录本身**。

如果在此删除阶段出现问题，将抛出IOException，并且测试或测试类将失败。

## 6. 总结

在本教程中，我们探讨了JUnit 5提供的TempDirectory扩展。

首先，我们介绍了这个扩展并了解了使用它需要哪些Maven依赖项。接下来，我们演示了几个如何在单元测试中使用该扩展的示例。

最后，有些值得需要注意的地方，即包括临时文件的创建位置以及删除过程中发生的情况。

与往常一样，本文的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/junit-5-basics)上找到。