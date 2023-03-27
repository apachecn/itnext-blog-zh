# 如何构建 Vue.js 项目

> 原文：<https://itnext.io/how-to-structure-a-vue-js-project-29e4ddc1aeeb?source=collection_archive---------0----------------------->

## 完美的 Vue.js 文件夹结构和组件架构，包含智能和非智能组件

![](img/4c29baf82f77c7e7aedb417e2b4f7beb.png)

Vue.js 不仅仅是一个宣传，它是一个很棒的前端框架。开始使用它并构建一个 web 应用程序非常容易。Vue.js 经常被描述为小型应用程序的框架，甚至有时因为它的小而成为 jQuery 的替代品！我个人认为，它也可以适用于更大的项目，在这种情况下，从组件架构的角度来看，很重要的一点是要很好地构建它。

在开始我的第一个大型 Vue.js 项目之前，我做了一些研究，以便找到完美的文件夹结构、组件架构和命名约定。我查阅了 Vue.js 文档、一些文章和许多 GitHub 开源项目。

我需要找到一些问题的答案。这就是你会在这篇文章中发现的:

*   你如何构建 Vue.js 项目中的文件和文件夹？
*   如何编写智能组件和非智能组件，将它们放在哪里？这是一个来自 React 的概念。
*   Vue.js 编码风格和最佳实践是什么？

我还将记录我得到灵感的来源和其他链接，以便更好地理解。

# Vue.js 文件夹结构

以下是 src 文件夹的内容。我建议您使用 [Vue CLI](https://github.com/vuejs/vue-cli) 启动该项目。我个人使用的是默认的 Webpack 模板。

```
.
├── app.css
├── App.vue
├── assets
│   └── ...
├── components
│   └── ...
├── main.js
├── mixins
│   └── ...
├── router
│   └── index.js
├── store
│   ├── index.js
│   ├── modules
│   │   └── ...
│   └── mutation-types.js
├── translations
│   └── index.js
├── utils
│   └── ...
└── views
    └── ...
```

每个文件夹的一些详细信息如下:

*   **资产** —您可以将导入到组件中的任何资产放在这里
*   **组件** —项目中所有非主视图的组件
*   **mixins**—mixins 是 javascript 代码中在不同组件中重用的部分。在 mixin 中，你可以从 Vue.js 中放入任何组件的方法，它们将与使用它的组件的方法合并。
*   **router** —您的项目的所有路径(在我的例子中，我在 index.js 中有它们)。基本上在 Vue.js 中一切都是组件。但不是所有东西都是一页。一个页面有一个类似“/仪表板”、“设置”或“搜索”的路径。如果元件有布线，则会对其进行布线。
*   **store(可选)**—mutation-type . js 中的 Vuex 常量，子文件夹 modules 中的 Vuex 模块(然后加载到 index.js 中)。
*   **翻译(可选)** — Locales 文件，我用的是 Vue-i18n，效果相当不错。
*   **utils(可选)** —我在一些组件中使用的函数，比如正则表达式值测试、常量或过滤器。
*   **视图** —为了让项目更快阅读，我将布线的组件分开，放在这个文件夹中。为我路由的组件不仅仅是一个组件，因为它们代表页面，并且它们有路由，我将它们放在“视图”中，然后当你检查一个页面时，你会转到这个文件夹。

您最终可以根据需要添加其他文件夹，如**过滤器、**或**常量、API。**

## 一些启发我的资源

*   [https://vuex.vuejs.org/en/structure.html](https://vuex.vuejs.org/en/structure.html)
*   [https://github.com/vuejs/vue-hackernews-2.0/tree/master/src](https://github.com/vuejs/vue-hackernews-2.0/tree/master/src)
*   [https://github . com/mchandleraz/real world-vue/tree/master/src](https://github.com/mchandleraz/realworld-vue/tree/master/src)

# 使用 Vue.js 的智能与非智能组件

智能和非智能组件是我从 React 学到的一个概念。智能组件也被称为容器，它们是处理状态变化的人，它们负责**事情如何工作**。相反，哑组件，也称为表示性的，只处理**外观和感觉**。

如果你熟悉 MVC 模式，你可以把**转储组件比作视图**，把**智能组件比作控制器**！

在 React 中，智能组件和非智能组件通常放在不同的文件夹中，而在 Vue.js 中，它们都放在同一个文件夹中:components。为了在 Vue.js 中进行区分，您将使用一个命名约定。假设您有一个哑卡组件，那么您应该使用以下名称之一:

*   基本卡
*   AppCard
*   VCard

如果您有一个使用 BaseCard 并向其添加了一些方法的智能组件，您可以根据您的项目对其进行命名:

*   档案卡
*   物品卡
*   新闻卡片

如果您的智能组件不仅仅是一个带有方法的“更智能”的 BaseCard，请使用任何适合您的组件的名称，而不要以 Base(或 App，或 V)开头，例如:

*   仪表板统计
*   搜索结果
*   用户配置文件

这个命名约定来自 Vue.js 官方样式指南，其中也包含命名约定！

# 命名规格

以下是来自 Vue.js 官方风格指南的一些约定，你需要用它们来组织好你的项目:

*   除了根“App”组件之外，组件名称应该总是由多个单词组成。例如，使用“用户卡”或“个人卡”代替“卡”。
*   每个组件都应该在自己的文件中。
*   单文件组件的文件名应该总是 PascalCase 或 kebab-case。使用“UserCard.vue”或“user-card.vue”。
*   每页只使用一次的组件应以前缀“the”开头，表示只能有一个。例如，对于导航栏或页脚，你应该使用“Navbar.vue”或“TheFooter.vue”。
*   子组件应包含其父组件名称作为前缀。例如，如果您希望在“用户卡”中使用“照片”组件，您可以将其命名为“用户卡照片”。这是为了更好的可读性，因为文件夹中的文件通常是按字母顺序排列的。
*   在组件的名称中总是使用全名而不是缩写。例如，不要使用“UDSettings”，而是使用“UserDashboardSettings”。

## Vue.js 官方风格指南

无论你是 Vue.js 的高级用户还是初学者，你都应该阅读这个 Vue.js 风格指南，它包含了很多提示和命名约定。它包含了很多该做和不该做的事情的例子。

[](https://vuejs.org/v2/style-guide/) [## 风格指南— Vue.js

### 这是 Vue 专用代码的官方风格指南。如果你在一个项目中使用 Vue，这是一个很好的参考来避免…

vuejs.org](https://vuejs.org/v2/style-guide/)