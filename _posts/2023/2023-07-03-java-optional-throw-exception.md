---
layout: post
title:  在Java 8的Optional中抛出异常
category: java
copyright: java
excerpt: Java Optional
---

## 1. 概述

在本教程中，我们将介绍如何在[Optional](https://www.baeldung.com/java-optional)为空时抛出自定义异常。

## 2. Optional.orElseThrow

简单地说，如果该值存在，则isPresent()将返回true，而调用get()将返回该值。否则，它会抛出NoSuchElementException。

还有一个方法**orElseThrow(Supplier<? extends X> exceptionSupplier)**允许我们提供一个自定义的Exception实例。只有存在时，此方法才会返回值。否则，它将抛出由提供的Supplier创建的异常。

## 3. 当值丢失时抛出异常

假设我们有一个返回可为空结果的方法：

```java
public String findNameById(String id) {
    return id == null ? null : "example-name";
}
```

现在我们将调用我们的findNameById(String id)方法两次，并使用ofNullable(T value)方法用Optional包装结果。

**Optional提供了一个用于创建新实例的静态工厂方法**，这个方法叫做ofNullable(T value)，然后我们可以调用orElseThrow。

我们可以通过运行此测试来验证行为：

```java
@Test
public void whenIdIsNull_thenExceptionIsThrown() {
    assertThrows(InvalidArgumentException.class, () -> Optional
        .ofNullable(personRepository.findNameById(null))
        .orElseThrow(InvalidArgumentException::new));
}
```

根据我们的实现，findNameById返回null。因此，InvalidArgumentException将从orElseThrow方法中抛出。

我们可以使用非空参数调用此方法。然后，我们不会得到InvalidArgumentException：

```java
@Test
public void whenIdIsNonNull_thenNoExceptionIsThrown() {
    assertAll(() -> Optional
        .ofNullable(personRepository.findNameById("id"))
        .orElseThrow(RuntimeException::new));
}
```

## 4. 当值存在时抛出异常

正如我们之前提到的，orElseThrow()返回包装的值(如果它存在)，否则抛出提供的异常。

现在，让我们看看当包含的值存在时如何抛出异常。

### 4.1 Optional.ifPresent()

简而言之，**这个方法允许我们在Optional的值不为null时执行特定的代码**。

例如，让我们看看如果我们找到具有特定id的User，如何使用此方法抛出[自定义异常](https://www.baeldung.com/java-new-custom-exception)。

首先，让我们创建UserFoundException异常：

```java
public class UserFoundException extends RuntimeException {

    public UserFoundException(String message) {
        super(message);
    }
}
```

**这里的基本思想是当findById()方法返回带有非空User对象的Optional时抛出UserFoundException**。

接下来，我们将创建另一个方法，该方法使用Optional的ifPresent()方法来抛出我们的自定义异常UserFoundException：

```java
public void throwExceptionWhenUserIsPresent(String id) {
    this.findById(id)
        .ifPresent(user -> {
            throw new UserFoundException("User with ID: " + user.getId() + " is found");
        });
}
```

现在，让我们创建一个测试用例来确认当包装的User存在时会引发我们的自定义异常：

```java
@Test
void givenExistentUserId_whenSearchForUser_thenThrowException() {
    final UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();
    String existentUserId = "2";

    assertThrows(UserFoundException.class, () -> userRepositoryWithOptional.throwExceptionWhenUserIsPresent(existentUserId));
}
```

我们还需要确保如果没有User可返回，则不会抛出UserFoundException：

```java
@Test
void givenNonExistentUserId_whenSearchForUser_thenDoNotThrowException() {
    final UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();
    String nonExistentUserId = "8";

    assertDoesNotThrow(() -> userRepositoryWithOptional.throwExceptionWhenUserIsPresent(nonExistentUserId));
}
```

**这种方法的唯一限制是我们丢失了包含的User-ifPresent()接收一个[Consumer](https://www.baeldung.com/java-8-functional-interfaces#Consumers)，但不返回任何内容**。

## 5. 总结 

在这篇简短的文章中，我们讨论了如何从Java 8 Optional中抛出异常。 

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-optional)上获得。