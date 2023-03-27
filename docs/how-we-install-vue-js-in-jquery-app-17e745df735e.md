# 我们如何在 jQuery app 中安装 Vue.js

> 原文：<https://itnext.io/how-we-install-vue-js-in-jquery-app-17e745df735e?source=collection_archive---------2----------------------->

![](img/ef0743ea5e1259d99c269e6f47ba0986.png)

当你学习 Vue.js 的时候，网上有那么多教程。我是在工作中维护 jQuery 代码的开发人员之一。当您完成了几个教程，并理解了 Vue.js 是什么时，您可能会有这样的问题:“那么，我如何在这个 jQuery 应用程序中添加 Vue.js 呢？”

这其实很简单，如果你已经开始用 [vuejs-templates](https://github.com/vuejs-templates/webpack) 学习 Vue.js，你会知道其中的 90%。你只需要 4 个步骤！

# 步骤 1:创建一个 Vue.js 组件

我创建的组件正在包装 [vue-multiselect](https://github.com/shentao/vue-multiselect) 。看起来是这样的:

任何 Vue.js 组件都可以。你可以通过教程添加你做的任何东西。

# 步骤 2:添加一段 JS 代码作为桥梁

根据我的经验，很难期望 jQuery 代码总是超级可读。我们希望避免混淆 Vue.js 在 jQuery 代码中的位置。代码连接了广阔的 jQuery 大陆和 Vue.js 岛。

你可以看到你的 Vue.js 组件的 props 是通过`data`传递的。现在`bind`的函数中有了你的 Vue 组件。

# 步骤 3:添加 Vue.js 组件的目的地

你只需要编辑一样东西:HTML——你必须已经知道如何在 Vue.js 应用中添加 Vue.js 组件。在 jQuery app 里也是一样。

# 步骤 4:在 jQuery 应用程序中添加 Vue.js 组件

确保在调用`multiselector.bind()`之前呈现 HTML。轻松点。

# 最后，如果您想将 jQuery 转换成 Vue.js

如果你想把你的 jQuery 应用转换成 Vue.js 应用，GitLab 的博客文章可能会给你一些好的想法。转变是如此吸引人，然而它需要大量的时间。如果你在一家初创公司工作，你永远不会有那样的资源。我相信在 jQuery 应用中一点一点地实现 Vue.js，然后如果你渴望这样做，最终用 Vue.js 取代一切，是最好的策略。这些博文解释得很好。

*   [我们为什么选择 Vue.js](https://about.gitlab.com/2016/10/20/why-we-chose-vue/)
*   [我们的大前端计划揭晓](https://about.gitlab.com/2017/02/06/vue-big-plan/)
*   [我们如何做 Vue:一年后](https://about.gitlab.com/2017/11/09/gitlab-vue-one-year-later/)

编码快乐！