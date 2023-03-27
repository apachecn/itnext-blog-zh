# Angular 应用程序的快速服务器第 5 部分:将非 www 请求重定向到 www 请求

> 原文：<https://itnext.io/express-server-for-an-angular-application-part-5-redirect-non-www-to-www-requests-b81a7794a4d7?source=collection_archive---------2----------------------->

Express.js 是为构建 web 应用程序而设计的 Node.js 的 web 应用程序框架，在这一系列文章中，我将一步一步地解释我是如何实现一个具有一些高级功能的 Express 服务器的，我将这些功能用于我用 Angular 7 制作的单页面应用程序。

![](img/b0c6f5d61fc008d9f6a4d58005cdf76c.png)

我将讨论我在不同文章中解决的 7 个主要问题:

1.  [入门](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)
2.  [从桶里取文件](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde)
3.  [将一些呼叫重定向到外部服务](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb)
4.  [将 http 重定向到 https 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)
5.  [将非 www 请求重定向到 www 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-redirect-non-www-to-www-requests-b81a7794a4d7)
6.  [限制连接并阻止 DDoS](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-limit-connections-and-prevent-ddos-e27902cc702b)
7.  [处理一些缓存](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-7-add-cache-f11e8060a2e3)

在第四篇文章中，我将讨论如何将 http 重定向到 https 请求。

# 参考知识库

该项目的一个工作示例可从以下网址获得:

[https://github . com/mzuccaroli/express _ server _ for _ angular _ example](https://github.com/mzuccaroli/express_server_for_angular_example)主分支包含最终的完整项目，但每篇文章都有一个专门的分支，对于这篇文章，引用是:[https://github . com/mzuccaroli/express _ server _ for _ angular _ example/tree/feature/non-www _ to _ www _ redirectes](https://github.com/mzuccaroli/express_server_for_angular_example/tree/feature/non-www_to_www_redirects)

# 关于环境

在本地开发过程中，Url 处理和操作可能会很棘手，就像本教程的前一部分“[如何将 http 请求重定向到 https](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93) ”一样，使用“_environment”变量将本地开发行为与生产行为分开是个好主意。更多[信息见本文](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)

# 重定向将非 www 请求重定向到 www 请求

为了 SEO 的目的，从一个唯一的 URL 为您的应用程序页面提供最佳服务，对您的应用程序服务器的一个很好的改进是非 www 请求的自动重定向。同样的方法也可以用于将 www 请求重定向到非 www 请求，这是您的选择。

让我们将这段代码添加到应用程序中的“app.all”声明之前:

正如你所看到的，代码非常清楚:express 拦截不包含“www”的请求，并将它们重定向到正确的 url。

像本教程的其他步骤一样，您可以使用

> $node server.js

去 [http://localhost:4100](http://localhost:4100) 看看结果。

我们创建一个好的 express 服务器的下一步是**通过对连接设置一些**限制来提高他的安全性**并防止 DDOS 攻击，这将在下一篇文章中讨论。**