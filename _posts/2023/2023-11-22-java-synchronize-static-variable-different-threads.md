---
layout: post
title: 在不同线程之间同步静态变量
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在Java中，需要同步访问[静态变量](https://www.baeldung.com/java-static)并不罕见。在这个简短的教程中，我们将介绍几种在不同线程之间同步对静态变量的访问的方法。

## 2. 关于静态变量

快速回顾一下，**静态变量属于类而不是类的实例，这意味着类的所有实例都具有相同的变量状态**。

例如，让我们考虑一个Employee类，其中包含static变量：

```java
public class Employee {
    static int count;
    int id;
    String name;
    String title;
}
```

在本例中，count变量是静态的，表示曾在公司工作过的员工总数。无论我们创建多少个Employee实例，它们都将共享相同的count值。

然后，我们可以向构造函数添加代码，以确保跟踪每个新员工的计数：

```java
public Employee(int id, String name, String title) {
    count = count + 1;
    // ...
}
```

虽然这种方法很简单，**当我们想要读取count变量时，它可能会出现问题**。在具有Employee类的多个实例的多线程环境中尤其如此。

下面，我们将看到同步访问count变量的不同方法。

## 3. 使用synchronized关键字同步静态变量

同步静态变量的第一种方法是使用Java的[synchronized关键字](https://www.baeldung.com/java-synchronized)，我们可以通过多种方式利用此关键字来访问静态变量。

首先，我们可以创建一个静态方法，在其声明中使用synchronized关键字作为修饰符：

```java
public Employee(int id, String name, String title) {
    incrementCount();
    // ...
}

private static synchronized void incrementCount() {
    count = count + 1;
}

public static synchronized int getCount() {
    return count;
}
```

**在本例中，synchronized关键字锁定类对象，因为变量是静态的**。这意味着无论我们创建多少个Employee实例，只要使用这两个静态方法，一次只有一个方法可以访问该变量。

其次，我们可以使用synchronized块在类对象上显式同步：

```java
private static void incrementCount() {
    synchronized(Employee.class) {
        count = count + 1;
    }
}

public static int getCount() {
    synchronized(Employee.class) {
        return count;
    }
}
```

请注意，**这在功能上与第一个示例等效，但代码更加明确**。

最后，我们还可以使用具有特定对象实例而不是类的同步块：

```java
private static final Object lock = new Object();

public Employee(int id, String name, String title) {
    incrementCount();
    // ...
}

private static void incrementCount() {
    synchronized(lock) {
        count = count + 1;
    }
}

public static int getCount() {
    synchronized(lock) {
        return count;
    }
}
```

有时首选这种方法的原因是因为锁是我们类私有的。在第一个示例中，**我们控制之外的其他代码也可能锁定我们的类**。有了私有锁，我们就可以完全控制它的使用方式。

Java synchronized关键字只是同步静态变量访问的一种方法，下面，我们将介绍一些也可以提供静态变量同步的Java API。

## 4. 同步静态变量的Java API

Java编程语言提供了多个有助于同步的API，让我们看一下其中的两个。

### 4.1 原子包装器

在Java 1.5中引入的[AtomicInteger](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html)类是同步静态变量访问的另一种方法。**此类提供原子读写操作，确保所有线程的底层值的视图一致**。

例如，我们可以使用AtomicInteger类型而不是int重写Employee类：

```java
public class Employee {
    private final static AtomicInteger count = new AtomicInteger(0);

    public Employee(int id, String name, String title) {
        count.incrementAndGet();
    }

    public static int getCount() {
        count.get();
    }
}
```

除了AtomicInteger之外，**Java还提供了long和boolean，以及引用类型**。所有这些包装类都是同步静态数据访问的出色工具。

### 4.2 可重入锁

同样在Java 1.5中引入的[ReentrantLock](https://www.baeldung.com/java-binary-semaphore-vs-reentrant-lock#reentrantLock)类是我们可以用来同步静态数据访问的另一种机制，**它提供与我们之前使用的synchronized关键字相同的基本行为和语义，但具有其他功能**。

让我们看一个示例，说明我们的Employee类如何使用ReentrantLock：

```java
public class Employee {
    private static int count = 0;
    private static final ReentrantLock lock = new ReentrantLock();

    public Employee(int id, String name, String title) {
        lock.lock();
        try {
            count = count + 1;
        } finally {
            lock.unlock();
        }

        // set fields
    }

    public static int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

关于此方法有几点需要注意。首先，它比其他的方法要冗长得多，**每次访问共享变量时，我们都必须确保在访问之前锁定并在访问之后解锁**。如果我们忘记在访问共享静态变量的每个地方执行此操作，这可能会导致程序错误。

此外，该类的文档建议使用try/finally块以正确锁定和解锁，这会增加额外的代码行和冗长。

也就是说，ReentrantLock类提供了synchronized关键字之外的其他行为。除此之外，**它还允许我们设置公平性标志并查询锁的状态，以获取有多少线程正在等待它的详细视图**。

## 5. 总结

在本文中，我们研究了在不同实例和线程之间同步访问静态变量的几种不同方法。我们首先查看了Java synchronized关键字，并查看了如何将其用作方法修饰符和静态代码块的示例。

然后，我们研究了Java并发API的两个功能：AtomicInteger和ReentrantLock。这两个API都提供了同步访问共享数据的方法，除了synchronized关键字的功能之外，还具有一些额外的好处。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上找到。