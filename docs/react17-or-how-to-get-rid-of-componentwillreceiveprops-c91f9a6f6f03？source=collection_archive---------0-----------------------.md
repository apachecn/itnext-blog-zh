# React17，或者如何去掉“componentWillReceiveProps”？

> 原文：<https://itnext.io/react17-or-how-to-get-rid-of-componentwillreceiveprops-c91f9a6f6f03?source=collection_archive---------0----------------------->

![](img/0c149cfab2f05ac7c19e356e31f0cec5.png)

正如你可能已经知道的，随着 *React 16.3* 的发布，一些遗留的生命周期被废弃了。

> 我们学到的最大教训之一是，我们的一些遗留组件生命周期倾向于鼓励不安全的编码实践。
> 
> 这些生命周期方法经常被误解和微妙地误用；此外，我们预计它们的潜在误用可能会给异步渲染带来更大的问题。

# 那么现在会发生什么呢？

从版本 17 开始，React 对某些生命周期的支持将发生变化:

*   `componentWillMount`
*   `componentWillReceiveProps`
*   `componentWillUpdate`

从现在开始，只有新的“UNSAFE_”生命周期名称才起作用。这给我们留下了几个选择:

*   我们可以将生命周期事件重命名为`UNSAFE_componentWillMount`、`UNSAFE_componentWillReceiveProps`、`UNSAFE_componentWillUpdate`，并希望一旦 v17 发布，什么都不会中断。甚至有这样的工具。
*   我们确实在尝试，并更改我们的组件以兼容 React 的未来版本。

我建议选择后者。

# 让我们开始吧

我们必须去掉三个生命周期，所以我将尝试分别讨论每一个。

## `componentWillMount`

这个很简单。用`constructor`代替就行了。

以前

在...之后

它看起来不错，但有一些警告。不建议在`constructor`中有副作用(api 获取，也许是一些计算)。最好把他们移到`componentDidMount`而不是*建造者*。不利的一面是，它是在`render()`之后执行的，所以根据副作用，你可能需要重新渲染。

[](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) [## 反应生命周期方法图

### 完全交互式和可访问的 React 生命周期方法图。

projects.wojtekmaj.pl](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) 

## `componentWillUpdate`

> 当接收到新的道具或状态时，在渲染之前调用`componentWillUpdate()`。利用这一机会，在更新发生之前做好准备。初始呈现时不调用此方法。

这个事件钩子基本上是`componentWillReceiveProps`，但是更差。它发生在渲染之前，但是你不能使用`**this**.setState()`。

但是它很有用，因为它让您可以在组件接收新的属性或状态之前控制组件的操作。我主要将它用于动画，因为它允许您为 DOM 节点预定义一些初始条件。

根据 React 文档，该方法可以由`componentDidUpdate()`代替。如果您用这种方法从 DOM 中读取(例如，保存一个滚动位置)，您可以将该逻辑移动到`getSnapshotBeforeUpdate()`。

```
getSnapshotBeforeUpdate(prevProps, prevState)
```

`getSnapshotBeforeUpdate()`在最近渲染的输出提交到 DOM 之前被调用。它返回快照*(默认值:NULL)，*，作为`componentDidUpdate(prevProps, prevState, snapshot)`的第三个参数

# `componentWillReceiveProps`

这是非常常用和误用的 react 生命周期事件。我想把这一部分分成两部分——副作用和状态。

## 副作用

考虑这个例子:我有一个内部通信集成，但是它需要在特定的路由上显示。

以前

在...之后

`componentDidUpdate`仅在更新发生并且 ***未*** 在初始渲染时被调用。

你可以用它做各种各样的副作用，从数据获取到 DOM 操作。你可以调用`**this**.setState()`,但是要注意它必须被包装在一个条件中，就像上面的例子一样，否则你会导致一个无限循环。

## 组件级状态

考虑这个例子:我有用于列表呈现组件，可以使用拖放操作对列表进行重新排序。出于性能原因，我决定在内部保留列表的副本，并且只更新 dragEnd 上的 *redux 状态*。这要求我保持两种状态同步。

以前

在...之后

> `getDerivedStateFromProps`在调用 render 方法之前被调用，无论是初始挂载还是后续更新。它应该返回一个对象来更新状态，或者返回 null 来不更新任何东西。

该方法仅**对状态处理**有用，所有其他副作用应移至`componentDidUpdate`。此外，它是一个`static`方法，对内部类方法的访问是有限的。

> 此方法不能访问组件实例。如果您愿意，您可以通过提取组件 props 的纯函数和类定义之外的 state，在`getDerivedStateFromProps()`和其他类方法之间重用一些代码。

假设我需要在两个数组之间做一些比较，在`getDerivedStateFromProps`里面。我不能使用`**this.**compareArrays()`、**、**来访问组件内部函数，所以在我的列表组件之外创建纯函数`compareArrays`没什么不好。

感谢您阅读这篇文章。所有报价均取自官方[反应文件](https://reactjs.org/docs/react-component.html)。

在 [LinkedIn](https://www.linkedin.com/in/giedrius-vickus) 上联系我或者在 [GitHub](https://github.com/giedriusvickus) 上关注我。