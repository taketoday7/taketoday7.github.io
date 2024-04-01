---
layout: post
title:  使用IntelliJ调试Java流
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

自从Java 8推出以来，很多人开始使用(新的)流功能。当然，有时我们的流操作无法按预期工作。

[IntelliJ](https://www.baeldung.com/intellij-basics)除了其[正常的调试选项](https://www.baeldung.com/intellij-debugging-tricks)外，还有一个专用的流调试功能。在这个简短的教程中，我们将探讨这个很棒的功能。

## 2. 流跟踪对话框

让我们首先展示如何打开Stream Trace对话框。**在调试窗口的工具栏中，有一个Trace Current Stream Chain图标，只有当我们的应用程序在Stream API调用中的断点处暂停时才会启用**：

![](/assets/images/2023/javastream/intellijdebuggingjavastreams01.png)

单击该图标将打开Stream Trace对话框。

该对话框有两种模式。我们将在第一个示例中查看Flat Mode。并且，在第二个示例中，我们将展示默认模式，即Split Mode。

## 3. 示例

现在我们已经在IntelliJ中引入了流调试功能，是时候处理一些代码示例了。

### 3.1 排序流的基本示例

让我们从一个简单的代码片段开始，以习惯Stream Trace对话框：

```java
int[] listOutputSorted = IntStream.of(-3, 10, -4, 1, 3)
    .sorted()
    .toArray();
```

最初。我们有一个未排序的int流。接下来，我们对该流进行排序并将其转换为数组。

当我们在Flat Mode下查看Stream Trace时，它向我们显示了发生的步骤的概览：

![](/assets/images/2023/javastream/intellijdebuggingjavastreams02.png)

在最左边，我们看到初始流。它包含我们编写它们的顺序的整数。

第一组箭头向我们展示了排序后所有元素的新位置。在最右边，我们看到了我们的输出。所有元素都按排序顺序出现在那里。

现在我们已经了解了基础知识，是时候举一个更复杂的例子了。

### 3.2 使用flatMap和filter的示例

下一个示例使用[flatMap](https://www.baeldung.com/java-difference-map-and-flatmap)。例如，Stream.flatMap可以帮助我们将[Optional](https://www.baeldung.com/java-optional)列表转换为普通列表。在下一个示例中，我们从一个Optional<Customer\>列表开始。然后我们将其映射到Customer列表并应用一些过滤：

```java
List<Optional<Customer>> customers = Arrays.asList(
    Optional.of(new Customer("John P.", 15)),
    Optional.of(new Customer("Sarah M.", 78)),
    Optional.empty(),
    Optional.of(new Customer("Mary T.", 20)),
    Optional.empty(),
    Optional.of(new Customer("Florian G.", 89)),
    Optional.empty()
);

long numberOf65PlusCustomers = customers
    .stream()
    .flatMap(c -> c
        .map(Stream::of)
        .orElseGet(Stream::empty))
    .mapToInt(Customer::getAge)
    .filter(c -> c > 65)
    .count();
```

接下来，让我们在Split Mode下查看Stream Trace，这使我们可以更好地了解此流。

在左侧，我们看到输入流。接下来，我们看到Optional<Consumer/>流到实际现有Customer流的平面映射：

![](/assets/images/2023/javastream/intellijdebuggingjavastreams03.png)

之后，我们将Customer流映射到他们的年龄：

![](/assets/images/2023/javastream/intellijdebuggingjavastreams04.png)

下一步将我们的age流过滤为大于65岁的age流：

![](/assets/images/2023/javastream/intellijdebuggingjavastreams05.png)

最后，我们计算age流中的元素数：

![](/assets/images/2023/javastream/intellijdebuggingjavastreams06.png)

## 4. 注意事项

在上面的示例中，我们看到了Stream Trace对话框提供的一些可能性。但是，有一些重要的细节需要注意。它们中的大多数是流工作方式的直接结果。

首先，**流总是需要执行终端操作**。这在使用Stream Trace对话框时没有什么不同。此外，我们**必须注意不消耗整个流的操作**-例如，anyMatch。在这种情况下，它不会显示所有元素-仅显示已处理的元素。

其次，**请注意流将被消耗**。如果我们将Stream与其操作分开声明，我们可能会遇到错误[“Stream已经被操作或关闭”](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception)。我们可以通过将流的声明与其用法联接来防止此错误。

## 5. 总结

在这个快速教程中，我们了解了如何使用IntelliJ的Stream Trace对话框。

首先，我们查看了一个显示排序和收集的简单案例。然后，我们研究了一个更复杂的场景，涉及flatMap、map、filter和count。

最后，我们研究了在使用流调试功能时可能会遇到的一些注意事项。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-3)上获得。
