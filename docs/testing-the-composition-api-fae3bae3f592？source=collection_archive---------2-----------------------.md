# 组合 API 的单元测试

> 原文：<https://itnext.io/testing-the-composition-api-fae3bae3f592?source=collection_archive---------2----------------------->

![](img/4294ec0a86ff45952efef481807fdd4d.png)

用 Vue 3 的[组合 API](https://vue-composition-api-rfc.netlify.com/#basic-example) 进行单元测试会怎么样？

为了让用户尽早尝试复合 API，Vue 团队发布了一个插件，让我们在 Vue 2 中试用。这里可以找到[。](https://github.com/vuejs/composition-api)

这是一篇超级快速的文章，验证了新的 Composition API 如何与 Vue 组件的官方单元测试库`vue-test-utils`一起使用。剧透:工作原理完全一样。

用组合 API 测试一个组件构建应该和测试一个标准组件没有什么不同，因为我们测试的不是实现，而是输出(组件做了什么，而不是如何做)。本文将展示一个使用 Vue 2 中的 Composition API 的组件的简单示例，以及测试策略如何与任何其他组件相同。

[![](img/8860e8ef1f39967845929ca6a9e3821a.png)](http://vuejs-course.com/)

来看看我的 [Vue.js 3 课程](https://vuejs-course.com/)！我们涵盖了组合 API、类型脚本、单元测试、Vuex 和 Vue 路由器。

# 测试组件

下面是 Composition API 的“Hello，World”，或多或少。有不懂的地方，[看 RFC](https://vue-composition-api-rfc.netlify.com/) 或者我的[文章](https://medium.com/js-dojo/compiling-vue-3-from-scratch-and-trying-the-new-composition-api-6d997f32e5b4)或者有 Google 有很多关于组合 API 的资源。

这里我们需要测试的两件事是:

1.  点击增量按钮会使`state.count`增加 1 吗？
2.  props 中收到的消息是否正确呈现(转换为大写字母)？

# 测试道具信息

测试消息是否被正确呈现是微不足道的。我们只是使用`propsData`来设置道具的值，如这里所描述的。

正如所料，这非常简单——不管我们以何种方式组成组件，我们都使用相同的 API 和相同的策略进行测试。您应该能够完全改变实现，而不需要接触测试。记住，要基于给定的输入(道具、触发事件)而不是实现来测试输出(通常是呈现的 HTML)。

# 测试按钮点击

编写一个测试来确保点击按钮增加`state.count`同样简单。请注意，测试标记为`async`；在[模拟用户输入](https://gist.github.com/lmiller1990/simulating-user-input.html#writing-the-test)中阅读更多关于为什么需要这样做的信息。

同样，完全没意思——我们`trigger`了点击事件，并断言呈现的`count`增加了。

# 结论

本文演示了使用组合 API 测试组件与使用传统选项 API 测试组件是如何相同的。想法和概念是一样的。要学习的要点是当编写测试时，根据输入和输出做出断言。`vue-test-utils`不关心你如何组成你的组件；所有现有的方法和 API 都适用于组合 API。好消息！

应该可以重构任何传统的 Vue 组件来使用组合 API，而不需要改变单元测试。如果你发现自己在重构时需要改变测试，你可能测试的是*实现*，而不是输出。

虽然这是一个令人兴奋的新特性，但是 Composition API 完全是附加的，所以不需要立即使用它，但是不管您的选择如何，记住一个好的单元测试断言组件的最终状态，而不考虑实现细节。

*原发表于* [*Vue 测试手册*](https://lmiller1990.github.io/vue-testing-handbook/) *。*