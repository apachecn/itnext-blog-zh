# React 生态系统指南

> 原文：<https://itnext.io/react-ecosystem-guide-4a5f85d17623?source=collection_archive---------2----------------------->

![](img/d5a7602e143b694540073bc3f5f72284.png)

为了能够找到一份体面的工作，需要学习哪些图书馆和东西？在这篇文章中，我试图解释你需要学习什么以及为什么。本文对每种工具都有两个列表:基础和高级。Basic 是给那些想知道如何使用一个工具而没有深入理解的人用的。对于不仅想工作和使用工具，而且知道它如何工作和为什么工作的人来说是高级的。我在这里只描述工具，你还需要学习 ES2015(6，7，8)才能使用 React。

[**反应过来**](https://reactjs.org/tutorial/tutorial.html) 是必须有的。不要认为我需要解释为什么你需要阅读和学习它。

**基础**:

1.  [组件状态和道具](https://reactjs.org/docs/faq-state.html)。
2.  [组件的生命周期。](https://reactjs.org/docs/react-component.html)
3.  [高阶组件](https://reactjs.org/docs/higher-order-components.html)。

**高级**:

1.  [单向数据流向](https://reactjs.org/docs/thinking-in-react.html)。
2.  F [勒克斯](https://facebook.github.io/flux/docs/in-depth-overview.html)。
3.  虚拟世界。
4.  生态循环。
5.  [反应元件和反应元件的区别](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)。

如果你想作为一个进步的 react 开发者，你需要理解 React 背后的东西，这些事情是基本的。它不仅仅是一个 UI 库，它还是一种思想。

[**Redux**](https://redux.js.org/) 是 React 应用程序的数据管理器。假设你需要获取数据并保存在某个地方，你总是可以使用组件状态，但有时你的应用程序变得越来越大，使用组件状态变得不方便，因为你需要在几个组件之间共享数据。

**基本:**

1.  [动作](https://redux.js.org/basics/actions)。
2.  [减速器](https://redux.js.org/basics/reducers)。
3.  [储存](https://redux.js.org/basics/store)。

**高级:**

1.  [核心概念](https://redux.js.org/introduction/core-concepts)。
2.  [三原则](https://redux.js.org/introduction/three-principles)。
3.  [数据流](https://redux.js.org/basics/data-flow)。
4.  [中间件](https://redux.js.org/advanced/middleware)。
5.  [异步动作](https://redux.js.org/advanced/async-actions)。
6.  [重新选择](https://github.com/reactjs/reselect)。缓存从存储中获取数据。
7.  [Redux act](https://github.com/pauldijou/redux-act) 。创建 redux 操作和 reducers 的助手。

我强烈推荐丹·阿布拉莫夫(redux 的作者之一)的这个[教程](https://egghead.io/courses/getting-started-with-redux)。

[**React-路由器**](https://reacttraining.com/react-router/) 是 React 应用的路由器。RR 有 4 个不同的版本，最后一个版本(4)在概念上与其他版本不同。我建议学习 RR 4，因为所有新项目都使用它，旧项目正在从以前的版本更新到最新版本。

**基本**:

1.  [简单教程](https://medium.com/@pshrmn/a-simple-react-router-v4-tutorial-7f23ff27adf)。

**高级**:

1.  [哲学](https://reacttraining.com/react-router/web/guides/philosophy)。
2.  [浏览器路由器](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/modules/BrowserRouter.js)。
3.  [内存路由器](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/modules/MemoryRouter.js)。
4.  [静态路由器](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/modules/StaticRouter.js)。
5.  [冗余路由器](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-redux/modules/ConnectedRouter.js)。

React 路由器是一个非常简单的工具。你只需要设置它，它真的很容易。

[**反应-国际**](https://github.com/yahoo/react-intl) 。国际化 react 应用程序。

**基础**:

1.  [入门](https://github.com/yahoo/react-intl/wiki#getting-started)。

**高级**:

1.  [国际化 API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) 。

[**重组**](https://github.com/acdlite/recompose) 。是一个有许多 hoc(高阶组件)的库。当您需要在您的哑组件中有一个状态，但您并不真的想把它变成 smart 时，使用它真的很方便。

[](https://redux-form.com/7.3.0/docs/gettingstarted.md/)**。是最受欢迎的表单生成器。它有很多很酷的特性，比如同步和异步验证，当表单提交时调度不同的动作等等。它还有两个不同的概念(redux-form < 5，redux-form > 5)。我也建议学习最新的。**

****基本**:**

1.  **[Api](https://redux-form.com/7.3.0/docs/api/) 。**
2.  **[例题](https://redux-form.com/7.3.0/examples/)。**

****高级**:**

1.  **值 l [如果循环](https://redux-form.com/7.3.0/docs/valuelifecycle.md/)。**

****网络包**。Webpack 是一个构建器。它也有 4 个不同的版本。我会推荐现在学习第 3 版，因为第 4 版刚刚出来，它还不牢固。您需要学习 webpack，因为您需要了解您的应用程序是如何构建的。你可以随时选择 [create-react-app](https://github.com/facebook/create-react-app) ，但是它可以做任何事情，如果你需要在你的应用构建中做一些改变，你会发现 webpack 知识非常有用。**

**没有必要将链接分成两个不同的组。你需要学的就是这里的[这里的](https://webpack.js.org/concepts/)。**

**[玩笑](https://facebook.github.io/jest/)。测试对学习非常重要。Jest 有一个专门测试 UI 的工具。它呈现 react 组件并将其输出保存到文件中。下次运行测试时，Jest 会将文件的输出与组件的输出进行区分。这是 e2e 测试。当您需要测试无论何时更改项目中的某些内容，UI 都能正确呈现时，这非常方便。**

****基本**:**

1.  **[快照测试](https://facebook.github.io/jest/docs/en/snapshot-testing.html)。**
2.  **[为 React 应用程序设置测试](https://facebook.github.io/jest/docs/en/tutorial-react.html)。**

****高级**:**

1.  **[匹配器](https://facebook.github.io/jest/docs/en/using-matchers.html)。**
2.  **[模拟功能](https://facebook.github.io/jest/docs/en/mock-functions.html)。**
3.  **[完整的 API 参考](https://facebook.github.io/jest/docs/en/api.html)。**

**还有另外一个测试 react 组件的库:[酶](https://github.com/airbnb/enzyme)。它也可以和笑话一起用，但通常，人们先用笑话。如果你需要更多的能力来渲染你的组件，那么你需要使用酶，否则，Jest 就足够了。**

**现在 react world 中有更多的工具，如果你学会了我上面描述的所有工具，你可能会对 GraphQL、Relay、React-Native 以及函数式编程风格、airbnb 代码风格、eslint、prettier 感兴趣。**

**这个列表会随着时间的推移而改变。如果你有什么东西要我放在这里，请留下私人便条。**