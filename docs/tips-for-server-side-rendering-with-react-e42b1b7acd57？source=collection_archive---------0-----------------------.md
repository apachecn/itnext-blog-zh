# 使用 React 进行服务器端渲染的技巧

> 原文：<https://itnext.io/tips-for-server-side-rendering-with-react-e42b1b7acd57?source=collection_archive---------0----------------------->

![](img/a6a0522ea351d9e00f78fcf9f79d7d0b.png)

嗨，今天我想分享一些当你用 [React](https://reactjs.org) 开发服务器端渲染应用的技巧。

## Web API 用法

使用 [Web API](https://developer.mozilla.org/en-US/docs/Web/API) 时要小心，因为它们不能在服务器端工作。这些是我们通常使用常见 Web API:

*   [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) (文档)
*   [窗口](https://developer.mozilla.org/en-US/docs/Web/API/Window)(窗口、[导航器](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)等)
*   [存储](https://developer.mozilla.org/en-US/docs/Web/API/Storage)(本地存储、会话存储等)

避免在这些生命周期中使用它们，因为它们是在服务器渲染期间调用的:

*   构造器
*   [getDerivedStateFromProps](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
*   组件将挂载(已弃用)

你可以在`[componentDidMount](https://reactjs.org/docs/react-component.html#componentdidmount)`和`[componentDidUpdate](https://reactjs.org/docs/react-component.html#componentdidupdate)`生命周期以及在`componentDidMount`阶段之后运行的任何类方法中安全地完成它。

> 我假设我们使用`[renderToString](https://reactjs.org/docs/react-dom-server.html#rendertostring)`和`[renderToNodeStream](https://reactjs.org/docs/react-dom-server.html#rendertonodestream)` API 进行服务器端渲染。[预渲染库](https://prerender.io/)不算做 SSR，因为只有客户端的生命周期仍然会被执行

所以，你可能会想:

> "但是我可以查一下`window`是否有空吗？"

嗯，看情况，但是**不要**这么做！

```
constructor(props) {
  super(props)
  this.state = {
    data: typeof window !== 'undefined' ? 'client' : 'server'
  }
}render()
  return <div>{this.state.data}</div>
}
```

您将在浏览器控制台上看到以下警告:

```
index.js:2178 Warning: Text content did not match. Server: "server" Client: "client"
```

那么这里出了什么问题呢？这是因为 React 期望在水合过程中，在**服务器**上呈现的那个**应该与在**客户端**上呈现的**相同。在进行 DOM 访问/操作之前使用窗口类型检查是可以的，但是要确保它不会影响客户端和服务器之间的渲染差异。

好吧，现在你可能会问:

> “但我需要基于窗口/浏览器 API 进行渲染……”

如果在某些情况下，您需要基于 DOM 对象或 Web APIs 进行渲染，那么您需要在`componentDidMount`生命周期之后进行渲染，并且在服务器上，您可以渲染占位符(加载组件，或者仅仅是`null`)

```
constructor(props) {
  super(props)
  this.state = {
    ssrDone: false
  }
}componentDidMount() {
  this.setState({ ssrDone: true, online: navigator.onLine })
}render() {
  if(!this.state.ssrDone) {
    return (
      <div>loading...</div>
    )
  }
  return <div>{this.state.online ? 'on' : 'off'}</div>
}
```

这在您使用非 SSR 友好的第三方 React 组件时尤其有用(这将在服务器渲染期间引发错误)。[官方 React 文档](https://reactjs.org/docs/react-dom.html#hydrate)也展示了相同的技术(名称为“两遍渲染”)，但它也声明:

> 请注意，这种方法会使您的组件变慢，因为它们必须渲染两次，所以请谨慎使用

确保**封装了**这些依赖于 DOM 的组件，这样双重渲染只会发生在它们身上，而不是整个页面。

## 水合警告

如果你收到一些水合警告，出于某种原因想忽略它们，你可以在元素上使用`[suppressHydrationWarning](https://reactjs.org/docs/dom-elements.html#suppresshydrationwarning)`道具。

## 奖金

使用`ReactDOM.render()`来水合服务器渲染的容器是不赞成的，并将在 React 17 中**移除**。用`[hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)`代替。(摘自 [React 官方文件](https://reactjs.org/docs/react-dom.html))

我想这些都是我的，希望对你们有用。

如果你有任何问题或想法，请发帖回复，如果你喜欢，请**拍手分享**给你的朋友！

谢谢你。