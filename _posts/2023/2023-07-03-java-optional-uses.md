---
layout: post
title:  Java中Optional的用途
category: java
copyright: java
excerpt: Java Optional
---

## 1. 概述

在本教程中，我们将介绍Java中Optional类的用途以及在构建应用程序时使用它的一些优势。

## 2. Java中Optional<T\>的用途

Optional是一个表示某些内容存在或不存在的类。从技术上讲，Optional是泛型T的包装类，如果T为null，则Optional实例为空。否则，它为包含值。

根据[Java 11文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html)，**Optional的目的是提供一种返回类型，该类型可以在返回null可能导致意外错误(例如臭名昭著的NullPointerException)的情况下表示值的缺失**。

### 2.1 有用的方法

Optional类提供了有用的方法来帮助我们使用该API，对于本文而言，重要的是of()、orElse()和empty()方法：

-   of(T value)返回一个Optional的实例，其中包含值
-   orElse(T other)返回Optional中的值，否则返回other
-   最后，empty()返回Optional的一个空实例

## 3. Optional的优势

我们已经了解了Optional的用途及其一些方法。但是，我们怎样才能从该类中受益呢？在本节中，我们将看到一些使用它的方法，这些方法可以帮助我们构建清晰而健壮的API。

### 3.1 Optional与Null

在创建Optional类之前，我们必须使用null来表示值的缺失。该语言并没有要求我们正确处理null情况，也就是说，**空值检查有时是必要的，但不是强制的**。因此，创建返回null的方法被证明是一种产生意外运行时错误(如NullPointerException)的方法。

另一方面，**应始终在编译时正确处理Optional的实例以获取其中的值**。这种在编译时处理Optional的义务导致更少的意外NullPointerException。

让我们尝试一个模拟User数据库的例子：

```java
public class User {

    public User(String id, String name) {
        this.id = id;
        this.name = name;
    }

    private String id;

    private String name;

    public String getName() {
        return name;
    }

    public String getId() {
        return id;
    }
}
```

让我们还定义Repository类，如果找到则返回一个用户。否则，它返回null：

```java
public class UserRepositoryWithNull {

    private final List<User> dbUsers = Arrays.asList(new User("1", "John"), new User("2", "Maria"), new User("3", "Daniel"));

    public User findById(String id) {

        for (User u : dbUsers) {
            if (u.getId().equals(id)) {
                return u;
            }
        }

        return null;
    }
}
```

现在，我们将编写一个单元测试，以显示如果我们不使用null检查来解决null，代码将如何因NullPointerException而中断：

```java
@Test
public void givenNonExistentUserId_whenSearchForUser_andNoNullCheck_thenThrowException() {
    UserRepositoryWithNull userRepositoryWithNull = new UserRepositoryWithNull();
    String nonExistentUserId = "4";

    assertThrows(NullPointerException.class, () -> {
        System.out.println("User name: " + userRepositoryWithNull.findById(nonExistentUserId).getName());
    });
}
```

**Java允许我们使用findById()返回的对象中的getName()方法，而无需进行空检查。在这种情况下，我们只能在运行时发现问题**。

为了避免这种情况，我们可以创建另一个Repository，如果我们找到一个用户，那么我们返回一个完整的Optional。否则，我们返回一个空的：

```java
public class UserRepositoryWithOptional {

    private final List<User> dbUsers = Arrays.asList(new User("1", "John"), new User("2", "Maria"), new User("3", "Daniel"));

    public Optional<User> findById(String id) {

        for (User u : dbUsers) {
            if (u.getId().equals(id)) {
                return Optional.of(u);
            }
        }

        return Optional.empty();
    }
}
```

现在，当我们重写单元测试时，我们看到当我们找不到任何User时，我们必须如何首先处理Optional以获得它的值：

```java
@Test
public void givenNonExistentUserId_whenSearchForUser_thenOptionalShouldBeTreatedProperly() {
    UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();
    String nonExistentUserId = "4";

    String userName = userRepositoryWithOptional.findById(nonExistentUserId)
        .orElse(new User("0", "admin"))
        .getName();

    assertEquals("admin", userName);
}
```

在上面的例子中，我们没有找到任何User，所以我们可以使用orElse()方法返回一个默认User。要获得它的值，必须在编译时正确处理Optional。这样，我们就可以减少运行时的意外错误。

除了使用orElse()方法提供默认值之外，我们还可以使用另外两个选项，即[使用orElseThrow()抛出异常](https://www.baeldung.com/java-optional-throw-exception)或[使用orElseGet()调用Supplier函数](https://www.baeldung.com/java-optional-or-else-vs-or-else-get)。

### 3.2 设计明确的意图API

正如我们之前所讨论的，null被广泛用来表示什么都没有。但是，null的含义只有创建API的人才能清楚，浏览该API的其他开发人员可能会发现null的不同含义。

可能名称“可选”是Optional在构建我们的API时成为有用工具的主要原因，**方法返回中的Optional提供了我们应该从该方法中期望的明确意图：它返回一些东西或什么都不返回**。不需要文档来解释该方法的返回类型，代码会自行解释。

使用返回null的Repository，我们可能会以最糟糕的方式发现null表示在数据库中找不到用户。或者它可能表示其他内容，比如连接数据库时出错，或者对象尚未初始化。很难知道。

另一方面，使用返回Optional实例的Repository，只需查看方法签名即可清楚地知道，我们可能会或可能不会在数据库中找到用户。

**设计清晰的API的一个重要实践是永远不要返回null Optional**，方法应始终使用静态方法返回Optional的有效实例。

### 3.3 声明式编程

使用Optional类的另一个很好的理由是能够使用一系列流式的方法，它提供了一个类似于集合中的stream()的“伪流”，但只有一个值。这意味着我们可以在其中的值上调用map()和filter()之类的方法，这有助于创建更具[声明性的程序，而不是命令式程序](https://www.baeldung.com/cs/imperative-vs-declarative-programming)。

假设要求是如果name以字母“M”开头，则将User名称的大小写更改为大写。

首先，让我们看看命令式的方式，使用不返回Optional的Repository：

```java
@Test
public void givenExistentUserId_whenFoundUserWithNameStartingWithMInRepositoryUsingNull_thenNameShouldBeUpperCased() {
    UserRepositoryWithNull userRepositoryWithNull = new UserRepositoryWithNull();

    User user = userRepositoryWithNull.findById("2");
    String upperCasedName = "";

    if (user != null) {
        if (user.getName().startsWith("M")) {
            upperCasedName = user.getName().toUpperCase();
        }
    }

    assertEquals("MARIA", upperCasedName);
}
```

现在，让我们看一下声明式方式，使用Repository的Optional版本：

```java
@Test
public void givenExistentUserId_whenFoundUserWithNameStartingWithMInRepositoryUsingOptional_thenNameShouldBeUpperCased() {
    UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();

    String upperCasedName = userRepositoryWithOptional.findById("2")
        .filter(u -> u.getName().startsWith("M"))
        .map(u -> u.getName().toUpperCase())
        .orElse("");

    assertEquals("MARIA", upperCasedName);
}
```

命令式方式需要嵌套两个if语句来判断对象是否不为null和过滤用户名。如果未找到用户，则大写字符串保持为空。

在声明式方式中，我们使用lambda表达式过滤名称并将大写函数映射到找到的User。如果未找到用户，我们将使用orElse()返回一个空字符串。

我们使用哪一个仍然是一个偏好问题，他们都达到了相同的结果，命令式方式需要更多挖掘才能理解代码的含义。例如，如果我们在第一个或第二个if语句中添加更多逻辑，它可能会对该代码的意图造成一些混淆。**在这种情况下，声明式编程明确了代码的意图**：返回大写的名称，否则返回空字符串。

## 4. 总结

在本文中，我们介绍了Optional类的用途以及如何有效地使用它来设计清晰而健壮的API。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-optional)上获得。