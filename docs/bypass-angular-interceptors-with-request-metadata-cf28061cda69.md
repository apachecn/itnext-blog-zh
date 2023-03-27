# 用请求元数据绕过角度拦截器

> 原文：<https://itnext.io/bypass-angular-interceptors-with-request-metadata-cf28061cda69?source=collection_archive---------0----------------------->

![](img/9e7623399f33e95cd0344afe6062fc17.png)

Angular 有一种非常强大的方式为应用程序发出的每个 http 请求提供公共逻辑。

使用`[HttpInterceptor](https://angular.io/guide/http#intercepting-requests-and-responses)`你可以添加标题、记录日志、处理错误……基本上任何你想对所有**http 请求做的事情。**

但是有时您想绕过拦截器来处理某些请求。到目前为止，您有几种方法可以做到这一点。

*   在请求中添加一些 http 头，并在拦截器中检查它。
*   为您的请求创建一个新的 HttpClient。
*   用令牌管理 HttpClient 注入。

这些都是试图让您的拦截器知道您的请求有一些需要区别对待的特殊情况的方法。

# HttpContext

有了 [Angular 12](https://blog.angular.io/angular-v12-is-now-available-32ed51fbfd49) ，有了一种新的方式将信息传递给你的拦截器，这看起来不像是你羞于做的事情。=)

现在可以使用 [HttpContext](https://angular.io/guide/http#passing-metadata-to-interceptors) 将元数据从请求传递给拦截器。

这是您的拦截器:

这是你的`httpClient`请求:

注意事项:

*   你可以在你觉得更有意义的地方声明你的`HttpContextToken`。
*   这个令牌可以是任何类型的数据。布尔值，数字，字符串，对象，数组...
*   这个上下文是可变的。因此您可以在请求/响应周期中更新它的值。

看到这终于在 Angular 实现了，我很激动！🎉

非常感谢每一个努力工作并且没有失去希望的人，让这个进入框架！

# 链接

 [## 有角的

### Angular 是一个构建移动和桌面 web 应用程序的平台。加入数百万开发者的社区…

angular.io](https://angular.io/guide/http#intercepting-requests-and-responses) [](https://github.com/angular/angular/pull/25751) [## feat(common): http 客户端请求元数据，用于 FDIM 拉请求#25751 的拦截器中

### 公关清单请检查您的公关是否满足以下要求:提交信息遵循我们的指导方针…

github.com](https://github.com/angular/angular/pull/25751)