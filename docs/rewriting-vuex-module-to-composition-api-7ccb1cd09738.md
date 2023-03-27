# 将 Vuex 模块重写为组合 API。

> 原文：<https://itnext.io/rewriting-vuex-module-to-composition-api-7ccb1cd09738?source=collection_archive---------0----------------------->

![](img/13fcddb05703537a0b492108ebb10c0d.png)

你好。在这篇文章中，我将向你展示如何将一些 Vuex 模块重写到 Vue 组合 API 中。这可能是一个很好的例子，说明如何用这个来自 Vue 3 版本的新的强大工具替换旧的、好的 Vue 状态管理系统。

这个模块来自一个简单的笔记本应用程序，是我为不久前举办的一些研讨会制作的。这里可以找到[。](https://github.com/lukasborawski/notebook)

那么这个模块是做什么的呢？简而言之，它聚合、保存和删除笔记。我们简单看一下。

好的，对于一些上下文，我们这里有 Typescript 和一些类型，你可以在下面找到。在 app 中，还有一个`$localForage` Nuxt 模块，在本地存储数据。在这里检查一下[。出于本文存储数据的目的，逻辑将被删除。](https://github.com/nuxt-community/localforage-module)

现在，我们来看一下本模块。当然，从顶部开始，我们有一个包含 notes 数组的状态。突变保持注释保存到状态功能中。然后，我们有添加、删除和读取存储笔记的操作。一个 getter 在最后接收当前注释。

好了，该动手了。

Composition API 允许的最主要和最重要的事情之一是将我们的公共业务逻辑分割并移动到称为 composables 的独立块(文件)中。然后在整个 app 中重用它们。

所以我们现在可以创建一个。将它作为一个`useNotes.ts`文件放到新文件夹`~/composables`中——我们使用的是 Nuxt 结构。首先复制将要使用的类型，就像使用 Vuex 模块一样。

一开始，我们必须重建国家。为此，我们将使用 Composition API 提供的一个名为`reactive`的新工具。

> `reactive`相当于当前 2.x 中的`Vue.observable()` API，重命名是为了避免与 RxJS observables 混淆。这里，返回的状态是一个所有 Vue 用户都应该熟悉的反应性对象。Vue 中反应状态的基本用例是我们可以在渲染过程中使用它。由于依赖关系跟踪，视图会在反应状态改变时自动更新。

**提示**:在这里也检查`ref`物体[。](https://composition-api.vuejs.org/api.html#ref)

代码:

值得注意的一点是，我们需要在主可组合函数之外定义我们的反应状态对象。我们需要完全的反应能力，并从其他组件获取这些数据。但是我们不需要导出它。

我们的组合时间到了。

在同一个文件中，我们将定义以下代码:

让我们深入研究一下。这里我们有一个简单的函数，它从先前定义的状态返回注释，以及保存、删除和获取注释的处理程序/操作。实际上，它们看起来和 Vuex 模块一模一样。注释是现在计算的值，它是从 Composition API 传递的，它相当于来自 Vue Options API 的众所周知的`computed`。

完成了。我们去掉了所有 Vuex 模块的复杂性——没有突变，没有动作，没有 getters。我们所需要的是一个功能组合，可以在应用程序中的任何地方重用。

此外，我们还提供了一些退货类型。至于笔记处理函数非常简单，我们现在使用的是`ComputedRef`的通用类型。从 Vue 的第 3 版开始，我们可以开箱即用了——太棒了。

现在我们可以将它与真正的组件一起使用。在我们的例子中，它将是一个`index`页面。来自`useNotes` composable 的数据将被传递，作为 prop 传播到子组件——关于通过 props 和 Composition API 链接数据的更多内容，敬请关注。

`index.vue`页面代码:

在 Vue 3 中，我们用`setup` [函数](https://composition-api.vuejs.org/api.html#setup)得到了这个新的可选语法。它允许我们将所有组件逻辑组合在一个地方，按逻辑块排序。最理想的情况是，您将整个业务代码放在组件之外，只需使用`setup`函数调用它。和我们的`index`页面示例一样，我们已经导入了`useNotes`可组合块来收集笔记。

这里你可能会注意到的一个新东西是这个新函数`onBeforeMount`。当然，这是一个钩子。有了组合 API，有了新重新定义的[钩子](https://composition-api.vuejs.org/api.html#lifecycle-hooks)，我们可以使用`setup`函数。

仅此而已。有争议？一点点？现在有了组合 API，我们可以摆脱几乎所有的 Vuex 复杂性。从技术角度来看，它几乎是一样的，但是定义它和用它来行动的方式会不那么复杂。只是我们都知道的函数。我们不需要突变、动作和 getters。更不用说我们根本不需要绘制它们的地图。现在只需要简单的导入就足够了，我们继续前进。Vuex 模块的最大优势——逻辑分离——我们仍然可以使用 Composition API。另一件事可能是速度和性能，但这需要一些基准来确认。试一试，你会很兴奋的。

完整的代码可在[这个](https://github.com/lukasborawski/notebook) repo 上用之前提到的一个简单的笔记本 app 获得。

谢谢，请慢用。