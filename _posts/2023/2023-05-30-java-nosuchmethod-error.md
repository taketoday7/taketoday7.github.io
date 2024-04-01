---
layout: post
title:  Java中的NoSuchMethodError
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将了解java.lang.NoSuchMethodError以及一些处理它的方法。

## 2. NoSuchMethodError

顾名思义，**NoSuchMethodError在未找到特定方法时发生**。此方法可以是实例方法或静态方法。

**在大多数情况下，我们能够在编译时捕获此错误**。因此，这不是一个大问题。但是，有时它可能会在运行时抛出，然后发现它变得有点困难。根据[Oracle文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/NoSuchMethodError.html)，如果类已被不兼容地更改，则此错误可能会在运行时发生。

因此，在以下情况下我们可能会遇到此错误。首先，**如果我们只对代码进行部分重新编译**。其次，**如果我们的应用程序中的依赖项存在版本不兼容**，例如外部jar。

请注意，NoSuchMethodError继承树包括[IncompatibleClassChangeError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/IncompatibleClassChangeError.html)和[LinkageError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/LinkageError.html)。这些错误与编译后不兼容的类更改有关。

## 3. NoSuchMethodError示例

让我们通过一个例子看看这个错误的实际效果。为此，我们将创建两个类。首先是SpecialToday，它将列出餐厅当天的特色菜： 


```java
public class SpecialToday {
    private static String desert = "Chocolate Cake";

    public static String getDesert() {
        return desert;
    }
}
```

第二个类MainMenu调用来自SpecialsToday的方法：


```java
public class MainMenu {
    public static void main(String[] args) {
        System.out.println("Today's Specials: " + getSpecials());
    }

    public static String getSpecials() {
        return SpecialToday.getDesert();
    }
}
```

这里的输出将是：

```text
Today's Specials: Chocolate Cake
```

接下来，我们将删除SpecialToday中的方法getDesert()并仅重新编译这个更新后的类。这次当我们运行MainMenu时，我们注意到以下运行时错误：

```text
Exception in thread "main" java.lang.NoSuchMethodError: SpecialToday.getDesert()Ljava/lang/String;
```

## 4. 如何处理NoSuchMethodError

现在让我们看看如何处理这个问题。对于上面的代码，让我们**做一个完整的清理编译**，包括这两个类。我们会注意到在编译时会捕获错误。**如果我们使用像Eclipse这样的IDE，它会在我们更新SpecialsToday时更早地被检测到**。

因此，如果我们的应用程序遇到此错误，作为第一步，我们将进行完全干净的编译。使用Maven，我们将运行mvn clean install命令。

有时问题出在我们应用程序的外部依赖项上。在这种情况下，我们将首先检查类路径加载器拉取的构建路径中jar的顺序。我们将跟踪并更新不一致的jar。

但是，如果我们在运行时仍然遇到这个错误，我们将不得不深入挖掘。我们必须**确保编译时和运行时类和jar具有相同的版本**。为此，我们可以**使用-verbose:class选项运行应用程序**来检查加载的类。我们可以按如下方式运行命令：

```text
$ java -verbose:class cn.tuyucheng.taketoday.exceptions.nosuchmethoderror.MainMenu
[0.014s][info][class,load] opened: /usr/lib/jvm/java-11-openjdk-amd64/lib/modules
[0.015s][info][class,load] opened: /usr/share/java/java-atk-wrapper.jar
[0.028s][info][class,load] java.lang.Object source: shared objects file
[0.028s][info][class,load] java.io.Serializable source: shared objects file
```

使用有关在运行时加载到各个jar中的所有类的信息，我们可以跟踪不兼容的依赖项。

我们还应该**确保两个或多个jar中没有重复的类**。在大多数情况下，Maven将帮助直接控制冲突的依赖项。此外，我们可以运行mvn dependency:tree命令来获取我们项目的依赖树，如下所示：

```text
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] -------------< cn.tuyucheng.taketoday.exceptions:nosuchmethoderror >--------------
[INFO] Building nosuchmethoderror 1.0.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ nosuchmethoderror ---
[INFO] cn.tuyucheng.taketoday.exceptions:nosuchmethoderror:jar:1.0.0
[INFO] \- org.junit:junit-bom:pom:5.7.0-M1:compile
```

我们可以在此命令生成的列表中检查库及其版本。此外，我们还可以使用Maven标签管理依赖项。使用[<exclusions\>标签](https://www.baeldung.com/maven-version-collision)，我们可以排除有问题的依赖项。使用[<optional\>标签](https://www.baeldung.com/maven-optional-dependency)，我们可以防止不需要的依赖项被捆绑在jar或war中。

## 5. 总结

在本文中，我们解决了NoSuchMethodError。我们讨论了此错误的原因以及处理它的方法。有关如何正确处理错误的更多详细信息，请参阅我们关于[捕获Java错误](https://www.baeldung.com/java-error-catch)的文章。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-3)上获得。
