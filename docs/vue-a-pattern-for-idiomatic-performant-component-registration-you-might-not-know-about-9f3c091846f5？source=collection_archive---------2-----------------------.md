# 一个您可能不知道的用于惯用的、高性能的组件注册的 Vue 模式

> 原文：<https://itnext.io/vue-a-pattern-for-idiomatic-performant-component-registration-you-might-not-know-about-9f3c091846f5?source=collection_archive---------2----------------------->

## …利用 Vue 和 Webpack 的酷功能！

![](img/89440570d2371eee388d08674363f216.png)

如果你玩过 Vue **单个文件组件**一点，你可能知道如何从另一个组件“调用”一个组件:

1.  `import`子组件。
2.  在父组件的`components`对象上注册它。
3.  将组件添加到模板/渲染函数中。

```
<template>
  <some-random-thing />
</template><script>
import **SomeRandomThing** from './components/SomeRandomThing'export default {
  components: {
    **SomeRandomThing**,
  },
}
</script>
```

这是一种常见的模式，最终可能会变得乏味。在这篇短文中，我们将学习一种(或两种)避免重复的模式。我们还将免费改进我们的应用程序性能。

让我们开始吧！

想象一个`Header`组件，它为我们的应用程序头保存信息和布局。现在想象一下，这些信息可能是用户相关的，也可能是公司相关的，这取决于……我不知道，一个设定值。随便啦。

现在想象我们有一个`UserInfo`和`CompanyInfo`组件。我们希望根据之前已经配置的设置值显示一个或另一个。

# 版本 1:好的方式

这是我们上面概述的方法。

这大概是所有人都会想到的“*默认*”方式(包括我！):

没什么特别的。我们导入两个组件，注册它们，然后根据某个属性值显示一个或另一个组件。

你可能到处都在使用这种“模式”。虽然它本身没有什么问题，但我们可以做得更好。

# 版本 2: <component>来援救了</component>

Vue 中有一个内置组件叫做[组件](https://vuejs.org/v2/guide/components.html#Dynamic-Components)。是的，试着在谷歌上搜索一下。

这个组件`<component />`充当另一个组件的占位符，并接受一个特殊的`:is`道具，该道具带有它应该呈现的组件的名称。

请注意，现在我们如何用所需组件的名称创建一个计算值，从而移除模板中的`v-if/v-else`逻辑，支持全能的`<component />`。我们甚至可以像往常一样传递一些道具。

是不是很酷？

嗯，确实是。但是那里仍然有一个**的主要痛点。**

我们必须导入并注册`:is`属性的所有有效值。我们必须导入并注册`UserInfo`和`CompanyInfo`。

除非有人允许我们动态地导入所有这些组件，这样我们就不必导入和注册它们了…

…哦，等等！

你是说“*动态导入*”吗？

我们抓住你了。

# 版本 3:动态导入+ `<component />`(还有免费的代码拆分！)

让我们看看 [**动态导入**](https://webpack.js.org/guides/code-splitting/#dynamic-imports) 和`<component />`如何一起玩:

使用上面的解决方案， **import** 变成了一个返回承诺的函数。如果承诺解决了(也就是说，没有任何东西被破坏和拒绝)，它将在**运行时**加载期望的模块。

那么，这里发生了什么？我们仍然使用我们的新朋友`<component/>`，但是这次我们提供的不是一个简单的字符串，而是一个完整的组件对象。什么？

如文档中所述，`:is`道具可以包含:

*   注册组件的名称，或
*   **组件的选项对象**

砰！一个“组件的选项对象”。这正是我们需要的！

注意我们是如何避免导入和注册组件的，因为我们的动态`import`是在运行时❤这样做的。

官方文档中有更多关于 Vue 和动态导入[的信息。](https://vuejs.org/v2/guide/components-dynamic-async.html)

## 抓到你了

注意，我们在动态导入语句的之外访问我们的属性`this.isCompany` **。**

这是强制性的，因为否则 Vue 不能发挥它的反应魔法，也不能在道具改变时更新我们的计算值。试试看，你会明白我的意思。

通过在动态导入之外访问我们的属性(通过创建一个简单的`name`变量), Vue 知道我们的`componentInstance`计算属性“依赖于”`this.isCompany`,所以当我们的属性改变时，它将有效地触发重新评估。

## 提醒一句*(8 月 4 日更新)*

当使用动态导入时，Webpack 将(在构建时)**为每个匹配导入函数**中表达式的文件创建一个块文件。

上面的例子有点做作，但是想象一下我的`/components`文件夹包含 800 个组件。那么 Webpack 将创建 800 个块。

因为这不是我们要找的(呵)，确保你[写更严格的表达式和/或遵循文件夹惯例](https://twitter.com/TheLarkInn/status/1025918613557981184)。例如，我倾向于将我想要拆分的组件分组到一个名为`/components/chunks`或`/components/bundles`的文件夹中，所以我知道哪些组件是 Webpack 拆分。

除了这个*gotches*，我们实现了一个**惯用的**，**更简洁的**模式。它有一个奇妙的副作用，使它真正发光:

## 我们的“条件”组件现在是代码分离的！

如果你像这样`npm run build`一个组件，你会注意到 Webpack 会为`UserInfo.vue`创建一个特定的包文件，为`CompanyInfo.vue`创建另一个包文件。默认情况下，Webpack [会这样做](https://webpack.js.org/guides/code-splitting/#dynamic-imports)。网络包是纯爱❤.

这很棒，因为我们的用户直到我们的应用程序请求它们的时候才会加载这些包，因此减少了我们最初的包大小，提高了我们应用程序的性能。

[代码拆分](https://webpack.js.org/guides/code-splitting/)dope。确保你熟悉它，因为如果你还没有使用它，你可以大大改善你的应用程序。去吧！

在这里，拿着这个代码沙箱，你可以随意使用这三种解决方案:

顺便说一下，您甚至可以使用 magic comments 为动态导入定制包名和加载策略。

如果你想了解更多关于代码分割、动态导入以及为什么你应该关心的信息，请听听来自 Webpack 核心团队的唤醒者 [Sean T. Larkin](https://medium.com/u/393110b0b9e4?source=post_page-----9f3c091846f5--------------------------------) :

希望有帮助！

*本帖刊登在*[*【105 期】官方 Vue.js 简讯*](https://www.getrevue.co/profile/vuenewsletter/issues/105-vue-js-sprint-sneak-peek-get-the-vuevixens-scholarship-for-vue-js-london-125646) 💃

我刚刚创办了自己的时事通讯！随意 [**订阅这里**](https://buttondown.email/afontcu) **。**