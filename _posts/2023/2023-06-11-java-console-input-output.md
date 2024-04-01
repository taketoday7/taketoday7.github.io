---
layout: post
title:  在Java中读取和写入用户输入
category: java
copyright: java
excerpt: Java Console
---

## 1. 概述

在这个快速教程中，我们将演示**在Java中使用控制台进行用户输入和输出的几种方法**。

我们将了解[Scanner](https://www.baeldung.com/java-scanner)类的一些用于处理输入的方法，然后我们将使用System.out显示一些简单的输出。

最后，我们将看到如何使用Console类，该类自Java 6起可用，用于控制台输入和输出。

## 2. 从System.in读取

对于我们的第一个示例，**我们将使用java.util包中的Scanner类从System.in获取输入**-“标准”输入流：

```java
Scanner scanner = new Scanner(System.in);
```

让我们**使用nextLine()方法将整行输入作为字符串读取并前进到下一行**：

```java
String nameSurname = scanner.nextLine();
```

我们还可以**使用next()方法从流中获取下一个输入标记**：

```java
String gender = scanner.next();
```

如果我们期望数字输入，我们可以**使用nextInt()将下一个输入作为int原始类型获取**，同样，我们可以**使用nextDouble()获取double类型的变量**：

```java
int age = scanner.nextInt();
double height = scanner.nextDouble();
```

Scanner类还提供hasNext_Prefix()方法，**如果下一个标记可以解释为相应的数据类型，则返回true**。

例如，我们可以使用hasNextInt()方法来检查下一个标记是否可以解释为整数：

```java
while (scanner.hasNextInt()) {
    int nmbr = scanner.nextInt();
    // ...
}
```

此外，我们可以**使用hasNext(Pattern pattern)方法来检查以下输入标记是否与模式匹配**：

```java
if (scanner.hasNext(Pattern.compile("www.tuyucheng.com"))) {         
    // ...
}
```

除了使用Scanner类之外，**我们还可以使用带有System.in的InputStreamReader从控制台获取输入**：

```java
BufferedReader buffReader = new BufferedReader(new InputStreamReader(System.in));
```

然后我们可以读取输入并将其解析为整数：

```java
int i = Integer.parseInt(buffReader.readLine());
```

## 3. 写入System.out

对于控制台输出，我们可以使用**System.out-PrintStream类的一个实例**，它是OutputStream的一种类型。

在我们的示例中，我们将使用控制台输出来提示用户输入并向用户显示最终消息。

**让我们使用println()方法打印一个String并终止该行**：

```java
System.out.println("Please enter your name and surname: ");
```

或者，**我们可以使用print()方法，它的工作方式与println()类似，但不会终止该行**：

```java
System.out.print("Have a good");
System.out.print(" one!");
```

## 4. 使用Console类进行输入和输出

在JDK 6及更高版本中，我们可以使用java.io包中的Console类来读取和写入控制台。

要获得Console对象，我们将调用System.console()：

```java
Console console = System.console();
```

接下来，让我们使用Console类的readLine()方法向控制台写入一行，然后从控制台读取一行：

```java
String progLanguauge = console.readLine("Enter your favourite programming language: ");
```

如果我们需要读取敏感信息，例如密码，我们可以**使用readPassword()方法提示用户输入密码，并在禁用回显的情况下从控制台读取密码**：

```java
char[] pass = console.readPassword("To finish, enter password: ");
```

**我们还可以使用Console类将输出写入控制台，例如，使用带有String参数的printf()方法**：

```java
console.printf(progLanguauge + " is very interesting!");
```

## 5. 总结

在本文中，我们展示了如何使用几个Java类来执行控制台用户输入和输出。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-console)上获得。