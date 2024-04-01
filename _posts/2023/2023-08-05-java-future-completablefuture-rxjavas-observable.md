---
layout: post
title:  Future、CompletableFuture和Rxjava的Observable之间的区别
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在Java中，我们可以通过多种方式异步运行任务。Java内置了[Future](https://www.baeldung.com/java-future)和[CompletableFuture](https://www.baeldung.com/java-completablefuture)，我们还可以使用[RxJava](https://www.baeldung.com/rx-java)库，它为我们提供了Observable类。在本文中，我们将研究这三者之间的差异以及各自的好处和潜在用例。

## 2. Future

**Future接口最初出现在Java 5中，提供的功能非常有限**。Future的实例是一个结果的占位符，该结果将由异步进程生成，并且可能尚不可用。提供了一小部分方法来帮助完成此进程，我们可以取消任务或获取已完成任务的结果，还可以检查任务是否已被取消或完成。

若要了解此操作的实际效果，让我们创建一个示例异步任务。我们将有一个对象和一个Callable，它的作用就像从数据库中检索该对象一样。我们的对象可以非常简单：

```java
class TestObject {
    int dataPointOne;
    int dataPointTwo;

    TestObject() {
        dataPointOne = 10;
    }
    // Standard getters and setters
}
```

因此，在调用构造函数时，我们返回一个包含数据点集之一的TestObject实例。现在，我们可以创建第二个实现Callable接口的类来为我们创建该对象：

```java
class ObjectCallable implements Callable<TestObject> {
    @Override
    TestObject call() {
        return new TestObject();
    }
}
```

设置这两个对象后，我们可以编写一个测试来使用Future获取TestObject：

```java
@Test
void whenRetrievingObjectWithBasicFuture_thenExpectOnlySingleDataPointSet() throws ExecutionException, InterruptedException {
    ExecutorService exec = Executors.newSingleThreadExecutor();
    Future<TestObject> future = exec.submit(new ObjectCallable());
    TestObject retrievedObject = future.get();
    assertEquals(10, retrievedObject.getDataPointOne());
    assertEquals(0, retrievedObject.getDataPointTwo());
}
```

在这里，我们创建了一个可以向其提交任务的ExecutorService。接下来，我们提交了ObjectCallable类并接收Future作为响应。最后，我们可以在Future上调用get()来获取结果。我们从断言中看到我们的对象填充了单个数据点。

## 3. CompletableFuture

**CompletableFuture是随Java 8一起发布的Future接口的实现，它扩展了Future的基本功能，使我们能够更好地控制异步操作的结果，最大的附加功能之一是将函数调用链接到初始任务的结果上的选项**。让我们通过重复我们在上一节中完成的任务来看看它的实际效果，但这一次，我们想在得到结果后对其进行水合。让我们创建一个具有水合方法的对象来填充TestObject中的第二个数据点：

```java
class ObjectHydrator {
    TestObject hydrateTestObject(TestObject testObject) {
        testObject.setDataPointTwo(20);
        return testObject;
    }
}
```

这次我们还需要从Supplier的实现中检索我们的初始TestObject：

```java
class ObjectSupplier implements Supplier<TestObject> {
    @Override
    TestObject get() {
        return new TestObject();
    }
}
```

准备好这两个类后，让我们使用它们：

```java
@Test
void givenACompletableFuture_whenHydratingObjectAfterRetrieval_thenExpectBothDataPointsSet() throws ExecutionException, InterruptedException {
    ExecutorService exec = Executors.newSingleThreadExecutor();
    ObjectHydrator objectHydrator = new ObjectHydrator();
    CompletableFuture<TestObject> future = CompletableFuture.supplyAsync(new ObjectSupplier(), exec)
        .thenApply(objectHydrator::hydrateTestObject);
    TestObject retrievedObject = future.get();
    assertEquals(10, retrievedObject.getDataPointOne());
    assertEquals(20, retrievedObject.getDataPointTwo());
}
```

这次我们可以从断言中看到，由于能够链接水合方法，我们已经在对象上设置了两个数据点。

## 4. RxJava的Observable

**[RxJava](https://www.baeldung.com/cs/reactive-programming)是一个库，允许我们按照响应式编程范例构建事件驱动和异步程序**。

要在项目中使用RxJava，我们需要将其导入到pom.xml中：

```xml
<dependency>
    <groupId>io.reactivex.rxjava3</groupId>
    <artifactId>rxjava</artifactId>
    <version>3.1.6</version>
</dependency>
```

最新版本可在[Maven仓库](https://mvnrepository.com/artifact/io.reactivex.rxjava3/rxjava)中找到。

这个库可以做很多事情，但这里我们将重点关注Observable类。**Observable根据需要或在数据可用时向观察者提供数据**，要异步运行任务，就像我们对Future和CompletableFuture所做的那样，我们可以创建一个Observable，它将在请求时从异步源生成数据：

```java
@Test
void givenAnObservable_whenRequestingData_thenItIsRetrieved() {
    ObjectHydrator objectHydrator = new ObjectHydrator();
    Observable<TestObject> observable = Observable.fromCallable(new ObjectCallable()).map(objectHydrator::hydrateTestObject);
    observable.subscribe(System.out::println);
}
```

在这里，我们从ObjectCallable类创建了一个Observable，并使用map()来应用我们的水化器，然后我们订阅Observable并提供一个方法来处理结果。在我们的例子中，我们只是将结果记录下来。这给出了与我们的CompletableFuture实现完全相同的最终结果。subscribe()方法与CompletableFutures get()的作用相同。 

虽然我们可以清楚地使用RxJava来实现与CompletableFuture相同的目的，但它的主要用例是它提供的大量其他功能。一个例子是以完全不同的方式再次执行相同的任务，我们可以创建一个Observable来等待数据到达，然后可以从其他地方将数据推送到它：

```java
@Test
void givenAnObservable_whenPushedData_thenItIsReceived() {
    PublishSubject<Integer> source = PublishSubject.create();
    Observable<Integer> observable = source.observeOn(Schedulers.computation());
    observable.subscribe(System.out::println, (throwable) -> System.out.println("Error"), () -> System.out.println("Done"));

    source.onNext(1);
    source.onNext(2);
    source.onNext(3);
    source.onComplete();
}
```

运行时，此测试会生成以下输出：

```text
1
2
3
Done
```

因此，我们可以订阅尚未生成任何内容的数据源，然后只需等待即可。数据准备就绪后，我们使用onNext()将其推送到源，并通过订阅收到警报。这是RxJava允许的响应式编程风格的一个示例，我们对外部来源推送给我们的事件和新数据做出反应，而不是我们自己请求。

## 5. 总结

在本文中，我们了解了早期Java的Future接口如何提供有用但有限的异步执行任务并稍后获取结果的能力。接下来，我们探讨了较新的实现CompletableFuture带来的好处，这使我们能够将方法调用串在一起，并对整个过程提供更好的控制。

最后，我们看到我们可以使用RxJava执行相同的工作，但也注意到它是一个广泛的库，允许我们做更多的事情。我们简要了解了如何使用RxJava将任务异步推送到观察者，同时无限期地订阅数据流。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-1)上找到。