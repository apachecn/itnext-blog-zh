# Vue 技巧:VueJS 的智能 api 模块

> 原文：<https://itnext.io/vue-tricks-smart-api-module-for-vuejs-b0cae563e67b?source=collection_archive---------0----------------------->

嗨，花儿们！

在我的朋友菲利普和他的面包店的故事之后，我们终于打开了坚实的原则的话题。为了继续提高我们的 VueJS 技能，我想开始一个新的部分，我称之为“Vue tricks ”,在这里我将分享我在工作中与惊人的 VueJS 的一些有趣的时刻。

![](img/407d44a0b872b9dc79414ef5ba95574d.png)

上次，Philipp 在他的面包店连接 API 服务时遇到了一些困难。今天我想谈谈 Vue 和 Vuex 的智能 API 模块。或许，有时或经常，你会注意到你的 api 连接代码不一致，看起来一团糟。在这篇文章中，我将分享如何创建一个新的 API 模块，您可以轻松地在您的 Vue 组件和 Vuex 操作中使用，只需输入

```
this.$api.tasks.fetch()
```

所有(在这种情况下)任务都将被提取到您的组件或存储中。

在这个项目中，我决定不使用我最喜欢的 TypeScript，因为，我想，许多人不会在 VueJS 中使用它，而只是专注于纯 JavaScript。我们开始吧！

# 先决条件

让我们使用 vue CLI[](https://cli.vuejs.org/)*开始并创建一个新的 Vue 项目*

```
*vue create vue-api-module*
```

*对我们来说，你使用哪个版本的 Vue 并不重要。您可以选择 Vue2+或 Vue3+*

*我们应该删除项目中不用的文件。清理之后(不要忘记删除 App.vue 中的所有导入),源目录如下所示:*

```
*--src
----store
------index.js
----App.vue
----main.js*
```

# *创建 API 模块*

*让我们从创建主 API 模块开始。*

*在我们的源文件夹中创建一个目录“ *services* ”，并将文件“ *api.js* ”放入其中:*

```
*--src
----services
------api.js
----store
------index.js
----App.vue
----main.js*
```

*我们从创建基类开始:*

*服务/api.js*

*在这个项目中，我们将使用免费的 API 服务[https://jsonplaceholder.typicode.com/](https://jsonplaceholder.typicode.com/)*

*请随意阅读如何使用该服务的文档。*

*让我解释一下我们的基类属性和方法。*

## *baseUrl*

*我们所有资源的基本 url。在您的项目中，您可能想要使用自己的 api 服务。良好的做法是使用。存储基本 url 的 env 文件。阅读更多关于。env 这里:[https://cli.vuejs.org/guide/mode-and-env.html](https://cli.vuejs.org/guide/mode-and-env.html)*

## *资源*

*特定资源，如用户、帖子、评论等。*

## *getUrl*

*一个助手函数，用于粘合基本 url、资源和(可选)id。*

## *处理错误*

*负责错误处理的功能。*

*现在，让我们创建两个子类:*

*服务/api.js*

*服务/api.js*

*它们之间的区别在于*readonlyaspiservice*将用于我们不想写入数据的只读资源。它只有两种方法:*

## *取得*

*去取许多物品*

## *得到*

*通过 id 获取单个对象*

**ModelApiService* 可以读写数据。它具有与 *ReadOnlyApiService* 相同的方法，但是在写入数据方面多了三种方法:*

## *邮政*

*创建资源实体。*

## *放*

*更新资源实体。*

## *删除*

*删除资源实体。*

*现在，让我们开始创建一个资源类。在这个项目中，出于不同的目的，我们将使用三种资源:用户、帖子和相册。*

*我们的用户资源将是只读的。*

*服务/api.js*

*我们只需要做两个动作:
1。在构造函数中指定资源的名称(在本例中为用户)。*

*2.继承我们想要使用的类。可能是 *ReadOnlyApiService* 或者 *ModelApiService* 。*

**UserApiService* 包含两个*只读*方法:*获取*和*获取*。无法使用写方法:*贴*，*贴*或*删*。*

*下一个是 *PostsApiService**

*服务/api.js*

*这个服务既能读又能写，并且有全部五种方法。*

*您还可以使用自定义方法轻松扩展您的服务。在 *AlbumsApiService* 中，我们将创建另外两个自定义方法: *uploadImage* 和 *triggerError**

*服务/api.js*

*AlbumApiService 现在有七种方法:五种基本方法和两种自定义方法。*

*在" *api.js* "文件的结尾，我们将导出我们的服务:*

*服务/api.js*

# *将 API 服务与商店和组件连接*

*是时候将我们的新 api 服务与 Vue 应用程序连接起来了。我们只需要把它注入到存储和组件中。*

*先说 Vuex 店。*

*创建一个新的目录“*plugins”*，并在里面放入文件“ *storePlugins.js* ”。*

```
*--src
----plugins
------storePlugins.js
----services
------api.js
----store
------index.js
----App.vue
----main.js*
```

*在" *storePlugins.js* "中，我们将把我们的 api 模块注入到商店中*

*插件/storePlugins.js*

*并在 *src/store/index.js* 中导入这些插件到 Vuex 实例中*

*商店/索引. js*

*让我们使用全局混合对 vue 组件做同样的工作。*

*在"*插件"*目录下新建一个文件" *mixins.js"**

```
*--src
----plugins
------mixins.js
------storePlugins.js
----services
------api.js
----store
------index.js
----App.vue
----main.js*
```

*创建一个全局 mixin $api*

*插件/mixins.js*

*并导入到" *main.js"* 中的 vue 实例中*

```
**...*
*import* "@/plugins/mixins";...*
```

*瞧，现在我们可以使用相同的语法在商店和组件中使用我们的$api 服务了*

*使用*

*下面是一个如何在组件中使用 API 模块的例子*

*App.vue*

*在 Vuex 商店内*

*存储/索引/js*

*你可以在我的 github 账户中找到所有带我评论的代码—[https://github.com/NovoManu/vue-api-module](https://github.com/NovoManu/vue-api-module)*

*如果您对本文或我的其他文章感兴趣，请随时关注我:*

*github:[https://github.com/NovoManu](https://github.com/NovoManu)*

*推特:[https://twitter.com/ManuSEngineer](https://twitter.com/ManuSEngineer)*

*我想对我的同事雅各布·斯卡莱基(【https://twitter.com/jskalc】)和塞尔吉·希林戈([【https://twitter.com/s_shilingov】](https://twitter.com/s_shilingov))说“*谢谢”*，他们给了我一些有趣的想法来写这篇文章。*

*在接下来的文章中，我想涵盖更多有趣的话题。
就这些了，乡亲们！*