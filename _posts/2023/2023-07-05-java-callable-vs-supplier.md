---
layout: post
title:  何时在Java中使用Callable和Supplier
category: java
copyright: java
excerpt: Java Lambda
---

## 1. 概述

在本教程中，我们将讨论结构相似但用法不同的Callable和Supplier[函数接口](https://www.baeldung.com/java-8-functional-interfaces)。

两者都返回一个类型值并且不接收任何参数，执行上下文是确定差异的判别式。

在本教程中，我们将重点介绍异步任务的上下文。

## 2. 模型

在开始之前，让我们定义一个类：

```java
public class User {

    private String name;
    private String surname;
    private LocalDate birthDate;
    private Integer age;
    private Boolean canDriveACar = false;

    // standard constructors, getters and setters
}
```

## 3. Callable

Callable是Java版本5中引入的接口，在版本8中演变为函数式接口。

**它的SAM(Single Abstract Method)是方法call()，返回一个泛型值，可能会抛出异常**：

```java
V call() throws Exception;
```

它旨在封装应该由另一个线程执行的任务，例如[Runnable](https://www.baeldung.com/java-runnable-callable)接口。这是因为Callable实例可以通过[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)执行。

因此，让我们定义一个实现：

```java
public class AgeCalculatorCallable implements Callable<Integer> {

    private final LocalDate birthDate;

    @Override
    public Integer call() throws Exception {
        return Period.between(birthDate, LocalDate.now()).getYears();
    }

    // standard constructors, getters and setters
}
```

当call()方法返回一个值时，主线程检索它以执行其逻辑。为此，我们可以使用[Future](https://www.baeldung.com/java-future)，这是一个在另一个线程上执行的任务完成时跟踪并获取值的对象。

### 3.1 单个任务

让我们定义一个只执行一个异步任务的方法：

```java
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    try {
        Future<Integer> ageFuture = executorService.submit(new AgeCalculatorCallable(user.getBirthDate()));
        user.setAge(age.get());
    } catch (ExecutionException | InterruptedException e) {
        throw new RuntimeException(e.getCause());
    }
    return user;
}
```

我们可以通过lambda表达式重写submit()的内部块：

```java
Future<Integer> ageFuture = executorService.submit(
    () -> Period.between(user.getBirthDate(), LocalDate.now()).getYears());
```

当我们尝试通过调用get()方法访问返回值时，我们必须处理两个受检异常：

-   InterruptedException：当线程处于休眠、活动或占用状态时发生中断时抛出
-   ExecutionException：当通过抛出异常中止任务时抛出。换句话说，它是一个包装器异常，中止任务的真正异常是cause(可以使用getCause()方法检查)

### 3.2 任务链

执行属于链的任务取决于先前任务的状态，如果其中之一失败，则无法执行当前任务。

因此，让我们定义一个新的Callable：

```java
public class CarDriverValidatorCallable implements Callable<Boolean> {

    private final Integer age;

    @Override
    public Boolean call() throws Exception {
        return age > 18;
    }
    // standard constructors, getters and setters
}
```

接下来，让我们定义一个任务链，其中第二个任务将前一个任务的结果作为输入参数：

```java
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    try {
        Future<Integer> ageFuture = executorService.submit(new AgeCalculatorCallable(user.getBirthDate()));
        Integer age = ageFuture.get();
        Future<Boolean> canDriveACarFuture = executorService.submit(new CarDriverValidatorCallable(age));
        Boolean canDriveACar = canDriveACarFuture.get();
        user.setAge(age);
        user.setCanDriveACar(canDriveACar);
    } catch (ExecutionException | InterruptedException e) {
        throw new RuntimeException(e.getCause());
    }
    return user;
}
```

在任务链中使用Callable和Future存在一些问题：

-   链中的每个任务都遵循“提交-获取”模式，在很长的任务链中，这会产生冗长的代码。
-   当链可以容忍任务失败时，我们应该创建一个专用的try/catch块。
-   调用时，get()方法会一直等待，直到Callable返回一个值，因此链的总执行时间等于所有任务执行时间的总和。但是，如果下一个任务仅依赖于前一个任务的正确执行，则链式过程会显著减慢。

## 4. Supplier

Supplier是一个函数接口，其SAM(单一抽象方法)是get()。

**它不接收任何参数，返回一个值，并且只抛出非受检的异常**：

```java
T get();
```

此接口最常见的用例之一是推迟某些代码的执行。

[Optional](https://www.baeldung.com/java-optional)类有一些方法接收Supplier作为参数，例如Optional.or()、Optional.orElseGet()。

因此，Supplier只有在Optional为空的时候才会执行。

我们还可以在异步计算上下文中使用它，特别是在[CompletableFuture](https://www.baeldung.com/java-completablefuture) API中。

某些方法接收Supplier作为参数，例如supplyAsync()方法。

### 4.1 单个任务

让我们定义一个只执行一个异步任务的方法：

```java
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    CompletableFuture<Integer> ageFut = CompletableFuture.supplyAsync(() -> Period.between(user.getBirthDate(), LocalDate.now())
        .getYears(), executorService)
        .exceptionally(throwable -> {throw new RuntimeException(throwable);});
    user.setAge(ageFut.join());
    return user;
}
```

在这种情况下，lambda表达式定义了Supplier，但我们也可以定义一个实现类。多亏了CompletableFuture，我们为异步操作定义了一个模板，使其更易于理解和修改。

join()方法提供Supplier的返回值。

### 4.2 任务链

我们还可以在Supplier接口和CompletableFuture的支持下开发一系列任务：

```java
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    CompletableFuture<Integer> ageFut = CompletableFuture.supplyAsync(() -> Period.between(user.getBirthDate(), LocalDate.now())
        .getYears(), executorService);
    CompletableFuture<Boolean> canDriveACarFut = ageFut.thenComposeAsync(age -> CompletableFuture.supplyAsync(() -> age > 18, executorService))
        .exceptionally((ex) -> false);
    user.setAge(ageFut.join());
    user.setCanDriveACar(canDriveACarFut.join());
    return user;
}
```

使用CompletableFuture–Supplier方法定义异步任务链可以解决之前使用Future–Callable方法引入的一些问题：

-   链中的每个任务都是独立的，因此，如果任务执行失败，我们可以通过exceptionally()块来处理它。
-   join()方法不需要在编译时处理受检的异常。
-   我们可以设计一个异步任务模板，完善每个任务的状态处理。

## 5. 总结

在本文中，我们讨论了Callable和Supplier接口之间的区别，重点关注异步任务的上下文。

**接口设计级别的主要区别是Callable抛出的是受检异常**。 

Callable不适用于函数上下文。它随着时间的推移而适应，函数式编程和受检异常不相容。

因此，任何函数式API(例如CompletableFuture API)总是接收Supplier而不是Callable。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-lambdas)上获得。