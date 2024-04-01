---
layout: post
title:  ArrayStoreException指南
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

**当试图在[对象数组](https://www.baeldung.com/java-arrays-guide)中存储不正确类型的对象时，Java会在运行时抛出ArrayStoreException**。由于ArrayStoreException是一个[非受检的异常](https://www.baeldung.com/java-checked-unchecked-exceptions)，因此通常不会处理或声明它。

在本教程中，我们将演示ArrayStoreException的原因、处理方法以及避免它的最佳实践。

## 2. ArrayStoreException产生的原因

当我们尝试在数组中存储不同类型的对象而不是声明的类型时，Java会抛出ArrayStoreException。

假设我们实例化了一个String类型的数组，然后尝试将Integer存储在其中。在这种情况下，在运行时会抛出ArrayStoreException：

```java
Object array[] = new String[5];
array[0] = 2;
```

当我们尝试在数组中存储不正确的值类型时，将在第二行代码处抛出异常：

```shell
Exception in thread "main" java.lang.ArrayStoreException: java.lang.Integer
    at cn.tuyucheng.taketoday.array.arraystoreexception.ArrayStoreExceptionExample.main(ArrayStoreExceptionExample.java:9)
```

由于我们将array声明为Object，因此**编译是没有错误的**。

## 3. 处理ArrayStoreException

这个异常的处理非常简单。与任何其他异常一样，它也需要**包含在[try-catch块](https://www.baeldung.com/java-exceptions)中进行处理**：

```java
try{
    Object array[] = new String[5];
    array[0] = 2;
}
catch (ArrayStoreException e) {
    // handle the exception
}
```

## 4. 避免此异常的最佳实践

**建议将数组类型声明为特定类，例如String或Integer，而不是Object**。当我们将数组类型声明为Object时，编译器将不会抛出任何错误。

但是**使用基类声明数组然后存储不同类的对象会导致编译错误**，让我们看一个简单的例子：

```java
String array[] = new String[5];
array[0] = 2;
```

在上面的示例中，我们将数组类型声明为String并尝试在其中存储一个Integer。这将导致编译错误：

```shell
Exception in thread "main" java.lang.Error: Unresolved compilation problem: 
    Type mismatch: cannot convert from int to String at cn.tuyucheng.taketoday.arraystoreexception.ArrayStoreExampleCE.main(ArrayStoreExampleCE.java:8)
```

**最好在编译时而不是运行时捕获错误**，因为我们对前者有更多的控制权。

## 5. 总结

在本教程中，我们了解了Java中ArrayStoreException的原因、处理和预防。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-guides)上获得。