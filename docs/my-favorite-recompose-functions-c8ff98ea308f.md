# 我最喜欢的重组函数

> 原文：<https://itnext.io/my-favorite-recompose-functions-c8ff98ea308f?source=collection_archive---------1----------------------->

![](img/723065c7aa4e4b2cdf0cc900067dd7c2.png)

[重新构图](https://github.com/acdlite/recompose)很棒。使用过 Lodash 或 Ramda 这样的库吗？重组是为了反应。

这

![](img/5082af87579c67d57cbff846b8470557.png)

变成了这个

![](img/235163a2907a9e6eebc500d2fc814355.png)

这种风格真的让我越来越喜欢，我想分享一些我最喜欢的功能。

# withHandlers()和 withState()

假设我们想在用户输入时显示他们的名字。我是根据[官方文件](https://github.com/acdlite/recompose/blob/master/docs/API.md#withhandlers)中的例子得出这个结论的。

传统上，我们的 JSX 可能看起来像

![](img/5b65cf3e88b7c996f7a2a8adbcb478bd.png)

我们需要状态管理来处理用户名，所以我们使用`this.state.value`。让我们在构造函数中定义该状态，并添加一个处理程序来更新它。

![](img/d45018cb5ce41f9a0e69e79a8c84cc01.png)

我们的状态被初始化为`{ value: '' }`，并且我们添加了一个`updateValue`方法。我们已经在 JSX 接上了。让我们试一试。

![](img/471393705bbf424cb60e56de21b07f0e.png)

太酷了，有用。当您键入时，用户名会实时更新。

这是一个可能的改编版本。我们的组件现在将接受一个名为`onChange`的道具。

![](img/93c0c1bdfd1e0975f540f2bbe8f65661.png)

我们将添加状态和一个使用`withState`更新它的处理程序。

![](img/a1c230a6c2d1c4465c8141a36b7ae45d.png)

`withState`接受三个参数，`stateName`、`stateUpdaterName`和`initialState`。

在本例中，我们创建了一个名为`value`的状态，将其初始化为一个空字符串，并创建了一个名为`updateValue`的函数来更新它。

现在为`withHandlers`。

![](img/49d8aa6b32556e13b8eb9935fd84b5dc.png)

`withHandlers`接受处理程序的对象:接受`props`并返回一个处理程序的高阶函数。即使在处理程序返回之后，它也可以通过闭包访问其父函数的`props`。

`updateValue`是由`withState`提供的更新函数，我们在`withHandlers`内部使用它来设置`event.target.value`为新的状态值。所有这些都发生在处理程序`onChange`中，它将作为道具传递给我们的组件！

记住现在每当用户更新输入框时,`MyForm`调用它的`onChange`道具。

![](img/8bf36e572d1964a66dd748f80ca05431.png)

我们的默认导出现在看起来像这样

![](img/ec038540bb5c4fc87eb4fd82d7d0b8cc.png)

在调用了`withState`和`withHandlers`之后，你得到了更高阶的组件。必须用另一个组件调用更高一级的组件。

这就是为什么我们把`MyForm`传给`addHandlers`，然后把`addHandlers(MyForm)`传给`addState`。一个装饰下一个，联合起来形成一个更大的实体。

我们的功能保持不变。

![](img/5ec5ca532dccaaa41b2388dacb5af489.png)

# 撰写()

如果你熟悉函数式编程，你可能会认出`compose()`。如果没有，我已经写了[一个帖子解释它和](https://medium.com/@yazeedb/pipe-and-compose-in-javascript-5b04004ac937) `[pipe()](https://medium.com/@yazeedb/pipe-and-compose-in-javascript-5b04004ac937)` [在这里。](https://medium.com/@yazeedb/pipe-and-compose-in-javascript-5b04004ac937)

`compose`结合了`n`函数，允许你更优雅地嵌套函数。让我们用它来重构我们增强的`MyForm`。

我们以前有过这个

![](img/ec038540bb5c4fc87eb4fd82d7d0b8cc.png)

我们将导入`compose`

![](img/c5b98aa46c6b2b71f6317722ea078950.png)

像这样使用它

![](img/f72986bb30b86f9726f42714d19262ed.png)

我在[重组文档](http://import { compose, withHandlers, withState } from 'recompose';)中看到的一个模式是将您的“增强”存储在一个变量中，然后导出该变量+您的组件。这里有一个例子。

![](img/213cef283d484b48fea9c380c385df6c.png)

我采用了这种模式，因为它对我来说很有意义。

你的组件是一个接受`props`并返回 JSX 的纯函数。

您的“增强”是高阶组件，旨在添加或改进组件的功能。

以这种方式定义和组合它们似乎是对这一想法的补充。

我再分享一个我最喜欢的高阶组件:`lifecycle`。

# 生命周期

您可能会从名字中猜到，`lifecycle`让您的纯功能组件访问 React 的生命周期挂钩，严格地说是针对类的。

实际上使用了一个类，但是你只处理函数。这才是重点。因为所有的东西都是作为一个函数暴露给你的，所以你只需要考虑这些。

让我们将其添加到我们的`enhanceComponent`高阶组件。我们先导入。

![](img/7604a7d5e5b72f0586be556919100fd3.png)

然后把它放在下面

![](img/06fbfcf94caa1d89f19ec452c2a6c284.png)

如您所见，`lifecycle`接受函数的对象，就像`withHandlers`一样，但是这些函数必须以有效的 React 生命周期钩子命名。

我们在屏幕上看到以下警告

![](img/7ef7f502ef6ff154d7a631bacc7c861e.png)

最好使用 [ES6 方法定义](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions)来允许访问`this.props`。如果你想从`props`调用一个处理程序，你需要`this`指向正确的上下文。

假设在`componentDidMount()`中，我们想将用户名设置为“生命周期挂钩！”

![](img/71dae29a99e925dd3f7146ca31f883d2.png)

导致了这个

![](img/e69d71dabd99ddef5debd8d355c8889a.png)

如果我们使用箭头函数，那就不行了

![](img/50dc4c9d781f3622790db4773a9b1db6.png)![](img/19525975f76d6cf9f4f8cdb3b50b030d.png)

如果我们`console.log(this)`我们得到如下

![](img/5af2f6c9dfdd33ed8e00661f406db20e.png)![](img/663ec71bb3d013b4cd9eda42b5a6a8d5.png)

不过，如果您使用 ES5 风格的函数，它也能工作。

![](img/f4b83ad0b99f9ffbb5411d783ca19598.png)![](img/c95e1d3950d1d2f1fb7e8b74e79bd60f.png)

现在我回避 ES5 函数，所以方法定义会做得很好。

# 其他实用功能

这些是我要重组的函数。

随着我开始再次使用 [RxJS Observables](http://reactivex.io/rxjs/) ，一些其他的重组函数引起了我的兴趣。

它们可能是未来一篇文章的主题，如果你感兴趣，这里有一个到重新组合 observable utils 文档的链接。

下次见！

保重，
Yazeed Bzadough