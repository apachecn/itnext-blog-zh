# Vue 窍门:VueJS 的智能路由器

> 原文：<https://itnext.io/vue-tricks-smart-router-for-vuejs-93c287f46b50?source=collection_archive---------0----------------------->

嗨，伙计们！

我们将继续为我们的 VueJS 应用程序添加更多功能。在讲述了 vue 布局的故事后，我想更深入地研究自动化，为这个应用程序创建一个自动路由器。我的朋友 Philipp 遇到了 vue 路由器的下一个问题:他的应用程序中有数百条路由，路由器文件包含数百行代码。我想如果应用程序中的页面多于几页，读者也会发现这个问题。一旦开发人员在应用程序中添加新页面，允许路由器自动创建路由将是一个好主意。

![](img/97914db83d80e5af62ef2042a1911b94.png)

让我们实现自动化。

# 先决条件

我们再次开始使用 Vue CLI[](https://cli.vuejs.org/)*创建一个新的应用程序*

```
*vue create vue-automatic-router*
```

*可以选择 Vue2+或者 Vue3+。该解决方案只需稍加调整，就可以在两个版本中使用。*

*让我们把这个项目中不用的文件清理掉。删除`assets`和`components`目录。创建`router`目录，将`router.js`重命名为`index.js`并移动到`router`目录。移除`Home.vue`中`logo`和`HelloWorld.vue`的所有链接，并将`Home.vue`重命名为`Index.vue`*

```
*--src
----router
------index.js
----views
------About.vue
------Index.vue
----App.vue
----main.js*
```

*路由器/index.js*

*这是我们的文件夹结构，我们准备好了。*

# *创建自动路由器*

## *基本实现*

*首先，在`router`目录中创建一个名为`routes.js`的新文件。*

*在内部添加下一个函数。*

*router/routes.js*

*该函数试图找到所有 vue 文件，从指向目录的开头`./`删除两个字符，删除文件扩展名`.vue`并使用斜杠`/`作为分隔符从文件的路径字符串创建一个数组。*

*现在我们应该从视图目录中导入所有的页面。客户端应用程序和浏览器不能直接访问文件系统的问题。我们可以尝试使用一些棘手的方法来访问文件系统，但幸运的是，由于我们使用的是 webpack，因此可以使用 webpack 的`[require.context](https://webpack.js.org/guides/dependency-management/#requirecontext)`函数。让我们这样做:*

*router/routes.js*

*明白我们到底在做什么。
我们在之前的函数`importAll`中添加了`require.context`作为参数*

*`require.context`的第一个参数是页面文件夹的相对路径。请确保您使用的结构与我相同，或者相应地调整路径。*

*第二个参数允许函数递归地检查内部文件夹。*

*在第三个例子中，我们指定文件扩展名，在我们的例子中是`.vue`。*

*现在 pages 变量是一个包含两个元素的数组*

```
*[["About"], ["Home"]]*
```

*是时候创建一个助手函数来从页面创建我们的路线了*

*router/routes.js*

*这个功能现在很简单，但在我们创建自动路由器的过程中，它会逐步扩展。现在我们只需检查文件名是否是`index`并将路径改为`/`而不是`/index`，否则它返回小写的路径名。对于`About.vue`是`/about`*

*在`routes.js`年底，我们需要导入我们的新路线*

*router/routes.js*

*在默认导出中，所有组件都是动态导入的。它需要从组件中获取路由的名称。稍后，我们将从这里的组件导入更多数据。*

*接下来，我们应该调整`router/index.js`和`main.js`来处理我们在`router/routes.js`中创建的承诺*

*路由器/index.js*

*我们不是导出路由器，而是导出代表我们的路由器的承诺数组，并等待所有路由都被导入。*

*主页. js*

*因为我们的路由器现在是异步的，所以我们应该在创建 Vue 实例之前创建一个异步包装器。*

*这是两个文件的 Vue3 实现*

*路由器. index.js*

*主页. js*

*是时候从`npm run serve`开始我们的项目了，要知道一切都在按预期进行。*

## *创建路由树*

*在大多数情况下，仅仅有简单的路线还不够，如`/` `/about` `/countacts`等等。通常，真正的 web 应用程序有更复杂的路由系统。现在让我们调整我们的`generateRoute`函数来处理更复杂的树。*

*如果你熟悉 NuxtJS 方法，我下面分享的方法你也会很熟悉。如果没有，我来解释一下它的工作原理。*

*路由是基于我们的文件结构创建的。在我们最初的例子中，我们只有两个页面组件`Index.vue`和`About.vue`。它为我们提供了两条路线:`/`和`/about`。如果你需要路线`/users`和`/user/profile`，你应该有下一个文件结构:*

```
*--src
----views
------users
--------Profile.vue
--------Index.vue*
```

*需要创建一个包含两个`vue`文件`Profile.vue`和`Index.vue`的文件夹`users`*

*让我们为`posts`资源创建一个典型的 CRUD 视图。我们不会得到 API 连接，因为这是我在这篇[文章](/vue-tricks-smart-api-module-for-vuejs-b0cae563e67b)中涉及的另一个主题。只需为该系统创建路线:*

*`/posts` —显示帖子列表*

*`/posts/create` —创建新帖子*

*`/posts/1` —显示 id === 1 的帖子*

*`/posts/edit/1` —编辑 id === 1 的帖子*

*我们将在下一小节中讨论动态路由，并集中讨论前两个:显示帖子列表和创建新帖子。*

*用两个文件`Index.vue`和`Created.vue`创建一个`posts`文件夹*

*视图/帖子/索引. vue*

*views/post/create . vue*

*它对我们的路由器没有任何作用，因为我们应该首先调整我们的`generateRoute`功能。*

*router/routes.js*

*如果您测试您的两条路线，您可以看到我们的组件的内容。路由器有下一个结构:*

```
*[
{
  component: About.vue,
  name: 'About',
  path: '/about'
}
{
  component: Index.vue,
  name: 'Home',
  path: '/'
}
{
  component: posts/Index.vue,
  name: 'Posts',
  path: '/posts'
}
{
  component: posts/Create.vue,
  name: 'PostCreate',
  path: '/posts/create'
}
]*
```

## *处理动态路由*

*通常，在构建应用程序的过程中，开发人员不仅需要处理静态路由，还需要处理动态路由。让我们将此功能添加到我们的自动路由器中。我们将再次调整我们的`generateRoute`函数。但在此之前，让我们在视图文件夹中创建两个新文件。在编辑`posts`文件夹中创建文件`_Id.vue`。是的，不是错别字。文件必须以`_`符号开始。它告诉我们的路由器这条路由是动态的。在`views/posts`文件夹中创建名为`edit`的文件夹。在里面添加文件`_Id.vue`。*

*views/posts/_Id.vue*

*views/posts/edit/_Id.vue*

*以下是`generateRoute`功能的最后调整*

*路由器中又增加了两条路由*

```
*[
...
{
  component: posts/_Id.vue,
  name: 'PostDetails',
  path: '/posts/:id'
}
{
  component: posts/edit/_Id.vue,
  name: 'PostEdit',
  path: '/posts/edit/:id'
}
]*
```

*用`npm run serve`测试*

## *处理嵌套路线*

*不太经常，但有时需要使用[嵌套路由](https://router.vuejs.org/guide/essentials/nested-routes.html)，也称为子路由。所以，我们再把手弄脏，加到我们的自动路由器上。*

*与动态路由一样，对于嵌套路由，我们将使用前缀让路由器知道它是嵌套路由。在嵌套路线的情况下，它是`^`符号。嵌套路由必须放置在与父路由相同的级别上。*

*首先，让我们将这个功能添加到我们的`router/routes.js`文件中。*

*该函数创建一个嵌套路由列表，其中的键是嵌套路由的父路径和值数组。让我们也扩展导出的函数:*

*router/routes.js*

*要看到这些变化，我们应该在一个新文件夹`users` `users/Index.vue`和`users/^Profile.vue`中再创建两个页面*

*你是否注意到这里需要另一个`router-view`来显示嵌套的路线？*

*另一个具有嵌套路由的路由已被添加到自动路由器中。*

```
*[
...
{
  children: [
    component: users/^Profile.vue,
    name: 'UserProfile',
    path: '/users/profile'
  ],
  component: users/Index.vue,
  name: 'Users',
  path: '/users'
}
]*
```

*我们结束了。我们介绍了一个新的自动路由器，它有助于开发者保持路由器的整洁，特别是当应用程序中有很多路由时，可以避免长路由文件地狱。*

## *奖金。添加布局和中间件。*

*在我们继续之前，我鼓励读者阅读这篇[文章](/vue-tricks-smart-layouts-for-vuejs-5c61a472b69b)来理解这里正在发生的事情。我们不会深入智能布局的主题，但将这两种方法结合成一个。正如您所记得的，要向路由添加布局，我们应该为路由指定`meta`属性，并在那里添加布局名称(以及中间件)。让我们在我们的自动路由器中也这样做。*

*我们应该调整`router/routes.js`文件中的导出功能。*

*[https://gist . github . com/novo manu/08269 F3 df 606 ca 5a e 4142 c 44982 c 6 bef](https://gist.github.com/NovoManu/08269f3df606ca5ae4142c44982c6bef)*

*为了处理中间件，也应该调整`router/index.js`文件*

*路由器/index.js*

*现在，您可以直接在组件中指定您的布局和中间件。*

*views/home.vue*

## *奖金。NMP 模块*

*如果您不想从头开始创建整个实现，请随意使用我创建的 npm 模块，可以在这里找到:*

*[](https://www.npmjs.com/package/vue-automatic-router) [## vue-自动路由器

### 受 NuxtJS 路由的启发，这个包将类似的系统添加到本地 VueJS 项目中。这意味着它不…

www.npmjs.com](https://www.npmjs.com/package/vue-automatic-router)* 

*我希望这种方法可以使你的项目更整洁，节省一些时间。无论如何，尝试一些新的东西来避免你的日常事务总是很有趣的。你可以在我的 github 账户上找到代码库—[https://github.com/NovoManu/vue-automatic-router-example](https://github.com/NovoManu/vue-automatic-router-example)*

*如果您对本文或我的其他文章感兴趣，请随时关注我:*

*github:【https://github.com/NovoManu *

*推特:【https://twitter.com/ManuSEngineer *

*乡亲们就这些了，
下篇见。*