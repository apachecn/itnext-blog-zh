# 角度应用——错误处理和优雅恢复

> 原文：<https://itnext.io/how-to-handle-your-run-time-errors-in-your-angular-application-8d72fefbf8a4?source=collection_archive---------0----------------------->

![](img/9bf320bb2389ea8d396f199eb6a82713.png)

由[拍摄的杰米街](https://unsplash.com/@jamie452?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)上[的 Unsplash](https://unsplash.com/search/photos/mistake?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## 我们真的需要它吗？

当我们开始设计我们的企业客户端时，我们要求我们的 UX 设计师给我们一些关于如何在应用程序中处理来自后端服务器的错误的设计提示。应该是弹出窗口，烤面包机，还是输入附近的消息错误？他的回答出乎我们的意料，“你的申请一定没有任何错误！”
我们解释说，在我们的应用程序中，用户可以做许多不允许做的无效操作，我们需要显示后端服务器的错误。毕竟，这是一个企业应用…
他的回答很公平，阻止用户做不允许做的事情。
那么，我们还需要担心什么呢？意外的错误，因为**对于预期的错误我们可以制定一个计划**，和一个好的 UI 流程。
**意外错误技术性很强，一个简单的用户对这些信息无能为力** (DB 锁、查询语法错误、空指针异常、连接断开……)。所以告诉用户“*出了问题*”就足够了。对于那些情况，显示一个弹出窗口或任何你想要的难看的解决方案是一个好的 UX 行为。

## 设计我们的错误处理器

我们希望创建一个服务来捕获应用程序中的任何意外错误，例如:

*   如果这是来自我们后端服务器的错误，那么我们希望向烤面包机显示一些消息，表明出现了错误，用户可能会稍后再试..当用户点击关闭它时，页面将被刷新(我们稍后会建议更好的解决方案)。
*   如果是未经授权的错误(会话已过期)，请将用户导航到登录页面。
*   如果禁止，请将用户导航到描述他/她需要更多权限才能执行此操作的页面。

在任何情况下，我们都会将错误记录到控制台，这样开发人员就可以调试它。

## 让我们一步一步来:

1.  实施`ErrorHandler`

Angular 在`@angular/core.`中有接口`ErrorHandler` ，让我们创建一个新的类来实现它。

2.实施`handleError(error**:** any)**:** void`

一些注意事项:

1.  我使用了两个外部库: [ng2-toastr](https://www.npmjs.com/package/ng2-toastr) 和 [http-status-codes](https://www.npmjs.com/package/http-status-codes) (如果您使用 typescript，您需要添加@types/http-status-codes)
2.  我们所有的后端错误都有一个字段`httpErrorCode.`,我们依靠它来区分不同类型的错误。如果没有这样的字段，那么可能是客户端代码导致的错误。

3.在 root 模块中提供我们的服务:(请 root，使其成为单例)

```
provide**:** ErrorHandler,useClass**:** MyAppErrorHandler
```

我们完了。！！

## 让我们更进一步:

刷新页面不是最佳解决方案。那么，什么更好呢？这取决于我们的用例。让我们给 UI 组件/服务在错误对象中发送一些回调的能力。我们的错误处理器服务将调用它，当用户消除错误时，它会出现在 toast 中。

在下面的代码中，我有一个删除项目的服务，调用一些后端 API 来删除它。如果出现错误，我会向错误对象添加一个回调函数，它知道如何重新添加已删除的项目。
场景将是:用户删除一个项目，得到一条消息:“*出错了，稍后再试*”，用户点击该消息，以消除项目被重新添加到列表中，就像在异常之前一样。

我们的处理程序将有所改变:我们将调用`error.callback?error.callback():window.location.reload;`，而不是简单的刷新:`window.location.reload`

## 够不够？

我更喜欢将错误结构化工作从组件逻辑中移除，并将其移到 HTTP 请求区域。该组件只知道准备回调并将其发送给生成 HTTP 的服务。这个函数将在错误对象中构造回调并重新抛出。

现在，我们的`deletItem` API 将更加简单:

后端 HTTP 请求将是:

如果您的 HTTP 请求有某种公共实用程序，您可以将这个逻辑移到那里，然后编写一次。在我的例子中，HTTP 请求是由我准备的一些模板生成的代码，所以我只需要用这个错误重组更改来更新模板。