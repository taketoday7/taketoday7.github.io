---
layout: post
title:  在Java中生成安全的AES密钥
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在本文中，我们将深入探讨AES或一般密码中密钥的用途。

## 2. AES

[高级加密标准(AES)](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf)是数据加密标准(DES)的继承者，由美国国家标准与技术研究院(NIST)于2001年发布。它被归类为对称块密码。

对称密码使用相同的密钥进行加密和解密，块密码意味着它适用于输入明文的128位块：

[![AES密钥](https://www.baeldung.com/wp-content/uploads/2022/01/AES-Key-1024x327.png)](https://www.baeldung.com/wp-content/uploads/2022/01/AES-Key.png)

### 2.1 AES变体

**根据密钥大小，AES支持三种变体：AES-128(128位)、AES-192(192位)和AES-256(256位)**。增加密钥大小会增加加密强度，因为更大的密钥大小意味着可能的密钥数量更大。因此，算法执行期间要执行的轮数也会增加，因此所需的计算量也会增加：

| 密钥大小 | 块大小 | 轮数 |
| -------- | ------ |----|
| 128      | 128    | 10 |
| 192      | 128    | 12 |
| 256      | 128    | 14 |

### 2.2 AES有多安全？

AES算法是公开信息-AES密钥是秘密的，必须知道才能成功解密。因此，它归结为破解AES密钥。假设密钥被安全保存，攻击者将不得不尝试猜测密钥。

让我们看看蛮力方法在猜测密钥方面的表现如何。

**AES-128密钥是128位，这意味着有2^128个可能的值**。搜索它需要[大量且不可行的时间和金钱](https://www.reddit.com/r/theydidthemath/comments/1x50xl/time_and_energy_required_to_bruteforce_a_aes256/)。因此，AES实际上是无法通过蛮力方法破解的。

已经有一些[非蛮力方法](https://threatpost.com/new-attack-finds-aes-keys-several-times-faster-brute-force-081911/75562/)，但这些方法只能将可能的密钥查找空间减少几位。

所有这一切意味着，**如果对密钥的了解为零，AES几乎是不可能破解的**。

## 3. 好密钥的属性

现在让我们看看在生成AES密钥时要遵循的一些重要准则。

### 3.1 密钥大小

由于AES支持三种密钥大小，因此我们应该为用例选择正确的密钥大小。AES-128是商业应用中最常见的选择，它提供了安全性和速度之间的平衡。[国家政府](https://web.archive.org/web/20101106122007/http://csrc.nist.gov/groups/ST/toolkit/documents/aes/CNSS15FS.pdf)通常使用AES-192和AES-256来获得最大的安全性。如果我们想要更高级别的安全性，我们可以使用AES-256。

量子计算机确实构成了能够减少大型密钥空间所需计算的威胁。因此，拥有AES-256密钥将更具前瞻性，尽管到目前为止，商业应用程序的任何威胁参与者都无法触及它们。

### 3.2 熵

熵是指密钥中的随机性，例如，如果生成的密钥不够随机，并且与时间相关、机器相关或字典单词有某种相关性，它就会变得脆弱。攻击者将能够缩小密钥搜索空间，从而削弱AES的强度。因此，**密钥真正随机至关重要**。

## 4. 生成AES密钥

现在，有了生成AES密钥的指南，让我们看看生成它们的各种方法。

对于所有代码片段，我们将密码定义为：

```java
private static final String CIPHER = "AES";
```

### 4.1 Random

让我们使用Java中的Random类来生成密钥：

```java
private static Key getRandomKey(String cipher, int keySize) {
    byte[] randomKeyBytes = new byte[keySize / 8];
    Random random = new Random();
    random.nextBytes(randomKeyBytes);
    return new SecretKeySpec(randomKeyBytes, cipher);
}
```

我们创建一个所需密钥大小的字节数组，并用从random.nextBytes()获得的随机字节填充它。然后使用随机字节数组创建SecretKeySpec。

Java Random类是一个[伪随机数生成器](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)(PRNG)，也称为**确定性随机数生成器**(DRNG)，这意味着它不是真正随机的。PRNG中的随机数序列可以完全根据其种子确定。Java不建议将[Random](https://docs.oracle.com/en/java/javase/16/docs/api/java.base/java/util/Random.html)用于加密应用程序。

话虽如此，**永远不要使用Random来生成密钥**。

### 4.2 SecureRandom

我们现在将使用Java中的SecureRandom类来生成密钥：

```java
private static Key getSecureRandomKey(String cipher, int keySize) {
    byte[] secureRandomKeyBytes = new byte[keySize / 8];
    SecureRandom secureRandom = new SecureRandom();
    secureRandom.nextBytes(secureRandomKeyBytes);
    return new SecretKeySpec(secureRandomKeyBytes, cipher);
}
```

与前面的示例类似，我们实例化一个所需密钥大小的字节数组。现在，我们不使用Random，而是使用SecureRandom为我们的字节数组生成随机字节。[Java推荐使用SecureRandom](https://docs.oracle.com/en/java/javase/16/docs/api/java.base/java/security/SecureRandom.html)为加密应用程序生成随机数，它最低限度地符合[FIPS 140-2](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-2.pdf)，加密模块的安全要求。

显然，在Java中，**SecureRandom是获取随机数的事实标准**。但这是生成密钥的最佳方式吗？让我们继续下一个方法。

### 4.3 KeyGenerator

接下来，让我们使用KeyGenerator类生成一个密钥：

```java
private static Key getKeyFromKeyGenerator(String cipher, int keySize) throws NoSuchAlgorithmException {
    KeyGenerator keyGenerator = KeyGenerator.getInstance(cipher);
    keyGenerator.init(keySize);
    return keyGenerator.generateKey();
}
```

我们为正在使用的密码获取了一个KeyGenerator实例。然后，我们使用所需的keySize初始化keyGenerator对象。最后，我们调用generateKey方法来生成我们的密钥。那么，它与Random和SecureRandom方法有何不同？

**有两个重要的区别值得强调**。

其一，无论是Random还是SecureRandom方法都无法判断我们是否根据Cipher规范生成了正确大小的密钥。只有当我们进行加密时，如果密钥的大小不受支持，我们才会遇到异常。

当我们初始化密码进行加密时，使用带有无效keySize的SecureRandom会抛出异常：

```java
encrypt(plainText, getSecureRandomKey(CIPHER, 111));
```

```text
java.security.InvalidKeyException: Invalid AES key length: 13 bytes
  at java.base/com.sun.crypto.provider.AESCrypt.init(AESCrypt.java:90)
  at java.base/com.sun.crypto.provider.GaloisCounterMode.init(GaloisCounterMode.java:321)
  at java.base/com.sun.crypto.provider.CipherCore.init(CipherCore.java:592)
  at java.base/com.sun.crypto.provider.CipherCore.init(CipherCore.java:470)
  at java.base/com.sun.crypto.provider.AESCipher.engineInit(AESCipher.java:322)
  at java.base/javax.crypto.Cipher.implInit(Cipher.java:867)
  at java.base/javax.crypto.Cipher.chooseProvider(Cipher.java:929)
  at java.base/javax.crypto.Cipher.init(Cipher.java:1299)
  at java.base/javax.crypto.Cipher.init(Cipher.java:1236)
  at cn.tuyucheng.taketoday.secretkey.Main.encrypt(Main.java:59)
  at cn.tuyucheng.taketoday.secretkey.Main.main(Main.java:51)
```

另一方面，使用KeyGenerator会在密钥生成过程中失败，这让我们可以更恰当地处理它：

```java
encrypt(plainText, getKeyFromKeyGenerator(CIPHER, 111));
```

```text
java.security.InvalidParameterException: Wrong keysize: must be equal to 128, 192 or 256
  at java.base/com.sun.crypto.provider.AESKeyGenerator.engineInit(AESKeyGenerator.java:93)
  at java.base/javax.crypto.KeyGenerator.init(KeyGenerator.java:539)
  at java.base/javax.crypto.KeyGenerator.init(KeyGenerator.java:516)
  at cn.tuyucheng.taketoday.secretkey.Main.getKeyFromKeyGenerator(Main.java:89)
  at cn.tuyucheng.taketoday.secretkey.Main.main(Main.java:58)
```

另一个关键区别是默认使用SecureRandom。KeyGenerator类是Java的加密包javax.crypto的一部分，它确保使用SecureRandom来实现随机性。我们可以看到KeyGenerator类中init方法的定义：

```java
public final void init(int keysize) {
    init(keysize, JCAUtil.getSecureRandom());
}
```

因此，使用KeyGenerator作为实践确保我们永远不会使用Random类对象来生成密钥。

### 4.4 基于密码的密钥

到目前为止，我们一直在从随机的和对人类不太友好的字节数组中生成密钥。基于密码的密钥(PBK)使我们能够根据人类可读的密码生成SecretKey：

```java
private static Key getPasswordBasedKey(String cipher, int keySize, char[] password) throws NoSuchAlgorithmException, InvalidKeySpecException {
    byte[] salt = new byte[100];
    SecureRandom random = new SecureRandom();
    random.nextBytes(salt);
    PBEKeySpec pbeKeySpec = new PBEKeySpec(password, salt, 1000, keySize);
    SecretKey pbeKey = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256").generateSecret(pbeKeySpec);
    return new SecretKeySpec(pbeKey.getEncoded(), cipher);
}
```

这里发生了很多事情，让我们分解一下。

**我们从人类可读的密码开始**，这是机密，必须保密。必须遵循密码准则，例如最小长度为8个字符、特殊字符的使用、大小写字母、数字的组合等。此外，[OWASP指南](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#implement-proper-password-strength-controls)建议检查已经暴露的密码。

用户友好的密码没有足够的熵。因此，**我们添加了额外的随机生成的字节，称为盐，以使其更难猜测**。[最小盐长度应为128位](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf)，我们使用SecureRandom来生成盐。盐不是机密，而是以明文形式存储。我们应该为每个密码成对生成盐，而不是全局使用相同的盐。这将防止[彩虹表](https://en.wikipedia.org/wiki/Rainbow_table)攻击，该攻击使用从预先计算的哈希表中查找来破解密码。

迭代次数是密钥生成算法应用转换函数的次数，它应该尽可能大。[推荐的最小迭代次数为1000次](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf)，较高的迭代次数会增加攻击者在对所有可能的密码执行暴力检查时的复杂性。

密钥大小与我们之前讨论的相同，对于AES可以是128、192或256。

我们已将上面讨论的所有四个元素包装到一个PBEKeySpec对象中。接下来，使用SecretKeyFactory，我们获得PBKDF2WithHmacSHA256算法的一个实例来生成密钥。

最后，使用PBEKeySpec调用generateSecret，我们基于人类可读的密码生成一个SecretKey。

## 5. 总结

生成密钥有两个主要依据，它可以是随机密钥或基于人类可读密码的密钥。我们已经讨论了生成随机密钥的三种方法，其中，KeyGenerator提供了真正的随机性，也提供了制衡。因此，KeyGenerator是更好的选择。

对于基于人类可读密码的密钥，我们可以使用SecretKeyFactory以及使用SecureRandom和高迭代次数生成的盐。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-3)上获得。