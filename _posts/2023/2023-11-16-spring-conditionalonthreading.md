---
layout: post
title:  Spring @ConditionalOnThreading注解
category: springboot
copyright: springboot
excerpt: Spring Boot 3
---

## 1. 简介

在这个简短的教程中，我们将研究一个相对较新的Spring Boot注解，称为@ConditionalOnThreading。

我们将介绍此注解的条件是什么以及如何满足它以创建bean。

## 2. 条件注解

尽管我们已经介绍了[Spring Boot中的条件注解](https://www.baeldung.com/spring-conditional-annotations)，但还是值得再次简要回顾一下它们。

条件注解提供了一种仅在满足各种特定条件时才在BeanFactory中注册Bean的方法，开发人员使用Condition接口为每个注解单独定义这些条件。

Spring Boot附带了一系列针对常见用例的预定义条件注解，常见的示例有@ConditionalOnProperty、@ConditionalOnBean和@ConditionalOnClass。

## 3. @ConditionalOnThreading理论

@ConditionalOnThreading只是Spring Boot中另一个预定义的条件注解，它是在3.2版本中添加的。该版本本身在撰写本文时是一个候选版本，为了访问这个候选版本，我们应该[使用专用的Spring依赖仓库](https://www.baeldung.com/spring-maven-repository)。

**仅当Spring配置为在内部使用特定类型的线程时，@ConditionalOnThreading注解才允许创建bean**。线程类型意味着平台线程或虚拟线程，回想一下，自Java 21以来，我们已经能够使用[虚拟线程而不是平台线程](https://www.baeldung.com/java-virtual-thread-vs-thread)。

因此，要将Spring配置为在内部使用虚拟线程，我们使用名为spring.threads.virtual.enabled的属性。如果此属性为true并且我们在Java 21或更高版本上运行，则@ConditionalOnThreading注解将允许创建bean。

## 4. 注解使用示例

现在让我们尝试编写一些示例来演示此注解的用例，假设我们有两个用@ConditionalOnThreading标注的bean：

```java
@Configuration
static class CurrentConfig {

    @Bean
    @ConditionalOnThreading(Threading.PLATFORM)
    ThreadingType platformBean() {
        return ThreadingType.PLATFORM;
    }

    @Bean
    @ConditionalOnThreading(Threading.VIRTUAL)
    ThreadingType virtualBean() {
        return ThreadingType.VIRTUAL;
    }
}

enum ThreadingType {
    PLATFORM, VIRTUAL
}
```

因此，通过ConditionalOnThreading注解，我们将创建这两个bean之一：platformBean或virtualBean。现在让我们创建一些测试来检查此注解的工作原理：

```java
ApplicationContextRunner applicationContextRunner = new ApplicationContextRunner().withUserConfiguration(CurrentConfig.class);

@Test
@EnabledForJreRange(max = JRE.JAVA_20)
public void whenJava20AndVirtualThreadsDisabled_thenThreadingIsPlatform() {
    applicationContextRunner.withPropertyValues("spring.threads.virtual.enabled=false").run(context -> {
        Assertions.assertThat(context.getBean(ThreadingType.class)).isEqualTo(ThreadingType.PLATFORM);
    });
}

@Test
@EnabledForJreRange(min = JRE.JAVA_21)
public void whenJava21AndVirtualThreadsEnabled_thenThreadingIsVirtual() {
    applicationContextRunner.withPropertyValues("spring.threads.virtual.enabled=true").run(context -> {
        Assertions.assertThat(context.getBean(ThreadingType.class)).isEqualTo(ThreadingType.VIRTUAL);
    });
}

@Test
@EnabledForJreRange(min = JRE.JAVA_21)
public void whenJava21AndVirtualThreadsDisabled_thenThreadingIsPlatform() {
    applicationContextRunner.withPropertyValues("spring.threads.virtual.enabled=false").run(context -> {
        Assertions.assertThat(context.getBean(ThreadingType.class)).isEqualTo(ThreadingType.PLATFORM);
    });
}
```

在这里，我们有一个ApplicationContextRunner实例，它为测试创建一个轻量级应用程序上下文，我们用它来设置Spring属性spring.threads.virtual.enabled的值。我们还使用@EnabledForJreRange注解对测试进行了标注，该注解允许我们仅在某些Java版本上运行测试。

我们可以注意到，要使ThreadingType bean为Virtual，我们必须将该属性设置为true且至少为Java 21。在其他情况下，注解条件为false。

## 5. 总结

在这篇简短的文章中，我们探讨了一个新的Spring注解@ConditionalOnThreading，它作为Spring Boot 3.2的一部分提供。仅当Spring配置为通过属性在内部使用特殊类型的线程时，此注解才允许创建bean。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-3-2)上获得。