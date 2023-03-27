# 在 Redux 中的减速器之间传递状态

> 原文：<https://itnext.io/passing-state-between-reducers-in-redux-318de6db06cd?source=collection_archive---------0----------------------->

![](img/18f54bce05bdea59b447d3e9db35977f.png)

## 想学 React？看看我为初学者设计的新课程 React:构建和应用，并学习基础知识。

[![](img/5c22a14a522ba5a7825571f3868cc6eb.png)](https://www.udemy.com/share/100GaX/)[![](img/3c7a53fe19b81397aed6147a0f577a60.png)](https://skl.sh/2CssGg0)

您是否曾经有过需要在另一个 reducer 中访问的来自一个 reducer 的状态片段？如果你使用 Redux 的[组合减速器](https://redux.js.org/api-reference/combinereducers)开箱即用，那么你可能会卡住，或者你可以使用[减速器](https://github.com/redux-utilities/reduce-reducers)，但这可能会超出你的需求。

> 相反，为什么不编写自己的联合减速器呢？

很容易忘记 reducer 只是一个具有以下格式的函数`(previousState, action) => newState`，没有理由不编写自己版本的`combineReducers`来处理这种情况。下面是我用来访问我的 reducer 中不同片的状态的模式。

例如，如果我们正在构建一个处理本地化的应用程序，我们可能会有以下状态形状:

在上面的状态中，`languages`是一个应用程序支持的语言数组，而`translations`是一个对象，其中每个键的值都是一个数组，包含每种支持的语言的翻译。如果我们用`combineReducers`来组装这个州，它看起来会像这样…

上面的工作很好，但是如果我们需要从`translations`减速器访问来自`languages`的数据会发生什么？如果你正在使用`combineReducers`，你将只能访问你的减速器的状态。相反，您可以使用下面的模式来创建一个定制的 reducer 函数，该函数组合您的 reducer，并在 reducer 之间传递所需的数据。

尽管这种模式足够好，但我不会滥用它。如果您发现您必须频繁地在多个 reducers 之间传递状态，这可能是您的状态形状需要工作的迹象。无论哪种方式，记住减速器只是功能——不要害怕实验！

**更多 React、Redux、JavaScript 阅读关注我上** [**中**](https://medium.com/@ryandrewjohnson) **或** [**推特**](https://twitter.com/ryandrewjohnson) **。**