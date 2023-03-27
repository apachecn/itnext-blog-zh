# 解释了使用 Dotnet 核心的跨来源请求

> 原文：<https://itnext.io/cross-origin-requests-with-dotnet-core-explained-e7284e1ec083?source=collection_archive---------0----------------------->

![](img/d63461b9fdf1a86236f0b8171ff18e72.png)

照片由[蒂姆·高](https://unsplash.com/@punttim?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/frustrated?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

跨源请求是 HTTP 规范的一部分，由于让两个网站相互对话看起来非常复杂，这使得许多开发人员达到了理智的极限。原则上，CORS 的存在实际上是为了让这变得更容易:它允许选择退出同源政策。这意味着如果没有 CORS，对另一个域进行 API 调用是绝对不可能的。

所以，显然 CORS 是来帮忙的，那又怎么样呢？CORS 规范允许**服务器**设置报头，该报头指示同源策略的一部分不应被实施。然而，这种实施发生在浏览器中，因此在**客户端**侧。这无疑是 CORS 最大的困惑来源:许多人要么试图修复 JavaScript 中的 CORS 错误(这没什么可说的)，要么试图找出他们后端的哪一部分出现了故障(这并没有导致故障)。在本文中，我将展示 CORS 是如何工作的，以及您需要做些什么来实现跨域请求。

# 预检和 HTTP 选项

CORS 请求分为两类:简单请求和非简单请求。对于非简单请求，浏览器将发出预检请求，询问服务器是否允许主请求。对于简单的请求，浏览器只是继续请求，然后拒绝调用。关于什么是简单请求，什么不是的完整定义可以在关于这个主题的优秀的 [MDN 页面](https://developer.mozilla.org/nl/docs/Web/HTTP/CORS#simple_requests)上找到。

服务器通过发送`Access-Control-Allow-*`头允许来自其他域的访问。最重要的一个是`Access-Control-Allow-Origin`头，它允许从其他域访问 URL，但是还有更多访问控制头。当一个预检发生时，你也会看到`Access-Control-Request-*`标题，用来告诉服务器一个客户想要哪个访问控制。这些是客户端发送的头，但是你永远不需要自己设置它们，它们也不会授予你实际的访问权限。

以下示例显示了 Dotnet 核心中的一个简单实现。Dotnet 有更好的方法来配置 CORS，而不是直接在控制器中设置头，但是我想首先向您展示最基本的底层实现，而不涉及很多框架魔法。

CORS 请求的简单工作示例

在此示例中，会发生以下情况:

*   浏览器向服务器发出`HTTP GET`请求。
*   服务器发回响应(正常情况下，头后面跟正文)。
*   标题一出现在浏览器中，就会用网站的当前来源验证`Access-Control-Allow-Origin`标题的值，如果不匹配，就会拒绝响应。

下一个示例显示了一个正在进行飞行前 CORS 请求。注意这里我们有两个动作:一个用于`HTTP OPTIONS`起飞前呼叫，一个用于`HTTP GET`‘正常’呼叫。

要求使用非标准标题进行飞行前检查的请求示例

在本例中，由于自定义标题`myheader`，浏览器需要发出预检请求，因此还会发生一些事情:

*   浏览器向服务器发出`HTTP OPTIONS`请求，以验证该请求是否被允许。在这个请求中，它将包括两个用于访问控制的报头:`Access-Control-Request-Headers: myheader`和`Access-Control-Request-Method: GET`。
*   浏览器用`Access-Control-Allow`头响应浏览器作为`Access-Control-Request`头发送的内容:`Access-Control-Allow-Headers: myheader`和`Access-Control-Allow-Origin: http://localhost:8080`。我们不必为该方法发送 allow 头，因为默认情况下已经允许了一个`GET`。如果授予的访问控制与请求的不匹配，浏览器将拒绝 API 调用。
*   然而，如果它们匹配，浏览器将发出常规的`HTTP GET`请求。
*   标题一出现在浏览器中，就会用网站的当前来源验证`Access-Control-Allow-Origin`的值，如果不匹配，就会拒绝响应。注意，它只是再次验证原点，而不是任何其他`Access-Control-Allow`头。

# 网络核心 CORS 政策

如前所述，有更好的方法来管理 Dotnet Core 中的 CORS，而不是到处手动设置标头。如链接文档中所述，有多种方法可以实现 CORS。我推荐的方法是使用带有动作属性的命名策略。通过这种方式，您可以将所有配置放在一个地方，但是您仍然有一个可以从其他域访问的终端的显式白名单。它还允许您为不同的域配置不同的配置。在我看来，这是安全性和便利性的最佳结合。

[](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-5.0) [## 在 ASP.NET 核心中启用跨来源请求(CORS)

### Rick Anderson 和 Kirk Larkin 撰写的这篇文章展示了如何在 ASP.NET 核心应用中启用 CORS。浏览器安全性…

docs.microsoft.com](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-5.0) 

我们需要做的第一件事是在应用程序的`Startup`中添加一个命名策略。除此之外，我们还需要在`app`上调用`UseCors`。策略的名称在常量中定义，因此我们可以在其他地方使用它来引用我们创建的策略:

为了使用这个策略，我们可以用`EnableCors`属性来注释一个动作(或者一个控制器)。当使用命名策略时，我们需要向它提供策略的名称，否则它会试图找到一个默认策略(我们没有设置)。

# 结论

我希望这篇文章能让你了解 CORS 是如何工作的，当你看到控制台中出现`‘Access to fetch at…’`错误时，你应该去哪里找，并向你展示如何在一个. net 核心应用程序中正确地配置 CORS。