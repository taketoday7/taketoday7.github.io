---
layout: post
title:  使用流在列表中查找最大日期
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在本文中，我们将首先创建一个带有日期的对象。然后，我们将了解如何使用[Stream](https://www.baeldung.com/java-8-streams)在这些对象的列表中找到最大日期。

## 2. 示例设置

**Java的原始[Date](https://www.baeldung.com/java-get-the-current-date-legacy) API仍在广泛使用**，因此我们将展示一个使用它的示例。但是，自Java 8以来，引入了[LocalDate](https://www.baeldung.com/java-8-date-time-intro)，并且弃用了大多数Date方法。因此，**我们还将展示一个使用LocalDate的示例**。

首先，让我们创建一个包含单独Date属性的基本Event对象：

```java
public class Event {
    Date date;

    // constructor, getter and setter
}
```

现在我们可以定义一个包含三个Event的列表：第一个发生在今天，第二个发生在明天，第三个发生在一周内。**要向Date添加天数，我们将使用[Apache Commons](https://www.baeldung.com/java-commons-lang-3)的DateUtils方法addDays()**：

```java
Date TODAY = new Date();
Event TODAYS_EVENT = new Event(TODAY);
Date TOMORROW = DateUtils.addDays(TODAY, 1);
Event TOMORROWS_EVENT = new Event(TOMORROW);
Date NEXT_WEEK = DateUtils.addDays(TODAY, 7);
Event NEXT_WEEK_EVENT = new Event(NEXT_WEEK);
List<Event> events = List.of(TODAYS_EVENT, TOMORROWS_EVENT, NEXT_WEEK_EVENT);
```

我们现在的目标是编写一个方法来确定NEXT_WEEK_EVENT是此Event列表中的最大日期。我们还将对LocalDate而不是Date执行相同的操作。我们的LocalEvent将如下所示：

```java
public class LocalEvent {
    LocalDate date;

    // constructor, getter and setter
}
```

这次构建Event列表更简单一些，因为LocalDate已经有一个内置的plusDays()方法：

```java
LocalDate TODAY_LOCAL = LocalDate.now();
LocalEvent TODAY_LOCAL_EVENT = new LocalEvent(TODAY_LOCAL);
LocalDate TOMORROW_LOCAL = TODAY_LOCAL.plusDays(1);
LocalEvent TOMORROW_LOCAL_EVENT = new LocalEvent(TOMORROW_LOCAL);
LocalDate NEXT_WEEK_LOCAL = TODAY_LOCAL.plusWeeks(1);
LocalEvent NEXT_WEEK_LOCAL_EVENT = new LocalEvent(NEXT_WEEK_LOCAL);
List<LocalEvent> localEvents = List.of(TODAY_LOCAL_EVENT, TOMORROW_LOCAL_EVENT, NEXT_WEEK_LOCAL_EVENT);
```

## 3. 获取最大日期

首先，**我们将使用[Stream API](https://www.baeldung.com/java-8-streams)来流式传输我们的Event列表**。然后，我们需要将Date getter应用于Stream的每个元素。因此，我们将获得一个包含事件日期的Stream。**现在我们可以为它使用max()函数。这将返回Stream中关于提供的[Comparator](https://www.baeldung.com/java-comparator-comparable)的最大日期**。

Date类实现Comparable<Date\>。因此，compareTo()方法定义了自然日期顺序。简而言之，可以在max()中等效地调用以下两个方法：

-   Date的compareTo()可以通过方法引用来引用
-   Comparator的naturalOrder()可以直接使用

最后要注意的是，如果给定的Event列表为null或者为空，我们可以直接返回null。这将确保我们在流式传输列表时不会遇到问题。

该方法最终如下所示：

```java
Date findMaxDateOf(List<Event> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
        .map(Event::getDate)
        .max(Date::compareTo)
        .get();
}
```

或者，使用naturalOrder()，它为：

```java
Date findMaxDateOf(List<Event> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
        .map(Event::getDate)
        .max(Comparator.naturalOrder())
        .get();
}
```

总而言之，我们现在可以快速测试我们的方法是否为我们的列表返回了正确的结果：

```java
assertEquals(NEXT_WEEK, findMaxDateOf(List.of(TODAYS_EVENT, TOMORROWS_EVENT, NEXT_WEEK_EVENT);
```

对于LocalDate，推理是完全一样的。LocalDate确实实现了ChronoLocalDate[接口](https://www.baeldung.com/java-interfaces)，该接口扩展了Comparable<ChronoLocalDate\>。因此，LocalDate的自然顺序由ChronoLocalDate的compareTo()方法定义。

因此，该方法可以写成：

```java
LocalDate findMaxDateOf(List<LocalEvent> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
        .map(LocalEvent::getDate)
        .max(LocalDate::compareTo)
        .get();
}
```

或者，以完全等效的方式：

```java
LocalDate findMaxDateOf(List<LocalEvent> events) {
    if (events == null || events.isEmpty()) {
        return null;
    }
    return events.stream()
        .map(LocalEvent::getDate)
        .max(Comparator.naturalOrder())
        .get();
}
```

我们可以编写以下测试来确认它是否有效：

```java
assertEquals(NEXT_WEEK_LOCAL, findMaxDateOf(List.of(TODAY_LOCAL_EVENT, TOMORROW_LOCAL_EVENT, NEXT_WEEK_LOCAL_EVENT)));
```

## 4. 总结

在本教程中，我们了解了如何获取对象列表中的最大日期。我们同时使用了Date和LocalDate对象。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。
