---
layout: post
title:  Java中的ASCII艺术
category: java
copyright: java
excerpt: Java Console
---

## 1. 概述

在本文中，我们将讨论使用Java语言的2D图形支持中的概念在Java中创建[ASCII](https://www.baeldung.com/cs/ascii-code)字符或字符串的图形打印。

## 2. 使用2D图形绘制字符串

在Graphics2D类的帮助下，可以将String绘制为图像，通过调用drawString()方法实现。

由于Graphics2D是抽象的，因此我们可以通过扩展它并实现与Graphics类关联的各种方法来创建一个实例。

虽然这是一项繁琐的任务，但通常通过在Java中创建BufferedImage实例并从中检索其底层Graphics实例来完成：

```java
BufferedImage bufferedImage = new BufferedImage(
    width, height, 
    BufferedImage.TYPE_INT_RGB);
Graphics graphics = bufferedImage.getGraphics();
```

### 2.1 用ASCII字符替换图像矩阵索引

在绘制字符串时，Graphics2D类使用一种简单的类似矩阵的技术，其中划分出设计字符串的区域被分配一个特定值，而其他区域被赋予0值。

为了能够用所需的ASCII字符替换雕刻区域，我们需要将雕刻区域的值检测为单个数据点(例如整数)，而不是RGB颜色值。

要将图像的RGB颜色表示为整数，我们将图像类型设置为整数模式：

```java
BufferedImage bufferedImage = new BufferedImage(
    width, height, 
    BufferedImage.TYPE_INT_RGB);
```

**基本思想是用所需的艺术特征替换分配给图像矩阵的非0索引的值**。

而表示0值的矩阵索引将被分配一个空格字符，整数模式的0等效值是-16777216。

## 3. ASCII艺术生成器

让我们考虑一个需要对“TUYUCHENG”字符串进行ASCII艺术处理的情况。

我们首先创建一个具有所需宽度/高度的空图像，并将图像类型设置为整数模式，如第2.1节所述。

为了能够在Java中使用2D图形的高级渲染选项，我们将Graphics对象强制转换为Graphics2D实例。然后我们在使用“TUYUCHENG”字符串调用drawString()方法之前设置所需的渲染参数：

```java
Graphics2D graphics2D = (Graphics2D) graphics;
graphics2D.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
graphics2D.drawString("TUYUCHENG", 12, 24);
```

在上面，12和24分别代表图像上文本打印开始点的x坐标和y坐标。

现在，我们有一个2D图形，其底层矩阵包含两种类型的判别值；非0和0索引。

但是为了让我们理解这个概念，我们将遍历二维数组(或矩阵)并将所有值替换为ASCII字符“*”：

```java
for (int y = 0; y < settings.height; y++) {
    StringBuilder stringBuilder = new StringBuilder();

    for (int x = 0; x < settings.width; x++) {
        stringBuilder.append("*");
    }

    if (stringBuilder.toString().trim().isEmpty()) {
        continue;
    }

    System.out.println(stringBuilder);
}
```

上面的输出只显示了一个星号(*)块，如下所示：

![](/assets/images/2023/java/asciiartinjava01.png)

如果我们通过仅将等于-16777216的整数值替换为“\*”并将其余值替换为“ ”来区分替换为“\*”：

```java
for (int y = 0; y < settings.height; y++) {
    StringBuilder stringBuilder = new StringBuilder();

    for (int x = 0; x < settings.width; x++) {
        stringBuilder.append(image.getRGB(x, y) == -16777216 ? "*" : " ");
    }

    if (stringBuilder.toString().trim().isEmpty()) {
        continue;
    }

    System.out.println(stringBuilder);
}
```

我们获得了一种不同的ASCII艺术，它对应于我们的字符串“TUYUCHENG”，但在一个像这样的倒雕中：

![](/assets/images/2023/java/asciiartinjava02.png)

最后，我们通过将等于-16777216的整数值替换为“ ”并将其余值替换为“*”来反转判别：

```java
for (int y = 0; y < settings.height; y++) {
    StringBuilder stringBuilder = new StringBuilder();

    for (int x = 0; x < settings.width; x++) {
        stringBuilder.append(image.getRGB(x, y) == -16777216 ? " " : "*");
    }

    if (stringBuilder.toString().trim().isEmpty()) {
        continue;
    }

    System.out.println(stringBuilder);
}
```

这为我们提供了所需字符串的ASCII艺术：

![](/assets/images/2023/java/asciiartinjava03.png)

## 4. 总结

在这个快速教程中，我们介绍了如何使用内置的2D图形库在Java中创建ASCII艺术。

Github上的源代码提供了一个实用函数，可以接收任何字符串。与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-console)上获得。