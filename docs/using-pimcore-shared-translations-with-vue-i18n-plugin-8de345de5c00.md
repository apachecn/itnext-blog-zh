# 通过 Vue i18n 插件使用 Pimcore 共享翻译

> 原文：<https://itnext.io/using-pimcore-shared-translations-with-vue-i18n-plugin-8de345de5c00?source=collection_archive---------1----------------------->

![](img/c2538e667c79462ffb21cfee95bd556b.png)

如今，大多数 web 应用程序都包含一些用 Javascript 框架构建的组件，如 Vue、React 或 Angular。将 Pimcore 后端与带有一些 Javascript 组件的响应式前端结合起来是一个非常强大且可伸缩的架构，我在我的大多数客户端项目中都在使用它。为什么？答案很简单——太棒了！Pimcore 是一个“数据管理”平台，一个用 Symfony 构建的 REST API(PIM core 由 Symfony 3.4 支持)和一个带有一些 Vue 组件的前端。

Pimcore 有一个很棒的功能，叫做“共享翻译”。它或多或少是 Symfony Translator 的扩展，但不是将翻译存储在文件系统上，而是存储在 SQL 数据库中。

> 这个解决方案是为前端到后端耦合的应用程序设计的。这意味着前端中的某些组件是用 Vue 构建的，而不是作为单页面应用程序构建的完整前端。

TL；DR；我们从数据库中读取翻译，对它们进行格式化，将它们推入 JavaScript `window`对象，然后在 Vue i18n 插件配置中使用来自`window`对象的翻译。

# 准备

为了将翻译从数据库转移到 Javascript `window`对象，我使用了一个名为`TranslationsDumper`的助手类。这个类从数据库中收集所有的翻译，并将它们作为一个递归数组返回，该数组按可用语言排序。

转储翻译的助手类

现在在你的`services.yml`文件中将这个类注册为服务，并将其添加到你的`config.yml`中的 Twig globals 中。

现在，我们需要以一种 JavaScript 组件可以全局使用的方式转储翻译。我是通过`window`对象来实现的。

> 重要提示:这个脚本标签需要放在包含您编译的 Vue 组件的脚本标签之前**。此外，我使用延迟枝扩展来延迟渲染这个块。点击此处阅读更多内容。[https://github.com/rybakit/twig-deferred-extension](https://github.com/rybakit/twig-deferred-extension)**

我们现在的现状是什么:

*   我们可以从数据库中读取我们所有的翻译
*   我们对翻译数据进行格式化，以便在后续步骤中使用
*   我们只打印 Pimcore 后端支持的语言的翻译
*   在我们的 JavaScript `window`对象中，所有翻译都是全局可用的。

# 让我们打印那些翻译！

首先，确保你已经安装了 i18n。如果没有，现在就做→[https://github.com/kazupon/vue-i18n](https://github.com/kazupon/vue-i18n)

下一步，我们需要在 Vue 组件中提供翻译。

**优势**

*   i18n 插件的语言环境链接到全局 Symfony 语言环境
*   翻译可以在 Pimcore 后端管理
*   使翻译作为 JavaScript 对象可用比通过 AJAX 调用获得翻译数据更快(没有闪烁…)