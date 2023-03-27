# 使用 React 路由器和悬念进行异步 React

> 原文：<https://itnext.io/async-react-using-react-router-suspense-a86ade1176dc?source=collection_archive---------1----------------------->

![](img/31d966ec54d9f80aa11223203c717e6e.png)

## 使用悬念和“懒惰”使 React 组件的异步加载像你期望的那样简单和直观。

几年前，我写了一篇关于 React 路由器 v4 中的异步 React 组件的文章，从那以后，React 走过了漫长的道路。

现在我们有了`Suspense`和`lazy`，但是我的老路子原创文章还是很受欢迎的。因为有一个本地解决方案可以解决这个问题，所以我决定写一篇关于如何用 React v16.6 做同样事情的新文章。

# 设置

*   反应:16.6.3
*   反应范围:16.6.3
*   react-router-dom: 4.3.1
*   web pack:4 . 27 . 1(create-react-app 2 . 1 . 1 也可以)

# 创建测试组件

首先，我们需要创建一个组件来呈现我们的应用程序。

我向你介绍，`TestComponent`:

# 同步方式

传统上，这是同步加载组件的方式:

它直接导入组件，并在加载`App`时立即使用。这很好，但是如果我们有太多导入的模块不能正常显示的话，它会有使我们的包变大的副作用。

# 异步(懒惰)方式

相反，让我们只在`App`加载之后获取`TestComponent`的 JS 文件:

这看起来很酷吧？我们从 React 调用`lazy`函数，而不是`import TestComponent`。它接受一个回调，在这个回调中，我们调用 ESModule `import`函数。

这一切都要归功于`Suspense`组件。因为`Suspense`的工作方式，你需要加一个`fallback`道具；一些组件，当你的组件还没有加载的时候显示出来。这就是为什么我们在`TestComponent`被拉入之前添加了一个`LoadingMessage`组件来显示。

使用`lazy`时，Webpack 会自动对您的包进行代码分割。这可能是最大的好处，因为只需几行 JavaScript 代码，您的初始包大小就可以小很多。

这与[我们使用的旧方法](https://medium.com/@Sawtaytoes/async-react-router-v4-components-c18792e6f331)完全不同，我们手动创建承诺，监听响应组件，自己做`Suspense`做的事情。

## 使用 CommonJS 中的“require”

`import`并不总是可用的，而是特定于 ESModules 的。相反，您可能会使用 CommonJS，所以您会想要使用`require`来代替。事情是，你不能把一个换成另一个，因为`import`返回一个承诺，而`require`直接返回你的组件。

在这种情况下，您会想要做这样的事情:

# 异步反应路由器

理想情况下，我们只有在特定的路线上才会装载`TestComponent`。为了处理这个用例，我们将使用`TestComponent`，以及一些其他的测试组件，并将它们封装在仅当您在那些特定的路线上时才加载的路线中。

通过这种方式，我们可以发送一个非常小的初始 JS 包，并在事后考虑如何获取真正的内容。这与您可能在 2-3 年前看到的老式 React 路由器 v1-v3 的`require.ensure`的行为相同。

这是全部的装备:

我们像往常一样在交换机中创建路由组件。使用`lazy`和`import`就像我们在第一个例子中所做的那样，我们能够动态地加载这三个测试组件，只有当您已经访问了其中一个路由时。从这里开始，将其推广到其他组件和子路由应该很简单。

问题是，导入语法相当混乱。您可以通过创建自己的`lazyImport`方法并使用它来清理它:

Webpack 需要静态编译，所以您需要完成我在这里展示的奇怪的模板字符串代码。但是如果你不想担心必须维护和实现你自己的解决方案，那么…这就变得棘手了。

事情不像你想的那样。

`import`的工作原理是从调用`import`函数的地方检查文件的加载位置。这意味着如果我要为它写一个库(我试过了)，它将只对相对于我在`node_modules/`中的库的目录的文件起作用，而不是在你的项目的根目录中，除非我直接传入`import`函数，这样它就知道在哪里相对导入。

有很多其他解决方案可以做同样的事情，比如围绕`lazy`和`import`函数创建 React 组件，但是这超出了像这样的简单示例的范围。

## CommonJS 问题再次罢工！

虽然这是可行的，并且允许代码分割，因为`import`允许动态字符串值，但它不会像您想象的那样使用`require`工作。除非你使用的是`require.ensure`，否则代码分割是不可能的，因为 Webpack 在幕后处理动态字符串导入的方式。

# 自己做

这里有很多代码，但是您可以在 [CodeSandbox](https://codesandbox.io/s/18rnr5p97q) 亲自尝试一下:

# [编辑]

react-router 4 . 4 . 0 目前有一个问题，它不能处理通过`Route`上的`component` prop 传入的延迟加载或内存化组件的情况。

您将会收到如下错误消息:

> 警告:失败的属性类型:提供给“Route”的“object”类型的属性“component”无效，应为“function”。

你可以在[GitHub](https://github.com/ReactTraining/react-router/issues/6420):
[https://github.com/ReactTraining/react-router/issues/6420](https://github.com/ReactTraining/react-router/issues/6420)上找到更多关于这个问题的信息

感谢[坤 GEO](https://medium.com/u/3aa9306ccdb5?source=post_page-----a86ade1176dc--------------------------------) 指出这一点！

# 更多阅读

如果您对与 React 相关的更多主题感兴趣，您应该查看我的其他文章:

*   [用减速器打造独角兽！](/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e)
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [你不应该需要从 React-Redux 中“连接”](https://medium.com/@Sawtaytoes/why-you-shouldnt-need-connect-from-react-redux-498876de9e4e)
*   使用 Redux 的秘密:createNamespaceReducer
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)