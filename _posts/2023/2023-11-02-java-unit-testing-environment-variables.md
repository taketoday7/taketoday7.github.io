---
layout: post
title: 如何在单元测试中Mock环境变量
category: unittest
copyright: unittest
excerpt: JUnit
---

## 1. 概述

当我们对依赖于环境变量的代码进行单元测试时，我们可能希望为它们提供特定的值作为测试实现的一部分。

Java不允许我们编辑环境变量，但是我们可以使用一些解决方法，以及一些可以帮助我们的库。

在本教程中，我们将了解单元测试中依赖环境变量的挑战、Java如何在最新版本中使这一过程变得更加困难，以及[JUnit Pioneer](https://junit-pioneer.org/)、[System Stubs](https://www.baeldung.com/java-system-stubs)、[System Lambda和System Rules](https://www.baeldung.com/java-system-rules-junit)库。我们将针对[JUnit 4](https://www.baeldung.com/tag/junit)、[JUnit 5](https://www.baeldung.com/junit-5)和[TestNG](https://www.baeldung.com/testng)进行研究。

## 2. 改变环境变量的挑战

在其他语言中，例如JavaScript，我们可以非常轻松地修改测试中的环境：

```javascript
beforeEach(() => {
    process.env.MY_VARIABLE = 'set';
});
```

Java更加严格。**在Java中，环境变量map是不可变的**。它是一个不可修改的Map，在JVM启动时初始化。尽管有充分的理由，但我们仍然希望在测试时控制我们的环境。

### 2.1 为什么环境是不可变的

在Java程序正常执行的情况下，如果修改像运行时环境配置这样的全局配置，可能会造成混乱。**当涉及多个线程时，这尤其危险**。例如，一个线程可能会与另一个线程同时修改环境，启动具有该环境的进程，并且任何冲突的设置都可能以意外的方式进行交互。

因此，Java的设计者保证了环境变量映射中全局值的安全。相反，**[系统属性](https://www.baeldung.com/java-system-get-property-vs-system-getenv)很容易在运行时更改**。

### 2.2 围绕不可修改Map处理

对于不可变的环境变量Map对象有一个解决方法，**尽管它是只读的UnmodifyingMap类型，但我们可以打破封装并使用[反射](https://www.baeldung.com/java-reflection)访问内部字段**：

```java
Class<?> classOfMap = System.getenv().getClass();
Field field = classOfMap.getDeclaredField("m");
field.setAccessible(true);
Map<String, String> writeableEnvironmentVariables = (Map<String, String>)field.get(System.getenv());
```

UnmodificableMap包装对象中的字段m是一个我们可以更改的可变Map：

```java
writeableEnvironmentVariables.put("tuyucheng", "has set an environment variable");

assertThat(System.getenv("tuyucheng")).isEqualTo("has set an environment variable");
```

实际上，在Windows上，有一个ProcessEnvironment的替代实现，它也考虑了不区分大小写的环境变量，因此使用上述技术的库也必须考虑到这一点。然而，原则上，这就是我们解决不可变环境变量Map的方法。

**在JDK 16之后，模块系统对JDK内部的保护变得更加严格，并且使用这种[反射访问](https://www.baeldung.com/java-illegal-reflective-access)变得更加困难**。

### 2.3 当反射访问不起作用时

自[JDK 17](https://openjdk.org/jeps/403)起，Java模块系统默认禁用其核心内部的反射修改。这些被认为是不安全的做法，如果将来内部发生变化，可能会导致运行时错误。

我们可能会收到这样的错误：

```shell
Unable to make field private static final java.util.HashMap java.lang.ProcessEnvironment.theEnvironment accessible: 
  module java.base does not "opens java.lang" to unnamed module @fdefd3f
```

这表明Java模块系统正在阻止使用反射。可以通过在pom.xml中的测试运行器配置中添加一些额外的命令行参数来修复此问题，以使用–add-opens来允许这种反射访问：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <argLine>
            --add-opens java.base/java.util=ALL-UNNAMED
            --add-opens java.base/java.lang=ALL-UNNAMED
        </argLine>
    </configuration>
</plugin>
```

这种解决方法允许我们编写代码并使用通过反射打破封装的工具。然而，我们可能希望避免这种情况，因为打开这些模块可能会导致不安全的编码实践，这些实践在测试时有效，但在运行时意外失败。我们可以选择不需要此解决方法的工具。

### 2.4 为什么我们需要以编程方式设置环境变量

我们的单元测试可以使用测试运行程序设置的环境变量来运行，如果我们有适用于整个测试套件的全局配置，这可能是我们的首选。

我们可以通过在pom.xml中的Surefire配置中添加一个环境变量来实现这一点：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <environmentVariables>
            <SET_BY_SUREFIRE>YES</SET_BY_SUREFIRE>
        </environmentVariables>
    </configuration>
</plugin>
```

然后这个变量对我们的测试可见：

```java
assertThat(System.getenv("SET_BY_SUREFIRE")).isEqualTo("YES");
```

但是，我们的代码可能会根据不同的环境变量设置而有不同的操作，我们可能更希望能够在不同的测试用例中使用环境变量的不同值来测试此行为的所有变体。

同样，我们在测试时可能会得到一些在编码时无法预测的值。一个很好的例子是我们在[Docker容器](https://www.baeldung.com/spring-boot-testcontainers-integration-test)中运行[WireMock](https://www.baeldung.com/introduction-to-wiremock)或测试数据库的端口。

### 2.5 从测试库获得正确的帮助

有几个测试库可以帮助我们在测试时设置环境变量，每个库都有自己的与不同测试框架和JDK版本的兼容性级别。

我们可以根据我们的首选工作流程、是否提前知道环境变量的值以及我们计划使用哪个JDK版本来选择正确的库。

我们应该注意到，所有这些库不仅仅涵盖环境变量。他们都采用在进行更改之前捕获当前环境并在测试完成后将环境恢复到原来状态的方法。

## 3. 使用JUnit Pioneer设置环境变量

JUnit Pioneer是JUnit 5的一组扩展，它提供了一种基于注解的方法来设置和清除环境变量。

我们可以添加[junit-pioneer](https://mvnrepository.com/artifact/org.junit-pioneer/junit-pioneer)依赖：

```xml
<dependency>
    <groupId>org.junit-pioneer</groupId>
    <artifactId>junit-pioneer</artifactId>
    <version>2.1.0</version>
    <scope>test</scope>
</dependency>
```

### 3.1 使用SetEnvironmentVariable注解

我们可以使用SetEnvironmentVariable注解来标注测试类或方法，并且我们的测试代码会使用环境中设置的值进行操作：

```kotlin
@SetEnvironmentVariable(key = "pioneer", value = "is pioneering")
class EnvironmentVariablesSetByJUnitPioneerUnitTest {
}
```

我们应该注意的是，key和value必须在编译时已知。

然后我们的测试可以使用环境变量：

```java
@Test
void variableCanBeRead() {
    assertThat(System.getenv("pioneer")).isEqualTo("is pioneering");
}
```

我们可以多次使用@SetEnvironmentVariable注解来设置多个变量。

### 3.2 清除环境变量

我们可能还希望清除系统提供的环境变量，甚至是为某些特定测试在类级别设置的一些环境变量：

```java
@ClearEnvironmentVariable(key = "pioneer")
@Test
void givenEnvironmentVariableIsClear_thenItIsNotSet() {
    assertThat(System.getenv("pioneer")).isNull();
}
```

### 3.3 JUnit Pioneer的局限性

**JUnit Pioneer只能与JUnit 5一起使用**。它使用反射，因此需要Java 16或更低版本，或者使用add-opens解决方法。

## 4. 使用System Stubs设置环境变量

System Stubs具有对JUnit 4、JUnit 5和TestNG的测试支持。与其前身System Lambda一样，它也可以在任何框架中的任何测试代码主体中独立使用。**System Stubs与JDK 11及以上的所有版本兼容**。

### 4.1 在JUnit 5中设置环境变量

为此，我们需要[System Stubs JUnit 5](https://mvnrepository.com/artifact/uk.org.webcompere/system-stubs-jupiter)依赖项：

```xml
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-jupiter</artifactId>
    <version>2.1.3</version>
    <scope>test</scope>
</dependency>
```

首先，我们需要将扩展添加到我们的测试类中：

```java
@ExtendWith(SystemStubsExtension.class)
class EnvironmentVariablesUnitTest {
}
```

我们可以使用我们希望使用的环境变量将EnvironmentVariables存根对象初始化为测试类的字段：

```java
@SystemStub
private EnvironmentVariables environment = new EnvironmentVariables("MY VARIABLE", "is set");
```

值得注意的是，**我们必须使用@SystemStub标注该对象**，以便扩展知道如何使用它。

然后，SystemStubsExtension在测试期间激活此替代环境，并在之后将其清除。**在测试过程中，EnvironmentVariables对象也可以被修改**，并且对System.getenv()的调用会接收最新配置。

让我们再看一个更复杂的情况，我们希望设置一个环境变量，其值仅在测试初始化时已知。在这种情况下，由于我们将在beforeEach()方法中提供一个值，因此我们不需要在初始化列表中创建该对象的实例：

```java
@SystemStub
private EnvironmentVariables environmentVariables;
```

当JUnit调用beforeEach()时，扩展已经为我们创建了对象，我们可以使用它来设置我们需要的环境变量：

```java
@BeforeEach
void beforeEach() {
    environmentVariables.set("systemstubs", "creates stub objects");
}
```

当我们的测试执行时，环境变量将被激活：

```java
@Test
void givenEnvironmentVariableHasBeenSet_thenCanReadIt() {
    assertThat(System.getenv("systemstubs")).isEqualTo("creates stub objects");
}
```

测试方法完成后，环境变量返回到修改之前的状态。

### 4.2 在JUnit 4中设置环境变量

为此，我们需要[System Stubs JUnit 4](https://mvnrepository.com/artifact/uk.org.webcompere/system-stubs-junit4)依赖：

```xml
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-junit4</artifactId>
    <version>2.1.3</version>
    <scope>test</scope>
</dependency>
```

**System Stubs提供了JUnit 4 Rule**。我们将其添加为测试类的一个字段：

```java
@Rule
public EnvironmentVariablesRule environmentVariablesRule = new EnvironmentVariablesRule("system stubs", "initializes variable");
```

这里我们使用环境变量对其进行了初始化，我们还可以在测试期间或在@Before方法中调用Rule上的set()来修改变量。

测试运行后，环境变量将处于活动状态：

```java
@Test
public void canReadVariable() {
    assertThat(System.getenv("system stubs")).isEqualTo("initializes variable");
}
```

### 4.3 在TestNG中设置环境变量

为此，我们需要[System Stubs TestNG](https://mvnrepository.com/artifact/uk.org.webcompere/system-stubs-testng)依赖：

```xml
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-testng</artifactId>
    <version>2.1.3</version>
    <scope>test</scope>
</dependency>
```

这提供了一个TestNG监听器，其工作方式类似于上面的JUnit 5解决方案。

我们将监听器添加到我们的测试类中：

```java
@Listeners(SystemStubsListener.class)
public class EnvironmentVariablesTestNGUnitTest {
}
```

然后我们添加一个用@SystemStub标注的EnvironmentVariables字段：

```java
@SystemStub
private EnvironmentVariables setEnvironment;
```

然后我们的beforeAll()方法可以初始化一些变量：

```java
@BeforeClass
public void beforeAll() {
    setEnvironment.set("testng", "has environment variables");
}
```

最后测试方法可以使用它们：

```java
@Test
public void givenEnvironmentVariableWasSet_thenItCanBeRead() {
    assertThat(System.getenv("testng")).isEqualTo("has environment variables");
}
```

### 4.4 不使用测试框架的System Stubs

System Stubs最初基于System Lambda的代码库，它附带的技术只能在单个测试方法中使用。这意味着测试框架的选择是完全开放的。

因此，System Stubs Core可用于在JUnit测试方法中的任何位置设置环境变量。

首先，让我们添加[system-stubs-core](https://mvnrepository.com/artifact/uk.org.webcompere/system-stubs-core)依赖：

```xml
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>system-stubs-core</artifactId>
    <version>2.1.3</version>
    <scope>test</scope>
</dependency>
```

现在，在我们的一种测试方法中，我们可以使用临时设置一些环境变量的构造来包围测试代码。首先我们需要从SystemStubs静态导入：

```java
import static uk.org.webcompere.systemstubs.SystemStubs.withEnvironmentVariables;
```

然后我们可以使用withEnvironmentVariables()方法来包装我们的测试代码：

```java
@Test
void useEnvironmentVariables() throws Exception {
    withEnvironmentVariables("system stubs", "in test")
        .execute(() -> {
            assertThat(System.getenv("system stubs")).isEqualTo("in test");
        });
}
```

在这里我们可以看到，assertThat()调用是对设置了变量的环境进行的操作。在execute()调用的闭包之外，环境变量不受影响。

我们应该注意，**这种技术要求我们的测试在测试方法上抛出Exception**，因为execute()函数必须处理可能调用带有受检异常的方法的闭包。

该技术还要求每个测试设置自己的环境，如果我们尝试使用生命周期大于单个测试的测试对象(例如Spring Context)，则该技术无法正常工作。

System Stubs允许其每个存根对象独立于测试框架进行设置和拆除。因此，我们可以使用测试类的beforeAll()和afterAll()方法来操作我们的EnvironmentVariables对象：

```java
private static EnvironmentVariables environmentVariables = new EnvironmentVariables();

@BeforeAll
static void beforeAll() throws Exception {
    environmentVariables.set("system stubs", "in test");
    environmentVariables.setup();
}

@AfterAll
static void afterAll() throws Exception {
    environmentVariables.teardown();
}
```

然而，测试框架扩展的好处是我们可以避免这种样板代码，因为它们为我们执行这些基础知识。

### 4.5 System Stubs的局限性

System Stubs的TestNG功能仅在版本2.1+版本中可用，并且仅限于Java 11及以上版本。

在其版本2发行版中，**System Stubs偏离了前面描述的常见的基于反射的技术**。它现在使用[ByteBuddy](https://www.baeldung.com/byte-buddy)来拦截环境变量调用。但是，如果项目使用低于11版本的JDK，则也无需使用这些更高版本。

System Stubs版本1提供与JDK 8到JDK 16的兼容性。

## 5. System Rules和System Lambda

System Rules是历史最悠久的环境变量测试库之一，它提供了用于设置环境变量的JUnit 4解决方案，其作者用System Lambda替换了它，以提供与测试框架无关的方法。它们基于相同的核心技术，用于在测试时替换环境变量。

### 5.1 使用System Rules设置环境变量

首先我们需要[system-rules](https://mvnrepository.com/artifact/com.github.stefanbirkner/system-rules)依赖：

```xml
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
```

然后我们将Rule添加到JUnit 4测试类中：

```java
@Rule
public EnvironmentVariables environmentVariablesRule = new EnvironmentVariables();
```

我们可以在@Before方法中设置值：

```java
@Before
public void before() {
    environmentVariablesRule.set("system rules", "works");
}
```

并在我们的测试方法中访问正确的环境：

```java
@Test
public void givenEnvironmentVariable_thenCanReadIt() {
    assertThat(System.getenv("system rules")).isEqualTo("works");
}
```

Rule对象environmentVariablesRule也允许我们在测试方法中立即设置环境变量。

### 5.2 使用System Lambda设置环境变量

为此，我们需要[system-lambda](https://mvnrepository.com/artifact/com.github.stefanbirkner/system-lambda)依赖：

```xml
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-lambda</artifactId>
    <version>1.2.1</version>
    <scope>test</scope>
</dependency>
```

正如System Stubs解决方案中已经演示的那样，我们可以将依赖于环境的代码放在测试中的闭包中。为此，我们应该静态导入SystemLambda：

```java
import static com.github.stefanbirkner.systemlambda.SystemLambda.withEnvironmentVariable;
```

然后我们就可以编写测试了：

```java
@Test
void enviromentVariableIsSet() throws Exception {
    withEnvironmentVariable("system lambda", "in test")
        .execute(() -> {
            assertThat(System.getenv("system lambda")).isEqualTo("in test");
        });
}
```

### 5.3 System Rules和System Lambda的限制

虽然这些都是成熟且广泛的库，但它们不能用于操作JDK 17及更高版本中的环境变量。

System Rules严重依赖于JUnit 4。我们无法使用System Lambda来设置测试夹具范围的环境变量，因此**它无法帮助我们进行Spring上下文初始化**。

## 6. 避免模拟环境变量

虽然我们已经讨论了在测试时修改环境变量的多种方法，但可能值得考虑这是否必要，甚至是否有益。

### 6.1 也许风险太大

正如我们在上面的每个解决方案中看到的那样，在运行时更改环境变量并不简单。如果存在多线程代码，情况可能会更加棘手。如果多个测试夹具在同一个JVM中并行运行(也许使用[JUnit 5的并发](https://www.baeldung.com/junit-5-parallel-tests)功能)，则存在不同测试可能试图以矛盾的方式同时控制环境的风险。

尽管上面的一些测试库在多个线程同时使用时可能不会崩溃，但很难预测从一个时刻到下一个时刻如何设置环境变量。更糟糕的是，一个线程可能会捕获另一个测试的临时环境变量，就好像它们是测试全部完成后让系统保持的正确状态一样。

作为另一个测试库的示例，当[Mockito Mock静态方法](https://www.baeldung.com/mockito-mock-static-methods)时，它会将其限制为当前线程，因为此类mock全局变量可能会破坏并发测试。因此，修改环境变量也会遇到完全相同的风险。一个测试可能会影响JVM的整个全局状态并在其他地方造成副作用。

同样，如果我们运行的代码只能通过环境变量进行控制，那么测试可能会非常困难，我们肯定可以通过设计来避免这种情况吗？

### 6.2 使用依赖注入

测试在构造时接收所有输入的系统比测试从系统资源中提取输入的系统更容易。

像Spring这样的依赖注入容器允许我们构建更容易在运行时隔离的情况下进行测试的对象。

我们还应该注意到，Spring将允许我们使用系统属性代替环境变量来设置其任何属性值。我们在本文中讨论的每个工具还支持在测试时设置和重置系统属性。

### 6.3 使用抽象

如果模块必须提取环境变量，也许它不应该直接依赖于System.getenv()，而可以使用环境变量读取器接口：

```java
@FunctionalInterface
interface GetEnv {
    String get(String name);
}
```

然后系统代码可以通过构造函数注入一个this对象：

```java
public class ReadsEnvironment {
    private GetEnv getEnv;

    public ReadsEnvironment(GetEnv getEnv) {
        this.getEnv = getEnv;
    }

    public String whatOs() {
        return getEnv.get("OS");
    }
}
```

在运行时，我们可以使用System::getenv实例化它，在测试时我们可以传入我们自己的替代环境：

```java
Map<String, String> fakeEnv = new HashMap<>();
fakeEnv.put("OS", "MacDowsNix");

ReadsEnvironment reader = new ReadsEnvironment(fakeEnv::get);
assertThat(reader.whatOs()).isEqualTo("MacDowsNix");
```

然而，这些环境变量的替代方案可能看起来非常繁重，让我们希望Java能够提供我们之前在JavaScript示例中看到的控制功能。同样，我们无法控制别人编写的代码，这可能依赖于环境变量。

因此，我们似乎不可避免地仍然会遇到我们希望能够在测试时动态控制某些环境变量的情况。

## 7. 总结

在本文中，我们研究了在测试时设置环境变量的选项。我们发现，当我们需要能够在运行时使这些变量灵活并可用于JDK 17及更高版本时，这变得更加困难。

然后，我们讨论了如果我们以不同的方式编写生产代码，是否可以完全避免这个问题。我们考虑了与在测试时修改环境变量相关的风险，尤其是并发测试。

我们还探索了四个最流行的用于在测试时设置环境变量的库：JUnit Pioneer、System Stubs、System Rules和System Lambda。其中每种方法都提供了不同的解决问题的方法，并且在JDK版本和测试框架之间具有不同的兼容性。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-libraries-2)上获得。