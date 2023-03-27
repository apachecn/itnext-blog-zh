# 下一代节点&快速 MVC 模式

> 原文：<https://itnext.io/a-new-and-better-mvc-pattern-for-node-express-478a95b09155?source=collection_archive---------1----------------------->

![](img/2a7cb4f2d499fe291a93c040e9c448d8.png)

[米尔科维](https://unsplash.com/@milkovi?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

似乎是很久以前的事了，我曾经写过关于[的文章，这是一个简单易用的 MVC 模式，可以用于 Node 和 Express。随着时间的推移，我已经开始发展这种模式来模仿企业框架中的模式，比如。NET Web API 2。](https://medium.com/@TaylorAckley/simple-and-minimalist-mvc-architecture-pattern-for-node-express-cb542287a144)

首先是一些免责声明。

首先，这真的不是 MVC 模式，因为没有视图。它更像是一个 MCS 模式，或者模型>控制器>商店。也就是说，我们将不讨论 MCS 的模型部分，只讨论控制器和商店。

第二，关注模式，而不是代码。为了简洁起见，对其进行了剪裁和编辑，然而，大部分内容被留了下来，以更清楚地说明数据是如何从模型上升到商店、控制器并被发送到以太网的。不管怎样，这些代码并不意味着要在实际的应用程序中使用。

# TL；速度三角形定位法(dead reckoning)

您可以在这个[示例存储库](https://github.com/TaylorAckley/NodeNgXBoilerPlate/tree/master/api/lib)中看到所讨论概念的示例

# 概观

该模式的总体思想是创建灵活的路由模式，通过所需的中间件功能对路由进行分组。

一旦路由收到请求，它就通过管道传递给控制器方法，该方法决定如何处理请求，并负责返回响应。

接下来，控制器将把请求对象分派给一个存储，存储处理我们的 CRUD 和业务逻辑操作。有一个商店来处理沉重的负担意味着我们可以保持我们的控制器轻，我们的商店是免费的不必要的混乱。这种解耦也使得我们的存储高度可测试。

为了进一步保持我们的语法尽可能的简洁，我们利用了本地 ES6 类。这并没有什么真正的优势，除了它有助于编写易于阅读的高度可组合的代码

```
// mehconst dataStore = {};dataStore.getData = async (a) => {// get some data
};dataStore.createData = async (a, b) => {// create some data
};module.exports = dataStore;
```

相对

```
// much cleaner!class DataStore {static async getData(a) {//.....get some data
    }static async createData(a, b) {//.....create some data
    }
}module.exports = DataStore;  // or module.exports = new DataStore() if you are not using static methods.
```

在第一个块中，你会注意到我在类名前使用了`static`关键字。`static`关键字仅仅意味着我不需要实例化一个新的类实例来使用这些方法，但是因为我不需要构造函数，所以我可以跳过实例化，直接访问我的类方法。

> 关键字`***static***`定义了一个类的静态方法。静态方法不会在类的实例上调用。相反，它们在类本身上被调用。这些通常是实用函数，例如创建或克隆对象的函数。
> https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Classes/static

在其他语言中，如。NET Core 中，静态的使用可能会导致问题，因为依赖注入主要是通过构造函数来处理的。因为 NodeJS 不像那样使用 DI，所以我们可以随意使用 statics，这实际上只涉及到风格问题。

# 结构

我们的目录结构大致如下所示。目录结构也模拟了模式的流程。

```
- server.js // Startup logic
--  /routes
---   routes.index.js  // index of Routers 
---   routes.unauth.js // route definitions for anonymous routes
---   routes.auth.js   // route definitions for authenticated routes
-  /controllers
--   account.controller.js // pipe request objects to  the account store, send response
-  /stores
--   account.store.js  // receive the request object, handle CRUD.
--   auth.store.js     // middleware to authenticate.
```

让我们从我们的`server.js`文件开始，开始监听请求和服务响应。这里没什么特别的！

接下来是我们的路由索引文件，它在前面的部分中被引用。

我们导出一个函数，该函数将 Express `app`实例作为一个参数，我们可以用它来创建一些通过中间件(如果需要，还有版本)将路由分组在一起的 [Express 路由器](https://expressjs.com/en/api.html#express.router)。

如果你看一下`app.use()`，我们会给它一个签名:

*   路线前缀(`api/v1)`
*   基于授权头的存在、它们的角色等，允许用户通过的一系列中间件功能。
*   包含在另一个文件中的路由定义。

这使得很容易看出哪些路由获得了哪些中间件，并保持事物的逻辑分组。如果您不仅想按中间件或版本对路由进行分组，还想按资源对路由进行分组，那么这种模式具有很强的适应性。

下面是路由索引中引用的身份验证中间件的示例。这篇文章实际上不是关于认证或者中间件如何工作的，但是为了清楚起见，我把它包括进来了。

这里的主要内容是`req`和`res`对象通过管道传输到中间件，您可以根据需要对它们进行操作，通过调用`next()`让请求通过，或者选择拒绝请求并在请求到达路由处理器之前返回响应。为了方便起见，我将解码后的授权头中的信息附加到了`req`对象上，这样用户的标识符就可以在路由处理程序中使用了。

接下来，我们设置未授权的路由，这些路由不需要对用户进行身份验证。

第一个参数是在`routes.index.js`中定义的父路由器中指定的根之后的路由模式。第二个参数是一个控制器方法(或函数)，它通过管道将`req`和`res`参数传递给控制器方法。我们还可以在每条路由的基础上指定一个中间件数组，将它作为第二个参数添加，并将方法定义推送到第三个参数。

如果您不熟悉如何指定一个需要参数而不指定所述参数的函数，基本的解释是，只要接收函数的签名匹配，参数就能通过。

现在，和上面一样，这里的区别是我们在`routes.index.js`中指定了认证中间件，这意味着这些路由是受保护的。

让我们进入正题，定义我们的控制器。下面我们有一个包含两个静态方法的类。我们可以走老路，在一个对象中声明我们所有的函数，但是类方法会产生更干净的代码。

我们使用静态方法并导出类定义，这意味着我们不需要类的实例。这是一种风格偏好，我们也可以初始化该类的一个实例，然后导出该实例。

```
module.exports = new AccountController();
```

控制器的作用是将资源集合在一起，组装一个响应并发送出去。我倾向于保持我的控制器方法最少，只负责弄清楚做什么和发送回一个响应。

我在这里使用了`async`关键词，但是你也可以使用基于承诺的方法。

终于——我们到达了商店。这是所有业务逻辑、垃圾和魔法发生的地方。

存储负责组装将返回给请求者的“有效负载”或可用数据。我们简单地将整个`req`对象管道化，这样所有重要的东西，比如参数和用户，在我们需要的时候都是可用的。

商店可以根据我们的需要而变得复杂或简单，你的大部分代码应该在商店里。我的方法倾向于保持方法小，并根据需要从同一个类中调用兄弟方法。这使得事情变得美好、整洁和可测试。

*就这样。完成了。非常感谢任何反馈。*