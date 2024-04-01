---
layout: post
title: SpringRunner与SpringBootTest
category: unittest
copyright: unittest
excerpt: JUnit 5
---

## 1. 概述

测试对于任何应用程序都至关重要，无论是单元测试还是集成测试。[SpringRunner](https://www.baeldung.com/junit-springrunner-vs-mockitojunitrunner#springrunner)和[SpringBootTest](https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)类构成了运行集成测试的基础。

在本教程中，我们将介绍这两者。我们将学习如何在代码中使用它们并了解它们的相似点和差异。

## 2. SpringRunner

[SpringRunner](https://www.baeldung.com/junit-springrunner-vs-mockitojunitrunner#springrunner)是[SpringJUnit4ClassRunner](https://www.baeldung.com/springjunit4classrunner-parameterized#springjunit4classrunner) 类的别名，适用于基于[JUnit 4](https://www.baeldung.com/junit-4-custom-runners)的测试类。它加载Spring TestContext，通过它Spring bean和配置可以与JUnit注解一起使用。我们需要JUnit 4.12或更高版本才能使用它。

要在代码中使用它，请使用@RunWith(SpringRunner.class)标注测试类：

```java
@RunWith(SpringRunner.class)
public class SampleIntegrationTest {

    @Test
    public void test() {
        // ...
    }
}
```

## 3. SpringBootTest

[SpringBootTest](https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)是[SpringRunner](https://www.baeldung.com/junit-springrunner-vs-mockitojunitrunner#springrunner)的替代方案，可与[JUnit 5](https://www.baeldung.com/junit-5)配合使用。它还用于运行集成测试和加载Spring TestContext。

它非常丰富，并通过其注解参数提供了许多配置。它支持各种Web环境模式，例如MOCK、RANDOM_PORT、DEFINED_PORT和NONE。我们可以通过在测试运行之前注入到Spring环境中的注解来传递应用程序属性：

```java
@SpringBootTest(
        properties = {"user.name=test_user"},
        webEnvironment = MOCK)
public class SampleIntegrationTest {

    @Test
    public void test() {
        // ...
    }
}
```

在类级别需要标注@SpringBootTest才能运行集成测试。

## 4. SpringRunner和SpringBootTest的比较

在下表中，我们比较了这两个注解的优缺点。

|         SpringRunner          |        SpringBootTest         |
|:-----------------------------:|:-----------------------------:|
| 用于运行集成测试并加载Spring TestContext | 用于运行集成测试并加载Spring TestContext |
|         JUnit注解也可以使用          |         JUnit注解也可以使用          |
|       需要JUnit 4.12或更高版本       |        需要JUnit 5或更高版本         |
|         就配置而言，API并不丰富         |        提供丰富的API来配置测试配置        |
|              不建议              |       推荐，因为它支持新功能并且易于使用       |

## 5. 总结

在这篇文章中，我们了解了[SpringRunner](https://www.baeldung.com/junit-springrunner-vs-mockitojunitrunner#springrunner)和[SpringBootTest](https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)。我们已经学会了如何使用它们，并对它们进行了比较，了解了它们的差异和相似之处。

**我们应该使用[SpringBootTest](https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)，因为它支持最新的[JUnit](https://www.baeldung.com/junit-5)，但是每当需要使用[JUnit 4](https://www.baeldung.com/junit-4-custom-runners)时，[SpringRunner](https://www.baeldung.com/junit-springrunner-vs-mockitojunitrunner#springrunner)也是不错的选择**。