# 为什么第三方 cookies 没有发送到您认为应该发送的地方

> 原文：<https://itnext.io/why-third-party-cookies-are-not-sent-where-you-think-they-should-1ebc60f4f314?source=collection_archive---------1----------------------->

![](img/a363179f60f7fcfaef09d99890988fcb.png)

# 什么是第三方 cookies

cookie 与一个域相关联。如果该域与您所在页面的域相同，则该 cookie 称为第一方 cookie。如果域不同，则它是第三方 cookie。

你可以按照以下步骤在 chrome 上查看第一方 cookies:打开开发控制台->`Application Tab`->-`Storage`->-`Cookies`。

# 我们需要第三方 cookies 的环境

使用 CORS 的跨来源请求。

# `OPTIONS`请求

CORS 预检请求是一种 CORS 请求，它使用特定的方法和报头检查服务器是否理解 CORS 协议。

这是一个 OPTIONS 请求，使用三个 HTTP 请求头:Access-Control-Request-Method、Access-Control-Request-Headers 和 Origin 头。

正常情况下，预检请求由浏览器自动发出。当请求被限定为“待预检”时出现，对于简单请求则省略。

# Cookies 属性

# httpOnly

使用`HttpOnly`属性来防止通过 JavaScript 访问 cookie 值。这是防止 XSS 袭击的一种方式。

# 领域

`Domain`属性指定允许哪些主机接收 cookie。如果未指定，则默认为设置 cookie 的相同来源，不包括子域。如果指定了域，则始终包括子域。

# 安全的

Cookies 只设置在 https 上。

# 同一地点

`SameSite`属性让服务器指定 cookies 是否/何时与跨来源请求一起发送(其中站点由可注册域定义)，这提供了一些针对跨站点请求伪造攻击的保护(CSRF)。

它有三个可能的值:严格、宽松和无。使用`Strict`，cookie 只被发送到产生它的同一个站点；`Lax`是类似的，除了当用户导航到 cookie 的源站点时发送 cookie，例如，通过跟随来自外部站点的链接；`None`指定在发起和跨站点请求上发送 cookies，但仅在安全上下文中发送(即，如果 SameSite=None，则还必须设置安全属性)。

# 发送第三方 cookies 的要求

1.  浏览器设置应支持设置第三方 cookies。对于 Chrome，通过路径检查设置:设置->隐私和安全-> cookie 和其他网站数据->允许所有 cookie
2.  服务器在`Set-Cookie`响应头中正确设置`Same-Site`属性，
3.  在`OPTIONS`预点火请求中设置`Access-Control-Allow-Credentails: true`。
4.  客户端应该在请求中显式包含凭据。

# 取得

```
fetch('https://example.com', {
  credentials: 'include' | 'same-origin' | 'omit'
});
```

# Axios

```
axios.defaults.withCredentials = true;
```

# 通知；注意

*   如果您想关注我的博客系列的最新消息/文章，请点击[【观看】](https://github.com/n0ruSh/blogs/)订阅