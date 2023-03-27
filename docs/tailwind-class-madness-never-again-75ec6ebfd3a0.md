# 顺风级疯狂。再也不会了？！

> 原文：<https://itnext.io/tailwind-class-madness-never-again-75ec6ebfd3a0?source=collection_archive---------1----------------------->

几天前，我姐姐请求帮助。她想为她经营的幼儿园创建一个小的联系/宣传页面。没有什么特别和花哨的，只是一个与一些事件图形和联系形式的常规登录页面。周末工作。

好吧，听起来像是一个非常容易和快速的任务来帮助我有需要的兄弟姐妹。首先想到的是 [Vue](https://vuejs.org) ，下一个是[顺风](https://tailwindcss.com)。经过进一步思考后，我发现这是一个将 [Vite](https://vitejs.dev) …用于现实项目的绝佳机会。

![](img/6c18d0992f2d9f4f8e372cce4021fb8c.png)

维克多·比斯特罗夫在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

所以我有我的 UI/前端部分的工具，剩下的是表单处理器和应用程序部署，服务平台。我可以从网上了解到这一切……决定已经做出。

安装和运行 Vite 非常简单。这个工具的性能令人难以置信，它的速度非常快，功能非常强大。事实证明，我也可以玩一下[脚本设置](https://v3.vuejs.org/api/sfc-script-setup.html)的概念。它“强制”了一种新的编写方法，但最终它非常方便，并且与[组合 API](https://v3.vuejs.org/api/composition-api.html) 流程一致。

好了，我的开发环境准备好了。现在我只需要开始创建实际的组件。因为我使用的是 Tailwind，所以我需要用我的默认风格值来设置一些初始配置。你知道，字体大小，颜色，和其他东西。好吧。

用一些网格类创建容器，很好，这里没什么特别的。是时候使用一些全局的、可重用的组件了，比如按钮。我们开始吧...🤕顺风公用事业类疯狂。为了定义一个简单的按钮，我不得不使用 20 多个类。闪回——每次当我使用 Tailwind 时，我都要处理这些超长的行标记——太可怕了。噩梦对吗？

**这必须结束！**至少对我的 Vue 应用来说是这样。**😃**

我知道，我可以使用 [apply](https://tailwindcss.com/docs/extracting-components#extracting-component-classes-with-apply) handler 并在 style 部分定义所有的类。但这根本不是解决问题。这也不那么全局，“重用”友好。所以我需要为我的全局组件定义一些全局变量，比如按钮、输入、块。但是如何配合顺风和 Vue 使用它们呢？很简单，只要我可以在 Vue 模板中绑定我的样式，我就可以将我的样式定义为一个对象，并在任何地方使用它们。当然，当我使用组合 API 时，我可以创建一些奇特的可组合的东西。这就是 [**vue-use-variant**](https://github.com/lukasborawski/vue-use-variant) 包的创建过程。

主要目标是创建全局可访问的对象或带有一些样式定义、组件变体的对象。就这么办吧。为了按钮。

太好了。👍我们有一些整体的按钮样式和一些变体定义——主要和次要的。典型的一个。😅

好了，现在是可组合的，以及我如何将它与上面的变体一起使用。

您的最终结果将如下所示。

```
font-bold rounded border-0 bg-blue hover:opacity-80 p-4 text-lg
```

就是这样。很棒吧？它干净、易读、方便、快捷，并且全球可用。阶级疯狂已经结束。🎉你可以用 **Ref** objects，用 **props** ，直接在组件内部使用。最后，你可以将它与**任何其他框架**一起使用。它是为 Vue 生态系统而构建的，但如果你愿意，它不会阻止你将它与 React 一起使用。

为所有可重用组件创建您的变体，并在您的应用程序中使用它们。通过这种方式，您将始终获得一个真实的来源，并且您的 UI 组件将始终具有相同的形状和视觉表示。

在此找到技术规范和一些指南[。可以从](https://github.com/lukasborawski/vue-use-variant) [**npm**](https://www.npmjs.com/package/vue-use-variant) 或者 [**纱**](https://www.npmjs.com/package/vue-use-variant) 安装。您可以使用存储库中提供的非常基本的**演示**。它是用 Vite 创造的，所以那可能是额外的兴奋剂。当然，我们非常欢迎任何建议和问题报告。

感谢阅读。尽情享受吧！✋也许会考虑请我喝杯咖啡。