---
layout: post
title:  Java中的Runtime.getRuntime().halt()与System.exit()
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在本教程中，我们将研究[System.exit()](https://www.baeldung.com/java-system-exit)、Runtime.getRuntime().halt()，以及这两种方法的相互比较。

## 2. System.exit()

**System.exit()方法停止正在运行的Java虚拟机**。但是，在停止JVM之前，它会调用关闭序列，也称为**有序关闭**。请参阅[本文](https://www.baeldung.com/adding-shutdown-hooks-for-jvm-applications)以了解有关添加关机钩子的更多信息。

JVM的关闭序列首先调用所有已注册的关机钩子并等待它们完成。然后，如果启用了finalization-on-exit，它会运行所有未调用的终结器。最后，它停止JVM。

该方法实际上在内部调用了Runtime.getRuntime().exit()方法。它以整数状态代码作为参数并具有void返回类型：

```java
public static void exit(int status)
```

如果状态码不为0，则表明程序异常停止。

## 3. Runtime.getRuntime().halt()

Runtime类允许应用程序与应用程序运行的环境进行交互。

它有一个halt方法，可用于**强行终止正在运行的JVM**。

与exit方法不同，此方法不会触发JVM关闭序列。因此，当我们调用halt方法时，**关机钩子或终结器都不会执行**。

此方法是非静态的，并且具有与System.exit()类似的签名：

```java
public void halt(int status)
```

与exit类似，该方法中的非0状态码也表示程序异常终止。

## 4. 示例

现在，让我们在关机钩子的帮助下看一个exit和halt方法的示例。

为简单起见，我们将创建一个Java类并在静态块中注册一个关机钩子。此外，我们将创建两个方法；一个调用exit方法，另一个调用halt方法：

```java
public class JvmExitAndHaltDemo {

    private static Logger LOGGER = LoggerFactory.getLogger(JvmExitAndHaltDemo.class);

    static {
        Runtime.getRuntime()
                .addShutdownHook(new Thread(() -> {
                    LOGGER.info("Shutdown hook initiated.");
                }));
    }

    public void processAndExit() {
        process();
        LOGGER.info("Calling System.exit().");
        System.exit(0);
    }

    public void processAndHalt() {
        process();
        LOGGER.info("Calling Runtime.getRuntime().halt().");
        Runtime.getRuntime().halt(0);
    }

    private void process() {
        LOGGER.info("Process started.");
    }
}
```

因此，为了首先测试exit方法，让我们创建一个测试用例：

```java
@Test
public void givenProcessComplete_whenExitCalled_thenTriggerShutdownHook() {
    jvmExitAndHaltDemo.processAndExit();
}
```

现在让我们运行测试用例，看看关机钩子被调用：

```text
12:48:43.156 [main] INFO cn.tuyucheng.taketoday.exitvshalt.JvmExitAndHaltDemo - Process started.
12:48:43.159 [main] INFO cn.tuyucheng.taketoday.exitvshalt.JvmExitAndHaltDemo - Calling System.exit().
12:48:43.160 [Thread-0] INFO cn.tuyucheng.taketoday.exitvshalt.JvmExitAndHaltDemo - Shutdown hook initiated.
```

同样，我们将为halt方法创建一个测试用例：

```java
@Test
public void givenProcessComplete_whenHaltCalled_thenDoNotTriggerShutdownHook() {
    jvmExitAndHaltDemo.processAndHalt();
}
```

现在，我们也可以运行这个测试用例，并看到没有调用关机钩子：

```text
12:49:16.839 [main] INFO cn.tuyucheng.taketoday.exitvshalt.JvmExitAndHaltDemo - Process started.
12:49:16.842 [main] INFO cn.tuyucheng.taketoday.exitvshalt.JvmExitAndHaltDemo - Calling Runtime.getRuntime().halt().
```

## 5. 何时使用exit和halt

正如我们之前看到的，System.exit()方法触发JVM的关闭序列，而Runtime.getRuntime().halt()突然终止JVM。

我们也可以通过使用操作系统命令来做到这一点。例如，我们可以使用SIGINT或Ctrl+C来触发有序关闭，如System.exit()和SIGKILL来突然杀死JVM进程。

因此，我们很少需要使用这些方法。话虽如此，当我们需要JVM运行已注册的关机钩子或向调用者返回特定状态代码时，我们可能需要使用exit方法，例如使用shell脚本。

但是，需要注意的是，如果设计不当，关机钩子可能会导致死锁。因此，**exit方法在等待注册的关机钩子完成时可能会被阻塞**。因此，解决这个问题的一种可能方法是使用halt方法强制JVM停止，以防退出阻塞。

最后，应用程序还可以限制这些方法的意外使用。这两个方法都调用[SecurityManager](https://www.baeldung.com/java-security-manager)类的checkExit方法。因此，要禁止exit和halt操作，应用程序可以使用SecurityManager类创建安全策略并从checkExit方法中抛出SecurityException。

## 6. 总结

在本教程中，我们借助示例研究了System.exit()和Runtime.getRuntime().halt()方法。此外，我们还讨论了这些方法的用法和最佳实践。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-1)上获得。