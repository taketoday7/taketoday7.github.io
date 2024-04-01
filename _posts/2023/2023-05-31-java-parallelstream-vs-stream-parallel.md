---
layout: post
title:  Java中parallelStream()和stream().parallel()的区别
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 简介

在本教程中，我们将探索Java [Stream](https://www.baeldung.com/java-streams) API的Collections.parallelStream()和stream().parallel()方法。Java在Java 8中将parallelStream()方法引入了Collection接口，将parallel()方法引入了BaseStream接口。

## 2. Java中的并行流与并行

并行流允许我们通过跨多个CPU核心[并行](https://www.baeldung.com/cs/concurrency-vs-parallelism)执行流操作来使用多核处理。数据被拆分成多个子流，并行执行预期的操作，最后将结果聚合回来形成最终输出。 

除非另有说明，否则在Java中创建的流在默认情况下始终是串行的。我们可以通过两种方式将流转换为并行流： 

-   调用Collections.parallelStream()
-   调用BaseStream.parallel()

如果流操作未指定，则Java编译器和运行时会在执行并行流操作时决定处理顺序以获得最佳并行计算优势。

例如，我们得到了一个很长的Book对象列表。我们必须确定在指定年份出版的书籍数量：

```java
public class Book {
    private String name;
    private String author;
    private int yearPublished;

    // getters and setters
}
```

我们可以在这里利用并行流，比串行地找到计数更高效。我们示例中的执行顺序不会以任何方式影响最终结果，使其成为并行Stream操作的完美[候选者](https://www.baeldung.com/java-when-to-use-parallel-stream)。

## 3. Collections.parallelStream()的用法

在我们的应用程序中使用并行流的方法之一是在数据源上调用parallelStream()。**此操作返回一个可能并行的Stream，其中提供的集合作为其源**。我们可以将其应用于我们的示例并查找特定年份出版的书籍数量：

```java
long usingCollectionsParallel(Collection<Book> listOfbooks, int year) {
    AtomicLong countOfBooks = new AtomicLong();
    listOfbooks.parallelStream()
        .forEach(book -> {
            if (book.getYearPublished() == year) {
                countOfBooks.getAndIncrement();
            }
        });
    return countOfBooks.get();
}
```

parallelStream()方法的默认实现从Collection的Spliterator<T\>接口创建一个并行Stream。[Spliterator](https://www.baeldung.com/java-spliterator)是一个对象，用于遍历和分区其源的元素。Spliterator可以使用其trySplit()方法将其源的某些元素分区，使其符合可能的并行操作条件。 

Spliterator API类似于Iterator，允许遍历其源的元素，旨在支持高效的并行遍历。Collection的默认Spliterator将在parallelStream()调用中使用。 

## 4. 在Stream上使用parallel()

我们可以通过首先将集合转换为Stream来获得相同的结果。我们可以通过调用parallel()将作为结果生成的顺序流转换为并行流。一旦我们有了一个并行流，我们就可以用与上面相同的方式找到我们的结果：

```java
long usingStreamParallel(Collection<Book> listOfBooks, int year) {
    AtomicLong countOfBooks = new AtomicLong();
    listOfBooks.stream().parallel()
        .forEach(book -> {
            if (book.getYearPublished() == year) {
                countOfBooks.getAndIncrement();
            }
        });
    return countOfBooks.get();
}
```

Streams API的BaseStream接口将在源集合的默认Spliterator允许的范围内拆分底层数据，然后使用[Fork-Join](https://www.baeldung.com/java-fork-join)框架将其转换为并行Stream。 

两种方法的结果相同。 

## 5. parallelStream()和stream().parallel()的区别

Collections.parallelStream()使用源集合的默认Spliterator来拆分数据源以启用并行执行。均匀地拆分数据源对于启用正确的并行执行很重要。不均匀拆分的数据源在并行执行方面比其顺序对应的数据源造成的危害更大。

在所有这些示例中，我们一直在使用List<Book\>来保存我们的书籍列表。现在让我们尝试通过重写Collection<T\>接口为我们的Book创建一个自定义Collection。 

**我们应该记住，覆盖接口意味着我们必须提供对基接口中定义的抽象方法的实现**。但是，我们看到诸如spliterator()、stream()和parallelStream()之类的方法作为接口中的默认方法存在。这些方法有一个默认提供的实现。但是，我们仍然可以用我们的版本覆盖这些实现。

我们将自定义的Book集合类称为MyBookContainer，并将定义我们自己的Spliterator： 

```java
public class BookSpliterator<T> implements Spliterator<T> {
    private final Object[] books;
    private int startIndex;
    
    public BookSpliterator(Object[] books, int startIndex) {
        this.books = books;
        this.startIndex = startIndex;
    }

    @Override
    public Spliterator<T> trySplit() {
        // Always Assuming that the source is too small to split, returning null
        return null;
    }

    // Other overridden methods such as tryAdvance(), estimateSize() etc
}
```

在上面的代码片段中，我们看到我们的Spliterator版本接收一个Object数组(在我们的例子中是Book)进行拆分，并且在trySplit()方法中，它总是返回null。 

我们应该注意，Spliterator<T\>接口的这种实现容易出错，并且不会将数据划分为相等的部分；而是返回null，从而导致数据不平衡。这仅用于表示目的。

我们将在我们的自定义Collection类MyBookContainer中使用这个Spliterator： 

```java
public class MyBookContainer<T> implements Collection<T> {
    private static final long serialVersionUID = 1L;
    private T[] elements;

    public MyBookContainer(T[] elements) {
        this.elements = elements;
    }

    @Override
    public Spliterator<T> spliterator() {
        return new BookSpliterator(elements, 0);
    }

    @Override
    public Stream<T> parallelStream() {
        return StreamSupport.stream(spliterator(), false);
    }

    // standard overridden methods of Collection Interface
}
```

我们将尝试将数据存储在我们的自定义容器类中并对其执行并行流操作：

```java
long usingWithCustomSpliterator(MyBookContainer<Book> listOfBooks, int year) {
    AtomicLong countOfBooks = new AtomicLong();
    listOfBooks.parallelStream()
        .forEach(book -> {
            if (book.getYearPublished() == year) {
                countOfBooks.getAndIncrement();
            }
        });
    return countOfBooks.get();
}
```

此示例中的数据源是MyBookContainer类型的一个实例。此代码在内部使用我们自定义的Spliterator来拆分数据源。生成的并行流充其量是一个顺序流。 

我们只是利用parallelStream()方法返回一个顺序流，尽管名字暗示了parallelStream。这是该方法不同于stream().parallel()的地方，后者总是尝试返回提供给它的流的并行版本。Java在其文档中记录了这种行为，如下所示：

```java
/**
 * @implSpec
 * The default implementation creates a parallel {@code Stream} from the
 * collection's {@code Spliterator}.
 *
 * @return a possibly parallel {@code Stream} over the elements in this
 * collection
 * @since 1.8
 */
default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}
```

## 6. 总结

在本文中，我们介绍了从集合数据源创建并行流的方法。我们还尝试通过实现自定义版本的Collection和Spliterator来找出parallelStream()和stream().parallel()之间的区别。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-5)上获得。
