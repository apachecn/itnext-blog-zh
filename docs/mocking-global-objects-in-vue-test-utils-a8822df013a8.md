# 在 vue-test-utils 中模仿 Vue.js 全局对象

> 原文：<https://itnext.io/mocking-global-objects-in-vue-test-utils-a8822df013a8?source=collection_archive---------1----------------------->

![](img/ebe471ee78559615813d14076cfd9462.png)

`vue-test-utils`提供了一种简单的方法来模拟连接到`Vue.prototype`的全局对象，既可以逐个测试，也可以为所有测试设置默认模拟。

以下示例中使用的测试可在处找到[。](https://github.com/lmiller1990/vue-testing-handbook/blob/master/demo-app/tests/unit/Bilingual.spec.js)

[![](img/8860e8ef1f39967845929ca6a9e3821a.png)](http://vuejs-course.com/)

来看看我的 [Vue.js 3 课程](https://vuejs-course.com/)！我们涵盖了组合 API、类型脚本、单元测试、Vuex 和 Vue 路由器。

# 模拟安装选项

模拟挂载选项是设置附属于`Vue.prototype`的任何属性的值的一种方式。这通常包括:

*   `$store`，用于 Vuex
*   `$router`，用于 Vue 路由器
*   `$t`，用于 vue-i18n

和许多其他人。

# vue-i18n 示例

与 Vuex 和 Vue 路由器一起使用将在相应章节中讨论，此处的[和此处的](https://lmiller1990.github.io/vue-testing-handbook/vuex-in-components.html)和。我们来看一个 [vue-i18n](https://github.com/kazupon/vue-i18n) 的例子。虽然可以为每个测试使用`createLocalVue`并安装`vue-i18n`，但这很快会变得很麻烦，并引入许多样板文件。首先，一个使用`vue-i18n`的`<Bilingual>`组件:

`vue-i18n`的工作方式是你在另一个文件中声明你的翻译，然后用`$t`引用它们。对于这个测试来说，翻译文件看起来像什么并不重要，但是对于这个组件来说，它看起来像这样:

根据语言环境，会呈现正确的翻译。让我们试着在测试中渲染组件，不要有任何嘲讽。

用`yarn test:unit`运行这个测试会抛出一个巨大的堆栈跟踪。如果仔细查看输出，您可以看到:

这是因为我们没有安装`vue-i18n`，所以全局`$t`方法不存在。让我们用`mocks`安装选项来模拟一下:

现在测试通过了！`mocks`选项有很多用途。我经常发现自己在嘲笑上面提到的三个包提供的全局对象。

# 使用配置设置默认模拟

有时您希望 mock 有一个缺省值，所以不要逐个测试地创建它。您可以使用由`vue-test-utils`提供的[配置](https://vue-test-utils.vuejs.org/api/#vue-test-utils-config-options) API 来完成这项工作。让我们扩展一下`vue-i18n`的例子。通过执行以下操作，您可以在任何地方设置默认模拟:

本指南的演示项目使用 Jest，所以我将在`jest.init.js`中声明默认的 mock，它在测试自动运行之前加载。我还将导入前面的示例翻译对象，并在模拟实现中使用它。

现在一个真正的翻译将被呈现，尽管使用了一个被模仿的`$t`函数。再次运行测试，这次使用`wrapper.html()`上的`console.log`并移除`mocks`安装选项:

测试通过，并呈现以下标记:

你可以在这里阅读关于使用`mocks`测试 Vuex [的内容。技术是一样的。](https://lmiller1990.github.io/vue-testing-handbook/vuex-in-components.html#using-a-mock-store)

# 结论

本指南讨论了:

*   使用`mocks`逐个测试地模仿一个全局对象
*   使用`config.mocks`设置默认模拟

最初发表于 [Vue 测试手册](https://github.com/lmiller1990/vue-testing-handbook)。