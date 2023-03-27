# 🔥微前端如何改变 Angular 的未来？

> 原文：<https://itnext.io/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad?source=collection_archive---------0----------------------->

让我们看看为什么 Angular 最适合微前端

![](img/9f1bb6944856a9abc981226417ed66ea.png)[](https://medium.com/subscribe/@easy-web) [## 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。

### 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。通过注册，您将创建一个中型帐户，如果您还没有…

medium.com](https://medium.com/subscribe/@easy-web) 

# **内容**

*   [简介 ](#dc6a)
*   [**微前端需要做什么？**](#1d51)
*   [**健壮的微前端架构**](#d2af)
*   [**微应用之间共享数据的方式**](#4b25)
*   [**为什么有棱角？**](#f5db)
*   [**如何建造？**](#743d)
*   [**结论**](#ad98)
*   [**了解更多**](#0bac)

# 介绍

每个组织在成长的同时，都面临着一系列的挑战来扩展产品:改组*团队*，选择正确的*技术栈*，项目间的*不一致*，以及*错误* *不容忍*。在现代世界中，我们将使用*微服务*来分离应用程序。它将完美地工作在服务器端，但是如何处理前端呢？它来了**微前端**，就像一个*微服务*，但它不拥有一个组件，而是拥有整个*业务* *子域*。让我们来看看它有什么好处，以及它将如何改变 Angular 的未来。

# 微前端有什么用？

**微前端** —是一种客户端架构设计，当单独的组件或页面托管在单独的域中，并集成到主**外壳应用**(主机应用)中。每个*微 app* 拥有整个业务子域，具有以下特征:

**归一队**所有。这并不意味着只有一个团队负责修复所有的 bug。更确切地说，这意味着每个子域都有一个*专用团队*，该团队携带关于业务逻辑和技术堆栈的域知识。隔离一部分业务是有好处的，单个人辞职不会对产品和业务有任何影响。你总是联系团队，而不是对业务子域有任何疑问的个人。团队中的每个人都必须通过文档和知识共享会议将个人知识传递给其他人。

**独立实现。**拥有*微应用*的团队拥有选择科技堆栈的全部自由。它可以是不同的框架、编码实践或架构。尽管如此，越自由-越灵活，强烈建议与拥有另一个*微应用*的其他团队结盟，至少在框架选择方面。通过这种方式，它将简化工程师在团队之间的转换，并使共享和集成带有可重用部件的库成为可能。

**独立部署。**每个*微应用* 都有需要单独部署，因为它们托管在单独的域中。它迫使你建立多个管道，这有时可能是一个挑战。首先，你需要注意顺序，以防一些微应用程序共享公共依赖、库、组件和实用程序。此外，确保每个管道尽可能地解耦。这将使并行运行它们成为可能，并充分提高构建速度。

**脱钩**。微应用程序必须尽可能地去耦合。理想情况下，它们之间不应该存在依赖关系。他们只能通过共享库共享数据，比如状态、UI 小部件、接口和 util 助手。这一部分需要在架构设计阶段进行额外的工作，以便构建适当的抽象。另一方面，重要的是找到平衡，而不是过度工程。

**容错。**微前端*架构最大的好处之一就是可靠性。如果其中一个*微 app* 坏了，另一个还能正常工作。这对大型 web 应用程序非常重要，因为一个应用程序有多个独立的大功能。让我们想象一下脸书，如果其中一个功能，比如群组，被关闭，用户仍然可以和朋友聊天。*

# 稳健的微前端架构

实现*微前端*架构的方法有哪些，有两种可能的方法:

**微前端作为组件。**在这种情况下，微 app 的粒度更大。它充当一个可跨应用程序重用的小部件。这种方法的问题是它使微应用程序耦合得更紧密。构建适当的抽象可能很复杂，并且很容易消除微前端架构的优势。

**微前端作为页面。**通过这种方式，我们为每个*微应用*设计了一个单独的页面。与之前的方法相比，它允许我们避免亲子间的紧密依赖。因此，这是实现微前端架构的最常见方式。

# 在微应用程序之间共享数据的方法

设计*微前端*的最大挑战之一是数据共享。数据可以有不同的类型，如**状态**、 **UI 组件、**或**实用函数。**确保您的微应用不包含**数据存储状态，**它必须被分离到一个可共享的数据层中。你可以把它想象成类似于 frontend——backend 通信，其中 frontend 是*微 app* ，back end(数据层)是可共享库。此外，UI 组件和实用程序也将和可重用的共享库一样。这个概念与 *monorepo* 原则紧密重叠。它通过共享数据来保证应用程序之间的一致性。

还有一些其他的微 app 之间的沟通方式我们需要提一下。首先，我们可以利用我们的浏览器存储能力。可能是 *localStorage* 、 *sessionStorage* 、 *cockies* 用于存储会话数据或用户元数据，也可能是 *indexedDB* 中一些更复杂的数据结构。除此之外，您还可以简单地通过路由器*查询参数*建立微应用之间的数据传输。

# 为什么有棱角？

Angular 提供了已经定义好的架构，完全符合 *monorepo* 原则和*微前端*最佳实践。

架构高层分为三大部分: ***工作区******项目*******库*** 。通过初始化一个角度 [*工作空间*](https://angular.io/cli/new#ng-new)——我们生成了一个 *monorepo* 应用程序的框架，它进一步由*项目*和*库组成。*每个 [*项目*](https://angular.io/cli/generate#application) 都可以是一个*微 app* ，它有一个单独的入口点，可以有一个灵活的、可定制的结构。那么 [*库*](https://angular.io/guide/libraries) 则是在项目之间共享可重用的数据，可能是我们的*数据层，* UI 组件，以及其他的实用函数。*

*有角的 [***模块***](https://angular.io/guide/architecture-modules) 是充当主要积木的。它们封装了整个特性、组件集或页面及其依赖项。这是将我们的微前端应用程序打包到其中的完美选择。*

*[***路由***](https://angular.io/guide/routing-overview) 特性自带 Angular 开箱即用，无需安装外部软件包。它提供了维持*微应用*之间页面转换所需的一切。 [*路由出口*](https://angular.io/guide/routing-overview) 将允许您配置*外壳 app* ，托管导航到微 app 的各个页面。您可以选择哪条路线将加载哪个角度模块。*

*最后是 [***懒加载***](https://angular.io/guide/lazy-loading-ngmodules#lazy-loading-feature-modules) ，在需要的时候加载有角度的*模块*，减少 app 的捆绑大小，加速页面加载。Angular 在*路由*和 [*惰性加载模块*](https://angular.io/guide/lazy-loading-ngmodules#lazy-loading-basics) 之间有一个原生集成。唯一的限制是它在编译时加载，但是我们需要的是在运行时动态加载模块。我们将很快向您展示如何去做。*

# *怎么建？*

*我们已经回顾了微前端的最佳实践，并弄清楚 angular 如何适合实现这样的架构。还剩下一些灰色地带。*

*首先是如何实时动态加载模块。这个问题可以用 webpack 提供的 [*模块联盟*](https://webpack.js.org/concepts/module-federation/) 插件轻松解决。它支持本地和远程模块独立和异步地构建，这对于我们的情况是最佳的。*

*第二个问题是，我们如何将所有这些资源组合在一起，以创建一个可行的原型，我们可以迭代这个原型。最快的方法是使用 NX cli，它会为我们生成所有的样板文件。为了了解更多，查看分步教程 [*如何用 angular 构建微前端，NgRx 状态共享，使用 NX 命令行*](https://medium.com/@easy-web/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03) 。*

# *结论*

*我们发现微前端架构与 Angular 紧密兼容。有多种工具可以将它们粘合在一起，比如模块联盟和 NX *monorepos* 。我们只能猜测 Angular 的未来版本是否会支持开箱即用的微前端架构。检查**角度 14** 是否支持微前端:*

*[](https://javascript.plainenglish.io/what-to-expect-from-angular-14-in-2022-is-micro-frontend-coming-7932566f773) [## 🔥2022 年 Angular 14 期待什么:微前端来了吗？🤫

### Angular 13 最近发布了，但变化更具进化性，我们能期待更多革命性的更新吗…

javascript.plainenglish.io](https://javascript.plainenglish.io/what-to-expect-from-angular-14-in-2022-is-micro-frontend-coming-7932566f773) 

这可能取决于社区。此外，随着实施工作的显著减少，它有很大的机会在未来变得更受欢迎。如果你想吸引更多的注意力，并希望 angular 团队在未来的版本中支持它，请分享这篇文章。如果你学到了新的东西，评论并点赞。

[](https://medium.com/subscribe/@easy-web) [## 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。

### 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。通过注册，您将创建一个中型帐户，如果您还没有…

medium.com](https://medium.com/subscribe/@easy-web) [](https://medium.com/@easy-web/membership) [## 通过我的推荐链接加入 Medium 维塔利·舍甫琴科

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

medium.com](https://medium.com/@easy-web/membership) 

# 了解更多信息

[](https://javascript.plainenglish.io/building-beautiful-tiktok-clone-with-angular-and-micro-frontend-part-1-bdd189425695) [## 🔥建造一个漂亮的抖音克隆体，有棱角和微前端

### 第 1 部分:不再有丑陋的教程，只有美丽的，真实世界的例子，通过构建抖音学习微前端

javascript.plainenglish.io](https://javascript.plainenglish.io/building-beautiful-tiktok-clone-with-angular-and-micro-frontend-part-1-bdd189425695) [](https://medium.com/@easy-web/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03) [## 🔥利用 NgRx 状态共享和 NX cli 构建角度微前端

### 如何在几乎不编码的情况下构建健壮的微前端架构；)

medium.com](https://medium.com/@easy-web/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03) [](https://javascript.plainenglish.io/what-to-expect-from-angular-14-in-2022-is-micro-frontend-coming-7932566f773) [## 🔥2022 年 Angular 14 期待什么:微前端来了吗？🤫

### Angular 13 最近发布了，但变化更具进化性，我们能期待更多革命性的更新吗…

javascript.plainenglish.io](https://javascript.plainenglish.io/what-to-expect-from-angular-14-in-2022-is-micro-frontend-coming-7932566f773)*