---
layout: post
title:  JMX基本介绍
category: java
copyright: java
excerpt: Java Performance
---

## 1. 概述

**Java Management Extensions(JMX)框架是在Java 1.5中引入的**，自一开始就得到了Java开发人员社区的广泛认可。

它提供了一个易于配置、可扩展、可靠且或多或少友好的基础架构，用于在本地或远程管理Java应用程序。该框架引入了用于实时管理应用程序的MBean的概念。

本文是初学者的分步指南，用于创建和设置基本MBean并通过JConsole管理它。

## 2. JMX架构

JMX架构遵循三层方法：

1.  **检测层**：向JMX代理注册的MBean，通过它管理资源
2.  **JMX代理层**：核心组件(MbeanServer)，维护托管MBean的注册表并提供访问它们的接口
3.  **远程管理层**：通常是客户端工具，如JConsole

## 3. 创建MBean类

在创建MBean时，我们必须遵循一种特定的设计模式。**模型MBean类必须实现具有以下名称的接口：“模型类名称”加上MBean**。

因此，让我们定义我们的MBean接口和实现它的类：

```java
public interface GameMBean {

    void playFootball(String clubName);

    String getPlayerName();

    void setPlayerName(String playerName);
}

public class Game implements GameMBean {

    private String playerName;

    @Override
    public void playFootball(String clubName) {
        System.out.println(this.playerName + " playing football for " + clubName);
    }

    @Override
    public String getPlayerName() {
        System.out.println("Return playerName " + this.playerName);
        return playerName;
    }

    @Override
    public void setPlayerName(String playerName) {
        System.out.println("Set playerName to value " + playerName);
        this.playerName = playerName;
    }
}
```

Game类重写了父接口的playFootball()方法。除此之外，该类还有一个成员变量playerName和它的getter/setter。

请注意，getter/setter也在父接口中声明。

## 4. 使用JMX代理进行检测

JMX代理是本地或远程运行的实体，它们提供对向其注册的MBean的管理访问。

让我们使用PlatformMbeanServer-JMX代理的核心组件并向其注册GameMBean。

我们将使用另一个实体ObjectName向PlatformMbeanServer注册Game类实例；这是一个由两部分组成的字符串：

-   **domain**：可以是任意字符串，但根据MBean命名约定，它应该具有Java包名称(避免命名冲突)
-   **key**：由逗号分隔的“key=value”对列表

在这个例子中，我们将使用：“cn.tuyucheng.taketoday.tutorial:type=basic,name=game”。

我们将从工厂类java.lang.management.ManagementFactory中获取MBeanServer。

然后我们将使用创建的ObjectName注册模型MBean：

```java
try {
    ObjectName objectName = new ObjectName("cn.tuyucheng.taketoday.tutorial:type=basic,name=game");
    MBeanServer server = ManagementFactory.getPlatformMBeanServer();
    server.registerMBean(new Game(), objectName);
} catch (MalformedObjectNameException | InstanceAlreadyExistsException | MBeanRegistrationException | NotCompliantMBeanException e) {
    // handle exceptions
}
```

最后，为了能够对其进行测试-我们将添加一个while循环以防止应用程序在我们可以通过JConsole访问MBean之前终止：

```java
while (true) {
}
```

## 5. 访问MBean

### 5.1 从客户端连接

1.  在Eclipse中启动应用程序
2.  启动JConsole(位于机器JDK安装目录bin文件夹下)
3.  Connection -> new Connection -> 选择本教程的本地Java进程 -> Connect -> Insecure SSl connection warning -> Continue with insecure connection
4.  建立连接后，单击“View”窗格右上角的“MBean”选项卡
5.  已注册的MBean列表将显示在左栏中
6.  点击cn.tuyucheng.taketoday.tutorial -> basic -> game
7.  game下面会有两行，属性和操作各一行

以下是该进程的JConsole部分的快速浏览：

[![编辑的 jmx 教程](https://www.baeldung.com/wp-content/uploads/2016/12/edited_jmx_tutorial-1024x575.gif)](https://www.baeldung.com/wp-content/uploads/2016/12/edited_jmx_tutorial.gif)

### 5.2 管理MBean

MBean管理的基础很简单：

-   属性可以读或写
-   可以调用方法并向它们提供参数或从它们返回值

让我们看看这对GameMBean在实践中意味着什么：

-   **attribute**：为playerName属性键入一个新值-例如“Messi”，然后单击**Refresh按钮**

以下日志将出现在Eclipse控制台中：

Set playerName to value Messi

-   **operations**：为方法playFootBall()的字符串参数键入一个值-例如“Barcelona”，然后单击method按钮，**将出现成功调用的窗口警报**

Eclipse控制台中会出现如下日志：

Messi playing football for Barcelona

## 6. 总结

本教程介绍了使用MBean设置支持JMX的应用程序的基础知识。此外，还讨论了如何使用典型的客户端工具(如JConsole)来管理经过检测的MBean。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-perf-1)上获得。