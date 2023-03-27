# 将 Vue 组件添加到旧的学校 web 应用程序中

> 原文：<https://itnext.io/adding-vue-components-to-old-school-web-apps-1f6c2339d599?source=collection_archive---------4----------------------->

![](img/1cc58d98451a96ba51a28541c5cbc9ff.png)

图片来源:[科技精神](https://techspirited.com/things-that-will-become-obsolete-in-next-10years)

Vue 对于单页面应用程序来说是一个极好的工具，但是试图将它集成到用 PHP 或 Python 编写的现有 CMS 中可能是一项令人生畏的任务(除非您正在使用 Laravel)。当你面对捆绑器、入口点、块、资产、公共路径和资源根时，尤其是在一个插件/插件驱动的架构(a-la Wordpress)中，这一切很快变得非常痛苦。

没有人想再处理 jQuery 了，尤其是在基于数据突变重新呈现组件的时候。乍一看，Vue 似乎是从 jQuery 迭代迁移到现代前端堆栈的一个可行的候选对象(比 React 或 Angular 更合适)。然而，当你开始这段旅程时，你会遇到许多问题和难以做出的决定。由于内联脚本和属性绑定，将整个页面作为 Vue 组件挂载通常是不可能的，并且会导致各种兼容性问题和冲突。试图基于 element #id 挂载多个 Vue 实例会导致大量重复的 JS 调用，并导致混乱的构建过程。如果项目使用 RequireJS，那么构建多个库目标还会有其他问题。棘手的问题不胜枚举。

如果我们可以构建一个单一的入口点作为调度程序，加载必要的 Vue 组件，并将占位符转换为 Vue 的挂载实例，会怎么样？

我最近参加了在 Brandung 举行的 Vue 会议，会上我们看到了一个有趣的方法，他们在 TYPO3 模板中加入了交互式 Vue 组件。我对这种方法很感兴趣，因为它似乎是我已经努力了一段时间的东西，我开始对它进行修补，看看我能想出什么。

假设我们想在页面上出现的所有卡片上添加一个 like 按钮，那么我们应该在卡片的 Twig 模板上添加这样的东西。如果您使用不同的模板语言或纯 HTML，请确保将 JSON 数据正确编码到元素属性中。

现在，让我们创建我们的`LikeButton` Vue 组件。这只是一个例子，所以有点傻——我想说的是，当存在可变的内部组件状态时，为什么你不想使用 jQuery。

我们现在要做的是在 DOM 中查询所有具有`data-vue`属性的元素，加载在`data-component`中引用的组件，准备道具，并用挂载的 Vue 实例替换我们的占位符元素。

如果您正在处理位于不同目录中的大量组件，您可以使用我编写的一个 [cli 脚本](https://www.npmjs.com/package/@hypejunction/vue-scanner)将所有组件解析到一个 JSON 对象中，并从中读取组件路径。

现在我们需要设置我们的构建过程。我喜欢使用`[laravel-mix](https://laravel-mix.com/)`，但是也欢迎你建立自己的构建过程。这里有一个`package.json`和`webpack.mix.js`的样本。我的项目叫`api_launcher`。我用我的项目名命名我的块，并使用`requestChunks`标志将每个单独的组件提取到它自己的块中，这有助于我保持运行时非常轻量级，只在需要安装组件和依赖项时加载它们。当在挂载期间导入组件时，webpack 使用已经通过`vue-scanner`定义并导入到我的索引中的正确块(在上面的例子中没有显示)。

如果这是您将在许多页面上使用的内容，请将编译好的`index.js`加载到您的全局模板的文档`<head>`中。在我的例子中，只要我有一个占位符`data-vue` div，我就加载 RequireJS 模块。

你现在要做的就是跑`npm run dev`或者`npm run prod`，你就万事俱备了。

使用过时的技术不一定会很痛苦:)

***更新 Sep 8:我在 PHP*** ***中拼凑了一个*** [***完全工作的例子。您会注意到，我对最初的方法做了一些更改——我意识到安装一个包装器组件会更容易，它将呈现一个动态组件*** `***<component :is="">***` ***，由 Vue 从其异步注册中解析，从而允许我们跳过需要查找和导入引用组件的部分。***](https://github.com/hypeJunction/vue-oldschool)

 [## hypeJunction/vue-oldschool

### 在 PHP 应用程序中使用 vue 的演示。在 GitHub 上创建一个帐户，为 hypeJunction/vue-oldschool 的发展做出贡献。

github.com](https://github.com/hypeJunction/vue-oldschool)