# 这到底是在反应什么？🥁🥁(Suspense) 🥁🥁

> 原文：<https://itnext.io/what-the-heck-is-this-in-react-suspense-c5e641e487a?source=collection_archive---------0----------------------->

暂停代码分割和数据获取。

![](img/adbaef074fd6517e0dcabf4b0c78f799.png)

诺德伍德主题公司在 [Unsplash](https://unsplash.com/search/photos/slow-loading?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

在 2018 年冰岛 JSConf 期间，[丹·阿布拉莫夫](https://twitter.com/dan_abramov)推出了 React 16 发布后的全新概念。在谈论资源加载时，他展示了一个*占位符*组件，呈现一个加载器。到现在一年了，这个组件已经演变成了 ***悬疑*** 。

*悬念*在 React 16.6 中已经发布，但仅针对代码拆分。让我们看看今天如何使用它，以及未来有哪些悬念。

# TL；速度三角形定位法(dead reckoning)

1.  [*暂停*](#7e67) 暂停你的组件渲染，渲染一个回退组件，直到满足一个条件。例如，它可以在等待资源时呈现加载程序，
2.  [现在- *暂挂*进行代码拆分](#4b8b):使用 React.lazy +动态导入
3.  [未来- *暂挂*获取数据](#ad68):你的组件抛出一个承诺，被*暂挂抓住，*等待它解决。等待时，*暂停*渲染一个回退组件。然后你的组件再次被渲染。

# 1.悬念是什么？

反应*暂停*允许你暂停组件渲染，直到满足一个条件。等待时，会呈现回退组件。基本上有两个主要的使用案例:

*   代码分割:条件是当用户想要访问你的应用程序时，下载它的一部分，
*   数据抓取:条件是数据的下载。

在这两种情况下，后备组件很可能是加载程序。

要实现这一点，您需要用*暂停*组件包装您将要暂停的组件。

悬念用法示例

在上面的例子中，可以暂停*口袋妖怪列表*组件渲染，等待口袋妖怪列表。在暂停期间，渲染一个*加载*组件。

加载程序不会立即呈现。为了获得更好的用户体验，React 在渲染回退之前会增加 100 毫秒的延迟。当资源很少或网络很快时，这避免了闪烁，在加载程序之后立即呈现真正的组件。

我们来看看如何用*悬念*进行代码拆分和数据取数。

# 2.代码拆分的当前悬念

**什么是代码分割？**

代码分割是一种通过创建多个块而不是一个大的唯一块来减少 javascript 包大小的技术。

一种方法是使用打包器，如 [Webpack](https://webpack.js.org/) 或 [Parcel](https://parceljs.org/) ，它们会根据[动态导入](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Instructions/import#Imports_dynamiques)的用法自动将包分成块。

动态导入示例

在上面的例子中，捆绑器将创建一个块，包含 *PokemonList* 代码(及其导入的依赖项)。在运行时，它将下载块，并返回一个承诺。这个承诺解决口袋妖怪列表组件，这是默认的出口。

**如何利用悬念展示一个装载机？**

React 附带了对惰性组件的支持。 *React.lazy* 允许您像使用静态导入组件一样使用动态组件，而不是处理动态导入承诺。

让我们看一些代码。

代码拆分示例的暂记

*React.lazy* 采用一个将执行动态导入的函数。它返回一个组件，该组件将在第一次呈现时运行该函数。得到的承诺状态被*暂记*使用

*   在块下载期间呈现回退(待定承诺)
*   下载后渲染真实的*口袋妖怪列表*组件(解决承诺)

我们可以看到使用*悬念*和 *React.lazy* 进行代码拆分的真正好处。代码在异步时感觉是同步的，我们不必编写大量样板文件来管理动态导入承诺及其组件使用。

这代表了*悬念*的稳定部分。但是它并不局限于代码分割。React 核心团队正在使用*悬念*获取数据。让我们看看它可能是什么样子。

# 3.数据获取的未来悬念

*请记住，数据提取的悬念仍在开发中，api 可能会改变。*

## **无*暂停*** 的数据提取加载器

让我们看一个没有*悬念*如何处理数据获取加载器的例子。要做到这一点，让我们更深入地了解我们的*口袋妖怪列表*组件。我们需要

1.  启动 *isLoading* 状态，告知正在获取数据，
2.  从缓存中读取数据，如果不存在，我们将获取它，相应地设置*正在加载*状态，
3.  根据*正在加载*状态，渲染*加载*或*列表*组件。

PokemonList 组件在获取数据时处理加载渲染

这里是一个简单的口袋妖怪服务的实现。没什么复杂的， *get()* 函数获取数据，并设置缓存值，这是通过 *readCache()* 函数公开的。

PokemonService 简单实现

我们已经可以在我们的组件中处理 loader，那么为什么我们将来还要使用*悬念*？

## **带*暂挂*** 的数据提取加载器

让我们看看如何在*悬念*中实现同样的行为。我们将从缓存中读取数据。但是如果它不在缓存中，我们就抛出一个获取数据的承诺。

如果缓存为空，则从缓存中读取会抛出一个承诺

只要记住*悬念*是我们的组件祖先之一

笼罩在悬念中的口袋妖怪

*悬念*使用[反应错误边界](https://reactjs.org/docs/error-boundaries.html)捕捉抛出的承诺，并为我们设置一个加载状态，渲染回退组件(*加载*)。当它被解析后，*口袋妖怪列表*组件会被再次渲染。

我们可以在*口袋妖怪列表*代码中看到好处。

不再有*加载*状态来初始化和设置，不再有*使用 Effect* 来获取数据，代码只是感觉同步。*悬念*处理所有异步状态，协调组件渲染。

让它工作的最后一步，我们需要改变呈现 React 应用程序的方式，以支持异步功能。

使用异步功能进行反应渲染

做出捡球的承诺看起来很奇怪。如果它困扰你，React 核心团队正在研究 [React-cache](https://github.com/facebook/react/tree/master/packages/react-cache) 来管理资源。

## 通过暂停和反应缓存获取数据

*请记住，react-cache 仍在开发中，lib 不稳定，api 会发生变化。*

*React-cache* 允许你创建与悬念兼容的资源用于数据获取。让我们看看如何用 *react-cache* 实现我们的 *PokemonService* 。

使用 react-cache 的口袋妖怪服务

让我们调整我们的 *PokemonList* 组件来适应这个新的服务 api。

使用新口袋妖怪服务的口袋妖怪列表

在引擎盖下， *react-cache* 将做我们的旧服务正在做的事情

1.  从缓存中返回数据，
2.  如果缓存不包含数据，它将抛出我们在创建资源时设置的获取承诺，
3.  *悬疑*会抓住它，处理加载器渲染。

# 结论

*悬念*帮助我们协调异步资源与我们的加载器组件渲染。我们已经可以用它来进行代码分割了。

当数据获取的*悬念*稳定后，我们将能够**专注于我们的业务特性**。*悬念*会替我们做，而不是一边取数据一边写重复的加载器代码。

# 谢谢你

非常感谢[安德烈亚斯·尼尔森](https://www.linkedin.com/in/andreas-nilsson333/)、[杨奇煜·拉西尼耶](https://twitter.com/frassinier)、[塞巴斯蒂安·罗曼](https://twitter.com/romainseb)和[塞巴斯蒂安·勒·穆伊尔](https://www.linkedin.com/in/sebastienlemouillour/)的校对！

# 参考

*   [丹·阿布拉莫夫在 JSConf 冰岛 2018 上的演讲](https://www.youtube.com/watch?v=nLF0n9SACd4)
*   杰瑞米·华格纳和艾迪·奥斯马尼的代码分解
*   [React 代码拆分文档](https://reactjs.org/docs/code-splitting.html)
*   [React 错误边界文档](https://reactjs.org/docs/error-boundaries.html)
*   [React-cache # create _ resource 实现](https://github.com/bvaughn/react/blob/a4603a04ab423d7a11f6c7cc1165fc1b03157e7e/packages/react-cache/src/ReactCache.js#L142)
*   [我的 github 存储库上的悬念示例](https://github.com/jsomsanith/demo-suspense)

# 关于我

我是前端建筑师@Talend。我参与的是[故事书](https://github.com/storybookjs/storybook)，是 [a11y 插件](https://github.com/storybookjs/storybook/tree/next/addons/a11y)，是[设计系统](https://github.com/storybookjs/design-system)。

在 Twitter[上关注我](https://twitter.com/jsomsanith)以获得关于 React 和可访问性的未来文章的通知:

*   [表单可访问性:实用指南](https://medium.com/@jimmy.somsanith/form-accessibility-a-practical-guide-4062b7e2dd14)