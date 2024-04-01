---
layout: post
title: Java 21中的未命名类和实例Main方法
category: java-new
copyright: java-new
excerpt: Java 21
---

## 1. 简介

Java 21已经到来，在新功能中，我们可以看到Java如何通过[未命名类和实例main方法](https://openjdk.org/jeps/445)变得越来越适合初学者。这些内容的引入是使Java成为一种更适合初学者的编程语言的关键一步。

在本教程中，我们将探讨这些新功能并了解它们如何使学生的学习曲线更加平滑。

## 2. 编写基本的Java程序

传统上，对于初学者来说，编写第一个Java程序比其他编程语言稍微复杂一些。一个基本的Java程序需要声明一个公共类，此类包含一个public static void main(String[] args)方法，作为程序的入口点。

所有这一切只是为了在控制台中输出一个“Hello world”：

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

Java 21极大地简化了我们编写简单程序的方式：

```java
void main() {
    System.out.println("Hello, World!");
}
```

我们将更详细地介绍如何使用新功能实现这种语法简化。

## 3. 实例Main方法

实例main()方法的引入允许开发人员利用更动态的方法来初始化他们的应用程序。

### 3.1 理解实例Main方法

这改变了Java程序声明其入口点的方式。事实上，Java早期要求公共类中存在带有String[]参数的[静态main()方法](https://www.baeldung.com/java-main-method)，正如我们在上一节中看到的那样。

这个新协议更加宽松，它允许使用具有不同[访问级别](https://www.baeldung.com/java-access-modifiers)的main()方法：public、protected或default(包)。

**此外，它不要求方法是静态的或具有String[]参数**：

```java
class HelloWorld {
    void main() {
        System.out.println("Hello, World!");
    }
}
```

### 3.2 选择启动协议

经过改进的启动协议会自动选择我们程序的入口，同时考虑可用性和访问级别。

**实例main()方法应始终具有非私有访问级别**。此外，启动协议遵循特定的顺序来决定使用哪种方法：

1. 在启动类中声明的static void main(String[] args)方法
2. 在启动类中声明的static void main()方法
3. 在启动类中声明或从超类继承的void main(String[] args)实例方法
4. void main()实例方法

**当类声明实例main()方法并继承[标准静态main()方法](https://www.baeldung.com/java-hello-world)时，系统将调用实例main()方法**。在这种情况下，[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)在运行时发出警告。

例如，假设我们有一个超类HelloWorldSuper，它实现了一个长期建立的main()方法：

```java
public class HelloWorldSuper {
    public static void main(String[] args) {
        System.out.println("Hello from the superclass");
    }
}
```

这个超类由HelloWorldChild类扩展：

```java
public class HelloWorldChild extends HelloWorldSuper {
    void main() {
        System.out.println("Hello, World!");
    }
}
```

让我们编译超类并使用--source 21和--enable-preview标志运行子类：

```shell
javac --source 21 --enable-preview HelloWorldSuper.java
java --source 21 --enable-preview HelloWorldChild
```

我们将在控制台中得到以下输出：

```shell
WARNING: "void HelloWorldChild.main()" chosen over "public static void HelloWorldSuper.main(java.lang.String[])"
Hello, World!
```

我们可以看到JVM警告我们程序中有两个可能的入口点。

## 4. 未命名类

未命名类是一项重要功能，旨在简化初学者的学习曲线。**它允许方法、字段和类在没有显式类声明的情况下存在**。

通常，在Java中，每个类都存在于包中，每个包也存在于模块中。但是，未命名的类存在于未命名的包和未命名的模块中。它们是final的，只能扩展Object类，而不实现任何接口。

鉴于这一切，我们可以声明main()方法，而无需在代码中显式声明该类：

```java
void main() { 
    System.out.println("Hello, World!");
}
```

利用这两个新功能，我们成功地将程序变得非常简单，任何开始使用Java编程的人都可以更容易地理解。

未命名的类几乎与显式声明的类完全相同。其他方法或变量被解释为未命名类的成员，因此我们可以将它们添加到我们的类中：

```java
private String getMessage() {
    return "Hello, World!";
}

void main() {
    System.out.println(getMessage());
}
```

尽管具有简单性和灵活性，但未命名类具有固有的局限性。

**直接构造或按名称引用是不可能的，并且它们不定义任何可从其他类访问的API**。这种不可访问性还会导致[Javadoc](https://www.baeldung.com/javadoc)工具在为此类类生成API文档时出现问题。但是，未来的Java版本可能会调整和增强这些行为。

## 5. 总结

在本文中，我们了解到Java 21通过引入未命名类和实例main()方法，在增强用户体验方面取得了重大进展，特别是对于那些刚刚开始编程之旅的用户来说。

通过简化编程的结构，这些功能使新手能够更快地专注于逻辑思维和解决问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-21)上获得。