# 用 Nuxt 去 Vue

> 原文：<https://itnext.io/going-vue-with-nuxt-f17333893a50?source=collection_archive---------4----------------------->

![](img/fdc53b241d5734cba27f47c716773401.png)

[Vue.js](https://vuejs.org/) 是构建网络应用的可靠选择。为了使用它，我们有两个主要工具:

*   [Vue CLI 3](https://cli.vuejs.org/)*Vue . js 开发的标准工具*
    快速设置 Vue 应用的官方解决方案。
*   [Nuxt](https://nuxtjs.org/)*Universal vue . js 应用*
    对，我们会谈到那个。

但是，ds Nuxt 与标准 Vue 应用有什么不同呢？

`**[TL;DR]**`

*   设置 Vue 应用程序的不同工具
*   关于配置的一些[约定](https://en.wikipedia.org/wiki/Convention_over_configuration)
*   从[单页应用(SPA)](https://en.wikipedia.org/wiki/Single-page_application) 转移到[通用网络应用](http://www.acuriousanimal.com/2016/08/10/universal-applications.html)的能力

`**[UPDATE]**`现在使用 [Nuxt 2](https://github.com/nuxt/nuxt.js/releases/tag/v2.0.0)

# 装置

有两种方法可以安装 Nuxt:

*   使用 vue-cli
*   一个基本的`npm install nuxt`或`yarn add nuxt`如果你是一个[纱线](https://yarnpkg.com/en/)人，创建一些文件夹并添加一些模块如果你想[使用预处理器](https://nuxtjs.org/faq/pre-processors#how-to-use-pre-processors-)

我更喜欢后者，因为它不依赖于任何全局依赖…而且这也是将 Nuxt 集成到现有项目的好方法。

对于 web 应用程序，我总是添加:

*   [Vue 路由器](https://router.vuejs.org/guide/)
*   [Vuex 商店](https://vuex.vuejs.org/guide/)
*   [Vue I18N](https://kazupon.github.io/vue-i18n/)

我 100%确定在某个时候我会需要它们。

让国际化尽快完成并不需要太多额外的努力，也避免了我以后加入国际化的无聊任务(一个文件接一个文件，添加 i18n 调用和键……)

# 应用程序结构

Vue 不强制任何类型的结构，但我们都喜欢&需要保持有组织性。

如果使用 Vue CLI，它将创建这种结构:

*   vue.config.js `vue configuration`
*   📁src
    –main . js`your applications' entry point` –router . js`configuring routes` –app . vue`main Vue component` 📁资产`all static files` 📁组件`other vue components` 📁视图`your pages' components` 📁存储
    - index.js `your Vuex Store`

Nuxt 将需要类似[的东西:](https://nuxtjs.org/guide/directory-structure)

*   nuxt.config.js `nuxt configuration`
*   📁静态`all static files`
*   📁页数`all page files`
*   📁布局`all layouts files`(我们稍后会谈到的一个新东西)
*   📁商店`your Vuex Store`
*   📁插件 [Vue 插件](https://vuejs.org/v2/guide/plugins.html#Using-a-Plugin)

这是一个更扁平的结构，名字很明显。

# 命令

Vue CLI 和 Nuxt 都提出了许多有用的命令。我只说主要的几个。它们都服务于相同的目的:

*   制作一个快速开发服务器开始编码
*   为生产而制造

Vue CLI 使用本地包 vue-cli-service 来启动魔术。

*   `vue-cli-service serve`开发服务器
*   `vue-cli-service build`为生产而构建

Nuxt 有等效的[命令](https://nuxtjs.org/guide/commands)。
无需安装额外的模块👍

*   `nuxt`开发服务器
*   `nuxt build`为生产而构建

我通常在所有项目中使用相同的 [npm 脚本](https://docs.npmjs.com/misc/scripts)别名:

之后，我可以做`yarn dev`开始编码& `yarn build`导出。这些命令将独立于底层的应用程序。

# 小型 Nuxt 概述

Nuxt 在某种程度上依赖于*约定而不是配置*。通过创建文件，Nuxt 会将它们集成到你的 Vue 应用程序中。

以下是它大放异彩的主要领域。

## 按指定路线发送

在标准的 Vue 应用中，您需要手动配置路由器。
这是`router.js`通常的样子:

这在重构时有一些缺点。
如果您想重命名路线，您必须:

*   重命名组件
*   修改`router.js`文件
    –更改路由名称
    –更改组件导入
    –更改 webpack 块名称

使用 Nuxt，这个路由将如下所示:

*   📁pages
    –index . vue
    –foo . vue
    –📁酒吧
    - _id.vue

**重命名路由现在只是更改文件/文件夹名称。**
和**页面代码开箱即可。**

## 商店

标准 Vuex 商店也是如此:

正如您在 Nuxt 中已经猜到的，它只是遵循与路由相同的原则:

*   📁商店
    –foo . js
    –bar . js

具有与路由相同的优点。如果你想对它有更多的控制，还有一个经典模式。

## 关于布局的一个注记

Nuxt 提供了一种轻松处理许多页面布局的方法。

我认为大多数时候你会坚持基本原则:

*   `layouts/default.vue`
*   `layouts/error.vue`

如果你想在一个常规的 Vue 应用程序中实现这一点，你必须手动将每一个页面组件包装在想要的布局组件中…这会使你的代码有点膨胀。

所以*不是必须有*，但肯定是一个很好的补充🏅。

## 插件(比如 vue I18N)

从 Vue 生态系统中集成更多的东西类似于在标准 Vue 应用程序中。

这里的很好地记录了这一点[。](https://nuxtjs.org/guide/plugins)

例如，在`plugin`文件夹中创建一个`i18n.js`文件…

…并更新`nuxt.config.js` …

…您可以在您的应用程序中使用`$t('my-i18n0key')`!

目前，Nuxt 不支持插件集成的`convention over configuration`模式😐所以你必须写一些样板代码。从好的方面来看，这一准则在未来不太可能改变。

但是看起来不必要的配置实际上有一个非常重要的目的。

Nuxt 允许我们构建`universal web applications`。这意味着它应该能够捆绑你的代码:

*   对于浏览器
*   对于服务器来说

如果你只是针对浏览器(SPA)，那就不用担心了。
**但是如果你在服务器上运行代码，你不希望它因为使用某些浏览器 API 而中断**。

Nuxt 通过[小型附加配置](https://nuxtjs.org/guide/plugins#client-side-only)来防止这种情况。

现在`browser.js`将从服务器捆绑包中删除，我们确信我们的代码不会因为 NodeJs 环境中缺少`window`对象而删除`throw`😅

# 原型设计和发展

在我看来，Nuxt 的主要优点是制作一个小原型并在此基础上构建直到第一个结果是多么方便。
同时确信我们可以让它在未来向任何方向发展。

## 单页应用程序

编写一个 SPA 可以让你很快地构建一个小程序，并让任何人都有机会在几乎真实的环境中使用它。

您可以通过将一些 JSON 文件放在`static`文件夹中来创建一个简单的静态 API，并且您可以通过使用带有 [vuex-persistedstate](https://www.npmjs.com/package/vuex-persistedstate) 的[本地存储 api](https://developer.mozilla.org/en-US/docs/Web/API/Storage/LocalStorage) 来持久化应用程序的状态

像 [firebase](https://firebase.google.com/) 、 [netlify](https://www.netlify.com/) 或 [github pages](https://pages.github.com/) 这样的托管解决方案提供了一种免费分享你的应用的方式。

## 通用 web 应用程序

现在您对自己的原型满意了，您可以通过将它集成到一个节点服务器来进一步推动它。
Nuxt 提供了一些模板来查看集成如何在许多框架中工作:

*   [快递模板](https://github.com/nuxt-community/express-template)
*   [koa 模板](https://github.com/nuxt-community/koa-template)

我会用 Koa🐨

在一份`server/index.js`文件中:

在`package.json`中，您应该更新您的脚本:

你将需要更新一点你现有的代码

*   注意你的商店指向你的新 API 的行为。
*   使用 [backpack](https://www.npmjs.com/package/backpack-core) 运行/编译您的服务器应用程序。
    这主要是因为我们在服务器上使用了 [ES 模块](https://nodejs.org/dist/latest-v10.x/docs/api/esm.html)，而 NodeJS 已经不在了。

除此之外，没什么可做的了。
一切都会按预期进行。

## UWA 的优势

构建一个通用的 Web 应用程序似乎是不必要的，但是它有一些优点:

*   更好的初始渲染时间
*   我可以制作一个没有浏览器 Javascript 的应用程序
    ——我相信[渐进增强](https://en.wikipedia.org/wiki/Progressive_enhancement)
    ——Android Chrome 可以在没有 Javascript 的情况下运行你的网站
*   应该有一个更好的搜索引擎优化(你可以阅读更多关于搜索引擎优化[这里](/seo-friendly-spas-d3c461a56217)

如果你想阅读更多关于这个主题的内容，你可以查看 [Stereobooster 的](https://dev.to/stereobooster)关于[服务器端渲染利弊](https://dev.to/stereobooster/server-side-rendering-or-ssr-what-is-it-for-and-when-to-use-it-2cpg)的文章。

# 还有…

这篇文章并没有详尽地列出 Nuxt 能给你提供什么。
这里是它提供的其他东西的快速列表:

*   开箱即用的 [HTML 元标签](https://nuxtjs.org/guide/views#html-head)和 [vue-meta](https://www.npmjs.com/package/vue-meta)
*   开箱即用[页面转换](https://nuxtjs.org/api/pages-transition)
*   开箱[加载进度条](https://nuxtjs.org/api/configuration-loading)
*   开箱 [asyncData](https://nuxtjs.org/api/) & [取](https://nuxtjs.org/api/pages-fetch)钩子。
    这提供了一种在呈现标记之前在服务器上获取异步数据的方法
*   一份很好的文件
*   感谢 nuxt 社区，这是一个很好的模块选择。
    这些提供了一个集成一些流行库的好方法(比如 [Axios](https://github.com/nuxt-community/axios-module)
*   等等。

# 结论

Nuxt 没有漂亮的 GUI😶但这是一段非常聪明的代码，让我在构建网站或应用程序时感觉更有效率。

我做了一个[小型演示库](https://github.com/Hiswe/nuxt-universal-application)，代码几乎和本文中使用的一样。

如果你想了解更多，这里是我最近发现的一些有用的链接:

*   来自 Nuxt.js 的 7 个前端架构课程作者[凯文·鲍尔](https://zendev.com/authors/kball.html)
    很好地分析了 Nuxt 可以如何帮助你
*   [在你的下一个 web 应用中使用 Nuxt.js 的 10 个理由](https://medium.com/vue-mastery/10-reasons-to-use-nuxt-js-for-your-next-web-application-522397c9366b)作者[吴镇男·索佐](https://medium.com/@dericksozo)
    关于 Nuxt 优点的另一个分析
*   [vue . js—1.5 年后再来](https://medium.com/@koreus/vue-js-there-and-back-again-in-1-5-years-756c1582aa96)作者 [Coreus](https://medium.com/@koreus) 。
    这不是关于 Nuxt，但它谈到了用 Vue 应用程序长期生存。
    还有一小部分关于[组件/视图文件夹结构](https://medium.com/@koreus/vue-js-there-and-back-again-in-1-5-years-756c1582aa96#5aea)的内容与我所说的有共鸣。
*   由 [Nuxt 社区](https://github.com/nuxt-community)收集的[教程列表](https://github.com/nuxt-community/awesome-nuxt#tutorials)

所以如果你用的是 Vue，你可能会想试试 Nuxt。

*原载于*[*his we . github . io*](https://hiswe.github.io/2018/12-vue-with-nuxt/)*。*