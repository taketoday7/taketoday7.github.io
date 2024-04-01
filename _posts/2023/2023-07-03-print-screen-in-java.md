---
layout: post
title:  如何在Java中打印屏幕
category: java
copyright: java
excerpt: Java OS
---

## 1. 概述

当你需要在桌面上执行打印屏幕操作时，键盘上有一个内置的“PrntScr”按钮可以帮助你完成此操作，有时这就足够了。

但是当你需要以编程方式执行该操作时，就会出现问题。简而言之，你可能需要使用Java将当前屏幕截图保存为图像文件。

让我们看看如何做到这一点。

## 2. Robot类

**Java java.awt.Robot类**是我们将要使用的主要API。此API包含一个名为“createScreenCapture”的方法，该方法在传递特定形状时获取屏幕截图：

```java
robot.createScreenCapture(rectangle);
```

由于上述方法返回一个java.awt.image.BufferedImage实例，你所要做的就是使用**javax.imageio.ImageIO**实用程序类将检索到的图像写入文件。

## 3. 捕获并保存图像文件

用于图像捕获和保存的Java代码如下：

```java
public void getScreenshot(int timeToWait) throws Exception {
    Rectangle rec = new Rectangle(Toolkit.getDefaultToolkit().getScreenSize());
    Robot robot = new Robot();
    BufferedImage img = robot.createScreenCapture(rectangle);
    
    ImageIO.write(img, "jpg", setupFileNamePath());
}
```

在这里，可以通过为java.awt.Rectangle实例设置所需的大小来捕获屏幕的一部分。在上面的示例中，通过设置当前屏幕大小，已将其设置为捕获全屏。

## 4. 总结

在本教程中，我们快速浏览了Java打印屏幕的用法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-os)上获得。