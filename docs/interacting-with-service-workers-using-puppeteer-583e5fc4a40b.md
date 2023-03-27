# 使用木偶师与服务人员互动

> 原文：<https://itnext.io/interacting-with-service-workers-using-puppeteer-583e5fc4a40b?source=collection_archive---------4----------------------->

![](img/78970542e9f0072ebe071fa7ce1393d7.png)

> **免责声明:**这篇文章假设你已经有了你最喜欢的测试者(我建议 Jest)的木偶师设置，并且你了解服务人员。

在为使用 SWs 的 [web 应用](https://github.com/humphd/next)编写集成测试时，我在 Jest 和 Puppeteer 的设置上遇到了问题。显然，目前木偶戏还没有与 SWs 交互的 API。参见[特征请求](https://github.com/GoogleChrome/puppeteer/issues/2634)和本[问题](https://github.com/GoogleChrome/puppeteer/issues/1193)。

应用:

*   您想要验证从 SW 绕过 UI 返回的内容。
*   您希望向 SW 发送非 GET 请求。目前，实现这一点的唯一方法是修改请求头，但是它关闭了软件请求拦截。

# 解决办法

解决方案以一点点“黑”的形式出现。有许多 puppeteer 函数允许我们在页面的上下文中运行 js 代码。看到他们[这里](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageemulateoptions)。

我们对`evaluate`方法特别感兴趣。它以一个`Promise`的形式返回在页面上执行的操作的结果，该结果用来自页面的数据进行解析。

让我们看一个例子，看看如何使用`fetch`接口直接从页面发出不同类型的请求。

> **注意:**节点中没有对`fetch`的原生支持。然而，也有类似的努力复制`fetch`的行为。

那么上面发生了什么呢？

1.  声明接收要传递给`fetch`的`url`和`config` ( [获取 API](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch) )的方法。
2.  从将解析为`fetch`结果的`evaluate`返回`Promise`。
3.  传递所有应该在`evaluate`的上下文中可用的参数。
4.  用步骤`3`中提供的参数列表声明要由`evaluate`执行的方法。
5.  运行`fetch`并返回`Promise`结果。

现在我们有了基础，让我们进入 SWs。任何与他们打过交道的人都会告诉你，关于工人，你需要知道的最重要的事情之一就是他们的生命周期。我不打算详细说明它是如何工作的，我只想说，在 SW 不活动之前，所有对它的请求都将返回 404。因此，在运行任何测试之前，我们需要确保页面已经准备好。

我们通过在页面上下文中注册侦听器来实现这一点，当所有 SW 都处于活动状态时，该上下文将进行解析。下面是一个片段:

1.  我使用 [getRegistrations](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/getRegistrations) 方法来获取与域相关的所有注册。
2.  遍历所有注册，将所有内容合并到一个`Promise`中，一旦所有 SW 都是活动的，这个【】就会被解析。
3.  检查当前软件注册是否已激活，如果是—解决，如果否—监听 [onupdatefound](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/onupdatefound) 事件，并让浏览器安装软件。

# 就是这样！🎉

我们现在需要做的就是在`beforeAll`中调用`waitForServiceWorkers`或者在你的框架中调用它的等价物！

*发现错误？请在评论中让我知道！*

*原载于*[*gist.github.com*](https://gist.github.com/cbc98054e837f365fad7e3f7f0538aa2)*。*