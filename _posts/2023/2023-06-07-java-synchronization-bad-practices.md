---
layout: post
title:  同步的不良做法
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

[Java中的同步](https://www.baeldung.com/java-synchronized#why-synchronization)对于摆脱多线程问题非常有帮助。然而，如果不经过深思熟虑地使用同步原则，它们可能会给我们带来很多麻烦。

在本教程中，我们将讨论与同步相关的一些不良实践，以及每个用例的更好方法。

## 2. 同步原理

一般来说，**我们应该只同步那些我们确定没有外部代码会锁定的对象**。

换句话说，**使用池化对象或可重用对象进行同步是一种不好的做法**。原因是JVM中的其他线程可以访问池化/可重用对象，并且外部/不受信任的代码对此类对象的任何修改都可能导致死锁和不确定的行为。

现在，让我们讨论基于某些类型(如String、Boolean、Integer和Object)的同步原则。

## 3. String

### 3.1 不良做法

[字符串文本是池化的](https://www.baeldung.com/java-string-pool)，通常在Java中重用。因此，不建议将String类型与[synchronized关键字](https://www.baeldung.com/java-synchronized#the-synchronized-keyword)一起用于同步：

```java
public void stringBadPractice1() {
    String stringLock = "LOCK_STRING";
    synchronized (stringLock) {
        // ... 
    }
}
```

同样，如果我们使用私有的最终字符串字面量，它仍然是从常量池中引用的：

```java
private final String stringLock = "LOCK_STRING";

public void stringBadPractice2() {
    synchronized (stringLock) {
        // ... 
    }
}
```

此外，为同步而调用String的intern被认为是不好的做法：

```java
private final String internedStringLock = new String("LOCK_STRING").intern();

public void stringBadPractice3() {
    synchronized (internedStringLock) {
        // ... 
    }
}
```

[根据Java文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#intern())，intern方法为我们提供了String对象的规范表示。换句话说，intern方法从池中返回一个字符串-并将其显式地添加到池中(如果它不存在)，它与这个字符串具有相同的内容。

因此，对于interned字符串对象，可重用对象上的同步问题仍然存在。

> 注意：**所有字符串字面量和字符串值的常量表达式都会自动interned**。

### 3.2 解决方案

避免在字符串字面量上进行同步的不良做法的建议是**使用new关键字创建字符串的新实例**。

让我们解决我们已经讨论过的代码中的问题。首先，我们将创建一个新的String对象以具有唯一引用(以避免任何重用)和它自己的内在锁，这有助于同步。

同时，我们将对象保持为私有和最终状态，以防止任何外部/不受信任的代码访问它：

```java
private final String stringLock = new String("LOCK_STRING");

public void stringSolution() {
    synchronized (stringLock) {
        // ...
    }
}
```

## 4. Boolean

具有两个值true和false的Boolean类型不适合用于锁定目的。与JVM中的字符串字面量类似，布尔字面量值也共享Boolean类的唯一实例。

让我们看一个在布尔锁对象上同步的错误代码示例：

```java
private final Boolean booleanLock = Boolean.FALSE;

public void booleanBadPractice() {
    synchronized (booleanLock) {
        // ...
    }
}
```

在这里，如果任何外部代码也在具有相同值的布尔字面量上同步，系统可能会变得无响应或导致死锁情况。

因此，我们不建议使用Boolean对象作为同步锁。

## 5. 包装类型

### 5.1 不良做法

与布尔字面量类似，包装类型可能会为某些值重用实例。原因是JVM缓存并共享可以表示为字节的值。

例如，让我们编写一个在包装类型Integer上同步的错误代码示例：

```java
private int count = 0;
private final Integer intLock = count;

public void boxedPrimitiveBadPractice() {
    synchronized (intLock) {
        count++;
        // ...
    }
}
```

### 5.2 解决方案

但是，与布尔字面量不同，包装类型的同步解决方案是创建一个新实例。

与String对象类似，我们应该使用new关键字创建Integer对象的唯一实例，该实例具有自己的内部锁，并保持其私有和最终状态：

```java
private int count = 0;
private final Integer intLock = new Integer(count);

public void boxedPrimitiveSolution() {
    synchronized (intLock) {
        count++;
        // ...
    }
}
```

## 6. 类同步

当类使用this关键字实现方法同步或块同步时，JVM将对象本身用作监视器(其内部锁)。

不受信任的代码可以获得并无限期地持有可访问类的内部锁。因此，这可能导致死锁情况。

### 6.1 不良做法

例如，让我们创建一个AnimalBadPractice类，其中包含一个同步的setName方法，以及一个包含同步块的setOwner方法：

```java
public class AnimalBadPractice {
    private String name;
    private String owner;

    // getters and constructors ...

    public synchronized void setName(String name) {
        this.name = name;
    }

    public void setOwner(String owner) {
        synchronized (this) {
            this.owner = owner;
        }
    }
}
```

现在，让我们编写一些糟糕的代码来创建AnimalBadPractice类的实例并在其上进行同步：

```java
public void classBadPractice() throws InterruptedException {
    AnimalBadPractice animalObj = new AnimalBadPractice("Tommy", "John");
    synchronized (animalObj) {
        while (true) {
            Thread.sleep(Integer.MAX_VALUE);
        }
    }
}
```

在这里，不受信任的代码示例引入了无限期延迟，从而阻止setName和setOwner方法实现获取相同的锁。

### 6.2 解决方案

**防止此漏洞的解决方案是私有锁对象**。

这个想法是**使用与我们类中定义的Object类的私有最终实例相关联的内部锁来代替对象本身的内部锁**。

此外，我们应该使用块同步来代替方法同步，以增加灵活性，将非同步代码排除在同步块之外。

因此，让我们对AnimalBadPractice类进行必要的更改：

```java
public class AnimalSolution {
    private final Object objLock1 = new Object();
    private final Object objLock2 = new Object();

    private String name;
    private String owner;

    // getters and constructors ...

    public void setName(String name) {
        synchronized (objLock1) {
            this.name = name;
        }
    }

    public void setOwner(String owner) {
        synchronized (objLock2) {
            this.owner = owner;
        }
    }
}
```

在这里，为了更好的并发性，我们通过定义多个私有最终锁对象来分离我们对setName和setOwner这两个方法的同步关注，从而细化了锁定方案。

此外，如果实现同步块的方法修改了静态变量，我们必须通过锁定静态对象来进行同步：

```java
private static int staticCount = 0;
private static final Object staticObjLock = new Object();

public void staticVariableSolution() {
    synchronized (staticObjLock) {
        staticCount++;
        // ... 
    }
}
```

## 7. 总结

在本文中，我们讨论了与某些类型(如String、Boolean、Integer和Object)上的同步相关的一些错误实践。

本文最重要的一点是，不建议使用池化或可重用对象进行同步。

此外，**建议在Object类的私有最终实例上进行同步**。这样的对象不会被外部/不受信任的代码访问，否则这些代码可能会与我们的公共类交互，从而减少此类交互可能导致死锁的可能性。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-advanced-4)上获得。