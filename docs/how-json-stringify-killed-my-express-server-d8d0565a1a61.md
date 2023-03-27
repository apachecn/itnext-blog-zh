# JSON.stringify 如何杀死我的 express 服务器

> 原文：<https://itnext.io/how-json-stringify-killed-my-express-server-d8d0565a1a61?source=collection_archive---------1----------------------->

## *通过简单的改变，从 express 服务器获得高达 300%的性能提升*

![](img/a7eb6cbc1a8ae65dee32a5e4ba188b88.png)

[Max 陈](https://unsplash.com/@maxchen2k?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

当您需要创建服务器时，Express 是最常用的 node.js 框架之一。

下面是一个简单的 express 服务器的例子，只有一个端点返回一个小的静态 JSON 响应。

运行上面的代码并多次使用 [autocannon](https://www.npmjs.com/package/autocannon) 对其进行基准测试显示，运行在 2018 MacBook pro 上的服务器在 11 秒内处理大约 19 万个请求~= 1900 RPS

```
➜ autocannon [http://localhost:3000/not_cached](http://localhost:3000/not_cached)
```

# 我们可以做得更好！

由于本例中的代码很少，所以只有一个地方可以获得更好的性能——express 源代码，即响应对象上 json 函数的定义(完整代码位于:[https://github . com/express js/express/blob/master/lib/response . js](https://github.com/expressjs/express/blob/master/lib/response.js))

最重要的部分发生在第 22 行，stringily — **对于我们使用的每个 res.json，返回值被 stringifed** 作为 http 响应发送。在对数据进行字符串化之后，设置内容类型并发送响应。

JSON.stringify 是一个 cpu 绑定的操作，而不是 node 最好的朋友，所以让我们试着只做一次。

我们可以将结果字符串化并保存到变量中，对于每个传入的请求，我们可以将内容类型设置为 application/json，并使用 end 方法将字符串直接写入套接字:

再次运行 autocannon 会在 11 秒内产生大约 35 万个请求~= 3500 RPS。80%的改善。

**但是等下你说，你答应我 300%改善的！！你是对的！**

性能差异很大程度上取决于返回的对象。我想展示的是，即使是一个小物件上的一个小变化也可能意义重大。

尝试对一个大的 json 对象(比如 500–600 Kb)做同样的事情，您将获得性能提升。实际上，使用 res.json 可能会导致您的服务器在有限的环境中崩溃，比如在 kubernetes 上运行的容器。

# **结论**

使用 express 时，如果您的服务器 RPS 性能很差，请尝试缓存任何共享响应并将字符串直接写入响应流，而不是使用 res.json，因为 RES . JSON 每次都使用 JSON.stringify。