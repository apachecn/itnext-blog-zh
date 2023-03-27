# 如何:在属性更改时更新状态

> 原文：<https://itnext.io/how-to-updating-state-on-prop-changes-2296a03ef08c?source=collection_archive---------0----------------------->

![](img/cca7a368949996be95dd8048a22a02a2.png)

Pierre Bamin 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

有时，复合意味着我们希望将关注点分离到可以隔离状态变化的程度。但是如果我们有一个改变的道具，并且我们想在它改变的时候更新状态，那该怎么办呢？这是一个很难回答的问题。有几种方法可以做到这一点，这取决于你想做什么。在 react 的未来版本中，旧的方法将被淘汰。这篇文章将向你展示给状态分配道具的正确方法，并在变化时更新它们。

# 旧方法(已弃用)

所以我们想给状态分配道具，但是我们希望它发生在挂载和每次更新的时候。函数 componentWillRecieveProps 为我们做了这些，但是它正在被淘汰。这样做的原因是它不安全，你可能会遇到问题。当您在这里设置状态时，它可以在准备设置状态之前运行。

```
state = {
  Show: false
}componentWillReceiveProps(nextProps) {
const { show } = this.props
 if (nextProps.show !== show) {
   this.setState({ show })
 }
}
```

因此，react 开发人员提出了一些替代方案。每种方法都有特定的情况。

# getDerivedStateFromProps

这是 componentWillReceiveProps 的第一个替换。

以下是如何使用该功能的示例:

```
state = {
 Show: false
}static getDerivedStateFromProps(nextProps, prevState) {
 return {
  show: nextProps.show,
 };
}
```

这里首先要注意的是它是一个静态方法。这意味着你不能直接访问类中的任何东西。第二件要注意的事情是它返回了一个新的状态对象。更新状态的方法是返回新的状态。

现在，react 团队非常明确地表示，这不是你需要用于所有道具和状态变化的方法。他们说，如果你属于另一个类别，你应该使用这些其他方法之一。这些笔记直接取自 [react 文档网站](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)。

*   如果您需要**执行副作用**(例如，数据获取或动画)来响应道具的更改，请使用 [componentDidUpdate](https://reactjs.org/docs/react-component.html#componentdidupdate) 生命周期。
*   如果您希望**仅在属性改变**时重新计算一些数据，那么[使用记忆助手代替](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)。
*   如果你想在道具改变时“重置”某个状态，可以考虑用一个键完全控制一个组件(T2 或 T4)。

当我写这篇文章时，我希望重新获取数据作为 prop 变化的副作用。这是我最后做的。

```
componentDidUpdate(nextProps) {
 const { show } = this.props
 if (nextProps.show !== show) {
  if (show) {
   getMoreData().then(resp => this.setState({ data: resp.data }))
  }
 }
}
```

这里重要的事情是检查以确保你是在你想更新的时候更新，而不是每次更新的时候。

# 结论

设置属性更改的状态可能很棘手。但是如果你遵循 react 团队设定的模式，它将为你节省大量的时间和技巧。祝你好运！

编码快乐！！

***Avery Duffin*** *是* [*【开发者午餐】*](https://www.developerswholunch.com) *的所有者和创始人，这是一个播客和博客，旨在帮助那些学习编程并寻求在软件开发领域建立职业生涯的人。他目前在万事达卡公司* *担任软件工程师，住在美国犹他州。你可以在*[*Twitter*](https://twitter.com/DuffinAvery)*和*[*linkedIn*](https://www.linkedin.com/in/avery-duffin-69317228/)*上和他联系。*