---
layout: post
title:  使用Mockito Mock私有字段
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们将学习如何使用[Mockito](https://www.baeldung.com/mockito-series) mock私有字段。Mockito是一种流行的mock框架，通常与JUnit一起使用以在Java中创建mock对象。它本身并不支持mock私有字段。

但是，我们可以使用不同的方法通过Mockito mock私有字段，让我们来看看其中的几个。

## 2. 项目设置

我们将创建一个带有私有字段的类和一个测试类来测试它。

### 2.1 源类

首先，我们将创建一个带有私有字段的简单类：

```java
public class MockService {
    private final Person person = new Person("John Doe");

    public String getName() {
        return person.getName();
    }
}
```

MockService类有一个类型为Person的私有字段person。它还有一个返回人名的方法getName()。如我们所见，person字段不存在setter方法，因此我们不能直接设置字段或者改变字段的值。

接下来，我们将创建Person类：

```java
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

Person类有一个私有字段name和该字段的getter方法。

### 2.2 测试类

接下来，我们将创建一个测试类来测试MockService类：

```java
public class MockServiceUnitTest {
    private Person mockedPerson;

    @BeforeEach
    public void setUp(){
        mockedPerson = mock(Person.class);
    }
}
```

我们创建一个Person类的实例并使用Mockito mock它。在接下来的部分中，我们将研究使用此mock实例替换MockService类的私有字段的方法。

## 3. 使用Java反射API启用Mock

设置私有字段的方法之一是使用[Java反射API](https://www.baeldung.com/java-reflection#Fields)。这是一个很好的方法，因为它不需要任何额外的依赖项。**我们可以首先使字段可访问，然后将字段的值设置为mock实例**。

让我们看一下执行此操作的代码：

```java
@Test
void givenNameChangedWithReflection_whenGetName_thenReturnName() throws Exception {
    Class<?> mockServiceClass = Class.forName("cn.tuyucheng.taketoday.mockprivate.MockService");
    MockService mockService = (MockService) mockServiceClass.getDeclaredConstructor().newInstance();
    Field field = mockServiceClass.getDeclaredField("person");
    field.setAccessible(true);
    field.set(mockService, mockedPerson);

    when(mockedPerson.getName()).thenReturn("Jane Doe");

    Assertions.assertEquals("Jane Doe", mockService.getName());
}
```

我们使用Class.forName()方法来获取MockService类的Class对象。然后，我们使用getDeclaredConstructor()方法创建MockService类的实例。

接下来，我们使用getDeclaredField()方法获取MockService类的person字段。**我们使用setAccessible()方法使字段可访问，并使用set()方法将字段的值设置为mock实例**。

最后，我们可以mock Person类的getName()方法并测试MockService类的getName()方法返回mock值。

## 4. 使用JUnit 5启用Mock

与Java反射API类似，[JUnit 5](https://www.baeldung.com/junit-5)也提供了实用方法来设置私有字段。我们可以使用JUnit 5的ReflectionUtils类为私有字段设置一个值：

```java
@Test
void givenNameChangedWithReflectionUtils_whenGetName_thenReturnName() throws Exception {
    MockService mockService = new MockService();
    Field field = ReflectionUtils
        .findFields(MockService.class, f -> f.getName().equals("person"),
            ReflectionUtils.HierarchyTraversalMode.TOP_DOWN)
        .get(0);

    field.setAccessible(true);
    field.set(mockService, mockedPerson);

    when(mockedPerson.getName()).thenReturn("Jane Doe");

    Assertions.assertEquals("Jane Doe", mockService.getName());
}
```

此方法的工作方式与前面的方法相同，主要区别在于我们如何获取该字段：

-   我们使用ReflectionUtils.findFields()方法来获取字段
-   它采用MockService类的Class对象和一个谓词来查找该字段，我们在这里使用的谓词是查找名称为“person”的字段
-   此外，我们需要指定HierarchyTraversalMode，当我们的类有层次结构并且我们想在层次结构中找到字段时，这很重要
-   在我们的例子中，我们只有一个类，因此我们可以使用任何TOP_DOWN或BOTTOM_UP模式

这为我们提供了字段，我们可以再次设置字段的值并执行测试。

## 5. 使用Spring测试启用Mock

如果我们的项目使用Spring，Spring Test提供了一个实用程序类[ReflectionTestUtils](https://www.baeldung.com/spring-reflection-test-utils)来设置私有字段。

### 5.1 依赖项

让我们首先将[Spring Test](https://mvnrepository.com/artifact/org.springframework/spring-test)依赖项添加到我们的项目中：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.25</version>
    <scope>test</scope>
</dependency>
```

或者，如果使用Spring Boot，我们可以使用[Spring Boot Starter Test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)依赖项来执行相同的操作。

### 5.2 测试用例

接下来，让我们在测试中使用此类来启用mock：

```java
@Test
void givenNameChangedWithReflectionTestUtils_whenGetName_thenReturnName() throws Exception {
    MockService mockService = new MockService();

    ReflectionTestUtils.setField(mockService, "person", mockedPerson);

    when(mockedPerson.getName()).thenReturn("Jane Doe");
    Assertions.assertEquals("Jane Doe", mockService.getName());
}
```

在这里，我们使用ReflectionTestUtils.setField()方法来设置私有字段。在内部，这也使用Java反射API来设置字段，但不需要样板代码。

## 6. 总结

在本文中，我们研究了使用Mockito mock私有字段的不同方法。我们探索了Java反射API、JUnit 5和Spring Test来mock私有字段。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/mocks-2)上获得。