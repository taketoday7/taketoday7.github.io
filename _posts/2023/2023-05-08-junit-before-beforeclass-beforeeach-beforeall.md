---
layout: post
title:  Before与BeforeClass vs BeforeEach与BeforeAll
category: unittest
copyright: unittest
excerpt: JUnit 5 @BeforeEach和@BeforeAll
---

## 1. 概述

在这个简短的教程中，我们将解释JUnit 4和5中@Before、@BeforeClass、@BeforeEach和@BeforeAll注解之间的区别-并通过实际代码说明如何使用它们。

## 2. @Before

**使用@Before注解标注的方法在每次测试之前运行**。当我们想在运行测试之前执行一些通用代码时，这很有用。

让我们初始化一个列表并添加一些元素：

```java
@RunWith(JUnit4.class)
public class BeforeAndAfterAnnotationsUnitTest {

    // ...

    private List<String> list;

    @Before
    public void init() {
        LOG.info("startup");
        list = new ArrayList<>(Arrays.asList("test1", "test2"));
    }

    @After
    public void teardown() {
        LOG.info("teardown");
        list.clear();
    }
}
```

**请注意，我们还添加了另一个用@After标注的方法，以便在执行每个测试后清除列表中的元素**。

现在让我们添加一些测试来检查列表的大小：

```java
@Test
public void whenCheckingListSize_thenSizeEqualsToInit() {
    LOG.debug("executing test");
    assertEquals(2, list.size());

    list.add("another test");
}

@Test
public void whenCheckingListSizeAgain_thenSizeEqualsToInit() {
    LOG.debug("executing another test");
    assertEquals(2, list.size());

    list.add("yet another test");
}
```

在这种情况下，**确保在运行每个测试之前正确设置测试环境至关重要，因为在每次测试执行期间都会修改列表**。

如果我们查看日志输出，我们可以验证init和teardown方法是否在每个测试方式中运行一次：

```text
... startup 
... executing another test 
... teardown 
... startup 
... executing test 
... teardown 
```

## 3. @BeforeClass

当我们想在每次测试之前执行一个耗时的通用操作时，**最好在使用@BeforeClass运行所有测试之前只执行一次**。

一些常见的耗时操作的示例是创建数据库连接或启动服务器。

让我们创建一个简单的测试类来模拟数据库连接的创建：

```java
@RunWith(JUnit4.class)
public class BeforeClassAndAfterClassAnnotationsUnitTest {

    // ...

    @BeforeClass
    public static void setup() {
        LOG.info("startup - creating DB connection");
    }

    @AfterClass
    public static void tearDown() {
        LOG.info("closing DB connection");
    }
}
```

请注意，**这些方法必须是静态的**，因此它们将在运行类的测试之前执行。

和以前一样，让我们也添加一些简单的测试：


```java
@Test
public void simpleTest() {
    LOG.info("simple test");
}

@Test
public void anotherSimpleTest() {
    LOG.info("another simple test");
}
```

此时，如果我们检查日志输出，我们可以看到setup和tearDown方法只运行了一次：

```text
... startup - creating DB connection
... simple test
... another simple test
... closing DB connection
```

## 4. @BeforeEach和@BeforeAll

**@BeforeEach和@BeforeAll是@Before和@BeforeClass的JUnit 5等价物**。为了避免混淆，这些注解已使用更清晰的名称重命名。

让我们使用这些新注解复制我们之前的类，从@BeforeEach和@AfterEach注解开始：

```java
class BeforeEachAndAfterEachAnnotationsUnitTest {

    // ...

    private List<String> list;

    @BeforeEach
    void init() {
        LOG.info("startup");
        list = new ArrayList<>(Arrays.asList("test1", "test2"));
    }

    @AfterEach
    void teardown() {
        LOG.info("teardown");
        list.clear();
    }

    // ...
}
```

如果我们观察日志，我们可以看到它与@Before和@After注解的工作方式相同：

```text
... startup 
... executing another test 
... teardown 
... startup 
... executing test 
... teardown 
```

最后，让我们对另一个测试类执行相同的操作，以理解@BeforeAll和@AfterAll注解的作用：

```java
class BeforeAllAndAfterAllAnnotationsUnitTest {

    // ...

    @BeforeAll
    static void setup() {
        LOG.info("startup - creating DB connection");
    }

    @AfterAll
    static void tearDown() {
        LOG.info("closing DB connection");
    }

    // ...
}
```

输出与旧注解相同：

```text
... startup - creating DB connection
... simple test
... another simple test
... closing DB connection
```

## 5. 总结

在本文中，我们演示了JUnit中@Before、@BeforeClass、@BeforeEach和@BeforeAll注解之间的区别，以及应该何时使用它们。

与往常一样，示例的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/junit-5-basics)上获得。