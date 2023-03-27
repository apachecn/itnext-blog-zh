# 创建一个自定义的 React 钩子来跟踪浏览器窗口宽度(用 TypeScript)

> 原文：<https://itnext.io/create-a-custom-react-hook-for-tracking-browser-window-width-with-typescript-6211901e9f9d?source=collection_archive---------11----------------------->

![](img/78c64429863071b13d4057e6dacf99cb.png)

最近我一直在使用定制的 React 钩子来清理我的代码库，它为我创造了奇迹！我已经设法将如此复杂的重复代码转移到可重用的定制钩子中。

在这篇文章中，我将通过一个例子来说明我是如何使用一个定制的钩子来跟踪我的窗口浏览器窗口的。我非常支持使用 TypeScript，所以我当然会在我的例子中使用 TypeScript。

# 在实现挂钩之前

对于那些不熟悉定制钩子的人来说，掌握它们的一个简单方法是识别它们解决的问题。这就是我们如何在不使用自定义钩子的情况下实现我们的功能。

假设我们有一个组件，我们想根据浏览器窗口的宽度来呈现一个不同的组件。

举个例子，

*   如果窗口宽度为`800px`或更大，那么我们想要渲染`Desktop`组件。
*   如果窗口宽度小于`800px`，那么我们要渲染`Mobile`组件。

这里有一个简单的解决方案:

```
import React, { useEffect, useState } from 'react'const MyComponent: FC = () => {
  const [width, setWidth] = useState(window.innerWidth)

  const handleResize = () => setWidth(window.innerWidth) useEffect(() => {
    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, []) return width < 800 ? <Mobile /> : <Desktop />
}
```

我们来破解这个密码，好吗？

在这个组件中，我们将窗口宽度存储在一个由 React 的`setState`变量存储的`width`变量中。

这个`setState` `width`值通过`setWidth`回调函数设置，该回调函数由`handleResize`函数调用。

我们使用`useEffect`向`window`添加一个事件监听器，每当调整窗口大小时，这个监听器就调用`handleResize`。这个事件侦听器是在组件挂载时添加的。当组件被卸载时，我们通过删除事件侦听器来进行清理。

最后，我们执行一个简单的三元语句，根据总是在调整窗口大小时更新的`width`值来决定呈现哪个组件。

这是一个非常基本的例子，但这样做是为了让我们能够专注于重要的内容，过滤掉不相关的信息。

# 这种做法有什么问题？

没什么…

是的，如果你只打算在这一个地方使用它，这没什么不好。但是，如果您想在项目的其他地方使用同样的东西，您将遇到以下两个问题:

*   您需要在想要使用该特性的每个组件中编写同样长的 6 行设置代码。
*   您将为使用该方法的每个组件设置一个新的事件侦听器。

如果我们有一些易于使用的可重复使用的解决方案来解决这些问题，那不是很好吗？

感谢 React 的一些便利特性，我们可以提出一个解决方案，每个组件只有一行设置，并且只添加一个所有组件都使用的事件侦听器。

# 解决方案

我先给你展示一下简单的实现，这样你就可以知道我们需要构建什么来完成我们的目标。

对解决方案进行编码后，我们将在另一个文件中开发一个`useViewport`钩子，工作方式如下:

```
import React, { FC } from 'react'
import { useViewport } from './use-viewport'const MyComponent: FC = () => {
  // See everything simplified to this one line.
  const { width } = useViewport() return width < 800 ? <Mobile /> : <Desktop />
}
```

简单多了，对吧？现在让我们创建我们的`useViewport`钩子并实现它！

为了防止多个事件侦听器被订阅，我们将利用 React 的上下文 API。

下面是`use-viewport`文件的完整实现:

```
import React, {
  createContext,
  FC,
  useContext,
  useEffect,
  useState,
} from 'react'interface IViewport {
  width: number
}const ViewportContext = createContext<IViewport>({
  width: window.innerWidth,
})export const ViewportProvider: FC = ({ children }) => {
  const [width, setWidth] = useState(window.innerWidth) const handleResize = () => setWidth(window.innerWidth) useEffect(() => {
    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, []) return (
    <ViewportContext.Provider value={{ width }}>
      {children}
    </ViewportContext.Provider>
  )
}export function useViewport() {
  return useContext<IViewport>(ViewportContext)
}
```

这里发生了很多事情。我们本质上要做的是将我们的整个应用程序包装在这个`ViewportProvider`中，这样事件监听器订阅只发生一次。

然后我们创建自定义的`useViewport`钩子来轻松访问组件中的宽度值。如果你不熟悉 React 的上下文 API，那么我强烈建议你阅读 React 的官方文档，以便更好地掌握它。虽然就我个人而言，我在构建定制钩子时通过使用上下文 API 学到了很多东西。

阅读上面的代码时，最重要的部分如下:

*   `ViewportContext`是使用 React 的`createContext`函数创建的。
*   `ViewportContext`只保存值`width`，以便我们创建的`useViewport`钩子可以使用它。对于我们的钩子实现来说，其他一切都不重要。
*   在`ViewportProvider`组件的后台更新`width`值。

# 最后的步骤

这部分很重要！我们的`useViewport`挂钩暂时还不能使用。我们需要在我们的应用程序中实现`ViewportProvider`组件，以便 hook 的实现实际上可以访问上下文。记住:React context——就像 Redux 一样——需要一个提供者。

为此，我建议在 React `App`的根附近实现。

类似这样的内容应该足够了:

```
import React from 'react' 
import ReactDOM from 'react-dom'import App from './app'
import ViewportProvider from './use-viewport'ReactDOM.render(
  <ViewportProvider>
    <App />
  </ViewportProvider>,
  document.getElementById('root')
)
```

就是这样！现在你应该可以轻松地在你的应用中使用你的`useViewport`钩子了🙂

希望你喜欢这本书，这对你的项目有所帮助，无论是大项目还是小项目。

PS:我把它叫做`useViewport`而不是`useWindowWidth`，因为我扩展了钩子的功能。我将很快写一篇第 2 部分的文章，包括一些关于如何使钩子更方便使用的改进。

*原载于 2020 年 4 月 13 日*[*【https://www.barrymichaeldoyle.com】*](https://www.barrymichaeldoyle.com/use-viewport/)*。*