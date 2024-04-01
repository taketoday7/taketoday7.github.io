---
layout: post
title:  使用JUnit对System.in进行单元测试
category: test-lib
copyright: test-lib
excerpt: System.in
---

## 1. 概述

当测试依赖于控制台用户输入的代码时，该过程可能会变得非常具有挑战性。此外，测试用户输入场景是基于控制台或独立应用程序的关键部分，因为我们需要确保正确处理不同的输入。

在本教程中，我们将介绍使用[JUnit](https://www.baeldung.com/junit)测试System.in的方法。

## 2. 理解System类

在深入研究之前，让我们先看一下[System](https://www.baeldung.com/java-lang-system)类，它是java.lang包中的最终类。

该类通过in和out变量提供对标准输入和输出流的访问。与out变量类似，System类具有表示标准错误输出流的err变量。

此外，这些变量允许我们读取和写入控制台。使用这些流，我们允许用户从控制台与我们的应用程序交互。

接下来，System.in返回一个已经打开的InputStream，可以从标准输入读取数据。使用System.in，我们可以将输入流从键盘重定向到CPU到我们的应用程序中。

## 3. 输入示例

让我们从本教程中将使用的简单示例开始：

```java
public static String readName() {
    Scanner scanner = new Scanner(System.in);
    String input = scanner.next();
    return NAME.concat(input);
}
```

Java提供了[Scanner](https://www.baeldung.com/java-scanner)类，它允许我们从各种来源读取输入，包括标准键盘输入。此外，它提供了读取用户输入的最简单方法。使用Scanner，我们可以读取任何原始数据类型或String。

在我们的示例方法中，我们使用next()方法来读取用户的输入。此外，该方法将输入中的下一个单词作为字符串读取。

## 4. 使用核心Java

对标准输入进行单元测试的第一种方法包括Java API提供的功能。

**我们可以利用System.in创建自定义输入流并在测试过程中模拟用户输入**。

但是，在编写单元测试之前，让我们在测试类中编写provideInput()辅助方法：

```java
void provideInput(String data) {
    ByteArrayInputStream testIn = new ByteArrayInputStream(data.getBytes());
    System.setIn(testIn);
}
```

在该方法内部，我们创建一个新的ByteArrayInputStream并将所需的输入作为字节数组传递。

**此外，我们使用System.setIn()方法将自定义输入流设置为System.in的输入**。

现在，让我们为示例方法编写一个单元测试。我们可以调用应用程序类的readName()方法，该方法现在读取我们的自定义输入：

```java
@Test
void givenName_whenReadFromInput_thenReturnCorrectResult() {
    provideInput("Tuyucheng");
    String input = Application.readName();
    assertEquals(NAME.concat("Tuyucheng"), input);
}
```

## 5. 使用System Rules库和JUnit 4

现在，让我们看看如何使用[System Rules](https://www.baeldung.com/java-system-rules-junit)库和JUnit 4测试标准输入。

首先，让我们向pom.xml添加所需的[依赖项](https://mvnrepository.com/artifact/com.github.stefanbirkner/system-rules)：

```xml
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
```

该库提供了用于测试依赖于System.in和System.out的代码的JUnit Rule。

此外，它允许我们在测试执行期间重定向输入和输出流，这使得模拟用户输入变得很容易。

其次，为了测试System.in，我们需要定义一个新的TextFromStandardInputStream Rule。我们将使用emptyStandardInputStream()方法用空输入流初始化变量：

```java
@Rule
public final TextFromStandardInputStream systemIn = emptyStandardInputStream();
```

最后，让我们编写单元测试：

```java
@Test
public void givenName_whenReadWithSystemRules_thenReturnCorrectResult() {
    systemIn.provideLines("Tuyucheng");
    assertEquals(NAME.concat("Tuyucheng"), Application.readName());
}
```

此外，我们使用ProvideLines()方法接收可变参数并将它们设置为输入。

此外，测试执行后会恢复原来的System.in。

## 6. 使用System Lambda库和JUnit 5

值得一提的是，System Rules默认不支持[JUnit 5](https://www.baeldung.com/junit-5)。但是，他们提供了一个[System Lambda](https://github.com/stefanbirkner/system-lambda)库，我们可以将其与JUnit 5一起使用。

我们需要在pom.xml中添加一个额外的[依赖项](https://mvnrepository.com/artifact/com.github.stefanbirkner/system-lambda)：

```xml
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-lambda</artifactId>
    <version>1.2.1</version>
    <scope>test</scope>
</dependency>
```

现在，让我们在测试中使用System Lambda：

```java
@Test
void givenName_whenReadWithSystemLambda_thenReturnCorrectResult() throws Exception {
    withTextFromSystemIn("Tuyucheng")
        .execute(() -> assertEquals(NAME.concat("Tuyucheng"), Application.readName()));
}
```

在这里，我们使用System Lambda类中提供的withTextFromSystemIn()静态方法来设置System.in中可用的输入行。

## 7. 使用System Stubs库和JUnit 4

此外，我们可以使用JUnit 4和[System Stubs](https://www.baeldung.com/java-system-stubs)库测试标准输入。

让我们添加所需的[依赖项](https://mvnrepository.com/artifact/uk.org.webcompere/system-stubs-junit4)：

```xml
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-junit4</artifactId>
    <version>2.0.2</version>
    <scope>test</scope>
</dependency>
```

接下来，让我们创建SystemInRule并传递所需的输入值：

```java
@Rule
public SystemInRule systemInRule = new SystemInRule("Tuyucheng");
```

现在，我们可以在单元测试中使用创建的Rule：

```java
@Test
public void givenName_whenReadWithSystemStubs_thenReturnCorrectResult() {
    assertThat(Application.readName()).isEqualTo(NAME.concat("Tuyucheng"));
}
```

## 8. 使用System Stubs库和JUnit 5

要使用System Stubs和JUnit 5测试System.in，我们需要添加另一个[依赖项](https://mvnrepository.com/artifact/uk.org.webcompere/system-stubs-jupiter)：

```xml
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-jupiter</artifactId>
    <version>2.0.2</version>
</dependency>
```

为了提供输入值，我们将使用withTextFromSystemIn()方法：

```java
@Test
void givenName_whenReadWithSystemStubs_thenReturnCorrectResult() throws Exception {
    SystemStubs.withTextFromSystemIn("Tuyucheng")
        .execute(() -> {
             assertThat(Application.readName()).isEqualTo(NAME.concat("Tuyucheng"));
        });
}
```

## 9. 总结

在本文中，我们学习了如何使用JUnit 4和JUnit 5测试System.in。

通过第一种方法，我们学习了如何使用核心Java功能自定义System.in。在第二种方法中，我们了解了如何使用System Rules库。接下来，我们学习了如何使用System Lambda库通过JUnit 5编写测试。最后，我们了解了如何使用System Stubs库。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-libraries-2)上找到。