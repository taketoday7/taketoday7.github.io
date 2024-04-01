---
layout: post
title:  JUnit 5与@RunWith注解
category: unittest
copyright: unittest
excerpt: JUnit 5 @RunWith
---

## 1. 概述

在本快速教程中，我们将讨论JUnit 5框架中@RunWith注解的使用。

在JUnit 5中，**@RunWith注解已被更强大的@ExtendWith注解所取代**。

但是，为了向后兼容，在JUnit 5中仍然可以使用@RunWith注解。

## 2. 使用基于JUnit 4的Runner运行测试

我们可以使用@RunWith注解在任何较老的JUnit版本环境中运行JUnit 5测试。

让我们看一个在仅支持JUnit 4的Intellij版本中运行测试的示例。

首先，让我们创建要测试的类：

```java
public class Greetings {
    public static String sayHello() {
        return "Hello";
    }
}
```

然后我们创建这个普通的JUnit 5测试：

```java
public class GreetingsTest {
    @Test
    void whenCallingSayHello_thenReturnHello() {
        assertTrue("Hello".equals(Greetings.sayHello()));
    }
}
```

最后，让我们添加此注解，以便我们能够运行测试：

```java
@RunWith(JUnitPlatform.class)
public class GreetingsUnitTest {
    // ...
}
```

JUnitPlatform类是一个基于JUnit 4的Runner，它允许我们在JUnit Platform上运行JUnit 4测试。

**请记住，JUnit 4并不支持新JUnit Platform的所有功能，因此该Runner的功能有限**。

如果我们在Intellij中检查测试结果，我们可以看到使用了JUnit 4 Runner：

![](/assets/images/2023/unittest/junit5runwith01.png)

## 3. 在JUnit 5环境中运行测试

现在让我们在支持JUnit 5的Intellij版本中运行相同的测试。在这种情况下，我们不再需要@RunWith注解，我们可以在没有Runner的情况下编写测试：

```java
class GreetingsUnitTest {

    @Test
    void whenCallingSayHello_thenReturnHello() {
        assertEquals("Hello", Greetings.sayHello());
    }
}
```

测试结果表明我们现在使用的是JUnit 5 Runner：

![](/assets/images/2023/unittest/junit5runwith02.png)

## 4. 从基于JUnit 4的Runner迁移

现在让我们将使用基于JUnit 4的Runner的测试迁移到JUnit 5。

我们以Spring测试为例：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { SpringTestConfiguration.class })
public class GreetingsSpringTest {
    // ...
}
```

**如果我们想将此测试迁移到JUnit 5，我们需要用新的@ExtendWith@RunWith注解**：

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { SpringTestConfiguration.class })
public class GreetingsSpringTest {
    // ...
}
```

SpringExtension类由Spring 5提供，并将Spring TestContext框架集成到JUnit 5中。@ExtendWith注解接收任何实现Extension接口的类。

## 5. 总结

在这篇简短的文章中，我们介绍了JUnit 4的@RunWith注解在JUnit5框架中的使用。

本文的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/junit-5-basics)上找到。