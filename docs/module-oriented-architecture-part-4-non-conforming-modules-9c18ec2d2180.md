# 面向模块的体系结构第 4 部分:非一致性模块

> 原文：<https://itnext.io/module-oriented-architecture-part-4-non-conforming-modules-9c18ec2d2180?source=collection_archive---------9----------------------->

刷新你的记忆:

[](https://medium.com/@poksi/module-oriented-architecture-part-3-modules-and-routing-241b06439a9f) [## 面向模块的体系结构第 3 部分:模块和路由

### 刷新你的记忆:

medium.com](https://medium.com/@poksi/module-oriented-architecture-part-3-modules-and-routing-241b06439a9f) 

可能会发生这样的情况，我们将不得不在我们的项目中包含代码，它不符合我们在本系列中介绍的架构。原因可能是多种多样的:我们可能有遗留代码，被关注点严重分离，也许我们有 ***React Native*** 与许多已经编写的 ***Javascript*** 代码桥接，我们不可能改变与原生端的接口，等等。

本章将展示，我们将如何使用`URLProtocol`和 iOS 的简单 URL 加载系统将我们的模块转换成经典的 HTTP API，从而使我们的模块开放并兼容任何代码，就像我们为应用程序外部的深层链接所做的一样。

# 简单的网络服务

我们创建了一个简单的`NetworkService`类，它为我们提供了两个公开的函数`get`和`post`。除此之外，我们还有两个私有函数。首先，`request`非常明显。唯一看起来有点猫腻的是`scheme`参数。它的处理非常简单:如果没有将`scheme`作为参数传递，那么它将默认设置为‘http’。我们现在对此很满意。

然后我们有了`service`函数，因为我们在 HTTP 方法`get`和`post`中重用了这段代码。但是有一行代码让这一切变得非常有趣:`configuration.protocolClasses?.append(URLRouter.self)`。`URLSessionConfiguration`的这个方法提供了注册我们自己的类`URLRouter`的可能性，该类继承自`URLProtocol`抽象类。让我们看看为什么需要这样做。

# URL 路由器

这是我们对 2 个协议的实现:`URLSessionDataDelegate`、`URLSessionTaskDelegate`以及最重要的，子类化抽象类`URLProtocol`(可能没有最好的名字)。对于曾经使用旧的 mocking 框架测试 URL 连接的所有人来说，`URLProtocol`通常用于它。我们中的许多人自己直接使用它来创建我们自己的网络存根基础结构。我不会讲太多细节，因为已经有很多关于它的文章了，但是让我解释一下这个协议在我们的例子中的作用。

所以，我们用`URLRouter`做的事情非常简单:

*   如果任务中的`URLRequest`具有`schema`，则签入`override class func canInit(with task: URLSessionTask) -> Bool`，这与我们在前一章中定义的内部模式相匹配
*   然后我们检查 URL 是否包含与我们的`ApplicationRouter`匹配的`host`和`path`
*   如果检查结果是肯定的，那么 URL 加载系统调用我们的`override func startLoading()`函数，我们只需通过 are `ApplicationRouter`调用一个适当的模块，工作就完成了！

嗯，不完全是，我们还没有在这里创建返回响应样板，但这不是现在最重要的。那么，下一步是什么？嗯，我猜是“淘气”模块！

# 不一致模块

我们已经创建了一个非常简单的类，叫做`NonConformingModule`，它没有实现我们从第一章开始讨论的任何架构。它唯一做的事情是用`login`来获取`bearerToken`，这就是使用`LoginModuleRouter`中的`login`函数的地方，我们之前提到过。

我们唯一需要做的就是实例化这个模块并调用`login`函数，下面将会发生:

*   `NonConformingModule`直接调用`NetworkService`作为普通的 HTTP 请求，但是它将“`tandem`”作为`schema`和`/login`方法的参数传递给`LoginModule`
*   `NetworkService`调用 iOS 的 URL 加载系统，就像调用任何其他 HTTP 请求一样。
*   注册的`URLRouter`类拦截`URLSessionTask`，发现来自 URL 的`scheme`与内部的匹配，确认拦截并创建自身的实例
*   `URLRouter`用 URL 调用`ApplicationRouter`打开`LoginModule`后者调用`LogineModuleRouter`剩下的你已经知道了…

这是一个展示，如何将 URL 作为一个通用访问对象，可以通过 URL 加载系统直接访问我们的模块架构基础设施。

# 演示项目

检查 tag 1.0 上的演示项目，并查看我们在前 4 章中已经讨论过的代码。

[](https://github.com/poksi592/module-architecture-demo/tree/1.0) [## poksi 592/模块-架构-演示

### 在 GitHub 上创建一个帐户，为模块架构演示开发做贡献。

github.com](https://github.com/poksi592/module-architecture-demo/tree/1.0) 

我们将继续讲述如何消除对显式路由器类的需求，并隐式实现路由。在[第 5 部分](https://medium.com/@poksi/module-oriented-architecture-part-5-implicit-routing-655b468ca1b4)中阅读相关内容。