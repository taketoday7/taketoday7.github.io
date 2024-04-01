---
layout: post
title:  安全上下文基础-User，Subject和Principal
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

安全性是任何Java应用程序的基本组成部分，我们可以找到许多可以处理安全问题的安全框架。此外，我们在这些框架中通常使用一些术语，如主题(Subject)、主体(Principal)和用户(User)。

在本教程中，**我们将解释安全框架的这些基本概念**。此外，我们将展示它们的关系和差异。

## 2. 主题

在安全上下文中，主题表示请求的来源。主题是获取有关资源的信息或修改资源的实体。此外，主体还可以是用户、程序、进程、文件、计算机、数据库等。

例如，一个人需要授权访问资源和应用程序以验证请求源。在这种情况下，这个人是一个主题。

让我们看一下基于[JAAS](https://www.baeldung.com/java-authentication-authorization-service)框架实现的示例：

```java
Subject subject = loginContext.getSubject();
PrivilegedAction privilegedAction = new ResourceAction();
Subject.doAsPrivileged(subject, privilegedAction, null);
```

## 3. 主体

身份验证成功后，我们有一个填充主体，其中包含许多关联身份，例如角色、社会安全号码(SSN)等。换句话说，这些标识符是主体，主体代表他们。

例如，一个人可能有一个帐号主体(“87654-3210”)和其他唯一标识符，以区别于其他主体。

让我们看看如何在成功登录后创建一个UserPrincipal并将其添加到一个Subject中：

```java
@Override
public boolean commit() throws LoginException {
    if (!loginSucceeded) {
        return false;
    }
    userPrincipal = new UserPrincipal(username);
    subject.getPrincipals().add(userPrincipal);
    return true;
}
```

## 4. 用户

通常，用户代表访问资源以执行某些操作或完成工作任务的人。

此外，我们可以将用户用作主体，另一方面，主体是分配给用户的身份。UserPrincipal是上一节中讨论的JAAS框架中用户的一个很好的例子。

## 5. 主题、主体、用户的区别

正如我们在上面的部分中看到的，我们可以使用主体来表示同一用户身份的不同方面。它们是主题的子集，而用户是指最终用户或交互操作员的主体的子集。

## 6. 总结

在本教程中，我们讨论了主题、主体和用户的定义，它们在大多数安全框架中都很常见。此外，我们展示了它们之间的区别。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-2)上获得。