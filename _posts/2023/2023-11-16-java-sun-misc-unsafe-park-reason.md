---
layout: post
title:  为什么sun.misc.Unsafe.park实际上不安全？
category: java-concurrency
copyright: java-concurrency
excerpt: Java Unsafe
---

## 1. 概述

Java提供了某些供内部使用的API，不鼓励在其他情况下不必要地使用。[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)开发人员给了包和类名称，例如[Unsafe](https://www.baeldung.com/java-unsafe)，这应该警告开发人员。但是，通常这并不能阻止开发人员使用这些类。

在本教程中，我们将探讨为什么Unsafe.park()实际上是不安全的。**目的不是吓唬人，而是为了教育和提供对park()和unpark(Thread)方法的相互作用的更好的见解**。

## 2. Unsafe

Unsafe类包含一个低级API，旨在仅与内部库一起使用。**但是， 即使在引入[JPMS](https://www.baeldung.com/java-9-modularity)后，sun.misc.Unsafe仍然可以访问**。这样做是为了保持向后兼容性并支持可能使用此API的所有库和框架。更详细的原因在[JEP 260](https://openjdk.org/jeps/260)中有解释。

在本文中，我们不会直接使用Unsafe，而是使用java.util.concurrent.locks包中的LockSupport类，该类包装了对Unsafe的调用：

```java
public static void park() {
    UNSAFE.park(false, 0L);
}

public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

## 3. park()与wait()

park()和unpark(Thread)功能类似于[wait()和notify()](https://www.baeldung.com/java-wait-notify)。让我们回顾一下它们的差异，并了解使用第一个而不是第二个的危险。

### 3.1 缺少监视器

与wait()和notification()不同，park()和unpark(Thread)不需要[监视器](https://www.baeldung.com/cs/monitor)。任何可以获得对停放线程的引用的代码都可以取消停放它，**这在低级代码中可能很有用，但可能会带来额外的复杂性和难以调试的问题**。 

监视器是用Java设计的，因此如果线程一开始没有获取它，就无法使用它。这样做是为了防止竞争条件并简化同步过程。让我们尝试在不获取线程监视器的情况下通知线程：

```java
@Test
@Timeout(3)
void giveThreadWhenNotifyWithoutAcquiringMonitorThrowsException() {
    Thread thread = new Thread() {
        @Override
        public void run() {
            synchronized (this) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    // The thread was interrupted
                }
            }
        }
    };

    assertThrows(IllegalMonitorStateException.class, () -> {
        thread.start();
        Thread.sleep(TimeUnit.SECONDS.toMillis(1));
        thread.notify();
        thread.join();
    });
}
```

尝试在不获取监视器的情况下通知线程会导致[IllegalMonitorStateException](https://www.baeldung.com/java-illegalmonitorstateexception)，这种机制强制执行更好的编码标准并防止可能出现的难以调试的问题。

现在，让我们检查park()和unpark(Thread)的行为：

```java
@Test
@Timeout(3)
void giveThreadWhenUnparkWithoutAcquiringMonitor() {
    Thread thread = new Thread(LockSupport::park);
    assertTimeoutPreemptively(Duration.of(2, ChronoUnit.SECONDS), () -> {
        thread.start();
        LockSupport.unpark(thread);
    });
}
```

我们只需很少的工作就可以控制线程，唯一需要的是对线程的引用。**这为我们提供了更多的锁定能力，但同时，它也让我们面临更多的问题**。

很明显为什么park()和unpark(Thread)可能对低级代码有帮助，但我们应该在通常的应用程序代码中避免这种情况，因为它可能会引入太多的复杂性和不清晰的代码。

### 3.2 有关上下文的信息

事实上，不涉及监视器也可能会减少有关上下文的信息。换句话说，该线程被停放，并且不清楚为什么、何时以及是否有其他线程因同样的原因被停放。让我们[运行](https://www.baeldung.com/java-start-thread)两个线程：

```java
public class ThreadMonitorInfo {
    private static final Object MONITOR = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread waitingThread = new Thread(() -> {
            try {
                synchronized (MONITOR) {
                    MONITOR.wait();
                }
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }, "Waiting Thread");
        Thread parkedThread = new Thread(LockSupport::park, "Parked Thread");

        waitingThread.start();
        parkedThread.start();

        waitingThread.join();
        parkedThread.join();
    }
}
```

让我们使用[jstack](https://www.baeldung.com/java-thread-dump#1-jstack)检查[线程转储](https://www.baeldung.com/java-thread-dump)：

```text
"Parked Thread" #12 prio=5 os_prio=31 tid=0x000000013b9c5000 nid=0x5803 waiting on condition [0x000000016e2ee000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:304)
        at cn.tuyucheng.taketoday.park.ThreadMonitorInfo$$Lambda$2/284720968.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:750)

"Waiting Thread" #11 prio=5 os_prio=31 tid=0x000000013b9c4000 nid=0xa903 in Object.wait() [0x000000016e0e2000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000007401811d8> (a java.lang.Object)
        at java.lang.Object.wait(Object.java:502)
        at cn.tuyucheng.taketoday.park.ThreadMonitorInfo.lambda$main$0(ThreadMonitorInfo.java:12)
        - locked <0x00000007401811d8> (a java.lang.Object)
        at cn.tuyucheng.taketoday.park.ThreadMonitorInfo$$Lambda$1/1595428806.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:750)
```

在分析线程转储时，很明显，停放的线程包含的信息较少。因此，它可能会造成某种线程问题(即使使用线程转储)也难以调试的情况。

**使用特定并发结构或特定锁的另一个好处是在线程转储中提供更多上下文，从而提供有关应用程序状态的更多信息**。许多JVM并发机制在内部使用park()，但是，如果线程转储说明该线程正在等待(例如，在[CyclicBarrier](https://www.baeldung.com/java-cyclic-barrier)上)，则它正在等待其他线程。

### 3.3 中断标志

另一个有趣的事情是处理[中断](https://www.baeldung.com/java-interrupted-exception)的差异，让我们回顾一下等待线程的行为：

```java
@Test
@Timeout(3)
void givenWaitingThreadWhenNotInterruptedShouldNotHaveInterruptedFlag() throws InterruptedException {

    Thread thread = new Thread() {
        @Override
        public void run() {
            synchronized (this) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    // The thread was interrupted
                }
            }
        }
    };

    thread.start();
    Thread.sleep(TimeUnit.SECONDS.toMillis(1));
    thread.interrupt();
    thread.join();
    assertFalse(thread.isInterrupted(), "The thread shouldn't have the interrupted flag");
}
```

如果我们从等待状态中断一个线程，wait()方法将立即抛出[InterruptedException](https://www.baeldung.com/java-interrupted-exception#what-is-an-interruptedexception)并清除[中断标志](https://www.baeldung.com/java-interrupted-exception#4-the-interrupt-status-flag)，这就是为什么最佳实践是[使用while循环](https://www.baeldung.com/java-wait-notify#1-why-enclose-wait-in-a-while-loop)检查等待条件而不是中断标志。

相反，停放的线程不会立即中断，而是按照其条件执行。此外，中断不会导致异常，线程只是从park()方法返回。**随后，中断标志不会重置，就像中断等待线程时发生的情况一样**：

```java
@Test
@Timeout(3)
void givenParkedThreadWhenInterruptedShouldNotResetInterruptedFlag() throws InterruptedException {
    Thread thread = new Thread(LockSupport::park);
    thread.start();
    thread.interrupt();
    assertTrue(thread.isInterrupted(), "The thread should have the interrupted flag");
    thread.join();
}
```

不考虑这种行为可能会在处理中断时导致问题。例如，如果我们在暂停线程上中断后不重置标志，则可能会导致微妙的错误。

### 3.4 优先许可

park和unpark的工作原理是[二进制信号量](https://www.baeldung.com/cs/binary-counting-semaphores#binary-semaphores)，因此，我们可以为线程提供抢占式许可。**例如，我们可以取消停放一个线程，这将给它一个许可，随后的停放不会暂停它，而是会获取许可并继续**：

```java
private final Thread parkedThread = new Thread() {
    @Override
    public void run() {
        LockSupport.unpark(this);
        LockSupport.park();
    }
};

@Test
void givenThreadWhenPreemptivePermitShouldNotPark()  {
    assertTimeoutPreemptively(Duration.of(1, ChronoUnit.SECONDS), () -> {
        parkedThread.start();
        parkedThread.join();
    });
}
```

该技术可用于一些复杂的同步场景。由于park使用二进制信号量，因此我们无法添加许可，并且两个unpark调用不会产生两个许可：

```java
private final Thread parkedThread = new Thread() {
    @Override
    public void run() {
        LockSupport.unpark(this);
        LockSupport.unpark(this);
        LockSupport.park();
        LockSupport.park();
    }
};

@Test
void givenThreadWhenRepeatedPreemptivePermitShouldPark()  {
    Callable<Boolean> callable = () -> {
        parkedThread.start();
        parkedThread.join();
        return true;
    };

    boolean result = false;
    Future<Boolean> future = Executors.newSingleThreadExecutor().submit(callable);
    try {
        result = future.get(1, TimeUnit.SECONDS);
    } catch (InterruptedException | ExecutionException | TimeoutException e) {
        // Expected the thread to be parked
    }
    assertFalse(result, "The thread should be parked");
}
```

在这种情况下，线程只有一个许可，第二次调用park()方法将停放该线程。如果处理不当，这可能会产生一些不良行为。

## 4. 总结

在本文中，我们了解了为什么park()方法被认为是不安全的。JVM开发人员出于特定原因隐藏或建议不要使用内部API，这不仅是因为它目前可能很危险并会产生意外结果，而且还因为这些API将来可能会发生变化，并且无法保证它们的支持。

此外，这些API需要对底层系统和技术进行广泛的了解，而这些系统和技术可能因平台而异，不遵循这一点可能会导致代码脆弱和难以调试的问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-sun)上获得。