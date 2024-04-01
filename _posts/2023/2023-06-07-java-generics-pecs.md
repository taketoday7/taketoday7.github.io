---
layout: post
title:  Java泛型PECS - Producer Extends Consumer Super
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将探讨[Java泛型](https://www.baeldung.com/java-generics)在生成和使用集合时的用法。

我们还将讨论extends和super关键字，我们将查看PECS(Producer Extends Consumer Super)规则的几个示例以确定如何正确使用这些关键字。

## 2. 生产者扩展

对于本文中的代码示例，我们将使用一个简单的数据模型，其中包含一个User基类和两个扩展它的类：Operator和Customer。

**重要的是要了解必须从集合的角度应用PECS规则**。换句话说，如果我们遍历一个List并处理它的元素，这个列表将充当我们逻辑的生产者：

```java
public void sendEmails(List<User> users) {
    for (User user : users) {
        System.out.println("sending email to " + user);
    }
}
```

现在，假设我们要对Operator列表使用sendEmail方法。Operator类扩展了User，因此我们可能希望它是一个简单、直接的方法调用。但是，不幸的是，我们会得到一个编译错误：

![](/assets/images/2023/javacollection/javagenericspecs01.png)

为了解决这个问题，我们可以按照PECS规则更新sendEmail方法。因为用户列表是我们逻辑的生产者，所以我们将使用extends关键字：

```java
public void sendEmailsFixed(List<? extends User> users) {
    for (User user : users) {
        System.out.println("sending email to " + user);
    }
}
```

因此，我们现在可以轻松地为任何泛型类型的列表调用该方法，只要它们继承自User类：

```java
List<Operator> operators = Arrays.asList(new Operator("sam"), new Operator("daniel"));
List<Customer> customers = Arrays.asList(new Customer("john"), new Customer("arys"));

sendEmailsFixed(operators);
sendEmailsFixed(customers);
```

## 3. Consumer Super

当我们向集合添加元素时，我们成为生产者，而列表将充当消费者。让我们编写一个方法来接收Operator列表并向其添加另外两个元素：

```java
private void addUsersFromMarketingDepartment(List<Operator> users) {
    users.add(new Operator("john doe"));
    users.add(new Operator("jane doe"));
}
```

如果我们传递一个Operator列表，这将完美地工作。但是，如果我们想使用它来将两个Operator添加到Users列表中，我们将再次遇到编译错误：

![](/assets/images/2023/javacollection/javagenericspecs02.png)

因此，我们需要更新该方法并使用super关键字使其接收Operator或其前身的集合：

```java
private void addUsersFromMarketingDepartmentFixed(List<? super Operator> users) {
    users.add(new Operator("john doe"));
    users.add(new Operator("jane doe"));
}
```

## 4. 生产与消费

可能存在我们的逻辑需要读取和写入集合的情况。在这种情况下，Collection将同时是生产者和消费者。

处理这些情况的唯一方法是使用基类，不使用任何关键字：

```java
private void addUsersAndSendEmails(List<User> users) {
    users.add(new Operator("john doe"));
    for (User user : users) {
        System.out.println("sending email to: " + user);
    }
}
```

**另一方面，使用同一个集合进行读取和写入将违反命令-查询分离原则，应避免使用**。

## 5. 总结

在本文中，我们讨论了Produce Extends Consumer Super规则，并学习了如何在处理Java集合时应用它。

我们探讨了集合作为我们逻辑的生产者或消费者的各种用法。在那之后，我们了解到，如果一个集合同时执行这两种操作，这可能表明我们的设计中存在代码味道。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-4)上获得。