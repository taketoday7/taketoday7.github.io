---
layout: post
title:  Java PriorityQueue指南
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个简短的教程中，我们将讨论[优先级队列](https://www.baeldung.com/cs/priority-queue)的Java实现。首先，我们将了解标准用法，并通过按自然顺序和逆序对队列进行排序来展示一些示例。

最后，我们将了解如何使用[Java Comparator](https://www.baeldung.com/java-comparator-comparable#comparator)定义自定义顺序。

## 2. java.util.PriorityQueue

java.util.PriorityQueue类是从JDK 1.5开始提供的，JDK 1.5还包含[AbstractQueue](https://www.baeldung.com/java-queue#abstract_queue)的其他实现。**正如我们可以从其名称推断的那样，我们使用PriorityQueue来维护给定集合中定义的顺序：队列的第一个元素(head)是相对于我们指定的顺序的最小元素。队列的每个检索操作(poll、remove或peek)都会读取队列的头部**。

在内部，PriorityQueue依赖于一个对象数组。如果初始指定的容量(在JDK 17中默认为11)不足以存储所有元素，则会自动调整此数组的大小。虽然为PriorityQueue提供初始容量不是强制性的，但如果我们已经知道集合的大小，则可以避免自动调整大小，这会消耗我们最好节省的CPU周期。

在[Javadoc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/PriorityQueue.html)中，指定此实现对入队和出队方法(offer、poll、remove和add)花费O(log(n))时间。这要归功于为Queue的每次修改不断维护的平衡二进制堆数据结构。相反，它被授予remove(Object)和contains(Object)方法的线性时间以及检索方法(peek、element和size)的常数时间。

## 3. 自然排序和逆序

在上一篇文章中，我们介绍了如何[插入到PriorityQueue中的元素如何根据其自然顺序进行排序](https://www.baeldung.com/java-queue#priority_queues)。这是因为使用空比较器初始化PriorityQueue将使用[compare](https://www.baeldung.com/java-comparator-comparable#comparable)操作直接对元素进行排序。

**例如，现在让我们看看通过提供标准整数自然排序比较器或null，队列将以相同的方式排序**：

```java
PriorityQueue<Integer> integerQueue = new PriorityQueue<>();
PriorityQueue<Integer> integerQueueWithComparator = new PriorityQueue<>((Integer c1, Integer c2) -> Integer.compare(c1, c2));

integerQueueWithComparator.add(3);
integerQueue.add(3);

integerQueueWithComparator.add(2);
integerQueue.add(2);

integerQueueWithComparator.add(1);
integerQueue.add(1);

assertThat(integerQueue.poll())
    .isEqualTo(1)
    .isEqualTo(integerQueueWithComparator.poll());

assertThat(integerQueue.poll())
    .isEqualTo(2)
    .isEqualTo(integerQueueWithComparator.poll());

assertThat(integerQueue.poll())
    .isEqualTo(3)
    .isEqualTo(integerQueueWithComparator.poll());
```

现在让我们创建一个以逆自然顺序排序的PriorityQueue。我们可以通过使用静态方法java.util.Collections.reverseOrder()来实现：

```java
PriorityQueue<Integer> reversedQueue = new PriorityQueue<>(Collections.reverseOrder());

reversedQueue.add(1);
reversedQueue.add(2);
reversedQueue.add(3);

assertThat(reversedQueue.poll()).isEqualTo(3);
assertThat(reversedQueue.poll()).isEqualTo(2);
assertThat(reversedQueue.poll()).isEqualTo(1);
```

## 4. 自定义排序

现在让我们尝试为自定义类定义一个特殊的顺序。首先，该类应该实现Comparable接口，或者我们应该在Queue的实例化中提供一个Comparator，否则将抛出[ClassCastException](https://www.baeldung.com/java-classcastexception)。

例如，让我们创建一个ColoredNumber类来演示此行为：

```java
public class ColoredNumber {

    private int value;
    private String color;

    public ColoredNumber(int value, String color) {
        this.value = value;
        this.color = color;
    }
    // getters and setters...
}
```

**当我们尝试在PriorityQueue中使用这个类时，它会抛出一个异常**：

```java
PriorityQueue<ColoredNumber> queue = new PriorityQueue<>();
queue.add(new ColoredNumber(3,"red"));
queue.add(new ColoredNumber(2, "blue"));
```

**这是因为PriorityQueue不知道如何通过将ColoredNumber对象与同一类的其他对象进行比较来对其进行排序**。

我们可以通过在构造函数中提供Comparator来提供排序，就像我们在前面的示例中所做的那样，或者我们可以实现Comparable接口：

```java
public final class ColoredNumberComparable implements Comparable<ColoredNumber> {
    // ...
    @Override
    public int compareTo(ColoredNumberComparable o) {
        if ((this.color.equals("red") && o.color.equals("red")) || (!this.color.equals("red") && !o.color.equals("red"))) {
            return Integer.compare(this.value, o.value);
        } else if (this.color.equals("red")) {
            return -1;
        } else {
            return 1;
        }
    }
}
```

这将授予每个元素将首先考虑“red”颜色，然后是自然排序中的值，这意味着所有红色对象将首先返回：

```java
PriorityQueue<ColoredNumberComparable> queue = new PriorityQueue<>();
queue.add(new ColoredNumberComparable(10, "red"));
queue.add(new ColoredNumberComparable(20, "red"));
queue.add(new ColoredNumberComparable(1, "blue"));
queue.add(new ColoredNumberComparable(2, "blue"));

ColoredNumberComparable first = queue.poll();
assertThat(first.getColor()).isEqualTo("red");
assertThat(first.getValue()).isEqualTo(10);

queue.poll();

ColoredNumberComparable third = queue.poll();
assertThat(third.getColor()).isEqualTo("blue");
assertThat(third.getValue()).isEqualTo(1);
```

关于多线程的最后一点说明：优先队列的这种Java实现不是同步的，这意味着多个线程不应同时使用Java PriorityQueue的同一个实例。

如果多个线程需要访问一个PriorityQueue实例，我们应该使用线程安全的[java.util.concurrent.PriorityBlockingQueue](https://www.baeldung.com/java-priority-blocking-queue)类。

## 5. 总结

在本文中，我们了解了Java PriorityQueue实现的工作原理。我们从该类的JDK内部结构及其性能写入和读取元素开始。然后，我们演示了具有自然排序和逆向排序的PriorityQueue。最后，我们提供了一个用户定义类的自定义Comparable实现并验证了它的排序行为。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-4)上获得。