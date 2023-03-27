# Angular 应用程序的 Express server 第 3 部分:将路由重定向到外部服务

> 原文：<https://itnext.io/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb?source=collection_archive---------3----------------------->

Express.js 是为构建 web 应用程序而设计的 Node.js 的 web 应用程序框架，在这一系列文章中，我将一步一步地解释我是如何实现一个具有一些高级功能的 Express 服务器的，我将这些功能用于我用 Angular 7 制作的单页面应用程序。

![](img/47a52539feb1d29dc5a9e3865f9905e8.png)

我将讨论我在不同文章中解决的 7 个主要问题:

1.  [入门](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)
2.  [从桶里取文件](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde)
3.  [将一些呼叫重定向到外部服务](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb)
4.  [将 http 重定向到 https 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)
5.  [将非 www 请求重定向到 www 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-redirect-non-www-to-www-requests-b81a7794a4d7)
6.  [限制连接并阻止 DDoS](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-limit-connections-and-prevent-ddos-e27902cc702b)
7.  [处理一些缓存](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-7-add-cache-f11e8060a2e3)

在第三篇文章中，我将讨论如何处理一些特殊的路由，并将它们重定向到外部服务。

# 参考知识库

该项目的一个工作示例可从以下网址获得:

[https://github . com/mzuccaroli/express _ server _ for _ angular _ example](https://github.com/mzuccaroli/express_server_for_angular_example)主分支包含最终的完整项目，但每篇文章都有一个专门的分支，这一分支的参考是:

[https://github . com/mzuccaroli/express _ server _ for _ angular _ example/tree/feature/redirect _ special _ URLs](https://github.com/mzuccaroli/express_server_for_angular_example/tree/feature/redirect_special_urls)

# 这个例子

一个经典的例子，也是我在我的服务器上添加这个特性的原因，是 **sitemap.xml 文件**。如果你的前端应用程序管理一些 **CMS 特性**并从 API 中检索内容，假设站点地图是由后端生成是完全合理的。这个问题有很多解决方案，比如由 Angular 解析的专用 api，但是最轻量级的解决方案是**将对站点地图的请求代理到你的后端**。

让我们假设您的后端服务器在一个已知的路径上提供了站点地图，并将这个变量添加到您的代码中:

> const _ API address = ' https://testapi/API/example/sitemap '；

只要对代码稍加改进，您就可以服务于这个文件。

# 代理外部路由

在声明“app”变量后，添加:

这个简单的代码片段**处理对 xml 文件**的所有请求，并通过管道传递外部调用的结果来服务它们。

这是一个**简单的实现**，但是你可以为更复杂的行为改变你的正则表达式或者后端调用。

# 测试一下

使用运行您的应用程序

> $node server.js

并转到[http://localhost:4100](http://localhost:4100)/sitemap . XML 和 express 将服务外部服务的结果[https://testapi/API/example/sitemap。](https://testapi/api/example/sitemap.)

在下一篇文章中，我将讨论 http 和 https 请求以及如何管理它们。