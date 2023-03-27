# Angular 应用程序的快速服务器第 1 部分:入门

> 原文：<https://itnext.io/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?source=collection_archive---------2----------------------->

Express.js 是为构建 web 应用程序而设计的 Node.js 的 web 应用程序框架，在这一系列文章中，我将一步一步地解释我是如何实现一个具有一些高级功能的 Express 服务器的，我将这些功能用于我用 Angular 7 制作的单页面应用程序。

![](img/7421c63c5d885fe59a01894e02f49c81.png)

我将讨论我在不同文章中解决的 7 个主要问题:

1.  [入门](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)
2.  [从桶里取文件](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde)
3.  [将一些呼叫重定向到外部服务](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb)
4.  [将 http 重定向到 https 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)
5.  [将非 www 请求重定向到 www 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-redirect-non-www-to-www-requests-b81a7794a4d7)
6.  [限制连接并阻止 DDoS](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-limit-connections-and-prevent-ddos-e27902cc702b)
7.  [处理一些缓存](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-7-add-cache-f11e8060a2e3)

在这第一篇文章中，我将讨论一个**超小型设置，以便开始使用 express** 。

# 参考知识库

该项目的一个工作示例可从以下网址获得:

[https://github . com/mzuccaroli/express _ server _ for _ angular _ example](https://github.com/mzuccaroli/express_server_for_angular_example)主分支包含最终的完整项目，但每篇文章都有一个专门的分支，这一篇的参考文献是:[https://github . com/mzuccaroli/express _ server _ for _ angular _ example/tree/feature/getting _ started](https://github.com/mzuccaroli/express_server_for_angular_example/tree/feature/getting_started)

# 入门指南

让我们从一个最小实现开始:一个用于 angular 应用程序的超级简单的 express 服务器，它提供来自本地文件夹的文件。

看一下代码:

如您所见，代码非常简单:导入后，您可以声明变量，如服务器的端口和包含应用程序的文件夹，然后看看两个主要行为。

# 静态文件

> app.server.get('*。*、express.static(_app_folder，{ maxAge:' 1y ' })；

这是用于处理静态文件的服务器的最简单实现:包含点 *'*的每条路线。* "*、类似于 *"/style.css"* 、 *"/favicon.ico"* 或 *"/assets/logo.png"* 的东西，会在 *dist/application* 文件夹中搜索相同的资源。

例如请求*"*[*http://localhost:4100/assets/logo . png*](http://localhost:4100/assets/logo.png)*"*将为用户提供*。/dist/application/assets/logo . png*文件如果存在。

# 角度应用

第二个主要行为是处理**所有其他请求**的行为，由*“app . all(' *)”*标识，这些请求将被重定向到 dist 文件夹的主文件，换句话说**所有不包含点的请求将由我们的 Angular 应用程序及其内部路由**处理。

您可以测试代码，只需将您的 angular 应用程序放在 server.js 文件的同一个文件夹中名为“dist”的文件夹中，并运行
$node server.js

现在你可以打开你的浏览器，进入 http://localhost:4100 ，测试你的静态文件和角度路径。

**这是第一步**，它将引导你为 angular 应用程序构建一个 express 服务器。您可以从参考资源库下载工作示例:[https://github . com/mzuccaroli/express _ server _ for _ angular _ example/tree/feature/getting _ started](https://github.com/mzuccaroli/express_server_for_angular_example/tree/feature/getting_started)

下一步是做一些修改，让您**在外部存储桶**上托管应用程序和资产。[点击此处查看**如何从铲斗**进行角度应用。](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde)