---
layout: post
title: 在集成测试中覆盖Spring Bean
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

我们可能希望在[Spring集成测试](https://www.baeldung.com/integration-testing-in-spring)中覆盖应用程序的一些Bean，通常，这可以使用专门为测试定义的[Spring Bean](https://www.baeldung.com/spring-bean)来完成。**但是，如果在Spring上下文中提供多个同名Bean，我们可能会遇到[BeanDefinitionOverrideException](https://www.baeldung.com/spring-boot-bean-definition-override-exception)**。

本教程将展示如何在Spring Boot应用程序中mock或stub集成测试Bean，同时避免出现BeanDefinitionOverrideException。

## 2. 测试中的Mock或Stub

**在深入研究细节之前，我们应该对如何在测试中使用[Mock或Stub](https://www.baeldung.com/cs/faking-mocking-stubbing)有信心**。这是一项强大的技术，可以确保我们的应用程序不易出现错误。

我们也可以将这种方法应用于Spring，但是，只有当我们使用Spring Boot时，才能直接mock集成测试bean。

或者，我们可以使用测试配置stub或mock bean。

## 3. Spring Boot应用示例

作为示例，让我们创建一个简单的Spring Boot应用程序，其中包含控制器、Service和配置类：

```java
@RestController
public class Endpoint {

    private final Service service;

    public Endpoint(Service service) {
        this.service = service;
    }

    @GetMapping("/hello")
    public String helloWorldEndpoint() {
        return service.helloWorld();
    }
}
```

/hello端点将返回我们要在测试期间替换的Service提供的字符串：

```java
public interface Service {
    String helloWorld();
}

public class ServiceImpl implements Service {

    public String helloWorld() {
        return "hello world";
    }
}
```

值得注意的是，我们将使用一个接口。因此，当需要时，我们将stub实现以获得不同的值。

我们还需要一个配置来加载Service bean：

```java
@Configuration
public class Config {

    @Bean
    public Service helloWorld() {
        return new ServiceImpl();
    }
}
```

最后，让我们添加@SpringBootApplication：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 4. 使用@MockBean进行覆盖

**[MockBean](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/MockBean.html)从Spring Boot 1.4.0版本开始可用，我们不需要任何测试配置**。因此，将@SpringBootTest注解添加到我们的测试类中就足够了：

```java
@SpringBootTest(classes = { Application.class, Endpoint.class })
@AutoConfigureMockMvc
class MockBeanIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private Service service;

    @Test
    void givenServiceMockBean_whenGetHelloEndpoint_thenMockOk() throws Exception {
        when(service.helloWorld()).thenReturn("hello mock bean");
        this.mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("hello mock bean")));
    }
}
```

我们相信与主配置没有冲突，**这是因为@MockBean将向应用程序中注入一个Service mock**。

最后，我们使用[Mockito](https://www.baeldung.com/mockito-series)来伪造服务返回：

```java
when(service.helloWorld()).thenReturn("hello mock bean");
```

## 5. 不使用@MockBean

让我们探索更多在不使用@MockBean的情况下覆盖Bean的选项，我们将研究四种不同的方法：Spring Profile、条件属性、@Primary注解和bean定义重写。然后我们可以stub或mock bean实现。

### 5.1 使用@Profile

**定义[Profile](https://www.baeldung.com/spring-profiles)是Spring的一种众所周知的做法**，首先，让我们使用@Profile创建配置：

```java
@Configuration
@Profile("prod")
public class ProfileConfig {

    @Bean
    public Service helloWorld() {
        return new ServiceImpl();
    }
}
```

然后，我们可以使用Service bean定义一个测试配置：

```java
@TestConfiguration
public class ProfileTestConfig {

    @Bean
    @Profile("stub")
    public Service helloWorld() {
        return new ProfileServiceStub();
    }
}
```

ProfileServiceStub服务将stub已定义的ServiceImpl：

```java
public class ProfileServiceStub implements Service {

    public String helloWorld() {
        return "hello profile stub";
    }
}
```

我们可以创建一个测试类，包括主配置和测试配置：

```java
@SpringBootTest(classes = { Application.class, ProfileConfig.class, Endpoint.class, ProfileTestConfig.class })
@AutoConfigureMockMvc
@ActiveProfiles("stub")
class ProfileIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void givenConfigurationWithProfile_whenTestProfileIsActive_thenStubOk() throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("hello profile stub")));
    }
}
```

我们在ProfileIntegrationTest中激活stub Profile，不会加载prod Profile。因此，测试配置将加载Service stub。

### 5.2 使用@ConditionalOnProperty

**与Profile类似，我们可以使用[@ConditionalOnProperty](https://www.baeldung.com/spring-conditionalonproperty)注解在不同的Bean配置之间切换**。

因此，我们的主配置中将有一个service.stub属性：

```java
@Configuration
public class ConditionalConfig {

    @Bean
    @ConditionalOnProperty(name = "service.stub", havingValue = "false")
    public Service helloWorld() {
        return new ServiceImpl();
    }
}
```

在运行时，我们需要将此条件设置为false，通常在application.properties文件中设置：

```properties
service.stub=false
```

相反，在测试配置中，我们希望触发Service加载。因此，我们需要满足以下条件：

```java
@TestConfiguration
public class ConditionalTestConfig {

    @Bean
    @ConditionalOnProperty(name="service.stub", havingValue="true")
    public Service helloWorld() {
        return new ConditionalStub();
    }
}
```

然后，我们还添加Service stub：

```java
public class ConditionalStub implements Service {

    public String helloWorld() {
        return "hello conditional stub";
    }
}
```

最后，让我们创建测试类。我们将service.stub条件设置为true并加载Service stub：

```java
@SpringBootTest(classes = {  Application.class, ConditionalConfig.class, Endpoint.class, ConditionalTestConfig.class }
        , properties = "service.stub=true")
@AutoConfigureMockMvc
class ConditionIntegrationTest {

    @AutowiredService
    private MockMvc mockMvc;

    @Test
    void givenConditionalConfig_whenServiceStubIsTrue_thenStubOk() throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("hello conditional stub")));
    }
}
```

### 5.3 使用@Primary

**我们还可以使用[@Primary](https://www.baeldung.com/spring-primary)注解，给定主配置，我们可以在测试配置中定义一个主Service，以更高的优先级加载**：

```java
@TestConfiguration
public class PrimaryTestConfig {

    @Primary
    @Bean("service.stub")
    public Service helloWorld() {
        return new PrimaryServiceStub();
    }
}
```

值得注意的是，bean的名称需要不同。否则，我们仍然会遇到最初的异常。我们可以更改@Bean的name属性或方法的名称。

同样，我们需要一个Service stub：

```java
public class PrimaryServiceStub implements Service {

    public String helloWorld() {
        return "hello primary stub";
    }
}
```

最后，让我们通过定义所有相关组件来创建测试类：

```java
@SpringBootTest(classes = { Application.class, NoProfileConfig.class, Endpoint.class, PrimaryTestConfig.class })
@AutoConfigureMockMvc
class PrimaryIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void givenTestConfiguration_whenPrimaryBeanIsDefined_thenStubOk() throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("hello primary stub")));
    }
}
```

### 5.4 使用spring.main.allow-bean-definition-overriding属性

如果我们无法应用之前的任何选项怎么办？**Spring提供了spring.main.allow-bean-definition-overriding属性，因此我们可以直接覆盖主配置**。

让我们定义一个测试配置：

```java
@TestConfiguration
public class OverrideBeanDefinitionTestConfig {

    @Bean
    public Service helloWorld() {
        return new OverrideBeanDefinitionServiceStub();
    }
}
```

然后，我们需要Service stub：

```java
public class OverrideBeanDefinitionServiceStub implements Service {

    public String helloWorld() {
        return "hello no profile stub";
    }
}
```

同样，让我们创建一个测试类。如果我们想重写Service bean，我们需要将属性设置为true：

```java
@SpringBootTest(classes = { Application.class, Config.class, Endpoint.class, OverribeBeanDefinitionTestConfig.class },
        properties = "spring.main.allow-bean-definition-overriding=true")
@AutoConfigureMockMvc
class OverrideBeanDefinitionIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void givenNoProfile_whenAllowBeanDefinitionOverriding_thenStubOk() throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("hello no profile stub")));
    }
}
```

### 5.5 使用Mock而不是Stub

到目前为止，在使用测试配置时，我们已经看到了使用Stub的示例。不过，我们也可以[Mock Bean](https://www.baeldung.com/java-spring-mockito-mock-mockbean)，这适用于我们之前见过的任何测试配置。不过，为了进行演示，我们将遵循Profile示例。

**这一次，我们使用Mockito mock方法返回一个Service，而不是Stub**：

```java
@TestConfiguration
public class ProfileTestConfig {

    @Bean
    @Profile("mock")
    public Service helloWorldMock() {
        return mock(Service.class);
    }
}
```

同样，我们创建一个测试类来激活mock Profile：

```java
@SpringBootTest(classes = { Application.class, ProfileConfig.class, Endpoint.class, ProfileTestConfig.class })
@AutoConfigureMockMvc
@ActiveProfiles("mock")
class ProfileIntegrationMockTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private Service service;

    @Test
    void givenConfigurationWithProfile_whenTestProfileIsActive_thenMockOk() throws Exception {
        when(service.helloWorld()).thenReturn("hello profile mock");
        this.mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(containsString("hello profile mock")));
    }
}
```

值得注意的是，其工作原理与@MockBean类似。但是，我们使用@Autowired注解将bean注入到测试类中。与Stub相比，这种方法更加灵活，允许我们在测试用例中直接使用when/then语法。

## 6. 总结

在本教程中，我们学习了如何在Spring集成测试期间覆盖bean。

我们了解了如何使用@MockBean。此外，我们使用@Profile或@ConditionalOnProperty创建了主配置在测试期间在不同的bean之间切换。此外，我们还了解了如何使用@Primary为测试bean提供更高的优先级。

最后，我们看到了一个简单的解决方案，使用spring.main.allow-bean-definition-overriding并覆盖主配置bean。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-testing-2)上找到。