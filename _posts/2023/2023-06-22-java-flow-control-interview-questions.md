---
layout: post
title:  Java流程控制面试题(+答案)
category: interview
copyright: interview
excerpt: Java
---

## 1. 概述

控制流语句允许开发人员使用决策、循环和分支来有条件地改变特定代码块的执行流。

在这篇文章中，我们将讨论一些在面试中可能会出现的流程控制面试问题。

## 2. 问题

### Q1. 描述if-then和if-then-else语句，什么类型的表达式可以用作条件？

这两个语句都指示我们的程序仅当特定条件的计算结果为true时才执行其中的代码。但是，if-then-else语句提供了辅助执行路径，以防if子句的计算结果为false：

```java
if (age >= 21) {
    // ...
} else {
    // ...
}
```

与其他编程语言不同，Java只支持布尔表达式作为条件。如果我们尝试使用不同类型的表达式，则会出现编译错误。

### Q2. 描述switch语句，switch子句中可以使用哪些对象类型？

switch允许根据变量的值选择多个执行路径。

每个路径都标有case或default，switch语句评估每个case表达式是否匹配，并执行匹配标签后面的所有语句，直到找到break语句。如果找不到匹配项，将改为执行default块：

```java
switch (yearsOfJavaExperience) {
    case 0:
        System.out.println("Student");
        break;
    case 1:
        System.out.println("Junior");
        break;
    case 2:
        System.out.println("Middle");
        break;
    default:
        System.out.println("Senior");
}
```

我们可以使用byte、short、char、int，它们的包装版本，枚举和字符串作为switch值。

### Q3. 当我们忘记在switch的case子句中加入break语句时会发生什么？

switch穿透。这意味着它将继续执行所有case标签，直到找到break语句，即使这些标签与表达式的值不匹配。

下面是一个演示这一点的示例：

```java
int operation = 2;
int number = 10;

switch (operation) {
    case 1:
        number = number + 10;
        break;
    case 2:
        number = number - 4;
    case 3:
        number = number / 3;
    case 4:
        number = number * 10;
        break;
}
```

运行代码后，number的值为20，而不是6。这在我们想要将同一操作与多个case相关联的情况下很有用。

### Q4. 什么时候使用Switch优于If-Then-Else语句，反之亦然？

switch语句更适合针对许多单个值测试单个变量或多个值将执行相同代码的情况：

```java
switch (month) {
    case 1:
    case 3:
    case 5:
    case 7:
    case 8:
    case 10:
    case 12:
        days = 31;
        break;
case 2:
    days = 28;
    break;
default:
    days = 30;
}
```

当我们需要检查值的范围或多个条件时，if-then-else语句更可取：

```java
if (aPassword == null || aPassword.isEmpty()) {
    // empty password
} else if (aPassword.length() < 8 || aPassword.equals("12345678")) {
    // weak password
} else {
    // good password
}
```

### Q5. Java支持哪些类型的循环？

Java提供三种不同类型的循环：for、while和do-while。

for循环提供了一种迭代一系列值的方法。当我们事先知道任务将重复多少次时，它最有用：

```java
for (int i = 0; i < 10; i++) {
     // ...
}
```

while循环可以在特定条件为true时执行语句块：

```java
while (iterator.hasNext()) {
    // ...
}
```

do-while是while语句的变体，其中布尔表达式的计算位于循环的底部。这保证代码将至少执行一次：

```java
do {
    // ...
} while (choice != -1);
```

### Q6. 什么是增强for循环？

增强for循环是for语句的另一种语法，旨在遍历集合、数组、枚举或任何实现Iterable接口的对象的所有元素：

```java
for (String aString : arrayOfStrings) {
    // ...
}
```

### Q7. 如何提前退出循环？

使用break语句，我们可以立即终止循环的执行：

```java
for (int i = 0; ; i++) {
    if (i > 10) {
        break;
    }
}
```

### Q8. 无标签和有标签的break语句有什么区别？

无标签的break语句终止最里面的switch、for、while或do-while语句，而有标签的break结束外部语句的执行。

让我们创建一个示例来演示这一点：

```java
int[][] table = { { 1, 2, 3 }, { 25, 37, 49 }, { 55, 68, 93 } };
boolean found = false;
int loopCycles = 0;

outer: for (int[] rows : table) {
    for (int row : rows) {
        loopCycles++;
        if (row == 37) {
            found = true;
            break outer;
        }
    }
}
```

当找到数字37时，有标签的break语句终止最外层的for循环，不再执行循环。因此，loopCycles以值5结束。

但是，无标签的break仅结束最内层的语句，将控制流返回到最外层的for继续循环到table变量中的下一行，使loopCycles以值8结束。

### Q9. 无标签和有标签的continue语句有什么区别？

无标签的continue语句跳到最内层的for、while或do-while循环中当前迭代的末尾，而有标签的continue跳到标有给定标签的外部循环。

下面是一个演示这一点的示例：

```java
int[][] table = { { 1, 15, 3 }, { 25, 15, 49 }, { 15, 68, 93 } };
int loopCycles = 0;

outer: for (int[] rows : table) {
    for (int row : rows) {
        loopCycles++;
        if (row == 15) {
            continue outer;
        }
    }
}
```

推理与上一个问题相同。有标签的continue语句终止最外层的for循环。

因此，loopCycles以值5结束，而无标签的版本仅终止最内层的语句，使loopCycles以值9结束。

### Q10. 描述try-catch-finally结构中的执行流程

当程序进入try块，并在其中抛出异常时，try块的执行被中断，控制流继续到catch块，该catch块可以处理抛出的异常。

如果不存在这样的块，则当前方法执行停止，并将异常抛给调用堆栈上的前一个方法。或者，如果没有发生异常，则忽略所有catch块，程序继续正常执行。

无论try块的主体内是否抛出异常，finally块总是被执行。

### Q11. finally块在哪些情况下可能不会被执行？

当JVM在执行try或catch块时终止，例如，通过调用System.exit()，或者当执行线程被中断或杀死时，finally块不会被执行。

### Q12. 执行以下代码的结果是什么？

```java
public static int assignment() {
    int number = 1;
    try {
        number = 3;
        if (true) {
            throw new Exception("Test Exception");
        }
        number = 2;
    } catch (Exception ex) {
        return number;
    } finally {
        number = 4;
    }
    return number;
}

System.out.println(assignment());
```

代码输出数字3。即使finally块始终会执行，这也只会在try块退出后发生。

在示例中，return语句在try-catch块结束之前执行。因此，finally块中对number的赋值不起作用，因为变量已经返回给assignment方法的调用代码。

### Q13. 在哪些情况下可以使用try-finally块，即使可能不会抛出异常？

当我们想要确保不会因遇到break、continue或return语句而意外绕过代码中使用的资源清理时，此块很有用：

```java
HeavyProcess heavyProcess = new HeavyProcess();
try {
    // ...
    return heavyProcess.heavyTask();
} finally {
    heavyProcess.doCleanUp();
}
```

此外，我们可能会面临无法在局部处理抛出的异常的情况，或者我们希望当前方法仍然抛出异常，同时允许我们释放资源：

```java
public void doDangerousTask(Task task) throws ComplicatedException {
    try {
        // ...
        task.gatherResources();
        if (task.isComplicated()) {
            throw new ComplicatedException("Too difficult");
        }
        // ...
    } finally {
        task.freeResources();
    }
}
```

### Q14. try-with-resources的原理？

try-with-resources语句在执行try块之前声明并初始化一个或多个资源，并在语句结束时自动关闭它们，无论该块是正常完成还是意外完成。任何实现AutoCloseable或Closeable接口的对象都可以用作资源：

```java
try (StringWriter writer = new StringWriter()) {
    writer.write("Hello world!");
}
```

## 3. 总结

在本文中，我们介绍了Java开发人员技术面试中出现的一些与控制流语句有关的最常见问题。这应该只被视为进一步研究的开始，而不是一个详尽的清单。