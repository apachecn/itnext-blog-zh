# ReactIntl +命名空间

> 原文：<https://itnext.io/reactintl-namespaces-2ba85d59133f?source=collection_archive---------6----------------------->

![](img/4e6b02a7e5f4e73fc49fdb9218c4b997.png)

[react ntl](https://github.com/yahoo/react-intl)是一个库，可以帮助您完成项目中与国际化和本地化相关的任务。

[*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Freactintl-namespaces-2ba85d59133f) 上分享这篇文章

它的好处是它试图以平台的方式解决问题，在引擎盖下它使用了 [ECMA-402](https://www.ecma-international.org/publications/standards/Ecma-402.htm) API，它现在填充了 [Intl.js](https://github.com/andyearnshaw/Intl.js/) 和 [FormatJS](https://formatjs.io/)

它的工作方式为您提供了两个主要组件 IntlProvider 和 FormattedMessage。IntlProvider 负责设置您的语言环境，并向 FormattedMessage 提供翻译的消息。FormattedMessage 用默认语言(引用语言)声明什么是您的资源文本，并格式化消息。稍后在构建过程中，你可以用 [babel 插件](https://github.com/yahoo/babel-plugin-react-intl)提取那些声明的消息资源，并将这些提取的文件发送给翻译器。

你可以像 ReactIntl 中的一个例子描述的那样使用它，在一个客户机上，你从一个已经在你的页面中声明的全局变量传递消息，这个全局变量是从服务器下载的

在服务器端，你发送从 index.html 翻译器收到的翻译信息

当可以预先加载所有资源时，这个解决方案是完美的。它的主要优点是什么？您不需要等待资源下载。当您更改语言时，只需用不同的语言环境重新加载您的页面。

当您有许多按需加载的模块时，您不想一次下载所有的资源，这就有问题了。你必须用翻译来维护文件，这对于开发者和翻译者来说都是一个痛苦，因为他们看不到 csv 文件的 xml、json 中的翻译上下文。

我遇到了这个问题，这就是为什么我想出了带有名称空间的 ReactIntl。你问的那些名称空间是什么…简单的推理方式是，当用户加载你的应用程序的一部分时，你想一起加载的一组翻译。

这个项目的灵感来自于扬·穆勒曼的《https://react.i18next.com》。

随着加载翻译的需求，你有很多新的可能性，你可以改善你的翻译过程。你不再需要通过电子邮件发送翻译文件，或者一些聊天和手动同步。你不必从你的应用程序中提取未翻译的信息，并把它们发送给某人进行翻译，你的应用程序可以自己完成这项工作。翻译人员可以使用您的应用程序和带翻译服务的上下文编辑器，只需点击应用程序的任何部分，就可以找到负责特定文本的资源。他们可以翻译它，刷新页面，并随时查看变化。他们看文本是否适合屏幕，他们看上下文。

当开发人员添加新消息时，它们会自动上传到翻译服务，您可以看到哪些消息是未翻译的。

我将在以后的文章中描述完整的过程。

第一个与[反应命名空间](https://github.com/maciejw/react-intl-namespaces)整合的翻译服务是 locize.com 的[。它很酷的一点是，他们有上下文编辑器，这个编辑器有另一个很酷的功能，除了许多其他功能，它可以帮助你处理](https://locize.com/) [ICU 格式](http://format-message.github.io/icu-message-format-for-translators/)的消息。在 [ReactIntl](https://formatjs.io/guides/message-syntax/) 中也支持这种格式，因为 ReactIntl 是使用 ICU 的 [FormatJS](https://formatjs.io/guides/message-syntax/) 的一部分。

反应命名空间如何工作？它位于 ReactIntl 之上，它将 IntlProvider 封装在 IntlNamespaceProvider 中，派生 FormattedMessage，并在此基础上添加 IntlBackendProvider

通常在 IntlProvider 上设置的内容可以在 IntlBackendProvider 上设置，使用 IntlNamespaceProvider 可以将资源分开并分组。消息 id 是有作用域的，所以两个名称空间可以有相同的 id。

为了使图片完整，你需要通过助手类 [ResourceProvider](https://github.com/maciejw/react-intl-namespaces/blob/master/packages/react-intl-namespaces/src/resourceProvider.ts#L81) 连接到一些后端，它负责协调资源下载和下载完成时的通知过程，它还具有一些功能，如下载另一种语言的资源，它可以检查给定语言的新翻译。

I 依赖于 [ResourceServer](https://github.com/maciejw/react-intl-namespaces/blob/master/packages/react-intl-namespaces/src/types.ts#L48) 接口的实现，该接口目前有一个 [locize.io api](https://docs.locize.com/api.html) 的实现。

我们需要做的只是配置我们的 LocizeClient 或任何其他资源服务器实现，用 getMessagesFromNamespaceFactory 将其连接到我们的应用程序，然后添加 MissingMessageFactory ant，就是这样。

在以后的文章中，我将解释如何将 locize-editor 与它集成，以及这个库还有哪些很酷的特性。

当我写这篇文章的时候，它已经有了版本 [0.1.0](https://github.com/maciejw/react-intl-namespaces/releases/tag/v0.1.0) 在 npm 上:

[](https://www.npmjs.com/package/react-intl-namespaces) [## react-intl-名称空间

### [react-intl]国际化库与[locize.com]在线翻译服务的集成。

www.npmjs.com](https://www.npmjs.com/package/react-intl-namespaces) [](https://www.npmjs.com/package/react-intl-namespaces-locize-client) [## react-intl-namespaces-locize-client

### [react-intl]国际化库与[locize.com]在线翻译服务的集成。

www.npmjs.com](https://www.npmjs.com/package/react-intl-namespaces-locize-client) [](https://www.npmjs.com/package/react-intl-namespaces-locize-editor) [## react-intl-namespaces-locize-editor

### [react-intl]国际化库与[locize.com]在线翻译服务的集成。

www.npmjs.com](https://www.npmjs.com/package/react-intl-namespaces-locize-editor) 

如果你觉得有趣，请在下面留下评论或 [github 问题](https://github.com/maciejw/react-intl-namespaces/issues/new)。

和平！