# react-router 中的命名 URL

> 原文：<https://itnext.io/named-urls-in-react-router-7949ff71f41d?source=collection_archive---------3----------------------->

![](img/edf876da7a4d79cd2433e3ecbd1f8d4e.png)

照片由 [Einar H. Reynis](https://unsplash.com/photos/CgSeT8__Wgw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/search/photos/signpost?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

react-router 是我最喜欢的 react 路由器组件，但是我真的错过了 Django 的一个特性:命名 url 模式。它们是什么，如何有用？

看看 react-router 文档中的任何例子:

```
import { Switch, Route } from 'react-router'

<Switch>
  <Route exact path="/" component={Home}/>
  <Route path="/about" component={About}/>
  <Route path="/:user" component={User}/>
  <Route component={NoMatch}/>
</Switch>
```

路径被定义为字符串，当我们想要呈现一个链接时，例如到`About`页面的链接，我们需要再次写入路径:

```
<Link to="/about">About</Link>
```

拼错 URL 很容易出错，重构 URL 很难，因为 URL 在几个文件中是重复的。

这里是命名的 URL。我们在一个地方定义所有的 URL，然后使用标识符而不是字符串。

首先我们需要一个`routes.js`文件来定义所有的 URL:

```
export default {
  home: "/",
  about: "/about",
  user: "/:user"
}
```

我们使用此配置来定义路由:

```
import { Switch, Route } from 'react-router'
import routes from './routes.js'<Switch>
  <Route exact path={routes.home} component={Home}/>
  <Route path={routes.about} component={About}/>
  <Route path={routes.user} component={User}/>
  <Route component={NoMatch}/>
</Switch>
```

和相同的配置来创建链接:

```
<Link to={route.about}>About</Link>
```

静态分析工具可以检查我们使用的是有效的路线，而且也是干的。

唯一的问题是带参数的路线，比如`/:user`。当创建链接时，我们需要向这个模式传递参数。这就是命名 url 库发挥作用的地方:

[](https://github.com/tricoder42/named-urls) [## tricoder 42/命名网址

### 命名 url——Javascript 中简单的命名 URL 模式

github.com](https://github.com/tricoder42/named-urls) 

named-urls 库导出`reverse`函数，该函数使用给定的参数将 url 模式转换为特定的 url:

```
<Link to={reverse(route.user, { user: "tricoder42" })}>About</Link>// reverse("/:user", { user: "tricoder42" }) === "/tricoder42"
```

就是这样！它很简单，完全独立于 react-router，但给你安全和干燥的 url 模式。这个库本身不到 100 行，所以没什么大不了的。如果你想知道代码分解是如何工作的，那么它不是。`routes.js`文件通常很小，考虑到这项任务的复杂性，保存的包大小不会很大。

您如何在 React 或 express 应用程序中管理路线？

嘿！我是一名全栈开发人员，在我的空闲时间里，我正在开发用于 JavaScript 的 i18n 库 [jsLingui](https://github.com/lingui/js-lingui) 。该库从头到尾解决了 i18n，提供了用于翻译的组件、用于管理消息目录的 CLI 以及离线编译消息以节省包大小。它是 react-intl 的替代方案，但也适用于普通 JS。看看 [React 教程](https://lingui.github.io/js-lingui/tutorials/react.html)或者说[和 react-intl](https://lingui.github.io/js-lingui/misc/react-intl.html) 有什么不同。

以下是您可能感兴趣的其他文章:

[](/react-patterns-lambda-components-and-render-props-c4dce3903a52) [## 反应模式—lambda 组件和渲染道具

### 观察开发人员如何在 React 中不断发现新的组件组合模式是很棒的。一些…

itnext.io](/react-patterns-lambda-components-and-render-props-c4dce3903a52) [](/react-patterns-lambda-components-and-render-props-c4dce3903a52) [## 反应模式—lambda 组件和渲染道具

### 观察开发人员如何在 React 中不断发现新的组件组合模式是很棒的。一些…

itnext.io](/react-patterns-lambda-components-and-render-props-c4dce3903a52) 

如果你对我做的事情和写的文章感兴趣，请在 Twitter 上关注我:

 [## 托马斯·埃利希(@托马斯 _ 埃利希)|推特

### 托马斯·埃利希的最新推文(@托马斯 _ 埃利希)。#铁人三项运动员，网站和移动应用程序开发人员，著有……

twitter.com](https://twitter.com/tomas_ehrlich)