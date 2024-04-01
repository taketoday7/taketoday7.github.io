---
layout: post
title:  通过二维Java数组对角线循环
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将了解如何沿对角线循环遍历二维数组。我们提供的解决方案可以用于任意大小的方形二维数组。

## 2. 二维数组

处理数组元素的关键是知道如何从该数组中获取特定元素。对于二维数组，我们使用行索引和列索引来获取数组的元素。对于这个问题，我们将使用下图来演示如何获取这些元素。

![](/assets/images/2023/javaarray/javaloopdiagonalarray01.png)

接下来，我们需要了解数组中有多少条对角线，如图所示。为此，我们首先获取数组的一维长度，然后使用它获取对角线(diagonalLines)的数量。

然后我们使用对角线的数量来获取中点，这将有助于搜索行和列索引。

在此示例中，中点为3：

```java
int length = twoDArray.length
int diagonalLines = (length + length) - 1
int midPoint = (diagonalLines / 2) + 1
```

## 3. 获取行列索引

要遍历整个数组，我们从1开始循环，直到循环变量小于或等于diagonalLines变量。

```java
for (int i = 1; i <= diagonalLines; i++) {
    // some operations
}
```

让我们也介绍一下对角线中元素数量的概念，称之为itemsInDiagonal。例如，上图中第3行有3个元素(g、e、c)，第4行有2个(h、f)。当循环变量i小于或等于midPoint时，该变量在循环中递增1。否则递减1。

在递增或递减itemsInDiagonal之后，我们便有了一个带有循环变量j的新循环。变量j从0开始递增，直到它小于itemsInDiagonal。

然后我们使用循环变量i和j来获取行和列索引。该计算的逻辑取决于循环变量i是否大于midPoint。当i大于midPoint时，我们还使用length变量来确定行和列索引：

```java
int rowIndex;
int columnIndex;

if (i <= midPoint) {
    itemsInDiagonal++;
    for (int j = 0; j < itemsInDiagonal; j++) {
        rowIndex = (i - j) - 1;
        columnIndex = j;
        items.append(twoDArray[rowIndex][columnIndex]);
    }
} else {
    itemsInDiagonal--;
    for (int j = 0; j < itemsInDiagonal; j++) {
        rowIndex = (length - 1) - j;
        columnIndex = (i - length) + j;
        items.append(twoDArray[rowIndex][columnIndex]);
    }
}
```

## 4. 总结

在本教程中，我们展示了如何使用有助于获取行和列索引的方法沿对角线循环遍历方形二维数组。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-multidimensional)上获得。