---
layout: post
title:  从同一Bean的另一个方法调用Spring @Cacheable
category: spring
copyright: spring
excerpt: Spring Cache
---

## 1. 简介

Spring提供了一种基于注解的方法来在Spring管理的bean上启用缓存。基于[AOP](https://www.baeldung.com/spring-aop)技术，通过在方法上添加@Cacheable注解，可以很容易地使方法可缓存。但是，当从同一个类中调用时，缓存将被忽略。

在本教程中，我们将解释为什么会发生这种情况以及如何解决它。

## 2. 重现问题

首先，我们创建一个[启用缓存的Spring Boot应用程序](https://www.baeldung.com/spring-cache-tutorial)。在本文中，我们创建了一个带有@Cacheable注解的square方法的MathService：

```java
@Service
@CacheConfig(cacheNames = "square")
public class MathService {
    private final AtomicInteger counter = new AtomicInteger();

    @CacheEvict(allEntries = true)
    public AtomicInteger resetCounter() {
        counter.set(0);
        return counter;
    }

    @Cacheable(key = "#n")
    public double square(double n) {
        counter.incrementAndGet();
        return n * n;
    }
}
```

其次，我们在MathService中创建一个方法sumOfSquareOf2，它调用square方法两次：

```java
public double sumOfSquareOf2() {
    return this.square(2) + this.square(2);
}
```

第三，我们为方法sumOfSquareOf2创建一个测试，以检查调用square方法的次数：

```java
@SpringBootTest(classes = Application.class)
class MathServiceIntegrationTest {

    @Resource
    private MathService mathService;

    @Test
    void givenCacheableMethod_whenInvokingByInternalCall_thenCacheIsNotTriggered() {
        AtomicInteger counter = mathService.resetCounter();

        assertThat(mathService.sumOfSquareOf2()).isEqualTo(8);
        assertThat(counter.get()).isEqualTo(2);
    }
}
```

由于同一个类的调用不会触发缓存，因此计数器的数量等于2，**这表明参数为2的方法square被调用了两次，缓存被忽略**。这不是我们的期望，因此我们需要确定此行为的原因。

## 3. 分析问题

[Spring AOP](https://www.baeldung.com/spring-aop)支持@Cacheable方法的缓存行为。如果我们使用IDE来调试这段代码，我们会找到一些线索。MathServiceIntegrationTest中的变量mathService指向MathService$$EnhancerBySpringCGLIB$$5cdf8ec8的实例，而MathService中的this则指向MathService的实例。

**MathService$$EnhancerBySpringCGLIB$$5cdf8ec8是Spring生成的代理类，它拦截MathService的@Cacheable方法上的所有请求，并使用缓存的值进行响应**。

另一方面，**MathService本身没有缓存的能力，所以同一个类内的内部调用不会得到缓存的值**。

现在我们了解了其中的机制，让我们寻找解决这个问题的方法。显然，最简单的方法是将@Cacheable方法移至另一个bean。但是，如果由于某种原因我们必须将方法保留在同一个bean中，我们有三种可能的解决方案：

- 自注入
- 编译时织入
- 加载时织入

在我们的[AspectJ简介](https://www.baeldung.com/aspectj)文章中，详细介绍了面向切面编程(AOP)和不同的织入方法。织入是一种插入代码的方法，当我们将源代码编译成.class文件时，就会发生这种情况。它包括AspectJ中的编译时织入、编译后织入和加载时织入。由于编译后织入用于第三方库的织入，这不是我们的情况，因此我们只关注编译时织入和加载时织入。

## 4. 解决方案1：自注入

[自注入](https://www.baeldung.com/spring-self-injection)是绕过[Spring AOP](https://www.baeldung.com/spring-aop)限制的常用解决方案，**它允许我们获取对Spring增强型bean的引用并通过该bean调用方法**。在我们的例子中，我们可以将mathService bean[自动注入](https://www.baeldung.com/spring-autowire)到名为self的成员变量，并通过self调用square方法，而不是使用this引用：

```java
@Service
@CacheConfig(cacheNames = "square")
@Scope(proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MathService {

    @Autowired
    private MathService self;

    // other code

    public double sumOfSquareOf3() {
        return self.square(3) + self.square(3);
    }
}
```

由于循环引用，@Scope注解有助于创建存根代理并将其注入到self，稍后将用相同的MathService实例填充它。测试表明square方法只执行一次：

```java
@Test
void givenCacheableMethod_whenInvokingByExternalCall_thenCacheIsTriggered() {
    AtomicInteger counter = mathService.resetCounter();

    assertThat(mathService.sumOfSquareOf3()).isEqualTo(18);
    assertThat(counter.get()).isEqualTo(1);
}
```

## 5. 解决方案2：编译时织入

顾名思义，编译时织入中的织入过程发生在编译时，**这是最简单的织入方法**。当我们同时拥有切面的源代码和我们在其中使用切面的代码时，AspectJ编译器将从源代码进行编译并生成织入类文件作为输出。

在Maven项目中，我们可以使用Mojo的AspectJ Maven插件，使用AspectJ编译器将AspectJ切面织入到我们的类中。对于@Cacheable注解，切面的源代码由库spring-aspects提供，因此我们需要将其添加为Maven依赖项和AspectJ Maven插件的切面库。

启用编译时织入需要三个步骤。首先，让我们通过在任何配置类上添加@EnableCaching注解来启用AspectJ模式的缓存：

```java
@EnableCaching(mode = AdviceMode.ASPECTJ)
```

其次，我们需要添加spring-aspects依赖项：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
</dependency>
```

第三，让我们为编译目标定义aspectj-maven-plugin：

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>${aspectj-plugin.version}</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <complianceLevel>${java.version}</complianceLevel>
        <Xlint>ignore</Xlint>
        <encoding>UTF-8</encoding>
        <aspectLibraries>
            <aspectLibrary>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
            </aspectLibrary>
        </aspectLibraries>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

当我们执行mvn clean compile时，上面显示的AspectJ Maven插件将织入切面。使用编译时织入，我们不需要更改代码，并且square方法只会执行一次：

```java
@Test
void givenCacheableMethod_whenInvokingByInternalCall_thenCacheIsTriggered() {
    AtomicInteger counter = mathService.resetCounter();

    assertThat(mathService.sumOfSquareOf2()).isEqualTo(8);
    assertThat(counter.get()).isEqualTo(1);
}
```

## 6. 解决方案3：加载时织入

加载时织入只是二进制织入，延迟到类加载器加载类文件并将类定义到JVM为止。可以使用AspectJ代理来启用AspectJ加载时织入，以参与类加载过程并在VM中定义任何类型之前织入它们。

启用加载时织入还需要三个步骤。首先，通过在任何配置类上添加两个注解来启用AspectJ模式和加载时织入器的缓存：

```java
@EnableCaching(mode = AdviceMode.ASPECTJ)
@EnableLoadTimeWeaving
```

其次，让我们添加spring-aspects依赖项：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
</dependency>
```

最后，我们为JVM指定javaagent选项-javaagent:path/to/aspectjweaver.jar或使用Maven插件来配置javaagent：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>${maven-surefire-plugin.version}</version>
            <configuration>
                <argLine>
                    --add-opens java.base/java.lang=ALL-UNNAMED
                    --add-opens java.base/java.util=ALL-UNNAMED
                    -javaagent:"${settings.localRepository}"/org/aspectj/aspectjweaver/${aspectjweaver.version}/aspectjweaver-${aspectjweaver.version}.jar
                    -javaagent:"${settings.localRepository}"/org/springframework/spring-instrument/${spring.version}/spring-instrument-${spring.version}.jar
                </argLine>
                <useSystemClassLoader>true</useSystemClassLoader>
                <forkMode>always</forkMode>
                <includes>
                    <include>cn.tuyucheng.taketoday.selfinvocation.LoadTimeWeavingIntegrationTest</include>
                </includes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

测试givenCacheableMethod_whenInvokingByInternalCall_thenCacheIsTriggered也将通过加载时织入。

## 7. 总结

在这篇文章中，我们解释了为什么当从同一个bean调用@Cacheable方法时缓存不生效。然后，我们分享了自注入和两种织入方案来解决这个问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-modules/spring-aop-2)上获得。