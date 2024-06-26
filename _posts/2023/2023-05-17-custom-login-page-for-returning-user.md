---
layout: post
title:  返回用户的自定义登录页面
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 简介

本文是我们正在进行的[Spring Security系列注册](https://www.baeldung.com/spring-security-registration)的续篇。

**在本文中，我们将了解如何为返回到我们应用程序的用户开发自定义登录页面**。用户将收到标准的“Welcome...”消息。

## 2. 添加一个长寿命的Cookie

**识别用户是否返回我们网站的一种方法是在用户成功登录后添加一个长期存在的cookie(例如30天)**。为了开发这个逻辑，我们需要实现我们的AuthenticationSuccessHandler，它会在身份验证成功后添加cookie。

让我们创建自定义MyCustomLoginAuthenticationSuccessHandler并实现onAuthenticationSuccess()方法：

```java
@Override
public void onAuthenticationSuccess(final HttpServletRequest request, final HttpServletResponse response, final Authentication authentication) throws IOException {
    addWelcomeCookie(gerUserName(authentication), response);
    redirectStrategy.sendRedirect(request, response, "/homepage.html?user=" + authentication.getName());
}
```

这里的重点是调用addWelcomeCookie()方法。

现在，让我们看一下添加cookie的代码：

```java
private String gerUserName(Authentication authentication) {
    return ((User) authentication.getPrincipal()).getFirstName();
}

private void addWelcomeCookie(String user, HttpServletResponse response) {
    Cookie welcomeCookie = getWelcomeCookie(user);
    response.addCookie(welcomeCookie);
}

private Cookie getWelcomeCookie(String user) {
    Cookie welcomeCookie = new Cookie("welcome", user);
    welcomeCookie.setMaxAge(60 * 60 * 24 * 30);
    return welcomeCookie;
}
```

我们设置了一个cookie，其键为“welcome”，值为当前用户的firstName。cookie设置为在30天后过期。

## 3. 在登录表单上读取Cookie

最后一步是在登录表单加载时读取cookie，如果存在，则获取显示问候消息的值。我们可以使用Javascript轻松做到这一点。

首先，让我们添加占位符“welcometext”以在登录页面上显示我们的消息：

```html
<form name='f' action="login" method='POST' onsubmit="return validate();">
    <span id="welcometext"> </span>

    <br /><br />
    <label class="col-sm-4" th:text="#{label.form.loginEmail}">Email</label>
    <span class="col-sm-8">
      <input class="form-control" type='text' name='username' value=''/>
    </span>
    ...
</form>
```

现在，让我们看看相应的Javascript：

```javascript
function getCookie(name) {
    return document.cookie.split('; ').reduce((r, v) => {
        const parts = v.split('=')
        return parts[0] === name ? decodeURIComponent(parts[1]) : r
    }, '')
}
    
function display_username() {
    var username = getCookie('welcome');
    if (username) {
        document.getElementById("welcometext").innerHTML = "Welcome " + username + "!";
    }
}
```

第一个函数只是读取用户登录时设置的cookie。第二个函数操作HTML文档以在存在cookie时设置欢迎消息。

函数display_username()在HTML <body\>标签的onload事件上被调用：

```html
<body onload="display_username()">
```

## 4. 总结

在这篇快速文章中，我们看到了通过修改Spring中的默认身份验证流程来自定义用户体验是多么简单。基于这个简单的设置可以完成很多复杂的事情。

此示例的登录页面可以通过/customLogin URL访问。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules)上获得。