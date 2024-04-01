---
layout: post
title:  Java Instrumentation指南
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在本教程中，我们将讨论[Java Instrumentation API](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html)。它提供了将字节码添加到现有已编译Java类的能力。

我们还将讨论Java代理以及我们如何使用它们来检测我们的代码。

## 2. 设置

在整篇文章中，我们将使用检测构建应用程序。

我们的应用程序将包含两个模块：

1.  允许我们取款的ATM应用程序
2.  以及一个Java代理，它允许我们通过测量花费的时间来衡量ATM的性能

**Java代理将修改ATM字节码，使我们无需修改ATM应用程序即可测量取款时间**。

我们的项目将具有以下结构：

```xml
<groupId>cn.tuyucheng.taketoday.instrumentation</groupId>
<artifactId>base</artifactId>
<version>1.0.0</version>
<packaging>pom</packaging>

<modules>
    <module>agent</module>
    <module>application</module>
</modules>
```

在详细介绍检测之前，让我们看看什么是Java代理。

## 3. 什么是Java代理

通常，java代理只是一个特制的jar文件。**它利用JVM提供的[Instrumentation API](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html)来更改加载到JVM中的现有字节码**。

为了让代理工作，我们需要定义两个方法：

-   premain：将在JVM启动时使用-javaagent参数静态加载代理
-   agentmain：将使用[Java Attach API](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html)将代理动态加载到JVM中

要记住的一个有趣概念是，JVM实现(如Oracle、OpenJDK等)可以提供动态启动代理的机制，但这不是必需的。

首先，让我们看看如何使用现有的Java代理。

之后，我们将研究如何从头开始创建一个以在我们的字节码中添加我们需要的功能。

## 4. 加载Java代理

为了能够使用Java代理，我们必须首先加载它。

我们有两种类型的加载：

-   静态：使用premain使用-javaagent选项加载代理
-   动态：利用agentmain通过[Java Attach API](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html)将代理加载到JVM中

接下来，我们将了解每种类型的加载并解释其工作原理。

### 4.1 静态加载

在应用程序启动时加载Java代理称为静态加载。**静态加载在执行任何代码之前在启动时修改字节码**。

请记住，静态加载使用premain方法，该方法将在任何应用程序代码运行之前运行，为了让它运行，我们可以执行：

```bash
java -javaagent:agent.jar -jar application.jar
```

需要注意的是，我们应该始终将–javaagent参数放在–jar参数之前。

以下是我们命令的日志：

```text
22:24:39.296 [main] INFO - [Agent] In premain method
22:24:39.300 [main] INFO - [Agent] Transforming class MyAtm
22:24:39.407 [main] INFO - [Application] Starting ATM application
22:24:41.409 [main] INFO - [Application] Successful Withdrawal of [7] units!
22:24:41.410 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
22:24:53.411 [main] INFO - [Application] Successful Withdrawal of [8] units!
22:24:53.411 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
```

我们可以看到premain方法何时运行以及MyAtm类何时被转换。我们还看到两条ATM取款交易日志，其中包含完成每个操作所花费的时间。

请记住，在我们的原始应用程序中，我们没有这个交易完成时间，它是由我们的Java代理添加的。

### 4.2 动态加载

**将Java代理加载到已运行的JVM中的过程称为动态加载**。代理是使用[Java Attach API](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html)附加的。

一个更复杂的场景是，当我们的ATM应用程序已经在生产环境中运行时，我们希望动态地添加交易的总时间，而不会使我们的应用程序停机。

让我们编写一小段代码来做到这一点，我们将这个类称为AgentLoader。为简单起见，我们将把这个类放在应用程序jar文件中。因此我们的应用程序jar文件既可以启动我们的应用程序，又可以将我们的代理附加到ATM应用程序：

```java
VirtualMachine jvm = VirtualMachine.attach(jvmPid);
jvm.loadAgent(agentFile.getAbsolutePath());
jvm.detach();
```

现在我们有了AgentLoader，我们启动应用程序，确保在交易之间的十秒暂停中，我们将使用AgentLoader动态附加我们的Java代理。

我们还要添加胶水，使我们能够启动应用程序或加载代理。

我们将这个类称为Launcher，它将成为我们的主要jar文件类：

```java
public class Launcher {
    public static void main(String[] args) throws Exception {
        if (args[0].equals("StartMyAtmApplication")) {
            new MyAtmApplication().run(args);
        } else if (args[0].equals("LoadAgent")) {
            new AgentLoader().run(args);
        }
    }
}
```

- 启动应用程序

    ```text
    java -jar application.jar StartMyAtmApplication
    22:44:21.154 [main] INFO - [Application] Starting ATM application
    22:44:23.157 [main] INFO - [Application] Successful Withdrawal of [7] units!
    ```

- 附加Java代理：

    第一次操作后，我们将Java代理附加到我们的JVM：

    ```text
    java -jar application.jar LoadAgent
    22:44:27.022 [main] INFO - Attaching to target JVM with PID: 6575
    22:44:27.306 [main] INFO - Attached to target JVM and loaded Java agent successfully
    ```

- 检查应用程序日志

    现在我们将代理附加到JVM，我们将看到第二次ATM取款操作的总完成时间。
    
    这意味着我们在应用程序运行时动态添加了功能：
    
    ```bash
    22:44:27.229 [Attach Listener] INFO - [Agent] In agentmain method
    22:44:27.230 [Attach Listener] INFO - [Agent] Transforming class MyAtm
    22:44:33.157 [main] INFO - [Application] Successful Withdrawal of [8] units!
    22:44:33.157 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
    ```

## 5. 创建Java代理

在学习了如何使用代理之后，让我们看看如何创建一个代理。我们将了解如何使用[Javassist](https://www.baeldung.com/javassist)更改字节码，并将其与一些检测API方法相结合。

由于Java代理使用[Java Instrumentation API](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html)，因此在深入创建代理之前，让我们先看看这个API中一些最常用的方法以及它们的作用的简短描述：

-   addTransformer：向检测引擎添加一个转换器
-   getAllLoadedClasses：返回JVM当前加载的所有类的数组
-   retransformClasses：通过添加字节码促进已加载类的检测
-   removeTransformer：注销提供的转换器
-   redefineClasses：使用提供的类文件重新定义提供的类集，这意味着该类将被完全替换，而不是像retransformClasses那样被修改

### 5.1 创建Premain和Agentmain方法

我们知道每个Java代理至少需要一个premain或agentmain方法。后者用于动态加载，而前者用于将Java代理静态加载到JVM中。

让我们在我们的代理中定义它们，以便我们能够静态和动态地加载这个代理：

```java
public static void premain(String agentArgs, Instrumentation inst) {
    LOGGER.info("[Agent] In premain method");
    String className = "cn.tuyucheng.taketoday.instrumentation.application.MyAtm";
    transformClass(className,inst);
}

public static void agentmain(String agentArgs, Instrumentation inst) {
    LOGGER.info("[Agent] In agentmain method");
    String className = "cn.tuyucheng.taketoday.instrumentation.application.MyAtm";
    transformClass(className,inst);
}
```

在每个方法中，我们声明要更改的类，然后使用transformClass方法向下挖掘以转换该类。

下面是我们定义的用于帮助我们转换MyAtm类的transformClass方法的代码。

在这个方法中，我们找到我们想要转换的类并使用transform方法。此外，我们将转换器添加到检测引擎中：

```java
private static void transformClass(String className, Instrumentation instrumentation) {
    Class<?> targetCls = null;
    ClassLoader targetClassLoader = null;
    // see if we can get the class using forName
    try {
        targetCls = Class.forName(className);
        targetClassLoader = targetCls.getClassLoader();
        transform(targetCls, targetClassLoader, instrumentation);
        return;
    } catch (Exception ex) {
        LOGGER.error("Class [{}] not found with Class.forName");
    }
    // otherwise iterate all loaded classes and find what we want
    for(Class<?> clazz: instrumentation.getAllLoadedClasses()) {
        if(clazz.getName().equals(className)) {
            targetCls = clazz;
            targetClassLoader = targetCls.getClassLoader();
            transform(targetCls, targetClassLoader, instrumentation);
            return;
        }
    }
    throw new RuntimeException("Failed to find class [" + className + "]");
}

private static void transform(Class<?> clazz, ClassLoader classLoader, Instrumentation instrumentation) {
    AtmTransformer dt = new AtmTransformer(clazz.getName(), classLoader);
    instrumentation.addTransformer(dt, true);
    try {
        instrumentation.retransformClasses(clazz);
    } catch (Exception ex) {
        throw new RuntimeException("Transform failed for: [" + clazz.getName() + "]", ex);
    }
}
```

有了这个，让我们为MyAtm类定义转换器。

### 5.2 定义我们的转换器

类转换器必须实现ClassFileTransformer并实现transform方法。

我们将使用[Javassist](https://www.baeldung.com/javassist)将字节码添加到MyAtm类，并添加包含ATW取款交易总时间的日志：

```java
public class AtmTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(
            ClassLoader loader,
            String className,
            Class<?> classBeingRedefined,
            ProtectionDomain protectionDomain,
            byte[] classfileBuffer) {
        byte[] byteCode = classfileBuffer;
        String finalTargetClassName = this.targetClassName
                .replaceAll("\\.", "/");
        if (!className.equals(finalTargetClassName)) {
            return byteCode;
        }

        if (className.equals(finalTargetClassName) && loader.equals(targetClassLoader)) {
            LOGGER.info("[Agent] Transforming class MyAtm");
            try {
                ClassPool cp = ClassPool.getDefault();
                CtClass cc = cp.get(targetClassName);
                CtMethod m = cc.getDeclaredMethod(WITHDRAW_MONEY_METHOD);
                m.addLocalVariable("startTime", CtClass.longType);
                m.insertBefore("startTime = System.currentTimeMillis();");

                StringBuilder endBlock = new StringBuilder();

                m.addLocalVariable("endTime", CtClass.longType);
                m.addLocalVariable("opTime", CtClass.longType);
                endBlock.append("endTime = System.currentTimeMillis();");
                endBlock.append("opTime = (endTime-startTime)/1000;");

                endBlock.append("LOGGER.info(\"[Application] Withdrawal operation completed in:" + "\" + opTime + \" seconds!\");");

                m.insertAfter(endBlock.toString());

                byteCode = cc.toBytecode();
                cc.detach();
            } catch (NotFoundException | CannotCompileException | IOException e) {
                LOGGER.error("Exception", e);
            }
        }
        return byteCode;
    }
}
```

### 5.3 创建代理Manifest文件

最后，为了获得一个工作的Java代理，我们需要一个包含几个属性的清单文件。

因此，我们可以在[Instrumentation包](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/package-summary.html)官方文档中找到清单属性的完整列表。

在最终的Java代理jar文件中，我们将以下行添加到清单文件中：

```manifest
Agent-Class: cn.tuyucheng.taketoday.instrumentation.agent.MyInstrumentationAgent
Can-Redefine-Classes: true
Can-Retransform-Classes: true
Premain-Class: cn.tuyucheng.taketoday.instrumentation.agent.MyInstrumentationAgent
```

我们的Java检测代理现已完成。要运行它，请参阅本文的[加载Java代理](https://www.baeldung.com/java-instrumentation#loading-a-java-agent)部分。

## 6. 总结

在本文中，我们讨论了Java Instrumentation API。我们研究了如何静态和动态地将Java代理加载到JVM中。

我们还研究了如何从头开始创建我们自己的Java代理。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-1)上获得。