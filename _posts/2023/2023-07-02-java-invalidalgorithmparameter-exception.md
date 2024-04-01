---
layout: post
title:  InvalidAlgorithmParameterException：Wrong IV Length
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

[高级加密标准](https://www.baeldung.com/java-aes-encryption-decryption)(AES)是一种广泛使用的对称分组密码算法，[初始化向量](https://www.baeldung.com/java-aes-encryption-decryption#3-initialization-vector-iv)(IV)在AES算法中起着重要作用。

在本教程中，我们将解释如何在Java中生成IV。此外，我们将描述**在生成IV并将其用于密码算法时如何避免InvalidAlgorithmParameterException**。

## 2. 初始化向量

AES算法通常有三个输入：明文、密钥和IV。它支持128、192和256位的密钥，以128位的块为单位对数据进行加密和解密。下图显示了AES输入：

[![图第 2 页](https://www.baeldung.com/wp-content/uploads/2020/12/Figures-Page-2.png)](https://www.baeldung.com/wp-content/uploads/2020/12/Figures-Page-2.png)

IV的目标是增强加密过程。在某些[AES操作模式](https://www.baeldung.com/java-aes-encryption-decryption#aes-variations)中，IV与密钥结合使用。例如，密码块链接(CBC)模式在其算法中使用IV。

通常，IV是由发送方选择的伪随机值。解密信息时加密的IV必须相同。

它与加密的块具有相同的大小。因此，IV的大小为16字节或128位。

## 3. 生成IV

建议使用java.security.SecureRandom类而不是java.util.Random来生成随机IV。此外，最好的做法是IV是不可预测的。此外，我们不应该在源代码中对IV进行硬编码。

要在密码中使用IV，我们使用IvParameterSpec类。让我们创建一个生成IV的方法：

```java
public static IvParameterSpec generateIv() {
    byte[] iv = new byte[16];
    new SecureRandom().nextBytes(iv);
    return new IvParameterSpec(iv);
}
```

## 4. 异常

AES算法要求IV大小必须为16字节(128位)。因此，**如果我们提供大小不等于16字节的IV，则会抛出InvalidAlgorithmParameterException**。

为了解决这个问题，我们必须使用大小为16字节的IV。有关在AES CBC模式下使用IV的示例代码片段可以在[本文](https://www.baeldung.com/java-aes-encryption-decryption#encryption-and-decryption)中找到。

## 5. 总结

总之，我们学习了如何在Java中生成初始化向量(IV)。此外，我们还描述了与IV生成相关的异常。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-algorithms)上获得。