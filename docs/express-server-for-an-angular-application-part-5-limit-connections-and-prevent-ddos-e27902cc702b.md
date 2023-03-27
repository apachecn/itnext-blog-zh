# 角度应用的快速服务器第 6 部分:限制连接和防止 DDoS

> 原文：<https://itnext.io/express-server-for-an-angular-application-part-5-limit-connections-and-prevent-ddos-e27902cc702b?source=collection_archive---------2----------------------->

Express.js 是为构建 web 应用程序而设计的 Node.js 的 web 应用程序框架，在这一系列文章中，我将一步一步地解释我是如何实现一个具有一些高级功能的 Express 服务器的，我将这些功能用于我用 Angular 7 制作的单页面应用程序。

![](img/c813521678c8b61a54e1fa756e718898.png)

我将讨论我在不同文章中解决的 7 个主要问题:

1.  [入门](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)
2.  [从桶里取文件](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde)
3.  [将一些呼叫重定向到外部服务](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb)
4.  [将 http 重定向到 https 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)
5.  [将非 www 请求重定向到 www 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-redirect-non-www-to-www-requests-b81a7794a4d7)
6.  [限制连接并阻止 DDoS](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-limit-connections-and-prevent-ddos-e27902cc702b)
7.  [处理一些缓存](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-7-add-cache-f11e8060a2e3)

在这篇文章中，我将讨论如何在短时间内限制单个 ip 的连接。

# 参考知识库

该项目的一个工作示例可从以下网址获得:

[https://github . com/mzuccaroli/express _ server _ for _ angular _ example](https://github.com/mzuccaroli/express_server_for_angular_example)主分支包含最终的完整项目，但每篇文章都有一个专用的分支，对于这篇文章的引用[https://github . com/mzuccaroli/express _ server _ for _ angular _ example/tree/feature/limiter](https://github.com/mzuccaroli/express_server_for_angular_example/tree/feature/limiter)

# 关于环境

对于这个特性，就像在我选择为开发和生产引入不同行为之前开发的特性一样，就像在本教程的前一部分“[如何将 http 请求重定向到 https](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93) ”中一样，我使用“_environment”变量将本地开发行为与生产行为分开。更多[信息见文章](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)

# 为什么要限制连接

**限制单个 ip 地址在短时间内连接的数量绝对是个好主意。**它防止服务器出现故障和不必要的过载，并提供防御 DDos 攻击的保护。Express 提供了一个非常易于使用的库，可以轻松完成工作。

# 如何限制连接

让我们从安装 express-rate-limit 开始，并将其添加到我们的 package.json 中

> $ npm 安装—节省快速费率限制

现在，您可以将“app”声明右侧的以下部分添加到代码中:

这就是启用限制所需的全部内容，您可以通过编辑用于更改时间量的“windowMs”值和用于设置连接限制的“max”值来调整限制器设置。这是对你的服务器的一个非常简单的修改，但是提供了一个很大的改进。

创建一个好的 express 服务器的下一步，也是最后一步，将是**添加一些缓存函数**来提高应用程序的速度，这将在下一篇文章中讨论。