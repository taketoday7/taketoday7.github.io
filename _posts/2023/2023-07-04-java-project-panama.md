---
layout: post
title:  Java Project Panama指南
category: java-new
copyright: java-new
excerpt: Java 20
---

## 1. 概述

在本教程中，我们将介绍[Project Panama](https://openjdk.java.net/projects/panama/)组件。我们将首先探讨[外部函数和内存API](https://openjdk.org/jeps/424)。然后，我们将了解[JExtract](https://jdk.java.net/jextract/)工具如何促进其使用。

## 2. 什么是Project Panama？

[Project Panama](https://openjdk.java.net/projects/panama/)旨在简化Java与外部(非Java)API之间的交互，即用C、C++等编写的本机代码。

到目前为止，使用[Java本机接口(JNI)](https://www.baeldung.com/jni)是从Java调用外部函数的解决方案，但是[JNI有一些缺点](https://www.baeldung.com/jni#disadvantages)，Project Panama通过以下方式解决了这些缺点：

-   无需在Java中编写中间本机代码包装器
-   用更面向未来的内存API替换[ByteBuffer API](https://www.baeldung.com/java-bytebuffer)
-   引入一种与平台无关、安全且内存高效的方法来从Java调用本机代码

为实现其目标，Panama提供了一组API和工具：

-   [外部函数和内存API](https://openjdk.org/jeps/424)：用于分配和访问堆外内存以及直接从Java代码调用外部函数
-   [Vector API](https://openjdk.org/jeps/426)：使高级开发人员能够用Java表达复杂的数据并行算法
-   [JExtract](https://jdk.java.net/jextract/)：一种从一组本机标头中机械地派生Java绑定的工具

## 3. 先决条件

要使用外部函数和内存API，让我们下载[Project Panama Early-Access Build](https://jdk.java.net/panama/)。在撰写本文时，我们使用的是Build 19-panama+1-13(2022/1/18)。接下来，我们[根据使用的系统设置JAVA_HOME](https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux)。

由于外部函数和内存API是[预览API](https://openjdk.java.net/jeps/12)，我们必须在启用预览功能的情况下编译和运行我们的代码，即通过向java和javac添加–enable-preview标志。

## 4. 外部函数和内存API

外部函数和内存API可帮助Java程序与Java运行时之外的代码和数据进行互操作。

它通过有效地调用外部函数(即JVM外部的代码)和安全地访问外部内存(即不受JVM管理的内存)来实现这一点。

它结合了两个早期的孵化API：[外部内存访问API](https://openjdk.java.net/jeps/393)和[外部链接器API](https://openjdk.java.net/jeps/389)。

API提供了一组类和接口来执行这些操作：

-   使用MemorySegment、MemoryAddress和SegmentAllocator分配外部内存
-   通过MemorySession控制外来内存的分配和释放
-   使用MemoryLayout操作结构化外部内存
-   通过[VarHandles](https://www.baeldung.com/java-variable-handles)访问结构化外部内存
-   借助Linker、FunctionDescriptor和SymbolLookup调用外部函数

### 4.1 外部内存分配

首先，让我们探讨一下内存分配。在这里，主要的抽象是MemorySegment。它对位于堆外或堆上的连续内存区域进行建模，MemoryAddress是段内的偏移量。简单的说，一个内存段是由内存地址组成的，一个内存段可以包含其他的内存段。

此外，内存段绑定到它们封装的MemorySession并在不再需要时释放。MemorySession管理段的生命周期并确保它们在被多个线程访问时被正确释放。

让我们在内存段中的连续偏移处创建4个字节，然后将浮点值设置为6：

```java
try (MemorySession memorySession = MemorySession.openConfined()) {
    int byteSize = 4;
    int index = 0;
    float value = 6;
    MemorySegment segment = MemorySegment.allocateNative(byteSize, memorySession);
    segment.setAtIndex(JAVA_FLOAT, index, value);
    float result = segment.getAtIndex(JAVA_FLOAT, index);
    System.out.println("Float value is:" + result);
}
```

在上面的代码中，confined内存会话限制对创建会话的线程的访问，而shared内存会话允许从任何线程进行访问。

此外，JAVA_FLOAT ValueLayout指定取消引用操作的属性：类型映射的正确性和要取消引用的字节数。

SegmentAllocator抽象定义了分配和初始化内存段的有用操作。当我们的代码管理大量堆外段时，它会非常有用：

```java
String[] greetingStrings = {"hello", "world", "panama", "tuyucheng"};
SegmentAllocator allocator = SegmentAllocator.implicitAllocator();
MemorySegment offHeapSegment = allocator.allocateArray(ValueLayout.ADDRESS, greetingStrings.length);
for (int i = 0; i < greetingStrings.length; i++) {
    // Allocate a string off-heap, then store a pointer to it
    MemorySegment cString = allocator.allocateUtf8String(greetingStrings[i]);
    offHeapSegment.setAtIndex(ValueLayout.ADDRESS, i, cString);
}
```

### 4.2 外部内存操作

接下来，我们深入研究内存布局的内存操作。MemoryLayout描述段的内容。它对操作本机代码的高级数据结构很有用，例如结构体、指针和指向结构体的指针。

让我们使用GroupLayout在堆外分配一个表示具有x和y坐标的点的[C结构体](https://www.w3schools.com/c/c_structs.php)：

```java
GroupLayout pointLayout = structLayout(
    JAVA_DOUBLE.withName("x"),
    JAVA_DOUBLE.withName("y")
);

VarHandle xvarHandle = pointLayout.varHandle(PathElement.groupElement("x"));
VarHandle yvarHandle = pointLayout.varHandle(PathElement.groupElement("y"));

try (MemorySession memorySession = MemorySession.openConfined()) {
    MemorySegment pointSegment = memorySession.allocate(pointLayout);
    xvarHandle.set(pointSegment, 3d);
    yvarHandle.set(pointSegment, 4d);
    System.out.println(pointSegment.toString());
}
```

值得注意的是，不需要计算偏移量，因为不同的[VarHandle](https://www.baeldung.com/java-variable-handles)用于初始化每个点坐标。

我们还可以使用SequenceLayout构造数据数组。以下是获取5点列表的方法：

```java
SequenceLayout ptsLayout = sequenceLayout(5, pointLayout);
```

### 4.3 来自Java的本机函数调用

外部函数API允许Java开发人员在不依赖第三方包装器的情况下使用任何本机库。它严重依赖[方法句柄](https://www.baeldung.com/java-method-handles)并提供三个主要类：Linker、FunctionDescriptor和SymbolLookup。

让我们考虑通过调用C printf()函数来打印“Hello world”消息：

```c
#include <stdio.h>
int main() {
    printf("Hello World from Project Panama Tuyucheng Article");
    return 0;
}
```

首先，我们在标准库的类加载器中查找函数：

```java
Linker nativeLinker = Linker.nativeLinker();
SymbolLookup stdlibLookup = nativeLinker.defaultLookup();
SymbolLookup loaderLookup = SymbolLookup.loaderLookup();
```

Linker是两个二进制接口之间的桥梁：JVM和C/C++本机代码，也称为[C ABI](https://en.wikipedia.org/wiki/Application_binary_interface)。

下面，我们需要描述函数原型：

```java
FunctionDescriptor printfDescriptor = FunctionDescriptor.of(JAVA_INT, ADDRESS);
```

值布局JAVA_INT和ADDRESS分别对应C printf()函数的返回类型和输入：

```c
int printf(const char * __restrict, ...)
```

接下来，我们得到[方法句柄](https://www.baeldung.com/java-method-handles)：

```java
String symbolName = "printf";
String greeting = "Hello World from Project Panama Tuyucheng Article";
MethodHandle methodHandle = loaderLookup.lookup(symbolName)
    .or(() -> stdlibLookup.lookup(symbolName))
    .map(symbolSegment -> nativeLinker.downcallHandle(symbolSegment, printfDescriptor))
    .orElse(null);
```

Linker接口支持向下调用(从Java代码调用本机代码)和向上调用(从本机代码调用回Java代码)。最后，我们调用本机函数：

```java
try (MemorySession memorySession = MemorySession.openConfined()) {
    MemorySegment greetingSegment = memorySession.allocateUtf8String(greeting);
    methodHandle.invoke(greetingSegment);
}
```

## 5. JExtract

使用JExtract，无需直接使用大部分外部函数和内存API抽象。让我们重新打印上面的“Hello World”示例。

首先，我们需要从标准库头文件生成Java类：

```shell
jextract --source --output src/main -t foreign.c -I c:\mingw\include c:\mingw\include\stdio.h
```

stdio的路径必须更新为目标操作系统中的路径。接下来，我们可以简单地从Java中导入本机printf()函数：

```java
import static foreign.c.stdio_h.printf;

public class Greetings {

    public static void main(String[] args) {
        String greeting = "Hello World from Project Panama Tuyucheng Article, using JExtract!";
        try (MemorySession memorySession = MemorySession.openConfined()) {
            MemorySegment greetingSegment = memorySession.allocateUtf8String(greeting);
            printf(greetingSegment);
        }
    }
}
```

运行代码会将问候语打印到控制台：

```shell
java --enable-native-access=ALL-UNNAMED --enable-preview --source 19 .\src\main\java\cn\tuyucheng\taketoday\java\panama\jextract\Greetings.java
```

## 6. 总结

在本文中，我们了解了Project Panama的主要功能。

首先，我们探索了使用外部函数和内存API进行本机内存管理。然后我们使用MethodHandles调用外部函数。最后，我们使用JExtract工具来隐藏隐藏外部函数和内存API的复杂性。

Project Panama还有很多值得探索的地方，特别是从本机代码调用Java、调用第三方库和[Vector API](https://openjdk.org/jeps/426)。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-panama)上获得。