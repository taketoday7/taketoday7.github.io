---
layout: post
title:  在Java中加密和解密文件
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在本教程中，我们将介绍如何使用现有的JDK API加密和解密文件。

## 2. 先编写测试

我们将从编写测试、TDD风格开始。由于我们将在这里处理文件，因此集成测试似乎是合适的。

由于我们只是使用现有的JDK功能，因此不需要外部依赖项。

首先，**我们将使用新生成的密钥对内容进行加密**(在本例中使用AES([高级加密标准](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard))作为对称加密算法)。

另请注意，我们在构造函数(AES/CBC/PKCS5Padding)中定义了完整的转换字符串，它是已用加密、分组密码模式和填充(algorithm/mode/padding)的串联。默认情况下，JDK实现支持许多不同的转换，但请注意，并非每个组合仍然可以按照今天的标准被认为是加密安全的。

我们假设**我们的FileEncrypterDecrypter类会将输出写入名为baz.enc的文件**。之后，**我们使用相同的密钥解密此文件**，并检查解密后的内容是否与原始内容相同：

```java
@Test
public void whenEncryptingIntoFile_andDecryptingFileAgain_thenOriginalStringIsReturned() {
    String originalContent = "foobar";
    SecretKey secretKey = KeyGenerator.getInstance("AES").generateKey();

    FileEncrypterDecrypter fileEncrypterDecrypter = new FileEncrypterDecrypter(secretKey, "AES/CBC/PKCS5Padding");
    fileEncrypterDecrypter.encrypt(originalContent, "baz.enc");

    String decryptedContent = fileEncrypterDecrypter.decrypt("baz.enc");
    assertThat(decryptedContent, is(originalContent));

    new File("baz.enc").delete(); // cleanup
}
```

## 3. 加密

我们将使用指定的转换字符串在FileEncrypterDecrypter类的构造函数中初始化Cipher。

这允许我们在指定错误转换的情况下尽早失败：

```java
FileEncrypterDecrypter(SecretKey secretKey, String transformation) {
    this.secretKey = secretKey;
    this.cipher = Cipher.getInstance(transformation);
}
```

然后我们**可以使用实例化的Cipher和提供的密钥来执行加密**：

```java
void encrypt(String content, String fileName) {
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);
    byte[] iv = cipher.getIV();

    try (FileOutputStream fileOut = new FileOutputStream(fileName); 
        CipherOutputStream cipherOut = new CipherOutputStream(fileOut, cipher)) {
        fileOut.write(iv);
        cipherOut.write(content.getBytes());
    }
}
```

**Java允许我们利用方便的CipherOutputStream类将加密内容写入另一个OutputStream**。

请注意，我们将IV([初始化向量](https://en.wikipedia.org/wiki/Initialization_vector))写入输出文件的开头。在此示例中，IV是在初始化Cipher时自动生成的。

使用CBC模式时必须使用IV，以便随机化加密输出。然而，IV不被视为机密，因此可以将其写在文件的开头。

## 4. 解密

为了解密，我们同样必须先读取IV。之后，我们可以初始化Cipher并解密内容。

我们可以再次**使用一个特殊的Java类CipherInputStream，它透明地处理实际的解密**：

```java
String decrypt(String fileName) {
    String content;

    try (FileInputStream fileIn = new FileInputStream(fileName)) {
        byte[] fileIv = new byte[16];
        fileIn.read(fileIv);
        cipher.init(Cipher.DECRYPT_MODE, secretKey, new IvParameterSpec(fileIv));

        try (CipherInputStream cipherIn = new CipherInputStream(fileIn, cipher);
                InputStreamReader inputReader = new InputStreamReader(cipherIn);
                BufferedReader reader = new BufferedReader(inputReader)
            ) {
            StringBuilder sb = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
            content = sb.toString();
        }

    }
    return content;
}
```

## 5. 总结

我们已经看到，我们可以使用标准JDK类(例如Cipher、CipherOutputStream和CipherInputStream)执行基本的加密和解密。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-1)上获得。

此外，你可以在[此处](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/crypto/Cipher.html)找到JDK中可用的Cipher列表。

最后，请注意，此处的代码示例并不是生产级代码，在使用它们时需要全面考虑系统的具体情况。