# 添加确认对话框以对事件做出反应

> 原文：<https://itnext.io/add-confirmation-dialog-to-react-events-f50a40d9a30d?source=collection_archive---------0----------------------->

一个简单呈现组件的例子，它为任何反应事件增加了一个确认步骤，比如表单提交或者按钮点击。

您已经做了无数次了——您有一个需要额外确认步骤的表单。也许是为了鼓励用户仔细检查输入的数据，或者只是礼貌地告诉他们将要发生什么。这里有一个很好的(不幸的是，有点粗糙的)方法来使用渲染道具组件。

# 出发点

假设您有一个表单:

```
<form onSubmit={**handleSubmit**}>
   // form
</form>
```

当提交表单时，`handleSubmit`事件处理程序被调用，发生了一些事情。现在您想添加一个确认步骤。您可能只是创建一个不同的处理程序，它打开一个 modal 并添加`handleSubmit`处理程序来提交 modal 中的按钮:

```
{isOpen && (
  <Dialog>
    Are you sure? <button onClick={**closeConfirmationModal**}>No</button>
    <button onClick={**handleSubmit**}>Yes</button>
  </Dialog>
)}<form onSubmit={**openConfirmationModal**}
   // ... form
</form>
```

…类似这样的东西。唯一的问题是如果你想在不同的组件中重用相同的行为。如何把它变成一个可复用的组件？

# 渲染道具组件

在考虑实现之前，让我们先画出可能的用例。

我们前一个例子可能是这样的:

```
<Confirm>
  {**confirm** => (
    <form onSubmit={**confirm**(handleSubmit)}>
      // ... form
    </form>
  )}
</Confirm>
```

整个确认处理都在`Confirm`组件内部，我们只是将`handleSubmit`事件处理程序作为回调传递给`confirm`处理程序。

最棒是，我们可以使用这个 render prop 组件来拦截任何 React 事件，比如按钮点击:

```
<Confirm>
  {**confirm** => (
    <button onClick={**confirm**(launch)}>Launch!</button>
  )}
</Confirm>
```

API 看起来很好很干净，那么有什么问题呢？合成事件的汇集。

# 履行

实现很简单。对于一个模态，我们将使用 Ryan Florence[制作的](https://medium.com/u/162352c45b6e?source=post_page-----f50a40d9a30d--------------------------------) [@reach/dialog](https://ui.reach.tech/dialog) 。首先，我们用`openConfirmationDialog`处理程序调用子进程，然后有条件地在它旁边呈现对话框:

```
<React.Fragment>
  {this.props.children(this.openConfirmationDialog)} {this.state.open && (
    <Dialog>
      <h1>{this.props.title}</h1>
      <p>{this.props.description}</p> <button onClick={this.hideConfirmationDialog}>Cancel</button>
      <button onClick={this.confirm}>OK</button>
    </Dialog>
  )}
</React.Fragment>
```

将[孩子作为函数](https://reactjs.org/docs/render-props.html)调用是一种常见的 React 模式。您可以将它视为局部高阶组件，而不是包装整个组件，您只需将呈现方法的一部分传递给它。

我们有一个呈现方法，让我们开始实现事件处理程序。

## 合成事件的汇集

如果您曾经试图在异步处理程序中传递 React 事件，您会在控制台中得到一个很好的错误，这是不可能的，因为 React 事件对象是重用的。这意味着当你试图在事件被触发之后访问它，它已经是一个完全不同的事件了。

这是很好的[文档](https://reactjs.org/docs/events.html#event-pooling)，它可能会在未来的版本中被删除，但现在我们必须处理它。

```
openDialogModal = callback => event => {
  // prevent default event action, e.g: form submission
  event.preventDefault() // we need to destructure the event
  // and get the current target value (e.g: select value
  event = {
    ...event,
    target: { ...event.target, value: event.target.value }
  } this.setState({
    open: true, // save original callback with event in closure
    callback: () => callback(event)
  })
}
```

这就是 *hackish* 的部分。

如果你看了文件，有一个提示:

> 如果您想以异步方式访问事件属性，您应该在事件上调用`event.persist()`，这将从池中移除合成事件，并允许用户代码保留对事件的引用。

不幸的是，`event.persist()`对阅读`event.target.value`不起作用。不知道为什么，但即使我没有得到任何关于事件被重用的错误或警告，它仍然返回当前的目标值。看起来，如果您拦截了将更新输入值的事件，并试图稍后读取该值，它将从当前 DOM 值中读取该值。

总之，回到例子。

## 剩余处理程序

两个失踪的经手人就简单多了。

第一个是`this.confirm`，它应该用原始事件调用原始处理程序并关闭模态:

```
confirm = () => {
  this.state.callback()
  this.hide()
}
```

我们用`openConfirmationModal`中的`event`解决了这个问题，所以现在我们只调用存储在闭包中的事件并隐藏模态。这将我们带到最后一个事件，它只是一个回调和关闭模式的清理:

```
hide = () => this.setState({ open: false, callback: null })
```

# 例子

如果你想尝试一下，这里有一个沙盒:

确认组件示例

有一个向表单提交添加确认步骤、更改选择值和单击按钮的例子。

# 结论

首先，我对这个简单的解决方案感到兴奋，它保持了组件的整洁，并且不需要对事件处理逻辑进行任何更改。

然后我发现了事件池的问题，但我可以忍受。

你有什么看法？有没有更好的方法坚持`event.target.value`？或者更好的处理确认的方法？请在这里或在 Twitter 上告诉我:

嘿！我是[全栈开发人员](https://tomasehrlich.cz/)，在空闲时间，我正在开发 [LinguiJS](https://github.com/lingui/js-lingui) ，这是一个面向 JavaScript 的 i18n 库。该库从头到尾解决了 i18n，提供了用于翻译的组件、用于管理消息目录的 CLI 以及离线编译消息以节省包大小。它是 react-intl 的替代方案，但也适用于 [vanilla JS](https://t.co/NigOTsQhaY) 。看看 [React 教程](https://lingui.github.io/js-lingui/tutorials/react.html)或者说[和 react-intl](https://lingui.github.io/js-lingui/misc/react-intl.html) 有什么不同。

如果你对 i18n 感兴趣，可以看看本周发布的 LinguiJS v2.7，它包含了用于国际化的宏(由 [Kent C. Dodds](https://medium.com/u/db72389e89d8?source=post_page-----f50a40d9a30d--------------------------------) 编写的 [babel-plugin-macros](https://github.com/kentcdodds/babel-plugin-macros) 的可插拔编译时插件):

如果你对我做的事情和写的文章感兴趣，请在 Twitter 上关注我:

[](https://twitter.com/tomas_ehrlich) [## 托马斯·埃利希(@托马斯 _ 埃利希)|推特

### 托马斯·埃利希的最新推文(@托马斯 _ 埃利希)。#铁人三项运动员，网站和移动应用程序开发人员，著有……

twitter.com](https://twitter.com/tomas_ehrlich)