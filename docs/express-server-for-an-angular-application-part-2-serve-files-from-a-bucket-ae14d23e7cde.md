# Angular 应用程序的快速服务器第 2 部分:从存储桶中提供文件

> 原文：<https://itnext.io/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde?source=collection_archive---------8----------------------->

Express.js 是为构建 web 应用程序而设计的 Node.js 的 web 应用程序框架，在这一系列文章中，我将一步一步地解释我是如何实现一个具有一些高级功能的 Express 服务器的，我将这些功能用于我用 Angular 7 制作的单页面应用程序。

![](img/3a494ff28fbeddfeb05488cd2505b1ed.png)

我将讨论我在不同文章中解决的 7 个主要问题:

1.  [入门](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)
2.  [从桶里取文件](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-2-serve-files-from-a-bucket-ae14d23e7cde)
3.  [将一些呼叫重定向到外部服务](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb)
4.  [将 http 重定向到 https 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-4-redirect-http-to-https-requests-d4f6e7ed7e93)
5.  [将非 www 请求重定向到 www 请求](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-redirect-non-www-to-www-requests-b81a7794a4d7)
6.  [限制连接并阻止 DDoS](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-5-limit-connections-and-prevent-ddos-e27902cc702b)
7.  [处理一些缓存](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-7-add-cache-f11e8060a2e3)

在第二篇文章中，我将讨论**如何提供托管在外部存储桶**上的文件。

# 参考知识库

该项目的一个工作示例可从以下网址获得:

[https://github . com/mzuccaroli/express _ server _ for _ angular _ example](https://github.com/mzuccaroli/express_server_for_angular_example)主分支包含最终的完整项目，但每篇文章都有一个专门的分支，对于这篇文章，引用是:[https://github . com/mzuccaroli/express _ server _ for _ angular _ example/tree/feature/angular _ from _ bucket](https://github.com/mzuccaroli/express_server_for_angular_example/tree/feature/angular_from_bucket)

# 为什么你应该使用水桶

当你在处理一个 **javascript 单页应用**时，比如 Angular、**，把所有静态文件放在一个专用的存储库**里是一个好主意，比如一个**亚马逊 s3 存储桶**。通过对你的[基本快速服务器](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)做一些小的修改，你将能够**从 angular 应用程序中分离出正在运行的服务器。通过这种方式，您将能够利用这两种解决方案的优点:在专用环境中运行一个小型的超级优化的 express 服务器，并在一个更便宜的桶中托管静态 js 文件和资产。**

让我们假设您的应用程序版本的副本部署在'[http://static . test . com . S3-website-eu-west-1 . Amazon AWS . com '](http://static.test.com.s3-website-eu-west-1.amazonaws.com')。你可以从我的[上一篇文章](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-1-getting-started-2cd27de691bd?fbclid=IwAR0FqPoqKeBps4QUM3E09aSGKDwjUNKcoZHxPKAB5iBt8dwUbuEol3b7IZ8)中创建的基本服务器开始，在代码中做一些修改。

首先，您可以将您的桶地址存储在一个变量中:

> const _ bucket address = '[http://static . test . com . S3-website-eu-west-1 . Amazon AWS . com '](http://static.test.com.s3-website-eu-west-1.amazonaws.com')；

# 从桶中提供静态文件

在您的代码中找到静态文件处理部分，并将其更改为:

正如您所看到的，这段代码将所有静态文件请求重定向到 bucket，充当了一个**反向代理**，您可以通过**在路径的正则表达式中识别特定的文件扩展名**来增加一点改进。通过这种方式，您可以直接从您的 bucket 中控制您想要提供哪些静态文件。

# 从存储桶提供应用程序路径

找到应用程序路径代码，并将其更改为:

主要的应用程序修改非常简单，请注意**如果您需要的话，您可以添加高级 url 处理**:一个典型的例子可能是在专用的 PAH 上提供不同的特定语言版本，如果您想了解更多关于使用 Angular 的多语言应用程序的信息，您可以查看本文

现在，您可以将您的角度应用程序放入您的桶中并运行

> $node server.js

打开你的浏览器进入[*http://localhost:4100*](http://localhost:4100)如你所见，应用程序运行良好。

请注意，这个小小的修改是对您的应用程序的**大改进:**

*   服务器代码和应用程序代码完全分离，可以使用不同的方法和管道进行部署
*   你的 express 服务器是一个小的 javascript 应用程序，很容易变成一个λ函数如果你使用的是 Amazon 环境，**这是无服务器方法的第一步**
*   服务器和应用程序的**解耦**使您能够根据需要添加一些**高级功能**:您的服务器可以处理不同存储桶中的多个应用程序，或者同一应用程序可以由不同环境中的不同服务器提供服务

快速旅程的下一步是处理一些特殊的路由，并将它们重定向到外部服务器:[将一些调用重定向到外部服务](https://medium.com/@marcozuccaroli/express-server-for-an-angular-application-part-3-redirect-routes-to-an-external-service-d4658ed575cb)