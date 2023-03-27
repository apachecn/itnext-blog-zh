# 不要在 React ref 回调中使用内联函数或 bind()

> 原文：<https://itnext.io/dont-use-inline-functions-or-bind-in-react-ref-callbacks-5559e4342ead?source=collection_archive---------1----------------------->

![](img/ea1c73088ca5f639dfd7a5f279feb2b2.png)

rawpixel.com 在 [Unsplash](https://unsplash.com/search/photos/polaroid?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上[拍照](https://unsplash.com/photos/2CilBNsOC3Q?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

React 使得*几乎*永远不用担心底层的 HTML/JavaScript 文档对象模型。React 的保留模式呈现允许你以声明的方式定义网页上的一切，包括交互和呈现状态。太棒了！

但是每隔一段时间，你都需要回到 DOM 来管理状态[命令](https://xkcd.com/149/)。您可能需要选择页面上的一些文本，或者管理表单元素的焦点。对于这些情况，React 提供了 [ref 回调](https://reactjs.org/docs/refs-and-the-dom.html)。

下面是一个非常简单的 React 组件类的例子，它使用了 ref 回调特性。

```
Import { Component } from ‘react’;class ExampleComponent extends Component {
  setFocus() {
    this.textInput.focus();
  } render() {
    return (
      <input type=”text”
             ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

这个例子是这样工作的:当`<input>`元素被呈现时，React 调用在`ref`属性中定义的函数，将`<input>`元素作为参数传递给该函数。

这与你在 React 官方文档和大多数关于 React 引用的教程中找到的例子非常相似。然而，如果你想做任何更复杂的事情，而不仅仅是保存对元素/组件的引用，**你不应该使用这种模式**。

我们使用[Daily.co](https://www.daily.co/)代码库中的引用来管理媒体和画布元素。我们经常需要在第一次安装一个元素时做一些事情。大概是这样的:

```
class CanvasWithAttitude extends Component {
  setupToDraw(canvas) {
    // ...
  }render() {
    return (
      <canvas ref={(canvas) => { this.setupToDraw(canvas); }} />
    );
  }
}
```

除了我们**不做那个**，因为用上面的代码，每次渲染时`setupToDraw()`函数实际上会被调用**两次。这绝对不是我们想要的。**

每次组件呈现时，React 都需要重新创建任何被定义为元素属性的内联或绑定函数。为了清除旧的引用，它首先用一个`null`参数调用内联或绑定函数。然后，它使用元素/组件参数再次调用该函数。(关于官方 React 文档中对此的解释，向下滚动到 [Refs 和 DOM 页面](https://reactjs.org/docs/refs-and-the-dom.html)的最底部。)

请注意，我写的是“内联或绑定”，所以这也是不对的:

```
<canvas ref={ this.setupToDraw.bind(this); } />
```

我们的目标是当组件挂载时，ref 回调被调用一次。

一种解决方法是使用 [ES7 类属性语法](https://babeljs.io/docs/plugins/transform-class-properties/)来定义函数。

```
class CanvasWithAttitude extends Component {
  setupToDraw = (canvas) => {
    // ...
  } render() {
    return (
      <canvas ref={ this.setupToDraw } />
    );
  }
}
```

这将为每个`CanvasWithAttitude`实例创建一个新的`setupToDraw()`函数，并将该函数赋给对象的`setupToDraw`属性。实际上，我们已经将内联函数定义移出了渲染函数，并让 JavaScript transpiler 在实例创建时不可见地为我们完成了这项工作。:-)

我们的许多类中都有这样的代码，但我不喜欢这种模式。JavaScript 对象模型中有很多极端情况，我发现很容易忘记上面的代码做了一些不明显的事情。首先，为该类的每个新实例创建了`setupToDraw()`函数，其次，该函数不作为类原型的一部分存在(例如，因此它不会被继承)。

我更喜欢这样写这个模式。

```
class CanvasWithAttitude extends Component {
  constructor() {
    super();
    this.setupToDrawBound = this.setupToDraw.bind(this);
  } setupToDraw(canvas) {
    // ...
  } render() {
    return (
      <canvas ref={ this.setupToDrawBound } />
    );
  }
}
```

这里，`setupToDrawBound()`仍然是一个新函数，每次构造实例时都会创建一个，但是`setupToDraw()`是一个普通的成员函数，在扫描代码时很容易推断出`bind()`的用法。最后，`setupToDraw()`和`setupToDrawBound`都被任何扩展`CanvasWithAttitude`的类继承。

我敢肯定，有其他的模式来做奇特的事情与反应参考。当一个组件被安装时，也有其他的方法来组织做一些事情。(例如，我们可以挂钩 React `componentDidMount()`生命周期方法。)我很想听听其他 React 开发者的想法——尤其是有趣用例的例子！