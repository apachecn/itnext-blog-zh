# Node.js 应用程序中的请求 Id 跟踪

> 原文：<https://itnext.io/request-id-tracing-in-node-js-applications-c517c7dab62d?source=collection_archive---------0----------------------->

![](img/649c183a76b9895ceed81a6ae66170ff.png)

照片由[菲利普·布朗](https://unsplash.com/@nebirdsplus?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

如果您曾经在 Node.js 中编写过后端应用程序，您就会知道通过日志条目跟踪相同的 HTTP 请求是一个问题。通常您的日志看起来像这样:

```
[07/Nov/2018:15:48:11 +0000] User sign-up: starting request validation[07/Nov/2018:15:48:11 +0000] User sign-up: starting request validation[07/Nov/2018:15:48:12 +0000] User sign-up: request validation success[07/Nov/2018:15:48:13 +0000] User sign-up: request validation failed. Reason:
...
```

这里，日志条目混在一起，没有办法确定它们中的哪一个属于同一个请求。虽然您可能更喜欢看到这样的内容:

```
[07/Nov/2018:15:48:11 +0000] [request-id:550e8400-e29b-41d4-a716-446655440000] User sign-up: starting request validation[07/Nov/2018:15:48:11 +0000] [request-id:340b4357-c11d-31d4-b439-329584783999] User sign-up: starting request validation[07/Nov/2018:15:48:12 +0000] [request-id:550e8400-e29b-41d4-a716-446655440000] User sign-up: request validation success[07/Nov/2018:15:48:13 +0000] [request-id:340b4357-c11d-31d4-b439-329584783999] User sign-up: request validation failed. Reason:
...
```

注意这里包含请求标识符的`[request-id:*]`部分。这些标识符将允许您过滤属于同一请求的日志条目。此外，如果您的应用程序由通过 HTTP 相互通信的微服务组成，请求标识符可以在 HTTP 头中发送，并用于跟踪链上所有微服务生成的日志中的请求链。对于诊断和监控目的来说，很难过高估计这个特性的价值。

作为一名开发人员，您可能希望在一个地方配置 web 框架和/或日志库，并自动生成请求 id 并打印到日志中。但很遗憾，可能是 Node.js 世界的问题。

在这篇文章中，我们将讨论这个问题和一个可能的解决方案。

# 好吧，但这是个问题吗？

在许多其他语言和平台中，如 JVM 和 Java Servlet 容器，HTTP 服务器是围绕多线程架构和阻塞 I/O 构建的，这不是问题。如果我们将处理 HTTP 请求的线程的标识符放入日志中，它已经可以作为跟踪特定请求的自然过滤参数。这个解决方案远非理想，但是可以通过使用[线程本地存储](https://en.wikipedia.org/wiki/Thread-local_storage) (TLS)来进一步增强。TLS 基本上是一种在与当前线程相关的上下文中存储和检索键值对的方法。在我们的例子中，它可以用来存储为每个新请求生成的 id(和任何其他诊断数据)。许多日志库都有围绕 TLS 构建的特性。作为一个例子，查看文档了解 [SLF4J 的映射诊断上下文](https://logback.qos.ch/manual/mdc.html)。

由于 Node.js 基于[事件循环](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)的异步特性，根本没有线程本地存储，因为 js 代码是在单线程上处理的。如果没有这个或类似的 API，您将不得不在整个请求处理调用中拖动一个包含请求 id 的上下文对象。

让我们看看它在一个简单的 Express 应用程序中是什么样子的:

在这个假想的应用程序中，我们必须将`req`对象传递给`fakeDbAccess`函数，这样我们就能够将请求 id 输出到日志中。想想这种方法在实际应用中是多么的多余和容易出错，因为实际应用通常有更多的路径和模块。

幸运的是，Node.js 社区的人们很久以前就在考虑 TLS 的替代方案。其中最流行的一种方法叫做连续本地存储(CLS)。

# CLS 来救援了！

CLS 的第一个实现是[延续本地存储](https://github.com/othiym23/node-continuation-local-storage)库。它对 CLS 有如下定义:

> 延续本地存储的工作方式类似于线程编程中的线程本地存储，但它基于节点样式的回调链，而不是线程。

如果你检查这个库的 API，你可能会发现它比 Java 的 TLS 要复杂一些。但核心上，还是挺像的。它允许您将上下文对象与一系列调用相关联，并在以后获取它。

最初的库是基于使用在 Node.js v0.12 之前可用的`process.addAsyncListener` API 及其用于 node v0.12+的 [polyfill](https://github.com/othiym23/async-listener) 。polyfill 做了大量的猴子补丁，旨在包装内置的节点 API。这就是为什么不应该考虑在 Node.js 的现代版本中使用原始库的主要原因。

幸运的是，CLS 库还有一个后续版本(或者更准确地说是一个分叉)，叫做 [cls-hooked](https://github.com/jeff-lewis/cls-hooked) 。在 Node.js > = 8.2.1 中，它使用了 [async_hooks](https://nodejs.org/api/async_hooks.html) ，一个节点的内置 API。尽管 API 仍处于试验阶段，但这种方法比使用 polyfill 的方法要好得多。如果你想了解更多关于异步钩子 API 的知识，可以看看这篇文章。

现在，当我们有了合适的工具，我们知道如何处理我们最初的问题，即跟踪应用程序日志中的请求 id。

# 为我的 Express/Koa/other-web-framework 提供现成的解决方案怎么样？

正如您已经知道的，如果您希望在 Node.js 应用程序中包含请求 id，您可以使用 cls-hooked 并将其与您正在使用的 web 框架集成。但是可能你会想用一个库来做这些事情。

最近，我在寻找这样的库，但没有找到一个好的任务匹配。是的，有几个集成了 web 框架的库，比如 Express 和 CLS。另一方面，有些库提供请求 id 生成中间件。但是我没有找到一个库可以结合 CLS 和请求 id 来解决请求 id 跟踪问题。

所以，女士们先生们，请认识一下 [cls-rtracer](https://github.com/puzpuzpuz/cls-rtracer) ，一个解决一个不那么小的问题的小库。它为 Express 和 Koa 提供了中间件，实现了基于 CLS 的请求 id 生成，并允许在您的调用链上的任何地方获取请求 id。

与 cls-rtracer 的集成基本上需要两步。第一个——将中间件连接到适当的位置。第二步—配置您的日志库。

这是基于 Express 的应用程序的外观:

运行时，此应用程序会生成类似于以下内容的控制台输出:

```
2018-12-06T10:49:41.564Z: The app is listening on 30002018-12-06T10:49:49.018Z [request-id:f2fe1a9e-f107-4271-9e7a-e163f87cb2a5]: Starting request handling2018-12-06T10:49:49.020Z [request-id:f2fe1a9e-f107-4271-9e7a-e163f87cb2a5]: Logs from fakeDbAccess2018-12-06T10:49:53.773Z [request-id:cd3a33a9-32cb-453b-a0f0-e36c65ff411e]: Starting request handling2018-12-06T10:49:53.774Z [request-id:cd3a33a9-32cb-453b-a0f0-e36c65ff411e]: Logs from fakeDbAccess2018-12-06T10:49:54.908Z [request-id:8b352536-d714-4838-a372-a8e2cfcb4f53]: Starting request handling2018-12-06T10:49:54.910Z [request-id:8b352536-d714-4838-a372-a8e2cfcb4f53]: Logs from fakeDbAccess
```

注意，集成本身包括附加由`rTracer.expressMiddleware()`函数调用生成的 Express 中间件，并通过`rTracer.id()`调用获得请求 id。所以，再简单不过了。你也可以在这里找到 Koa 应用[的例子。](https://github.com/puzpuzpuz/cls-rtracer/tree/master/examples)

希望 cls-rtracer 将帮助您解决请求 id 跟踪问题，并使诊断 Node.js 应用程序变得更加容易。

我必须注意到使用异步钩子会对性能产生一定的影响。但是，对于大多数应用程序来说，这并不重要。查看[此评论](https://medium.com/@marek.kajda/hi-andrey-e3d04ec1a87e)和后续(感谢[马雷克·卡伊达](https://medium.com/u/a5985fe05072?source=post_page-----c517c7dab62d--------------------------------)！).

P.P.S .随时请求对其他 web 框架的支持并报告发现的问题。