---
layout: post
title:  在Java中将浮点数转换为字节数组
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在这个快速教程中，我们将探讨一些使用Java将浮点数转换为字节数组的示例，反之亦然。

如果我们将int或long转换为字节数组，这很简单，因为Java按位运算符仅适用于整数类型。但是，对于一个浮点数，我们需要使用另一层转换。

例如，我们可以使用Float类或java.nio包的ByteBuffer类提供的API。

## 2. 浮点数到字节数组的转换

正如我们所知，Java中浮点数的大小是32位，类似于int。因此，我们可以使用Java的Float类中提供的floatToIntBits或floatToRawIntBits函数。然后移动位以返回字节数组。单击[此处](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op3.html)了解有关移位操作的更多信息。

两者之间的区别在于floatToRawIntBits也保留非数字(NaN)值。在这里，移动位是通过一种称为[缩小原始转换](https://docs.oracle.com/javase/specs/jls/se10/html/jls-5.html#jls-5.1.3)的技术完成的。

首先让我们看一下使用Float类函数的代码：

```java
public static byte[] floatToByteArray(float value) {
    int intBits =  Float.floatToIntBits(value);
    return new byte[] { (byte) (intBits >> 24), (byte) (intBits >> 16), (byte) (intBits >> 8), (byte) (intBits) };
}
```

其次，使用ByteBuffer的一种简洁的转换方式：

```java
ByteBuffer.allocate(4).putFloat(value).array();
```

## 3. 字节数组到浮点数的转换

现在让我们使用Float类函数intBitsToFloat将字节数组转换为浮点数。

但是，我们需要先使用左移将字节数组转换为int位：

```java
public static float byteArrayToFloat(byte[] bytes) {
    int intBits = bytes[0] << 24 | (bytes[1] & 0xFF) << 16 | (bytes[2] & 0xFF) << 8 | (bytes[3] & 0xFF);
    return Float.intBitsToFloat(intBits);  
}
```

使用ByteBuffer将字节数组转换为浮点数非常简单：

```java
ByteBuffer.wrap(bytes).getFloat();
```

## 4. 单元测试

让我们看一下用于实现的简单单元测试用例：

```java
public void givenAFloat_thenConvertToByteArray() {
    assertArrayEquals(new byte[] { 63, -116, -52, -51}, floatToByteArray(1.1f));
}

@Test
public void givenAByteArray_thenConvertToFloat() {
   assertEquals(1.1f, byteArrayToFloat(new byte[] { 63, -116, -52, -51}), 0);
}
```

## 5. 总结

我们已经看到了浮点到字节转换的不同方式，反之亦然。

Float类提供函数作为此类转换的解决方法。但是，ByteBuffer提供了一种简洁的方法来做到这一点。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-convert)上获得。