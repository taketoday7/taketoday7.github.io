---
layout: post
title:  在Spring中获取当前的ApplicationContext
category: spring
copyright: spring
excerpt: Spring
---

## 一、概述

在这个简短的教程中，我们将了解如何在 Spring 应用程序中获取当前的[ApplicationContext 。](https://www.baeldung.com/spring-application-context)

## 2.应用上下文

ApplicationContext表示 Spring IoC 容器，其中包含应用程序创建的所有 beans。它负责实例化、配置和创建 bean。此外，它还从 XML 或 Java 中提供的配置元数据中获取 bean 的信息。

ApplicationContext表示[BeanFactory](https://docs.spring.io/spring-framework/docs/1.2.9/javadoc-api/org/springframework/beans/factory/BeanFactory.html)的子接口。除了BeanFactory的功能外，它还包括消息解析和国际化、资源加载和事件发布等功能。此外，它还具有加载多个上下文的功能。

每个 bean 在容器启动后实例化，因为它使用预加载。

我们可能希望使用此容器来访问我们应用程序中的其他 bean 和资源。我们将学习两种在 Spring 应用程序中获取当前ApplicationContext引用的方法。

## 3.应用上下文Bean

获取当前ApplicationContext 的最简单方法是使用[@Autowired注解](https://www.baeldung.com/spring-autowire)将其注入到我们的 bean 中。

首先，让我们声明实例变量并使用@Autowired注解对其进行注解：

```java
@Component
public class MyBean {

    @Autowired
    private ApplicationContext applicationContext;

    public ApplicationContext getApplicationContext() {
        return applicationContext;
    }
}
```

我们可以使用@Inject注解代替@Autowired。

为了验证容器是否正确注入，让我们创建一个测试：

```java
@Test
void whenGetApplicationContext_thenReturnApplicationContext(){
    assertNotNull(myBean);
    ApplicationContext context = myBean.getApplicationContext();
    assertNotNull(context);
}
```

## 4.ApplicationContextAware接口_

获取当前上下文的另一种方法是实现[ApplicationContextAware](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContextAware.html)接口。它包含setApplicationContext()方法，Spring 在创建ApplicationContext后调用该方法。

此外，当应用程序启动时，Spring 会自动检测此接口并注入对ApplicationContext 的引用。

现在，让我们创建实现ApplicationContextAware接口的ApplicationContextProvider类：

```java
@Component
public class ApplicationContextProvider implements ApplicationContextAware {
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ApplicationContextProvider.applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
}
```

我们将applicationContext实例变量声明为静态的，这样我们就可以在任何类中访问它。此外，我们创建了一个静态方法来检索对ApplicationContext 的引用。

现在，我们可以通过调用静态的getApplicationContext()方法来获取当前的ApplicationContext对象：

```java
@Test
void whenGetApplicationContext_thenReturnApplicationContext() {
    ApplicationContext context = ApplicationContextProvider.getApplicationContext();
    assertNotNull(context);
}
```

此外，通过实现该接口，bean 可以获得对ApplicationContext 的引用并访问其他 bean 或资源。

为此，首先，让我们创建ItemService类：

```java
@Service
public class ItemService {
    // ...
}
```

其次，要从上下文中获取ItemService bean，让我们调用ApplicationContext上的getBean()方法：

```java
@Test
void whenGetBean_thenReturnItemServiceReference() {
    ApplicationContext context = ApplicationContextProvider.getApplicationContext();
    assertNotNull(context);

    ItemService itemService = context.getBean(ItemService.class);
    assertNotNull(context);
}
```

## 5.结论

在这篇简短的文章中，我们学习了如何在我们的 Spring Boot 应用程序中获取当前的ApplicationContext。总而言之，我们可以直接注入ApplicationContext bean 或实现ApplicationContextAware接口。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-modules/spring-core-2)上获得。