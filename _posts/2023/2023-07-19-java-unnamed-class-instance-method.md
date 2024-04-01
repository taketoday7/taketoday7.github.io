---
layout: post
title:  Java未命名类和实例main方法
category: java-new
copyright: java-new
excerpt: Java 21
---

## 1. 概述

Java被认为是一种非常冗长的语言，即使对于简单的[Hello World程序](https://howtodoinjava.com/java/basics/java-hello-world-program/)和小型概念验证工作也是如此。从[Java 21](https://howtodoinjava.com/java/java-21-new-features/)开始，我们可以使用未命名的类和实例main方法，这允许我们用最少的语法引导类。它对于刚刚开始学习Java并希望尝试该语言语法以快速学习的初学者来说将受益匪浅。

## 2. 变化一览

以下是[JEP 445](https://openjdk.org/jeps/445)中这一变化的要点，注意：

- 无需类声明
- 不再需要Java关键字public和static
- main方法参数args是可选的

![img](https://howtodoinjava.com/wp-content/uploads/2023/07/Java-21-Unnamed-Classes-and-Instance-Main-Methods.jpg)

以下代码是一个功能齐全的类，它将输出“Hello, World!”打印到控制台。我们可以将这个类存储在任何Java文件中，例如HelloWorld.Java，然后我们可以运行它。

```java
void main() {
	System.out.println("Hello, World!");
}
```

这是Java 21中的预览语言功能，默认情况下处于禁用状态。使用--enable-preview标志来使用此功能。

```shell
$ java.exe --enable-preview --source 21 HelloWorld.Java

// Prints
Hello, World!
```

相比之下，在Java 21之前，我们必须编写以下类来执行相同的打印语句：

```java
public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

## 2. 编写main()方法的灵活方式

现在，Java编译器/运行时不会强迫我们为[main()方法](https://howtodoinjava.com/java/basics/main-method/)编写严格匹配的语法，尽管仍然允许在未命名的类中编写它。

这意味着我们可以在一个类中编写多个main()方法，Java编译器将按以下顺序搜索main()方法。无论JVM在序列中找到的第一个main()方法是什么，它都会使用它来启动程序：

- static void main(String[] args)方法
- 没有任何参数的static void main()方法
- 没有static关键字的void main(String[] args)实例方法
- 没有static关键字和参数的void main()实例方法

请注意，在所有上述情况下，该方法必须是非私有的，即它必须具有访问修饰符public、protected或package。

例如，在下面的程序中，我们创建了两个main方法。

```java
void main(String[] args) {
    System.out.println("Main method with args");
}

void main() {
    System.out.println("Main method without args");
}
```

在序列中，void main(String[] args)位于void main()之前，因此程序输出将是：

```text
Main method with args
```

请注意，上面的实例main()方法也可以用普通类编写。下面的Java程序是一个有效的程序，它将执行该main()方法。

```java
public class Main {

    void main() {
        // ...
    }
}
```

## 3. 创建和使用未命名类的规则

要正确创建未命名类，需要遵守某些规则。

### 3.1 不允许使用package声明

未命名的类始终是未命名包的成员，所以我们不能在其中声明package语句。

```java
package cn.tuyucheng.taketoday.java21;

void main() {
	System.out.println("Hello, World!");
}
```

程序输出：

```text
Main.java:1: error: unnamed class should not have package declaration
package cn.tuyucheng.taketoday.java21;
^
```

### 3.2 不允许使用构造函数

由于我们无法通过名称引用未命名的类，因此我们也无法创建[构造函数](https://howtodoinjava.com/java/oops/java-constructors/)。

这也意味着我们无法创建未命名类的实例，这就是为什么这些类仅适合简单的POC和学习目的。

### 3.3 main()方法必须存在

由于我们无法创建未命名类的直接实例，因此为了使其可用，它必须有一个main()方法来执行其代码。如果没有main方法，Java编译器将抛出错误。

```java
void display() {
	System.out.println("Hello, World!");
}
```

Java编译器将失败并显示以下错误：

```text
Main.java:4: error: unnamed class does not have main method in the form of void main() or void main(String[] args)
void display() {
     ^
...
error: compilation failed
```

### 3.4 未命名类无法扩展或实现

未命名的类始终是final并且不能扩展另一个类(Object除外)或实现接口。

缺少类名根本不允许使用语法来扩展或实现任何内容。

### 3.5 访问静态成员

由于未命名的类不能用名称来引用，因此我们不能用类名来引用它们的[静态方法和变量](https://howtodoinjava.com/java/keywords/java-static-keyword/)。我们可以直接访问它们或使用this关键字。

```java
static void main() {

    if(staticVariable) {
        System.out.println("static variable");
    }
    
    staticMethod();
}

static boolean staticVariable = true;

static void staticMethod(){
    System.out.println("static method");
}
```

程序输出：

```text
static variable
static method
```

## 4. 总结

Java 21未命名类功能是对该语言的一个重要补充，这将使那些激发学习Java以及来自具有类似快速入门语法的其他语言(例如Python)的程序员受益。同样，实例main()方法将有助于减少旨在学习语言基础知识和简单证明工作的简单Java程序的礼仪量。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-21)上找到。