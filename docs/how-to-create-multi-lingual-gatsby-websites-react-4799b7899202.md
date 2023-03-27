# 如何创建多语言的盖茨比反应网站

> 原文：<https://itnext.io/how-to-create-multi-lingual-gatsby-websites-react-4799b7899202?source=collection_archive---------3----------------------->

## 开发多语言 Gatsby 网站的常见陷阱

![](img/8303357cdface9dbf4e257ae1818b286.png)

照片由[凯尔·格伦](https://unsplash.com/@kylejglenn?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

你是否在做一个必须使用多种语言的盖茨比网站？根据我们的经验，我们将深入解释如何实现这一点，同时揭示常见的陷阱。

在本文中，我们将使用 i18n 处理 json 翻译文件。

# 选择插件

有几个插件可用于国际化盖茨比网站。我们发现`gatsby-plugin-intl`似乎是最受欢迎的，在 [npm](https://www.npmjs.com/package/gatsby-plugin-intl) 上有大约 6k 的周下载量。它在幕后使用`react-intl`，在 [npm](https://www.npmjs.com/package/react-intl) 上有大约 95 万的周下载量！

从实施来看，它做了很多开箱即用的工作，设置非常简单，如[设置说明页](https://www.gatsbyjs.com/plugins/gatsby-plugin-intl/)所示。然而，不要让这愚弄你，有一些重要的项目你应该知道，因为一些具体的工作方式将需要使用本模块的一些变通办法。

# 设置 gatsby-plugin-intl

文档对于初始设置非常清楚，但是让我们一起来看一下。

首先，你需要安装 npm 包，`npm install --save gatsby-plugin-intl`。

其次，我们需要一个存储翻译文件的地方。通常，这存储在`/src/intl/[languagecode].json`中。用这个结构创建空的 json 文件。

在`gatsby-config.js`中设置插件配置，将下面的配置粘贴到你的插件数组中。我在插件列表的开头和结尾都没有发现任何问题，但是如果你在设置上有任何错误，你可能想在你的插件列表中移动它，以防有任何冲突的 Gatsby 插件。

注意，languages 数组需要包含与您之前创建的 i18n json 文件相同的名称。

设置现在已经完成，通过快速启动来确保您的 Gatsby 实例仍然正确运行。

# 使用国际化文件

接下来，我们可以开始使用翻译。假设我们有以下简单的翻译 json 文件`/src/intl/en.json`。稍后我们将讨论嵌套内容。也为另一种选择的语言创建一个类似的文件，例如，`/src/intl/nl.json`如果你也想用荷兰语。

## 使用 intl 挂钩

打开任何 React 组件，在那里您想要使用来自翻译的标签。首先，从`gatsby-plugin-intl`包装中导入`useIntl`挂钩。

然后，我们可以使用这个钩子获取`intl`对象，如下所示。

确实非常简单。从这一点开始，我们可以使用`formatMessage`函数或者`FormattedMessage` React 组件来获取翻译。

现在轮到你了，试一下，看看你是否得到了正确的翻译！从`localhost/en/`文件夹切换到`localhost/nl/`文件夹将为您提供您在配置中定义的另一种语言的翻译。

## 使用 Intl HOC(高阶组件)

使用来自`gatsby-plugin-intl`包的特设版本没有太大的不同。我们先导入`injectIntl`函数。

然后，当我们导出我们的组件时，我们用`injectIntl`函数包装它。

然后我们将收到作为道具的`intl`对象。

# 创建到其他页面的链接

由于 URL 结构有了细微的变化，包括路径中的区域设置，您也需要注意这一点。

或者，您可以使用`intl.locale`作为值，并自己生成路径，比如`/${intl.locale}/contact`，或者您可以使用`gatsby-plugin-intl`包中专门制作的组件。这里有一个例子:

# 常见陷阱

第一次设置`gatsby-plugin-intl`时有一些陷阱。让我们来看看最重要的几个。

## 嵌套翻译

首先不解释这类 i18n json 文件中允许的结构。事实上，目前允许**一级嵌套**。例如:

然而，嵌套超过一层**似乎还不支持**！遗憾的是，目前我们只停留在一层嵌套上。如果您尝试这样做，您可能会得到一个神秘的跨环境反应错误。深入挖掘，你会发现错误日志提到本地化没有找到。

## 带列表的翻译

在 i18n 文件中定义数组没有像我们预期的那样工作。消息被正确定义，但是`formatMessage`方法此时不能返回数组。例如:

我们可以看到，我们可以通过指定完整路径来检索项目，但是不可能一次检索完整的数组。

一种解决方法是不使用`/src/intl`文件，而是创建自己的文件结构。然后使用`intl.locale`值，您可以导入正确的文件。这是一个想法，虽然我们还没有测试这在 SEO 方面意味着什么，标签是否会被立即解析。

## 更新翻译文件

更新 i18n 转换文件后，gatsby 服务器需要完全重启。`gatsby develop`服务器热重装不起作用！

# 结论

`gatsby-plugin-intl`很棒，它自动化了开发多语言网站的许多任务。但是，您应该确切地知道如何使用它，i18n 文件的结构是什么，等等。

仍然遗憾的是，它不支持多层嵌套，以及翻译文件中的数组。拥有这种支持将是一个很好的补充，有助于使用更动态的内容格式。

# 进一步阅读

总而言之，我们对`gatsby-plugin-intl`很满意，但这不是唯一可用的实现。另一个插件`gatsby-plugin-i18n`的周下载量约为 5k，略低于 T7。

由 [David Dal Busco](https://medium.com/u/94f0c8061324?source=post_page-----4799b7899202--------------------------------) 撰写的关于`gatsby-plugin-i18n`的完整指南。我建议看一看他的文章，了解实现国际化的不同方式，以便做出决定并选择符合您需求的插件。

[](https://betterprogramming.pub/internationalization-with-gatsby-ae3991c39e92) [## 与 Gatsby 的国际化

### 如何用 gatsby-plugin-i18n 和 react-intl 国际化你的 Gatsby 网站

better 编程. pub](https://betterprogramming.pub/internationalization-with-gatsby-ae3991c39e92) 

我们强烈建议阅读 npm 上提供的文档，以了解完整的功能集。在本文中，我们只强调了我们认为最重要的特性。

[](https://www.npmjs.com/package/gatsby-plugin-intl) [## 盖茨比插件国际

### 国际化你的盖茨比网站。把你的 gatsby 网站变成一个国际化的框架

www.npmjs.com](https://www.npmjs.com/package/gatsby-plugin-intl) 

[订阅我的媒介](https://kevinvr.medium.com/membership)到**解锁** **所有** **文章**。通过使用我的链接订阅，你是支持我的工作，没有额外的费用。你会得到我永远的感激。