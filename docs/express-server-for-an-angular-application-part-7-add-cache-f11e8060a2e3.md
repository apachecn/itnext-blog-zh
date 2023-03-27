# Angular 应用程序的快速服务器第 7 部分:添加缓存

> 原文：<https://itnext.io/express-server-for-an-angular-application-part-7-add-cache-f11e8060a2e3?source=collection_archive---------2----------------------->

Express.js 是为构建 web 应用程序而设计的 Node.js 的 web 应用程序框架，在这一系列文章中，我将一步一步地解释我是如何实现一个具有一些高级功能的 Express 服务器的，我将这些功能用于我用 Angular 7 制作的单页面应用程序。

![](img/3fe8d6ab4e60d998c6ff732dbbf122ca.png)

我将讨论我在不同文章中解决的 7 个主要问题:

1.  [入门](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)
2.  [从桶里取文件](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde)
3.  [将一些呼叫重定向到外部服务](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb)
4.  [将 http 重定向到 https 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)
5.  [将非 www 请求重定向到 www 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-redirect-non-www-to-www-requests-b81a7794a4d7)
6.  [限制连接并阻止 DDoS](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-limit-connections-and-prevent-ddos-e27902cc702b)
7.  [处理一些缓存](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-7-add-cache-f11e8060a2e3)

在本文中，我将讨论如何给 express server 添加一个缓存层。

# 参考知识库

该项目的一个工作示例可从以下网址获得:

[https://github . com/mzuccaroli/express _ server _ for _ angular _ example](https://github.com/mzuccaroli/express_server_for_angular_example)主分支包含最终的完整项目，但每篇文章都有一个专用的分支，对于这篇文章，引用[https://github . com/mzuccaroli/express _ server _ for _ angular _ example/tree/develop](https://github.com/mzuccaroli/express_server_for_angular_example/tree/develop)

# 关于环境

对于这个特性，就像在我选择为开发和生产引入不同行为之前开发的特性一样，就像在本教程的前一部分“[如何将 http 请求重定向到 https](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93) ”中一样，我使用“_environment”变量将本地开发行为与生产行为分开。更多[信息见文章](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)

# 为什么缓存

根据定义，Angular 应用程序是一组静态文件，其中一些文件，如字体、图像和主 js，可能会很大，因此 express 服务器的主要工作是不断地获取这组有限的文件，并将它们提供给用户。
**这是添加一个小缓存层**的理想情况，它将大大提高你的服务器性能。

# 添加缓存层

主要思想是为桶获取的请求执行内存缓存，这些文件在数量上是有限的并且是静态的，所以我们不需要担心大量的内存使用。在 nodejs 中执行它而不添加复杂层的工具是*节点缓存*，它将在键值存储中缓存您的响应对象。

首先安装节点缓存

> npm 安装内存-缓存-保存

然后将其导入您的服务器:

现在，您可以添加缓存处理函数:

正如您可以看到的，这两个函数主要是在内存缓存中放置和检索对象，getCache 函数是根据快速调用链编写的，因此您可以在每次处理这样的请求时调用它:

注意“cache.getcache()”函数是 app.all()函数的一个参数，而 cache.handleResponse()是在第二个函数中显式调用的，因此，如果所需的对象在缓存中，则直接由 getcache 函数提供，否则，如果没有缓存的对象，handleResponse 函数将从桶中检索它，放入内存缓存中，并将其提供给主函数。

您可以在所有处理程序中添加缓存方法，并大幅提高服务器性能。你可以在这里看到完整的工作示例:[https://github . com/mzuccaroli/express _ server _ for _ angular _ example](https://github.com/mzuccaroli/express_server_for_angular_example)

# 关于字体文件的说明

本文和参考示例**的灵感来自于一个在生产环境**中运行的真实项目，因此在**压力测试**阶段，我注意到了字体文件的一个问题:在某些情况下，缓存的字体文件被损坏，我未能理解这个问题的真正原因，所以我将它们从缓存中排除，对 handleresponse()函数做了如下小修改:

它并不完美，但工作得很好，正如你所看到的**这不是“hello world”**演示，而是在生产中运行的真实代码，如果将来我会解决字体问题，这篇文章将会更新，如果你找到了解决方案，请写信给我！