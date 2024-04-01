---
layout: post
title:  标记和过滤JUnit测试
category: unittest
copyright: unittest
excerpt: JUnit 5 @Tag
---

## 1. 概述

使用Maven作为CI构建的一部分自动执行我们所有的JUnit测试是很常见的。然而，这通常很耗时。

因此，**我们通常希望过滤测试并在构建过程的各个阶段执行单元测试或集成测试，或者两者都执行**。

在本教程中，我们将介绍一些针对[JUnit 5](https://www.baeldung.com/junit-5)测试用例的过滤技术。在接下来的部分中，我们还将介绍JUnit 5之前的各种过滤机制。

## 2. Junit 5标签

### 2.1 使用@Tag标注Junit测试

使用JUnit 5，我们可以通过在唯一的标签名称下标记测试的子集来过滤测试。例如，假设我们使用JUnit 5实现了单元测试和集成测试。我们可以在两组测试用例上添加标签：

```java
@Test
@Tag("IntegrationTest")
public void testAddEmployeeUsingSimpleJdbcInsert() {
}

@Test
@Tag("UnitTest")
public void givenNumberOfEmployeeWhenCountEmployeeThenCountMatch() {
}
```

此后，**我们可以单独执行特定标签名称下的所有测试**。我们也可以标记类而不是方法，从而在一个标签下包含一个类中的所有测试。

在接下来的几节中，我们将介绍过滤和执行标记的JUnit测试的各种方法。

### 2.2 使用测试套件过滤标签

JUnit 5允许我们实现测试套件，通过这些套件我们可以执行标记的测试用例：

```java
@SelectPackages("cn.tuyucheng.taketoday.tags")
@IncludeTags("UnitTest")
public class EmployeeDAOSuiteTest {
}
```

现在，**如果我们运行这个套件，标签UnitTest下的所有JUnit测试都将被执行**。同样，我们可以使用@ExcludeTags注解排除测试。

### 2.3 使用Maven Surefire插件过滤标签

为了在Maven构建的各个阶段过滤JUnit测试，我们可以使用Maven Surefire插件。**Surefire插件允许我们在插件配置中包含或排除标签**：

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <groups>UnitTest</groups>
    </configuration>
</plugin>
```

如果我们现在执行这个插件，它将执行所有标记为UnitTest的测试。同样，我们可以排除标签名下的测试用例：

```xml
<excludedGroups>IntegrationTest</excludedGroups>
```

### 2.4 使用IDEA过滤标签

IDEA允许按标签名过滤JUnit测试。这样我们就可以直接从我们的IDE执行一组特定的标记测试。

IntelliJ允许通过自定义Run/Debug配置进行此类过滤：

![](/assets/images/2023/unittest/junit5tagstest01.png)

如上图所示，我们选择使用Tags来执行测试，并在标签表达式中选择了要执行的标签。

JUnit 5允许使用各种[标签表达式](https://junit.org/junit5/docs/current/user-guide/#running-tests-tag-expressions)来过滤标签。例如，**要运行除集成测试之外的所有内容，我们可以使用!IntegrationTest作为标签表达式**。或者为了同时执行UnitTest和IntegrationTest，我们可以使用"UnitTest|IntegrationTest"。

## 3. Junit 4类别

### 3.1 JUnit 4测试分类

JUnit 4允许我们通过将它们添加到不同的类别来执行JUnit测试的子集。因此，我们可以执行特定类别的测试用例，同时排除其他类别。

**我们可以通过实现[标记接口](https://www.baeldung.com/java-marker-interfaces)来创建尽可能多的类别，其中标记接口的名称代表类别的名称**。对于我们的示例，我们将实现两个类别，UnitTest：

```java
public interface UnitTest {
}
```

和IntegrationTest：

```java
public interface IntegrationTest {
}
```

现在，我们可以通过使用@Category注解来对JUnit 4测试进行分类：

```java
@Test
@Category(IntegrationTest.class)
public void testAddEmployeeUsingSimpleJdbcInsert() {
}

@Test
@Category(UnitTest.class)
public void givenNumberOfEmployeeWhenCountEmployeeThenCountMatch() {
}
```

在我们的示例中，我们将@Category注解放在测试方法上。同样，我们也可以在测试类上添加此注解，从而将该类中的所有测试方法归为一个分类。

### 3.2 Categories Runner

为了执行一个类别中的JUnit测试，我们需要实现一个测试套件类：

```java
@RunWith(Categories.class)
@IncludeCategory(UnitTest.class)
@SuiteClasses(EmployeeDAOCategoryIntegrationTest.class)
public class EmployeeDAOUnitTestSuite {
}
```

该测试套件可以从IDE执行，并将执行UnitTest类别下的所有JUnit测试。同样，我们也可以在套件中排除一类测试：

```java
@RunWith(Categories.class)
@ExcludeCategory(IntegrationTest.class)
@SuiteClasses(EmployeeDAOCategoryIntegrationTest.class)
public class EmployeeDAOUnitTestSuite {
}
```

### 3.3 在Maven中排除或包含类别

最后，我们还可以在Maven中包含或排除JUnit测试的类别。因此，我们可以在不同的Maven Profile中执行不同类别的JUnit测试。

我们将为此使用Maven Surefire插件：

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <groups>cn.tuyucheng.taketoday.categories.UnitTest</groups>
    </configuration>
</plugin>
```

同样，我们可以从Maven构建中排除一个类别：

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <excludedGroups>cn.tuyucheng.taketoday.categories.IntegrationTest</excludedGroups>
    </configuration>
</plugin>
```

这类似于我们在上一节中讨论的标签，**唯一的区别是我们将标签名称替换为Category实现的完全限定名称**。

## 4. 使用Maven Surefire插件过滤JUnit测试

我们上面讨论的两种方法都是用JUnit库实现的。过滤测试用例的一种与实现无关的方法是遵循命名约定。对于我们的示例，我们将使用UnitTest作为单元测试的类名后缀，使用IntegrationTest作为集成测试的类名后缀。

现在我们将使用[Maven Surefire插件](https://www.baeldung.com/maven-surefire-plugin)来执行单元测试或集成测试：

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <excludes>
            **/*IntegrationTest.java
        </excludes>
    </configuration>
</plugin>
```

**这里的excludes标签过滤所有集成测试并仅执行单元测试**。这样的配置将节省大量的构建时间。

此外，我们可以在具有不同排除或包含的各种Maven Profile中执行Surefire插件。

**虽然Surefire非常适合用于过滤测试，但建议使用[Failsafe插件](https://www.baeldung.com/maven-failsafe-plugin)在Maven中执行集成测试**。

## 5. 总结

在本文中，我们介绍了一种使用JUnit 5标记和过滤测试用例的方法。**我们使用了@Tag注解，还介绍了通过IDE或在使用Maven的构建过程中过滤具有特定标签的JUnit测试的各种方法**。

所有示例都可以在[Github](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/junit-5-basics)上找到。