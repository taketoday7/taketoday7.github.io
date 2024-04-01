---
layout: post
title:  为什么不推荐字段注入？
category: spring
copyright: spring
excerpt: Spring DI
---

## 1. 概述

当我们在IDE中运行代码分析工具时，它可能会针对带有@Autowired注解的字段发出“不推荐字段注入”的警告。

在本教程中，我们将探讨为什么不推荐使用字段注入以及我们可以使用哪些替代方法。

## 2. 依赖注入

对象使用它们的依赖对象而不需要定义或创建它们的过程称为[依赖注入](https://www.baeldung.com/spring-dependency-injection)，它是Spring框架的核心功能之一。

我们可以通过三种方式注入依赖对象，使用：

-   构造函数注入
-   Setter注射
-   字段注入

这里的第三种方法涉及使用[@Autowired](https://www.baeldung.com/spring-autowire)注解将依赖项直接注入到类中。尽管这可能是最简单的方法，但我们必须了解它可能会导致潜在的问题。

更重要的是，即使是[官方的Spring文档](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)也不再提供字段注入作为DI选项之一。

## 3. 空安全

**如果依赖项未正确初始化，字段注入会产生NullPointerException风险**。

让我们定义EmailService类并使用字段注入添加EmailValidator依赖项：

```java
@Service
public class EmailService {

    @Autowired
    private EmailValidator emailValidator;
}
```

现在，让我们添加process()方法：

```java
public void process(String email) {
    if(!emailValidator.isValid(email)){
        throw new IllegalArgumentException(INVALID_EMAIL);
    }
    // ...
}
```

仅当我们提供EmailValidator依赖项时，EmailService才能正常工作。**但是，使用字段注入，我们没有提供直接的方法来实例化具有所需依赖项的EmailService**。

此外，我们可以使用默认构造函数创建EmailService实例：

```java
EmailService emailService = new EmailService();
emailService.process("test@tuyucheng.com");
```

执行上面的代码会导致NullPointerException，因为我们没有提供它的强制依赖项EmailValidator。

现在，**我们可以使用构造函数注入来降低NullPointerException的风险**：

```java
private final EmailValidator emailValidator;

public EmailService(final EmailValidator emailValidator) {
    this.emailValidator = emailValidator;
}
```

通过这种方法，我们公开了所需的依赖项。此外，我们现在要求客户提供强制依赖项。换句话说，如果不提供EmailValidator实例，就无法创建EmailService的新实例。

## 4. 不变性

**使用字段注入，我们无法创建不可变类**。

我们需要在声明最终字段时实例化它们或通过构造函数。**此外，一旦调用了构造函数，Spring就会执行自动装配**。因此，不可能使用字段注入自动装配最终字段。

由于依赖项是可变的，因此我们无法确保它们在初始化后将保持不变。此外，重新分配非最终字段可能会在运行应用程序时产生意想不到的副作用。

或者，我们可以对强制依赖项使用构造函数注入，对可选依赖项使用setter注入。这样，我们可以确保所需的依赖项保持不变。

## 5. 设计问题

现在，让我们讨论一下字段注入时可能出现的一些设计问题。

### 5.1 违反单一责任

作为[SOLID原则](https://www.baeldung.com/solid-principles)的一部分，[单一职责原则](https://www.baeldung.com/java-single-responsibility-principle)规定每个类应该只有一个职责。换句话说，一个类应该只负责一个操作，因此只有一个改变的理由。

当我们使用字段注入时，我们可能最终会违反单一职责原则。**我们可以轻松地添加不必要的更多依赖项，并创建一个完成多个任务的类**。

另一方面，如果我们使用构造函数注入，我们会注意到如果构造函数具有多个依赖项，我们可能会遇到设计问题。此外，如果构造函数中的参数超过7个，IDE也会发出警告。

### 5.2 循环依赖

简单地说，当两个或多个类相互依赖时，就会发生[循环依赖](https://www.baeldung.com/circular-dependencies-in-spring)。由于这些依赖关系，无法构造对象，并且执行可能会以运行时错误或无限循环结束。

使用字段注入可能会导致循环依赖被忽视：

```java
@Component
public class DependencyA {

    @Autowired
    private DependencyB dependencyB;
}

@Component
public class DependencyB {

    @Autowired
    private DependencyA dependencyA;
}
```

**由于依赖项是在需要时而不是在上下文加载时注入的，因此Spring不会抛出BeanCurrentlyInCreationException**。

使用构造函数注入，可以在编译时检测循环依赖，因为它们会产生无法解决的错误。

此外，如果我们的代码中存在循环依赖，则可能表明我们的设计存在问题。因此，如果可能的话，我们应该考虑重新设计我们的应用程序。

**但是，自Spring Boot 2.6版本以来，[默认情况下不再允许循环依赖](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6-Release-Notes#circular-references-prohibited-by-default)**。

## 6. 测试

单元测试揭示了字段注入方法的主要缺点之一。

假设我们想编写一个单元测试来检查EmailService中定义的process()方法是否正常工作。

首先，我们想mock EmailValidation对象。但是，由于我们使用字段注入插入了EmailValidator，因此我们不能直接用mock版本替换它：

```java
EmailValidator validator = Mockito.mock(EmailValidator.class);
EmailService emailService = new EmailService();
```

此外，在EmailService类中提供setter方法会引入额外的漏洞，因为其他类(不仅仅是测试类)可以调用该方法。

**但是，我们可以通过反射来实例化我们的类**。例如，我们可以使用Mockito：

```java
@Mock
private EmailValidator emailValidator;

@InjectMocks
private EmailService emailService;

@BeforeEach
public void setup() {
    MockitoAnnotations.openMocks(this);
}
```

在这里，Mockito将尝试使用@InjectMocks注解注入mock。**但是，如果字段注入策略失败，Mockito不会报告失败**。

另一方面，使用构造函数注入，我们可以在不反射的情况下提供所需的依赖项：

```java
private EmailValidator emailValidator;

private EmailService emailService;

@BeforeEach
public void setup() {
    this.emailValidator = Mockito.mock(EmailValidator.class);
    this.emailService = new EmailService(emailValidator);
}
```

## 7. 总结

在本文中，我们了解了不推荐使用字段注入的原因。

总而言之，我们可以对必需的依赖项使用构造函数注入，对可选的依赖项使用setter注入，而不是字段注入。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-modules/spring-di-4)上获得。