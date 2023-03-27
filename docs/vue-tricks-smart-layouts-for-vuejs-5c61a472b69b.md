# Vue 技巧:VueJS 的智能布局

> 原文：<https://itnext.io/vue-tricks-smart-layouts-for-vuejs-5c61a472b69b?source=collection_archive---------0----------------------->

![](img/407d44a0b872b9dc79414ef5ba95574d.png)

嗨，亲爱的朋友们！

今天，我想和你谈谈你的 VueJS 应用程序的布局系统。我的朋友 Philipp 问我如何为他的面包店应用程序添加不同的布局。在谷歌搜索了几个小时后，他发现了一些有趣的技巧，并问我对 VueJS 的布局有什么看法。在这篇文章中，我决定分享我对这个话题的想法，并向你展示如何为 VueJS 项目创建智能布局的技巧。

但是在我们开始之前，问你自己一个问题:

![](img/e1138aec930b56b948633bdcf28c1efd.png)

"当前的 VueJS 布局有什么问题？"

答案很简单:

![](img/2509ffda299b4ca576af1fdc60685480.png)

“他们不存在”

VueJS 开发者通常如何解决这个问题？最常见的方法是创建高阶组件并将其添加到所有页面中。

高阶组件示例

我认为这种方法存在两个问题:

1.  我们应该在不同的页面上反复导入我们的布局。
2.  我们应该用布局组件包装我们内容。
3.  这使得我们的代码有点复杂，并迫使组件负责渲染布局。

一方面，这应该不是什么大事，但我对 NuxtJS 添加布局的方法印象深刻，并希望创建类似的东西。如果你不熟悉 NuxtJS 的方式，我可以说两个字:“太神奇了”。你可以将你的布局定义为一个组件的属性，你不应该添加更高阶的组件来将布局添加到你的应用程序中。但是现在让我们忘记 NuxtJS，看看我们可以用 VueJS 中的布局做什么。

# 先决条件

让我们使用 vue CLI[](https://cli.vuejs.org/)*开始并创建一个新的 Vue 项目*

```
*vue create vue-layouts*
```

*在 Vue2+和 Vue3+之间随意选择。*

# *清理项目*

*我们应该清理我们的项目并删除由 vue cli 创建的未使用的文件。*

*首先，移除 *HelloWorld.vue* 组件和组件文件夹。在这个项目中不需要它。在 *Home.vue* 中，我们应该移除对 *HelloWorld.vue* 组件的导入。*

*删除包含内容的资产文件夹，并创建一个新文件 *views/Contacts.vue* 。*

*移除未使用的文件后，我们的项目看起来像:*

```
*--src
----views
------About.vue
------Contacts.vue
------Home.vue
----App.vue
----main.js
----router.js*
```

*App.vue*

*views/Home.vue*

*views/About.vue*

*视图/联系人. vue*

*router.js*

*现在我们准备创建我们的布局。*

# *创建布局构造器*

*作为第一步，我们需要创建一个名为*“layouts/applayout . vue”*的新组件(不要忘记创建*“layouts”*文件夹)。
下面是 Vue 2 版本的实现:*

*layouts/AppLayout.vue*

*如你所见，这是一个非常简单的组件，但这是我们布局系统的核心。让我们更深入地看看它是如何工作的。*

*在模板部分，我们添加了动态组件。作为动态组件的内容，我们添加了名为“布局”的计算属性。该计算属性查看当前路线并检查那里的布局，或者使用我们在常量中定义的默认布局 *"AppLayoutDefault"* 。如果找到了路线，computed property 会尝试找到适当的布局并将其加载到动态组件中。*

*这是 Vue 3 的实现:*

*我们不是在 getter 中导入组件，而是在 watcher 中导入组件，并通过数据将其传递给动态组件。*

*下面是 Vue 3 合成 API 实现(感谢克里斯托佛·罗恩·李的想法【https://twitter.com/KristofferRoenL[】)](https://twitter.com/KristofferRoenL)*

# *创建布局*

*现在，我们要创建我们的布局。对于这个项目，我们将为每个页面创建三种不同的布局。当然，对于您的项目，您可以根据需要选择任意多的布局。*

*但在我们继续布局之前，让我们稍微重构一下代码，将路由器链接从 *App.vue* 移到一个新组件*“layoutlinks/AppLayoutLinks”*。*

*layouts/AppLayoutLinks.vue*

*App.vue*

*在这个小的重构之后，我们应该创建我们的三个布局和默认布局。让我们从默认的一个开始。对我来说，这非常简单。你可能想把它创造得更漂亮。*

*AppLayoutDefault.vue*

*请注意，您的默认布局的名称必须与我们的布局构造函数 *AppLayout.vue.* 中的 *defaultLayout* 常量相同，在我们的例子中是 *"AppLayoutDefault"* (没有文件扩展名)。*

*以及我们的三个主要布局*

*layouts/AppLayoutHome.vue*

*layouts/AppLayoutAbout.vue*

*layouts/AppLayoutContacts.vue*

*我没有花太多时间来创建漂亮的布局。唯一的区别是不同的颜色。但是我相信，对于本文的目的来说，这已经足够了。*

# *将布局移动到路由器级别*

*是时候玩我们的新布局了。让我们再来看看我们的路由器。花点时间想想我们可以在哪里添加我们的布局。*

*没错，我们可以将它添加到我们正在使用的每条路线的元属性中。你可能已经注意到，在我们的布局构造函数中，我们使用了 *this。$route.meta.layout* 属性。你可以在这里阅读关于路线属性*

*所以，让我们现在就做，并添加我们的布局到每条路线*

*router.js*

# *最后一笔*

*你现在可以说:“嗯，还是不行”。是的，我们只需要做最后一件事——将我们的布局构造函数与应用程序连接起来。有两种方法:*

*首先，使用旧的好的高阶组件将我们的 *router-view* 封装在 *App.vue* 中。就像这样:*

*App.vue*

*是的，我添加了一个高阶组件，尽管我在文章的开头说过。但是正如你所看到的，它只进口了一次，我认为它完全没问题。*

*还有一个办法。我们可以创建一个全局 vue 组件并将其包含到 vue 应用程序中，而不是在 *App.vue* 中导入我们的 *AppLayout.vue* 。我们应该更改我们的 *main.js* 文件。*

*下面是 Vue 2 版本:*

*main.js Vue 2*

*和版本 Vue 3:*

*main.js Vue 3*

*清除回 *App.vue* 文件*

# *测试默认布局*

*为了确保一切按预期工作，让我们创建一个新的测试路线，并检查默认布局是否正常工作。
在 router.js 中添加新路由*

```
*{
  path: '/test',
  name: 'Test',
  component: () => *import*('@/views/Home.vue')
}*
```

*如果您现在访问“/test”route，您可以看到我们的默认布局，因为我们没有在路由器中定义任何布局。*

# *结论*

*我发现这个布局系统在我的 VueJS 应用程序中使用非常方便。我看到了一些好处:*

1.  *容易实现。您可以在创建新项目后的几分钟内添加它。*
2.  *好用。布局只需在适当的路线中指定即可。但是如果你忘记这样做，默认的布局仍然有效。*
3.  *使我们的组件更干净，没有至少一个导入语句和模板包装。*
4.  *将所有布局逻辑移至布线器层，而不是元件层。*

*希望你喜欢这篇文章，并找到一些可以在日常工作中使用的有用技巧。你可以在我的 github 账户上找到代码库—[https://github.com/NovoManu/vue-smart-layouts](https://github.com/NovoManu/vue-smart-layouts)*

*如果您对本文或我的其他文章感兴趣，请随时关注我:*

*github:[https://github.com/NovoManu](https://github.com/NovoManu)*

*https://twitter.com/ManuSEngineer*

*乡亲们就这些了，
下篇见。*