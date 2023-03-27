# 如何构建我的 Vue.js 项目

> 原文：<https://itnext.io/how-to-structure-my-vue-js-project-e4468db005ac?source=collection_archive---------0----------------------->

嗨，Vue.js 的粉丝们。我注意到，当我试图学习新东西时，我不知道该如何组织我的项目。做这件事的最佳实践是什么？我应该把我的文件放在哪里，以避免项目中出现混乱？
这个故事主要是献给刚开始学习 Vue.js 的开发人员，但是 Vue.js 的老手们也可以从一个全新的角度来看待这个话题。

![](img/be09becd9f39d94883361cac3ed248c2.png)

让我们从使用 [Vue CLI](https://cli.vuejs.org/guide/installation.html) 全新安装一个新项目开始。

```
vue create my-awesome-app
```

安装完成后，您可以看到下一个文件夹结构(这也取决于您在 Vue CLI 中选择的选项。对于这个项目，我选择了所有可能的选项):

```
--public
----img
------icons
----favicon.ico
----index.html
----robots.txt
--src
----assets
------logo.png
----components
------HelloWorld.vue
----router
------index.ts
----store
------index.ts
----views
------About.vue
------Home.vue
----App.vue
----main.ts
----registerServiceWorkers.ts
----shims-vue.d.ts
--tests
----e2e
----unit
--.browserslistrc
--.eslintrc.js
--.gitignore
--babel.config.js
--cypress.json
--jest.config.js
--package.json
--package-lock.json
--README.md
--tsconfig.json
```

这是一个非常标准的结构，但不适合中型或大型应用程序。我们将专注于`src`文件夹，但在继续之前，让我们快速浏览一下其他文件夹。

`public`如果需要文件夹正在使用:

*   生成输出中具有特殊名称的文件
*   图像的动态参考
*   如果库与 Webpack 不兼容

更多关于如何使用公共文件夹的信息，你可以在这里找到。

`tests/e2e`进行端到端测试。

`tests/unit`用于单元测试。

# 源文件夹结构

让我们开始构建我们的新项目和`src`文件夹。下面是我想介绍的结构:

```
src
--assets
--common
--layouts
--middlewares
--modules
--plugins
--router
--services
--static
--store
--views
```

我们将逐一浏览所有文件夹，了解我们为什么需要它。

## 资产

在这个目录中，我们将存储所有的资产文件。在这里，我们想保存字体，图标，图像，风格等。

## 普通的

该文件夹用于保存常用文件。通常可以分成多个内部文件夹:`components` `mixins` `directives`等，或者单个文件:`functions.ts` `helpers.ts` `constants.ts` `config.ts`等。把一个文件放入这个文件夹的主要原因是在很多地方都要用到它。例如:在`src/common/components`中，您可以存储`Button.vue`——在整个应用程序中使用的共享组件。在`helpers.ts`中，你可以编写一个公共函数在多个地方使用它。

## 布局

我已经在我之前的一篇[文章](/vue-tricks-smart-layouts-for-vuejs-5c61a472b69b)中提到了布局问题。您可以将应用程序布局保存在此目录中。比如:`AppLayout.vue`。

## 中间件

该目录与 vue 路由器紧密配合。您可以将导航卫士存放在此文件夹中。下面是一个简单的单一中间件的例子

```
*export default function* checkAuth(next, isAuthenticated) {
  *if* (isAuthenticated) {
    next('/')
  } *else {* next('/login');
  }
}
```

并在 vue 路由器内部使用它

```
*import* Router *from* 'vue-router'
import checkAuth from '../middlewares/checkAuth.js'
const isAuthenticated = true*const* router= *new* Router({
  ***routes:*** [],
  mode: 'history'
})router.beforeEach((to, from, next) => {
  *checkAuth(next,* isAuthenticated*)*
});
```

关于路由器和中间件更多高级主题，您可以在这篇文章[中找到。](/vue-tricks-smart-router-for-vuejs-93c287f46b50)

## 模块

这是我们应用程序的核心。在这里，我们存储了所有的模块——应用程序中逻辑上分离的部分。我鼓励你在每个模块中创建:

*   一个内部组件文件夹，您可以在其中保存您的 vue 组件
*   tests 文件夹(我更喜欢将所有相关的测试保存在模块中)
*   store.ts 或 store 目录，在这里保存您的存储模块
*   与此模块相关的其他文件。它可以是仅与模块相关的辅助函数。

例如，这是一个`orders`模块的例子，其中存储了应用程序中与订单相关的所有组件:订单列表、订单细节等。订单 vuex 商店模块。其他相关文件。它可能看起来像:

```
src
--modules
----orders
------__tests__
------components
--------OrdersList.vue
--------OrderDetails.vue
------store
--------actions.ts
--------getters.ts
--------mutations.ts
--------state.ts
------helpers.ts
------types.ts
```

## 插件

在这个文件夹中，你可以创建和连接你所有的插件。下面是 Vue 2 中插件连接的一个例子

```
import MyPlugin from './myPlugin.ts'Vue.use(MyPlugin, { someOption: true })
```

在 Vue 3 中，你将在 main.ts 中连接你的插件，但是你仍然可以在插件文件夹中创建你的插件。

更多关于插件的信息，你可以在这里阅读和[这里](https://v3.vuejs.org/guide/plugins.html#using-a-plugin)。

## 服务

需要此文件夹来存储您的服务。例如，您可以创建和保存 API 连接服务、localStorage manager 服务等。

更多关于 vue api 模块的信息，你可以在这里阅读。

## 静态

通常，您不需要此文件夹。它可以用来保存一些伪数据。

## 路由器

在这个目录中，您可以存储所有与 vue-router 相关的文件。它可能只是 index.ts，路由器和路由都在一个地方(在这种情况下，将该文件存储在`src`根目录下可能是个好主意)。为了避免混乱，我更喜欢将路由器和路由分成两个不同的文件。

在这篇[文章](/vue-tricks-smart-router-for-vuejs-93c287f46b50?source=your_stories_page-------------------------------------)中，你可以了解如何为你的应用创建一个自动路由器。

## 商店

这是 vuex 存储目录，您可以在其中保存所有与 vuex 相关的文件。因为你想分离你的 vuex 模块，在这个文件夹中你应该存储根状态、动作、突变和 getters，并自动连接所有来自`modules`目录的 vuex 模块。

## 视图

这是我们应用程序中第二重要的文件夹。在这里，我们存储应用程序路径的所有入口点。例如，在您的应用程序中，您可以拥有`/home` `/about` `/orders`条路线。分别在`views`文件夹里你应该有`Home.vue` `About.vue` `Orders.vue`。

你可能会争辩说，如果我们可以把`views`和`modules`存储在同一个地方，为什么我们要把它们分开呢？我看到一些专业人士来区分它们:

*   更清晰的文件结构
*   让你快速了解你在申请中有哪些路线
*   很容易理解哪个文件是页面上的根，以及它从哪里开始工作

# 结论

在这篇文章中，我分享了我创建 vue 应用程序的方法，这是基于我在许多不同的 vue 应用程序上的工作经验。当然，这只是建议，你可以自由地全部使用，部分使用或完全不使用。我希望这篇文章能帮助一些开发人员以一种新的方式构建他们的项目，并给出更多的想法。

如果您对本文或我的其他文章感兴趣，请随时关注我:

github:[https://github.com/NovoManu](https://github.com/NovoManu)

推特:[https://twitter.com/ManuSEngineer](https://twitter.com/ManuSEngineer)

这是所有的乡亲。
见下篇。