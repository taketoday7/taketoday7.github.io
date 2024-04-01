---
layout: post
title:  如何在Java中运行Shell命令
category: java
copyright: java
excerpt: Java OS
---

## 1. 概述

在本文中，我们将学习如何**从Java应用程序执行shell命令**。

首先，我们将使用Runtime类提供的exec()方法。然后，我们将了解更可自定义的ProcessBuilder。

## 2. 操作系统依赖性

**Shell命令依赖于操作系统，因为它们的行为因系统而异**。因此，在我们创建任何进程来运行我们的shell命令之前，我们需要了解运行JVM的操作系统。

此外，在Windows上，shell通常称为cmd.exe。相反，在Linux和MacOS上，shell命令是使用/bin/sh运行的。为了在这些不同的机器上兼容，我们可以通过编程方式附加cmd.exe(如果在Windows机器上)或/bin/sh。例如，我们可以通过读取System类的“os.name”属性来检查运行代码的机器是否是Windows机器：

```java
boolean isWindows = System.getProperty("os.name")
    .toLowerCase().startsWith("windows");
```

## 3. 输入输出

通常，我们需要连接进程的输入和输出流。具体来说，InputStream充当标准输入，OutputStream充当进程的标准输出。**我们必须始终使用输出流**，否则，我们的进程将不会返回并将永远挂起。

让我们实现一个名为StreamGobbler的常用类，它使用一个InputStream：

```java
private static class StreamGobbler implements Runnable {
    private InputStream inputStream;
    private Consumer<String> consumer;

    public StreamGobbler(InputStream inputStream, Consumer<String> consumer) {
        this.inputStream = inputStream;
        this.consumer = consumer;
    }

    @Override
    public void run() {
        new BufferedReader(new InputStreamReader(inputStream)).lines().forEach(consumer);
    }
}
```

此类实现了Runnable接口，这意味着任何[Executor](https://www.baeldung.com/java-executor-service-tutorial)都可以执行它。

## 4. Runtime.exec()

接下来，我们将使用exec()方法生成一个新进程，并使用之前创建的StreamGobbler。

例如，我们可以列出用户主目录中的所有目录，然后将其打印到控制台：

```java
String homeDirectory = System.getProperty("user.home");
Process process;
if (isWindows) {
    process = Runtime.getRuntime()
        .exec(String.format("cmd.exe /c dir %s", homeDirectory));
} else {
    process = Runtime.getRuntime()
        .exec(String.format("/bin/sh -c ls %s", homeDirectory));
}
StreamGobbler streamGobbler = new StreamGobbler(process.getInputStream(), System.out::println);
Future<?> future = Executors.newSingleThreadExecutor().submit(streamGobbler);

int exitCode = process.waitFor();
assert exitCode == 0;

future.get(); // waits for streamGobbler to finish
```

在这里，我们使用newSingleThreadExecutor()创建了一个新的子进程，然后使用submit()运行包含shell命令的进程。此外，submit()返回一个[Future](https://www.baeldung.com/guava-futures-listenablefuture#1-future)对象，我们利用它来检查进程的结果。另外，请确保在返回的对象上调用get()方法以等待计算完成。

> **注意：JDK 18弃用了Runtime类中的exec(String command)**。

### 4.1 管道处理

目前，无法使用exec()处理管道。幸运的是，管道是shell的一个特性。因此，我们可以在要使用管道的地方创建整个命令并将其传递给exec()：

```java
if (IS_WINDOWS) {
    process = Runtime.getRuntime()
        .exec(String.format("cmd.exe /c dir %s | findstr \"Desktop\"", homeDirectory));
} else {
    process = Runtime.getRuntime()
        .exec(String.format("/bin/sh -c ls %s | grep \"Desktop\"", homeDirectory));
}
```

在这里，我们列出了用户家目录中的所有目录并搜索“Desktop”文件夹。

## 5. ProcessBuilder

**或者，我们可以使用[ProcessBuilder](https://www.baeldung.com/java-lang-processbuilder-api)，它优于Runtime方法**，因为我们可以自定义它而不是仅仅运行一个字符串命令。

简而言之，通过这种方法，我们能够：

-   使用directory()更改shell命令正在运行的工作目录
-   通过向environment()提供键值映射来更改环境变量
-   以自定义方式重定向输入和输出流
-   使用inheritIO()将它们都继承到当前JVM进程的流中

同样，我们可以运行与上一个示例相同的shell命令：

```java
ProcessBuilder builder = new ProcessBuilder();
if (isWindows) {
    builder.command("cmd.exe", "/c", "dir");
} else {
    builder.command("sh", "-c", "ls");
}
builder.directory(new File(System.getProperty("user.home")));
Process process = builder.start();
StreamGobbler streamGobbler = new StreamGobbler(process.getInputStream(), System.out::println);
Future<?> future = Executors.newSingleThreadExecutor().submit(streamGobbler);
int exitCode = process.waitFor();
assert exitCode == 0;
future.get(10, TimeUnit.SECONDS)
```

## 6. 总结

可以看到，我们可以通过两种不同的方式在Java中执行shell命令。

通常，如果我们计划自定义生成进程的执行，例如，更改其工作目录，我们应该考虑使用ProcessBuilder。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-os)上获得。