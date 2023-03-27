# 从快车到 Koa

> 原文：<https://itnext.io/from-express-to-koa-f3be4afdfd39?source=collection_archive---------2----------------------->

![](img/f7a137b292f6c1a5b74e5d7ab6a4e10c.png)

nodeJs web 框架有几个，最流行的一个是 [express.js](https://expressjs.com/) 。我想解释一下为什么在写服务器代码时，我决定从 *express.js* 转移到 [koa](https://koajs.com/) 。

要理解这篇文章，您应该了解:

*   [JavaScript](https://developer.mozilla.org/bm/docs/Web/JavaScript)
    –什么是[承诺](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
    –如何用 [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 写承诺
*   [nodeJs](https://nodejs.org/en/)
    ——如何搭建一个简单的[express . js](https://expressjs.com/)server
    ——什么是 [express 中间件](https://expressjs.com/en/guide/writing-middleware.html)

# TL；速度三角形定位法(dead reckoning)

随着 NodeJS 版本 7 的出现，出现了对 `[async/await](http://node.green/#ES2017-features-async-functions)` [函数](http://node.green/#ES2017-features-async-functions)的[支持。Koa 只是和他们玩得更自然，➡️使用 Koa。](http://node.green/#ES2017-features-async-functions)

# 表达

我们将编写一条简单的路线，它将:

1.  查询数据库获取一些资料
2.  将结果传递给第二个数据库调用
3.  然后发送最终结果作为响应

## 节点样式回调

这里我们将使用 nodeJs 风格的回调签名:
一个回调函数，以`error`为第一个自变量& `result`为第二个自变量。

到目前为止一切顺利。幸运的是，我们的数据库对象也支持承诺。

## 承诺

下面的代码与上面的代码一样，但是:

*   我们可以简化我们的代码
*   我们不再复制错误控制
*   `catch`不会处理同步错误

到目前为止一切顺利。幸运的是，因为我们使用 nodeJS > = 7，所以我们可以使用 async/await。

## 异步/等待

下面的代码与上面的代码一样，但是:

*   我们有一个不太麻烦的代码
*   我们仍然没有复制错误控制
*   任何在`try/catch`内部的错误都会被处理

到目前为止一切顺利。但是如果我们增加更多的路线，情况会变得更糟:

你看到了吗？
我们写了一遍又一遍`try {} catch(error){ next(error) }`没什么大不了的，但是结尾很无聊……

但幸运的是，我们可以为此编写一个包装函数！

## 更好的异步/等待

因此，让我们编写我们的包装器:

一篇更详细的文章是由[亚历克斯·巴泽诺夫](https://medium.com/@Abazhenov/using-async-await-in-express-with-node-8-b8af872c0016)撰写的

最后，让我们在代码中使用它:

到目前为止一切顺利。但是，我们仍然需要编写一些样板文件来处理这个问题…

KOA 来了！

# 寇阿相思树

## 什么是 KOA？

简而言之，是 express.js 背后的同一个团队使用 JavaScript 语言编写了一个 web 框架。
**它的核心是使用 async/await**
你可以在这里找到[的完整介绍](https://koajs.com/#introduction)

用 Koa 设置服务器非常简单。
对于路由，由于默认不提供任何东西，我们将使用 [koa-router](https://www.npmjs.com/package/koa-router)

## 设置路由器和错误中间件

这段代码应该足够了:

## 书写我们的路线

这就是我们编写应用程序代码的方式:

在我看来更瘦😀

*   没有重复的`try/catch`
*   不需要编写异步中间件
*   不需要将我们所有的路由处理器都打包到那个中间件中
*   处理同步/异步错误

# 关于 Koa 生态系统

目前，Koa 还没有 express.js 那么多的中间件。

但是*必须有*中间件已经在这里了，编写自己的中间件相当容易。
我从来没有发现自己在 Koa 身上无法达到自己想要的情况。

所以如果你喜欢`async/await`代码风格，给 Koa 一个尝试🙂

*原载于*[*hiswe . github . io*](https://hiswe.github.io/2018/07-from-express-to-koa/)*。*