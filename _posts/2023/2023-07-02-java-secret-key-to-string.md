---
layout: post
title:  Java中密钥和字符串转换
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在现实生活中，我们会遇到几种出于安全目的需要加密和解密的情况，我们可以使用密钥轻松实现这一点。因此，为了加密和解密密钥，我们必须知道将密钥转换为字符串的方式，反之亦然。在本教程中，我们将介绍Java中的密钥和字符串转换。此外，我们将通过示例介绍在Java中创建密钥的不同方法。

## 2. 密钥

密钥是用于加密和解密消息的信息或参数。在Java中，我们有一个SecretKey接口，将其定义为机密(对称)密钥。此接口的目的是对所有密钥接口进行分组(并为其提供类型安全)。

在Java中有两种生成密钥的方法：从随机数生成或从给定密码派生。

**在第一种方法中，密钥是从加密安全(伪)随机数生成器(如SecureRandom类)生成的**。

为了生成密钥，我们可以使用KeyGenerator类。让我们定义一个生成SecretKey的方法-参数n以位为单位指定密钥的长度(128、192或256)：

```java
public static SecretKey generateKey(int n) throws NoSuchAlgorithmException {
    KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
    keyGenerator.init(n);
    SecretKey originalKey = keyGenerator.generateKey();
    return originalKey;
}
```

**在第二种方法中，密钥是使用基于密码的密钥派生函数(如PBKDF2)从给定密码派生的**。我们还需要一个盐值来将密码转换为密钥，盐也是一个随机值。

我们可以使用SecretKeyFactory类和PBKDF2WithHmacSHA256算法从给定的密码生成密钥。

让我们定义一种从给定密码生成SecretKey的方法，迭代次数为65536次，密钥长度为256位：

```java
public static SecretKey getKeyFromPassword(String password, String salt) throws NoSuchAlgorithmException, InvalidKeySpecException {
    SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
    KeySpec spec = new PBEKeySpec(password.toCharArray(), salt.getBytes(), 65536, 256);
    SecretKey originalKey = new SecretKeySpec(factory.generateSecret(spec).getEncoded(), "AES");
    return originalKey;
}
```

## 3. SecretKey与String转换

### 3.1 SecretKey转换为String

我们会将SecretKey转换为字节数组，然后，我们将使用Base64编码将字节数组转换为字符串：

```java
public static String convertSecretKeyToString(SecretKey secretKey) throws NoSuchAlgorithmException {
    byte[] rawData = secretKey.getEncoded();
    String encodedKey = Base64.getEncoder().encodeToString(rawData);
    return encodedKey;
}
```

### 3.2 String转换为SecretKey

我们将使用Base64解码将编码的字符串密钥转换为字节数组。然后，使用SecretKeySpecs，我们将字节数组转换为SecretKey：

```java
public static SecretKey convertStringToSecretKeyto(String encodedKey) {
    byte[] decodedKey = Base64.getDecoder().decode(encodedKey);
    SecretKey originalKey = new SecretKeySpec(decodedKey, 0, decodedKey.length, "AES");
    return originalKey;
}
```

让我们快速验证转换：

```java
SecretKey encodedKey = ConversionClassUtil.getKeyFromPassword("Baeldung@2021", "@$#baelDunG@#^$");
String encodedString = ConversionClassUtil.convertSecretKeyToString(encodedKey);
SecretKey decodeKey = ConversionClassUtil.convertStringToSecretKeyto(encodedString);
Assertions.assertEquals(encodedKey, decodeKey);
```

## 4. 总结

总之，我们已经学习了如何在Java中将SecretKey转换为String，反之亦然。此外，我们还讨论了在Java中创建SecretKey的各种方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-3)上获得。