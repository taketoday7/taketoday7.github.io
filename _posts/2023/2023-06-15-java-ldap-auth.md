---
layout: post
title:  使用纯Java的LDAP身份验证
category: java
copyright: java
excerpt: Java JNDI
---

## 1. 概述

在本文中，我们将介绍如何使用纯Java通过[LDAP](https://ldap.com/learn-about-ldap/)对用户进行身份验证。此外，我们将探讨如何搜索用户的[专有名称](https://ldap.com/ldap-dns-and-rdns/)(DN)。这很重要，因为LDAP需要DN来验证用户。

为了执行搜索和用户身份验证，我们将使用Java命名和目录接口([JNDI](https://www.baeldung.com/jndi))的目录服务访问功能。

首先，我们将简要讨论什么是LDAP和JNDI。然后我们将讨论如何通过JNDI API使用LDAP进行身份验证。

## 2. 什么是LDAP？

**轻量级目录访问协议(LDAP)定义了客户端发送请求和接收来自目录服务的响应的方式**，我们将使用此协议的目录服务称为LDAP服务器。

**LDAP服务器提供的数据存储在基于[X.500](https://docs.oracle.com/javase/jndi/tutorial/ldap/models/x500.html)的信息模型中**，这是一组用于电子目录服务的计算机网络标准。

## 3. 什么是JNDI？

**JNDI为应用程序提供标准API以发现和访问命名和目录服务**。它的根本目的是为应用程序提供一种访问组件和资源的方法，这是本地和网络上的。

命名服务是此功能的基础，因为它们提供对分层命名空间中按名称的服务、数据或对象的单点访问。为这些本地或网络可访问资源中的每一个指定的名称是在托管命名服务的服务器上配置的。

**我们可以使用JNDI的命名服务接口访问目录服务，如LDAP**。这是因为目录服务只是一种特殊类型的命名服务，它使每个命名条目都具有与其关联的属性列表。

除了属性之外，每个目录条目可能有一个或多个子项，这使得条目可以分层链接。在JNDI中，目录条目的子项表示为其父上下文的子上下文。

JNDI API的一个主要优点是它独立于任何底层服务提供者实现，例如LDAP。因此，我们可以使用JNDI来访问LDAP目录服务，而无需使用特定于协议的API。

使用JNDI不需要外部库，因为它是Java SE平台的一部分。此外，作为Java EE的核心技术，它被广泛用于实现企业级应用程序。

## 4. 使用LDAP进行身份验证的JNDI API概念

在讨论示例代码之前，让我们介绍一些有关使用JNDI API进行基于LDAP的身份验证的基础知识。

要连接到LDAP服务器，我们首先需要创建一个JNDI [InitialDirContext](https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/InitialDirContext.html)对象。这样做时，我们需要将环境属性作为[Hashtable](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Hashtable.html)传递到其构造函数中以对其进行配置。

除此之外，我们需要向此Hashtable添加我们希望用来进行身份验证的用户名和密码的属性。为此，我们必须将用户的DN和密码分别设置为Context.SECURITY_PRINCIPAL和Context.SECURITY_CREDENTIALS属性。

InitialDirContext实现主要的JNDI目录服务接口[DirContext](https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/DirContext.html)。通过这个接口，我们可以使用我们的新上下文在LDAP服务器上执行各种目录服务操作，这些包括将名称和属性绑定到对象以及搜索目录条目。

值得注意的是，JNDI返回的对象将具有与其底层LDAP条目相同的名称和属性。因此，要搜索一个条目，我们可以使用它的名称和属性作为条件来查找它。

通过JNDI检索目录条目后，我们可以使用[Attributes](https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/Attributes.html)接口查看其属性。此外，我们可以使用Attribute接口来检查它们中的每一个。

## 5. 如果我们没有用户的DN怎么办？

有时我们没有立即可用的用户DN来进行身份验证。为了解决这个问题，我们首先需要使用管理员凭据创建一个InitialDirContext。之后，我们可以使用它从目录服务器中搜索相关用户并获取他的DN。

然后，一旦我们有了用户的DN，我们就可以通过创建一个新的InitialDirContext来验证他的身份，这次使用他的凭据。为此，我们首先需要在环境属性中设置用户的DN和密码。之后，我们需要在创建InitDirContext时将这些属性传递给InitDirContext的构造函数。

现在我们已经讨论了使用JNDI API通过LDAP对用户进行身份验证，让我们看一下示例代码。

## 6. 示例代码

在我们的示例中，我们将使用[ApacheDS](https://directory.apache.org/apacheds/)目录服务器的[嵌入式版本](https://directory.apache.org/apacheds/advanced-ug/7-embedding-apacheds.html)。这是一个使用Java构建的LDAP服务器，旨在在单元测试中以嵌入式模式运行。

让我们看看如何设置它。

### 6.1 设置嵌入式ApacheDS服务器

要使用嵌入式ApacheDS服务器，我们需要定义Maven[依赖项](https://search.maven.org/classic/#search|ga|1|g%3A"org.apache.directory.server"ANDa%3A"apacheds-test-framework")：

```xml
<dependency>
    <groupId>org.apache.directory.server</groupId>
    <artifactId>apacheds-test-framework</artifactId>
    <version>2.0.0.AM26</version>
    <scope>test</scope>
</dependency>
```

接下来，我们需要使用JUnit 4创建单元测试类。要在此类中使用嵌入式ApacheDS服务器，我们必须声明它从ApacheDS库扩展AbstractLdapTestUnit。由于这个库还不兼容JUnit 5，因此我们需要使用JUnit 4。

此外，我们需要在单元测试类声明上方包含[Java注解](https://directory.apache.org/apacheds/advanced-ug/7-embedding-apacheds.html#basic-test)以配置服务器。我们可以从[完整的代码示例](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jndi/src/test/java/cn/tuyucheng/taketoday/jndi/ldap/auth/JndiLdapAuthManualTest.java)中看到为它们提供哪些值，我们将在稍后探讨。

最后，我们还需要将[users.ldif](https://github.com/eugenp/tutorials/blob/master/core-java-modules/core-java-jndi/src/test/resources/users.ldif)添加到类路径中。这样，当我们运行代码示例时，ApacheDS服务器就可以从该文件中加载[LDIF](https://datatracker.ietf.org/doc/html/rfc2849)格式的目录条目。这样做时，服务器将加载用户Joe Simms的条目。

接下来，我们讨论将对用户进行身份验证的示例代码。要针对LDAP服务器运行它，我们需要将代码添加到单元测试类中的方法中。这将使用文件中定义的DN和密码通过LDAP对Joe进行身份验证。

### 6.2 验证用户

为了对用户Joe Simms进行身份验证，我们需要创建一个新的InitialDirContext对象。这将创建与目录服务器的连接，并使用用户的DN和密码通过LDAP对用户进行身份验证。

为此，我们首先需要将这些环境属性添加到Hashtable中：

```java
Hashtable<String, String> environment = new Hashtable<String, String>();

environment.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
environment.put(Context.PROVIDER_URL, "ldap://localhost:10389");
environment.put(Context.SECURITY_AUTHENTICATION, "simple");
environment.put(Context.SECURITY_PRINCIPAL, "cn=Joe Simms,ou=Users,dc=tuyucheng,dc=com");
environment.put(Context.SECURITY_CREDENTIALS, "12345");
```

接下来，在一个名为authenticateUser的新方法中，我们通过将环境属性传递到其构造函数来创建InitialDirContext对象。然后，我们将关闭上下文以释放资源：

```java
DirContext context = new InitialDirContext(environment);
context.close();
```

最后，我们将对用户进行身份验证：

```java
assertThatCode(() -> authenticateUser(environment)).doesNotThrowAnyException();
```

现在我们已经介绍了用户身份验证成功的情况，让我们检查它何时失败。

### 6.3 处理用户认证失败

应用与之前相同的环境属性，让我们使用错误的密码使身份验证失败：

```java
environment.put(Context.SECURITY_CREDENTIALS, "wrongpassword");
```

然后，我们将检查使用此密码验证用户是否按预期失败：

```java
assertThatExceptionOfType(AuthenticationException.class).isThrownBy(() -> authenticateUser(environment));
```

接下来，让我们讨论在没有用户DN的情况下如何对用户进行身份验证。

### 6.4 以管理员身份查找用户的DN

有时，当我们想要对用户进行身份验证时，并没有他的DN。在这种情况下，我们首先需要创建一个具有管理员凭据的目录上下文来查找用户的DN，然后使用它对用户进行身份验证。

和以前一样，我们首先需要在Hashtable中添加一些环境属性。但这次，我们将使用管理员的DN作为Context.SECURITY_PRINCIPAL，以及他的默认管理员密码作为Context.SECURITY_CREDENTIALS属性：

```java
Hashtable<String, String> environment = new Hashtable<String, String>();

environment.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
environment.put(Context.PROVIDER_URL, "ldap://localhost:10389");
environment.put(Context.SECURITY_AUTHENTICATION, "simple");
environment.put(Context.SECURITY_PRINCIPAL, "uid=admin,ou=system");
environment.put(Context.SECURITY_CREDENTIALS, "secret");
```

接下来，我们将创建一个具有这些属性的InitialDirContext对象：

```java
DirContext adminContext = new InitialDirContext(environment);
```

这将创建一个目录上下文，连接到以管理员身份验证的服务器。这为我们提供了搜索用户DN的安全权限。

现在我们将根据用户的[CN(即他的常用名)](https://ldapwiki.com/wiki/CommonName)定义搜索过滤器。

```java
String filter = "(&(objectClass=person)(cn=Joe Simms))";
```

然后，使用此过滤器搜索用户，我们将创建一个[SearchControls](https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/SearchControls.html)对象：

```java
String[] attrIDs = { "cn" };
SearchControls searchControls = new SearchControls();
searchControls.setReturningAttributes(attrIDs);
searchControls.setSearchScope(SearchControls.SUBTREE_SCOPE);
```

接下来，我们将使用过滤器和SearchControls搜索用户：

```java
NamingEnumeration<SearchResult> searchResults = adminContext.search("dc=tuyucheng,dc=com", filter, searchControls);
  
String commonName = null;
String distinguishedName = null;
if (searchResults.hasMore()) {
    
    SearchResult result = (SearchResult) searchResults.next();
    Attributes attrs = result.getAttributes();
    
    distinguishedName = result.getNameInNamespace();
    assertThat(distinguishedName, isEqualTo("cn=Joe Simms,ou=Users,dc=tuyucheng,dc=com")));

    commonName = attrs.get("cn").toString();
    assertThat(commonName, isEqualTo("cn: Joe Simms")));
}
```

现在我们有了用户的DN，让我们对用户进行身份验证。

### 6.5 使用用户查找的DN进行身份验证

现在可以使用用户的DN进行身份验证，我们将现有环境属性中的管理员DN和密码替换为用户的DN和密码：

```java
environment.put(Context.SECURITY_PRINCIPAL, distinguishedName);
environment.put(Context.SECURITY_CREDENTIALS, "12345");
```

然后，有了这些，让我们对用户进行身份验证：

```java
assertThatCode(() -> authenticateUser(environment)).doesNotThrowAnyException();
```

最后，我们将关闭管理员上下文以释放资源：

```java
adminContext.close();
```

## 7. 总结

在本文中，我们讨论了如何使用JNDI通过LDAP使用用户的DN和密码来验证用户。

此外，我们研究了如果没有DN，如何查找它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jndi)上获得。
