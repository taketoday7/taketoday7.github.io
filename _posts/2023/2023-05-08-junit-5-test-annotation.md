---
layout: post
title:  JUnit 5 @Test注解
category: unittest
copyright: unittest
excerpt: JUnit 5 @Test
---

## 1. 概述

在本文中，我们将快速回顾一下JUnit的@Test注解。此注解为执行单元和回归测试提供了强大的工具。

## 2. Maven配置

要使用[最新版本的JUnit 5](https://central.sonatype.com/artifact/org.junit.jupiter/junit-jupiter-engine/5.9.3)，我们需要添加以下Maven依赖项：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

我们使用test范围是因为我们不希望Maven在最终构建中包含此依赖项。

由于Surefire插件本身并不完全支持JUnit 5，我们**还需要添加一个[提供程序](https://central.sonatype.com/artifact/org.apache.maven.plugins/maven-surefire-plugin/3.1.0)**，它告诉Maven在哪里可以找到我们的测试：

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.19.1</version>
    <dependencies>
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-surefire-provider</artifactId>
            <version>1.0.2</version>
        </dependency>
    </dependencies>
</plugin>
```

在我们的配置中，我们将使用Surefire 2.19.1，因为**在撰写本文时，版本2.20.x与junit-platform-surefire-provider不兼容**。

## 3. 被测方法

首先，让我们构建一个简单的方法，我们将在测试场景中使用它来演示@Test注解的功能：

```java
public boolean isNumberEven(int number) {
    return number % 2 == 0;
}
```

如果传递的参数是偶数，则此方法应返回true，否则返回false。现在，让我们看看它是否按预期的方式工作。

## 4. 测试方法

对于我们的示例，我们要特别检查两种情况：

+ 当给定偶数时，该方法应返回true
+ 当给定奇数时，该方法应返回false

这意味着实现代码将使用不同的参数调用我们的isNumberEven()方法，并检查结果是否符合我们的预期。

**为了使测试能够被识别，我们添加@Test注解**。我们可以在一个类中编写很多测试方法，但最好只将相关的测试放在一起。还要注意，**测试方法不能是私有的，也不能返回值**，否则它会被忽略。

```java
private final NumbersBean bean = new NumbersBean();

@Test
void givenEvenNumber_whenCheckingIsNumberEven_thenTure() {
    boolean result = bean.isNumberEven(8);
    assertTrue(result);
}

@Test
void givenOddNumber_whenCheckingIsNumberEven_thenFalse() {
    boolean result = bean.isNumberEven(1);
    assertFalse(result);
}
```

如果我们现在运行Maven build，**Surefire插件将遍历src/test/java下的类中的所有带有@Test注解的方法并执行它们**，如果发生任何测试失败，则会导致构建失败。

如果你使用JUnit 4，请注意，**在此版本中，注解不接受任何参数**。要检查超时或抛出的异常，我们可以使用断言来代替：

```java
@Test
void givenLowerThanTenNumber_whenCheckingIsNumberEven_thenResultUnderTenMillis() {
    assertTimeout(Duration.ofMillis(10), () -> bean.isNumberEven(3));
}

@Test
void givenNull_whenCheckingIsNumberEven_thenNullPointerException() {
    assertThrows(NullPointerException.class, () -> bean.isNumberEven(null));
}
```

## 5. 总结

在这个快速教程中，我们演示了如何使用@Test注解实现和运行一个简单的JUnit测试。

有关JUnit框架的更多信息可以在[这篇文章](https://www.baeldung.com/junit-5)中找到，它提供了一般介绍。

示例中使用的所有代码都可以在[GitHub项目](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/junit-5-basics)中找到。