---
layout: post
title:  用于加密的初始化向量
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在本教程中，我们将讨论如何将[初始化向量(IV)](https://en.wikipedia.org/wiki/Initialization_vector)与加密算法结合使用。我们还将讨论使用IV时的最佳实践。

本文假设你对密码学有基本的了解。

对于所有示例，我们将以不同的模式使用[AES算法](https://www.baeldung.com/java-aes-encryption-decryption)。

## 2. 加密算法

任何密码算法都需要一些数据或明文和密钥来生成加密文本或密文。并且，它还使用生成的密文和相同的密钥来生成解密数据或原始明文。

例如，块密码算法通过加密和解密固定长度的块来提供安全性。我们使用不同的加密模式对整个数据重复应用一种算法，并规定要使用的IV类型。

在块密码的情况下，我们使用相同大小的块。如果明文大小小于块大小，我们使用填充。有些模式不使用填充，因为它们使用块密码作为流密码。

## 3. 初始化向量(IV)

我们在加密算法中使用IV作为起始状态，将其添加到密码中以隐藏加密数据中的模式。这有助于避免在每次调用后重新发布新密钥的需要。

### 3.1 IV的属性

对于大多数加密模式，我们使用唯一序列或IV。而且，我们永远不应该使用相同的密钥重复使用相同的IV。这确保了相同的明文加密具有不同的密文，即使我们使用相同密钥对其进行多次加密也是如此。

让我们看一下IV的一些特征，具体取决于加密模式：

-   它必须是不重复的
-   基于加密模式，它也需要是随机的
-   不必保密
-   它需要是一个加密随机数
-   无论密钥长度如何，AES的IV始终为128位

### 3.2 生成IV

我们可以直接从Cipher类中获取IV：

```java
byte[] iv = cipher.getIV();
```

如果我们不确定默认实现，我们总是可以编写我们的方法来生成IV。如果我们不提供显式IV，则使用Cipher.getIV()隐式获取IV。我们可以使用任何方法来生成IV，只要它符合上面讨论的属性。

首先，让我们使用SecureRandom创建一个随机IV：

```java
public static IvParameterSpec getIVSecureRandom(String algorithm) throws NoSuchAlgorithmException, NoSuchPaddingException {
    SecureRandom random = SecureRandom.getInstanceStrong();
    byte[] iv = new byte[Cipher.getInstance(algorithm).getBlockSize()];
    random.nextBytes(iv);
    return new IvParameterSpec(iv);
}
```

接下来，我们将通过从Cipher类获取参数来创建IV：

```java
public static IvParameterSpec getIVInternal(Cipher cipher) throws InvalidParameterSpecException {
    AlgorithmParameters params = cipher.getParameters();
    byte[] iv = params.getParameterSpec(IvParameterSpec.class).getIV();
    return new IvParameterSpec(iv);
}
```

我们可以使用上述任何一种方法来生成随机的、不可预测的IV。但是，**对于某些模式，如GCM，我们将IV与计数器一起使用**。在这种情况下，我们使用前几个字节，主要是12个字节用于IV，接下来的4个字节用于计数器：

```java
public static byte[] getRandomIVWithSize(int size) {
    byte[] nonce = new byte[size];
    new SecureRandom().nextBytes(nonce);
    return nonce;
}
```

在这种情况下，我们需要确保我们不会重复计数器并且IV也是唯一的。

最后，**虽然不推荐，但我们也可以使用硬编码IV**。

## 4. 在不同模式下使用IV

众所周知，加密的主要功能是屏蔽明文，使攻击者无法猜测。因此，我们使用不同的密码模式来掩盖密文中的模式。

ECB、CBC、OFB、CFB、CTR、CTS和XTS等模式提供机密性，但是这些模式不能防止篡改和修改。我们可以添加消息认证码(MAC)或数字签名进行检测。我们使用各种实现方式，在经过身份验证的加密(AE)下提供组合模式，CCM、GCM、CWC、EAX、IAPM和OCB是几个例子。

### 4.1 电子密码本(ECB)模式

[电子密码本模式](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB))使用密钥单独加密每个块，这始终将相同的明文加密成相同的密文块，从而不能很好地隐藏模式。因此，我们不将它用于加密协议。解密也同样容易受到重放攻击。

要以ECB模式加密数据，我们使用：

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, key);
ciphertext = cipher.doFinal(data);
```

要以ECB模式解密数据，我们编写：

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, key);
plaintext = cipher.doFinal(cipherText);
```

我们没有使用任何IV，因此相同的明文将产生相同的密文，从而使其容易受到攻击。**尽管ECB模式是最脆弱的，但它仍然是许多提供商的默认加密模式。因此，我们需要更加警惕显式设置加密模式**。

### 4.2 网络区块链(CBC)模式

[网络块链接模式](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_block_chaining_(CBC))使用IV来防止相同的明文产生相同的密文，**我们需要注意IV是可靠的随机或唯一的。否则，我们将面临与ECB模式相同的漏洞**。

让我们使用getIVSecureRandom获取随机IV：

```java
IvParameterSpec iv = CryptoUtils.getIVSecureRandom("AES");
```

首先，我们将使用IV使用CBC模式加密数据：

```java
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, key, iv);
```

接下来，让我们使用IvParameterSpec对象传递相同的IV进行解密：

```java
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, key, new IvParameterSpec(iv));
```

### 4.3 网络反馈(CFB)模式

[网络反馈模式](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_feedback_(CFB))是最基本的流媒体模式，它就像一个自同步流密码。与CBC模式不同，我们在这里不需要任何填充。在CFB模式下，我们使用IV作为密码生成流的来源。**同样，如果我们对不同的加密使用相同的IV，则密文中可能会出现相似之处。这里也像CBC模式一样，IV应该是随机的。如果IV是可预测的，那么我们就失去了保密性**。

让我们为CFB模式生成一个随机IV：

```java
IvParameterSpec iv = CryptoUtils.getIVSecureRandom("AES/CFB/NoPadding");
```

另一种极端情况是，如果我们使用全零IV，那么在CFB-8模式下，某些密钥可能会生成全零IV和全零明文。在这种情况下，1/256密钥不会生成加密，这将导致明文作为密文返回。

**对于CBC和CFB模式，重用IV会揭示有关两条消息共享的公共块的信息**。

### 4.4 计数器(CTR)和输出反馈(OFB)模式

[计数器模式](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_(CTR))和[输出反馈模式](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Output_feedback_(OFB))使块密码成为同步流密码，每种模式都会生成密钥流块。在这种情况下，我们使用特定的IV初始化密码。我们这样做主要是为了将12个字节分配给IV，将4个字节分配给计数器。这样，我们就可以加密长度为2^32块的消息。

在这里，让我们创建一个IV：

```java
IvParameterSpec ivSpec = CryptoUtils.getIVSecureRandom("AES");
```

对于CTR模式，**初始比特流取决于IV和密钥。在这里，重用IV也会导致密钥比特流重用。反过来，这将导致破坏安全性**。

如果IV不是唯一的，则计数器可能无法为与重复计数器块对应的块提供预期的机密性。但是，其他数据块不受影响。

### 4.5 伽罗瓦/计数器(GCM)模式

[伽罗瓦/计数器模式](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf)是一种AEAD加密模式，它将计数器模式加密与身份验证机制相结合。并且，它可以保护明文和其他经过身份验证的数据(AAD)。

**但是，GCM中的这种身份验证取决于IV的唯一性。我们使用随机数作为IV，如果我们重复一个IV，那么我们的实现可能容易受到攻击**。

由于GCM使用AES进行加密，因此IV或计数器为16字节。因此，我们使用前12个字节作为IV，最后4个字节作为计数器。

要在GCM模式下创建IV，我们需要设置GCMParameterSpec。让我们创建一个IV：

```java
byte[] iv = CryptoUtils.getRandomIVWithSize(12);
```

首先，让我们获取Cipher的实例并使用IV对其进行初始化：

```java
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
cipher.init(Cipher.ENCRYPT_MODE, key, new GCMParameterSpec(128, iv));
```

现在，我们将使用用于解密的IV创建并初始化Cipher：

```java
cipher.init(Cipher.DECRYPT_MODE, key, new GCMParameterSpec(128, iv));
```

在这里，我们也需要一个唯一的IV，否则就可以破译明文。

### 4.6 总结IV

下表总结了不同模式所需的IV类型：

| 模式      | 初始化向量              |
|---------|--------------------|
| ECB     | 无IV                |
| CBC/CFB | 随机不可预测IV           |
| OFB/CTR | 唯一IV(计数器)          |
| GCM     | 唯一IV(12字节)和计数器(4字节) |


正如我们所见，重复使用具有相同密钥的IV会导致安全性下降。如果可能，我们应该使用更高级的模式，如GCM。此外，CCM等某些模式在标准JCE发行版中不可用。在这种情况下，我们可以使用[BouncyCastle](https://www.bouncycastle.org/fips-java/BCFipsIn100.pdf) API来实现它。

## 5. 总结

在本文中，我们展示了如何在不同的加密模式下使用IV，我们还讨论了使用IV时的问题和最佳实践。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-3)上获得。