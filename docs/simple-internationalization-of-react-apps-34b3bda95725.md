# React 应用程序的简单国际化

> 原文：<https://itnext.io/simple-internationalization-of-react-apps-34b3bda95725?source=collection_archive---------0----------------------->

![](img/8587496e2f5ce6c2103093ba35435382.png)

由[罗曼·维涅斯](https://unsplash.com/photos/ywqa9IZB-dU?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/dictionary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

> **注:**有没有单语的 app？继续读！像复数、数字和日期格式这样的东西即使对于单语项目也仍然有用。

国际化是一种完美的方式，可以让你的项目为更多的观众所接受。

`react-intl`说到 React apps 的 i18n 还是标配。尽管它已经有一年没有维护了，但是这个社区是如此的强大，以至于它越来越受欢迎，新的独立工具也在不断地被发布。

我想向您介绍我在过去一年半时间里一直在研究的另一种 i18n 解决方案— [LinguiJS](https://github.com/lingui/js-lingui) 。底层 API 与`react-intl`非常相似，使我的项目迁移更容易，但除此之外，还有完全不同的开发人员体验。

## 语言简介

我开始使用 LinguiJS 作为一个实验，如何使用 Babel 简化 i18n。我花了几个小时调试消息语法中的语法错误，还花了大量时间构建缺失的工具。经过几次迭代，我得到了一个非常类似于`react-intl`的库，但是在引擎盖下优化了几个`react-intl`缺少的特性。

公开宣布他们正在使用 [LinguiJS](https://github.com/lingui/js-lingui) 的项目并不多，但是这个库已经有了很多兴奋的用户。加入 [LinguiJS showroom](https://lingui.js.org/misc/showroom.html) 的最新项目是 [MyMusicTaste](https://www.mymusictaste.com/) ，一款翻译成 13 种语言的打字稿 app。

让我们继续学习教程。我假设对 i18n 一无所知，从头开始:

# 什么是国际化

W3C 有一个我一直使用的美丽定义:

> 国际化是指产品、应用程序或文档内容的设计和开发，能够为不同文化、地区或语言的目标受众轻松实现本地化。— [W3C Web 国际化常见问题解答](https://www.w3.org/International/questions/qa-i18n)

这是什么意思？当你开发一个应用程序，你想让它支持多种语言，你需要找到一种方法来轻松地交换文本的翻译。您应该能够在不修改代码的情况下做到这一点，只需为所有消息提供不同的字典。翻译人员只需很少或根本不需要编程技能就能翻译词典。

换句话说:**国际化**就是你作为一个开发者所做的事情，允许翻译人员**将你的应用程序本地化**成多种语言。

# 步骤 1 —安装

让我们从安装所需的工具开始。 [LinguiJS](https://github.com/lingui/js-lingui) 附带有用的 [CLI](https://lingui.js.org/tutorials/cli.html) 我们可以用它来安装所有需要的包:

```
npm install -g @lingui/cli
lingui init
```

如果你不信任它，只需运行`lingui init --dry-run`来检查它将安装什么。

命令安装`@lingui/core`和`@lingui/react`。第一个包包含所有 i18n 功能，可以用于任何项目。第二个包提供了建立在核心功能之上的 React 组件，主要是为了利用 React 生命周期方法和优化重新呈现。

它还安装了我们需要添加到我们的`.babelrc`配置中的`@lingui/babel-preset-react`:

```
{
  **"presets"**: [
    "env",
    "react",
    "@lingui/babel-preset-react"
  ]
}
```

**注:**这个巴别预设完全可选，但推荐。XXX

# 步骤 2 —准备应用程序

首先，我们需要能够从外部字典加载消息，并设置我们将在应用程序中使用的语言。这是`I18nProvider`的工作。你只需要一次，它应该是你应用程序中最顶层的组件(类似于[的`Provider`react-redux](https://github.com/reduxjs/react-redux)或者 [react-apollo](https://github.com/apollographql/react-apollo) 的 ApolloProvider)。

```
*// index.js*
**import** React from 'react'
**import** { render } from 'react-dom'
**import** Inbox from './Inbox.js'

**import** { I18nProvider } from '@lingui/react'const catalogs = {}  // don't worry about it now**const** App = () => (
  <**I18nProvider** language="en" catalogs={catalogs}>
    <**Inbox** />
  </**I18nProvider**>
)

render(<**App** />, document.getElementById('app'))
```

因为我们没有任何字典(也称为消息目录)，所以现在让`catalogs` prop 为空。我们晚点回来。`I18nProvider`设置我们接下来要使用的 i18n 上下文。

# 步骤 3——国际化

有趣的部分来了。我们将把我们的简单应用程序变成一个可以翻译的应用程序。这是我们的应用程序:

```
**const** Inbox = ({ messages, markAsRead, user }) => {
   **const** messagesCount = messages.length
   **const** { name, lastLogin } = user

   **return** (
      <**div**>
        <**h1**>Message Inbox</**h1**>

        <**p**>
          See all <**Link** to="/unread">unread messages</**Link**>{" or "}
          <**a** onClick={markAsRead}>mark them</**a**> as read.
        </**p**>

        <**p**>
          {
            messagesCount === 1
              ? "There's {messagesCount} message in your inbox."
              : "There're {messagesCount} messages in your inbox."
          }
        </**p**>

        <**footer**>
          Last login on {lastLogin}.
        </**footer**>
      </**div**>
   )
}
```

假设我们想翻译标题**消息收件箱**。我们需要将它包装成组件，组件从字典中加载翻译。

[LinguiJS](https://github.com/lingui/js-lingui) 对简单消息使用`[<Trans>](https://lingui.js.org/ref/react.html#trans)`组件。我们需要做的就是将我们的消息包装在这个组件中:

```
<h1><Trans>Message Inbox</Trans></h1>
```

它是如何工作的？`[<Trans>](https://lingui.js.org/ref/react.html#trans)`组件将在字典中查找**消息收件箱**或将其替换为翻译。这是默认的方法，当*我们使用消息作为消息 id*时。邮件 ID 是在字典中标识邮件的信息。例如，捷克语词典会是这样的:

```
{
  "Message Inbox": "Příchozí pošta"
}
```

> 消息 ID 由开发人员定义(在源代码中)，翻译由翻译人员定义(在字典中)。

## 消息或密钥作为消息 id？

您可能听说过使用消息作为 id 是一种反模式，最好使用自定义键，如下所示:

```
<h1><Trans id="Inbox.message">Message Inbox</Trans></h1>
```

然后，字典会是这样的:

```
{
  "Inbox.message": "Příchozí pošta"
}
```

我就不争论哪个更好了。我一直在从事两种方法都使用的大型项目，但没有一个是完美的。它们各有利弊，最终问题不是你用什么作为消息 id，而是你如何在你的开发人员和翻译人员之间建立翻译工作流程。

重要的一点是: [LinguiJS](https://github.com/lingui/js-lingui) 支持这两者，并让您，作为一名开发人员，选择最适合您和您的团队的方式。

## 带富文本格式的翻译

让我们通过下一条消息继续我们的旅程:

```
<**p**>
  See all <**Link** to="/unread">unread messages</**Link**>{" or "}
  <**a** onClick={markAsRead}>mark them</**a**> as read.
</**p**>
```

这一个有点棘手，因为消息包含 React 组件。理想情况下，我们希望在没有任何标记干扰的情况下翻译“查看所有未读邮件或将它们标记为已读”。不幸的是，我们不能轻易预测翻译后的消息看起来会是什么样子，所以我们需要包含最少的标记。幸运的是，<trans>组件自动为我们做了这件事:</trans>

```
<**p**>
   <**Trans**>
      See all <**Link** to="/unread">unread messages</**Link**>{" or "}
      <**a** onClick={markAsRead}>mark them</**a**> as read.
   </**Trans**>
</**p**>
```

没错。我们需要做的就是将消息包装在<trans>中。消息看起来会是什么样子？包括属性在内的 React 组件被删除，翻译器只获得相关信息。邮件 ID 将如下所示:</trans>

```
See all <0>unread messages</0> or <1>mark them</1> as read.
```

我曾经做过在消息中包含 html 属性的项目，当我们在改变类或 href 后必须更新翻译时，这总是困扰着我。在这种情况下，translator 得到了他们需要的一切，我可以使用内置元素和反应组件，而没有任何限制。

和前面的例子一样，如果您喜欢用键作为消息 id，也可以这样做:

```
<**p**>
   <**Trans id**="Inbox.actionsDescription">
      See all <**Link** to="/unread">unread messages</**Link**>{" or "}
      <**a** onClick={markAsRead}>mark them</**a**> as read.
   </**Trans**>
</**p**>
```

现在翻译者将得到:

```
{
  "Inbox.actionsDescription": 
    "See all <0>unread messages</0> or <1>mark them</1> as read."
}
```

## 复数

下一条信息也很棘手，但方式不同。

```
<**p**>
  {
    messagesCount === 1
      ? "There's {messagesCount} message in your inbox."
      : "There're {messagesCount} messages in your inbox."
  }
</**p**>
```

看到了吗？有时信息依赖于一个变量。不仅仅是在消息内部包含一个变量，而是使用完全不同的形式。这是一种常见的语言特征，称为复数，通常我们在收件箱中使用简化的消息来避免这个问题。不幸的是，这只适用于有两种复数形式(单数、复数)的语言。在捷克语中，我们有 3 种复数形式，有些语言甚至更多。

现在怎么办？我们不能对每种语言都有不同的`if`条件。这需要针对每种语言重构我们的应用程序。相反，我们将使用国际化消息格式来处理所有需要的语言特性。

我给你介绍一下 [ICU 消息格式](https://lingui.js.org/ref/message-format.html)。这种格式足够简单，即使没有编程知识的翻译人员也可以使用，但功能足够强大，可以支持不同语言的所有语法差异。

我们的消息以 ICU 消息格式表示，如下所示:

```
{messagesCount, plural,
   one {There's # message in your inbox}
   other {There're # messages in your inbox}
}
```

我们看到有一个变量`messageCount`，接着是格式化程序`plural`，最后是这个格式化程序的参数。`one`用于单数，`other`用于复数。在捷克语中，这将是:

```
{messagesCount, plural,
   one {Máte # zprávu v příchozí poště}
   few {Máte # zprávy v příchozí poště}
   other {Máte # zpráv v příchozí poště}
}
```

看出区别了吗？除了一个词之外，这些信息几乎完全相同！

我猜你现在感到很困惑，因为这一切看起来很复杂。但是再一次， [LinguiJS](https://github.com/lingui/js-lingui) 将事情简化了很多。

这次我们要用[<>](https://lingui.js.org/ref/react.html#plural)的复数代替[<>](https://lingui.js.org/ref/react.html#trans)的反式成分:

```
<**p**>
   <**Plural**
      value={messagesCount}
      one="There's # message in your inbox"
      other="There're # messages in your inbox"
   />
</**p**>
```

我们用复数> 组件替换了 conditional，并在`one`和`other`道具中添加了消息。这就是我们需要做的。LinguiJS 自动生成 ICU MessageFormat 格式的语法有效的消息。

我们也不关心其他语言的复数。作为开发人员，我们只需要提供源代码中使用的语言的复数形式(本例中为英语)。其他一切都由翻译和图书馆处理。

## 引擎盖下的复数

[<复数>](https://lingui.js.org/ref/react.html#plural) 组件对于不同复数形式的翻译(例如捷克语中的一个、几个、其他)是如何工作的并不明显，所以让我们看看它是如何工作的。

[<复数>](https://lingui.js.org/ref/react.html#plural) 组件在 Babel 插件中解析，替换为 [< Trans >](https://lingui.js.org/ref/react.html#trans) 组件:

```
<**p**>
   <**Trans**
      id="{messagesCount, plural, one {There's # message in your inbox} other {There're # messages in your inbox}}"
      values={{ messagesCount }}
   />
</**p**>
```

复数的格式化由核心库使用语言特定的复数规则来处理。当我们从翻译器获得翻译后的消息时，我们不必改变我们的 [<复数>](https://lingui.js.org/ref/react.html#plural) 组件。它只是作为 ICU MessageFormat 的一种更简单的语法。

值得一提的是，自定义键也支持复数:

```
<**p**>
   <**Plural** id="Inbox.messageCount"
      value={messagesCount}
      one="There's # message in your inbox"
      other="There're # messages in your inbox"
   />
</**p**>
```

翻译者会得到:

```
{
  "Inbox.messageCount": 
    "{messagesCount, plural, one {There's # message in your inbox} other {There're # messages in your inbox}}"
}
```

复数有点难以理解，但是非常容易使用。我试图深入解释它们是如何工作的，但大多数信息只与翻译人员相关。作为开发人员，我们关心的只是默认语言使用的复数形式。

## 日期和数字格式

最后，我们将看到应用程序中的最后一条消息。在上一节中，我谈到了复数，这是一种语言的特征。这次我们将关注日期和数字格式，这取决于国家和文化。

在美国，通常用格式`MM/DD/YYYY`写日期，而在捷克语中，我们通常写`YYYY/MM/DD`。同样，作为开发人员，我们不应该被要求知道我们需要使用什么样的日期格式。那是翻译的工作。我们只需将消息包装在我们习惯的 [< Trans >](https://lingui.js.org/ref/react.html#trans) 组件中，并使用 [< DateFormat >](https://lingui.js.org/ref/react.html#dateformat) 组件进行日期格式化。

原始邮件:

```
<**footer**>
  Last login on {lastLogin}.
</**footer**>
```

…看起来会像这样:

```
<**footer**>
   <**Trans**>
      Last login on <**DateTimeFormat** value={lastLogin} />.
   </**Trans**>
</**footer**>
```

翻译器将处理消息`Last login on {lastLogin, date}.``lastLogin`变量使用`date`格式化器格式化，该格式化器使用 [Intl。DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat) 隐藏起来(因此允许相同的格式选项)。

数字格式非常相似，我们只是用 [<数字格式>](https://lingui.js.org/ref/react.html#numberformat) 代替。

# 步骤 4-创建消息目录

我们的工作大部分已经完成。一旦我们的应用程序国际化了，我们需要创建一个**消息目录**(我称之为**字典**)并发送给翻译人员。

CLI 又来了。命令解析源代码，找到所有需要翻译的消息，并将其提取到外部文件中。

首先，我们需要定义我们将在`[lingui add-locale](https://lingui.js.org/ref/cli.html#add-locale)`命令中使用的语言。先说英语和捷克语:

```
lingui add-locale en cs
```

我们会得到一个确认和一个类似 git 的帮助，告诉我们下一步该怎么做:

```
Added locale en.
Added locale cs.

(use "lingui extract" to extract messages)
```

运行`[lingui extract](https://lingui.js.org/ref/cli.html#extract)`在`locale`目录下创建一个消息目录；

```
lingui extract
```

该命令输出关于总翻译和缺失翻译的统计信息:

```
Extracting messages from source files…
Collecting all messages…
Writing message catalogs…
Messages extracted!

Catalog statistics:
┌──────────┬─────────────┬─────────┐
│ Language │ Total count │ Missing │
├──────────┼─────────────┼─────────┤
│ cs       │     4       │   4     │
│ en       │     4       │   4     │
└──────────┴─────────────┴─────────┘

(use "lingui add-locale <locale>" to add more locales)
(use "lingui extract" to update catalogs with new messages)
(use "lingui compile" to compile catalogs for production)
```

默认情况下，提取的空目录看起来像这样，但是 [LinguiJS](https://github.com/lingui/js-lingui) CLI 也支持一个 [po 文件](https://lingui.js.org/ref/conf.html#po):

```
{
  **"Message Inbox"**: "",
  **"See all <0>unread messages</0> or <1>mark them</1> as read."**: "",
  **"{messagesCount, plural, one {There's {messagesCount} message in your inbox.} other {There're {messagesCount} messages in your inbox.}}"**: "",
  **"Last login on {lastLogin,date}."**: "",
}
```

我们在源代码中使用英语，因为我们不使用关键字作为消息 id，所以我们不需要翻译英语消息目录！让我们通过将`lingui` [配置](https://lingui.js.org/ref/conf.html)添加到我们的`package.json`来修复它(如果您使用密钥作为消息 id，请跳过这一步):

```
{
  "lingui": {
    "sourceLocale": "en",
  }
}
```

另外，让我做你的捷克语翻译。这是捷克语的目录:

```
{
  **"Message Inbox"**: 
    "Přijaté zprávy", **"See all <0>unread messages</0> or <1>mark them</1> as read."**:
    "Zobrazit všechny <0>nepřečtené zprávy</0> nebo je <1>označit</1> jako přečtené.", **"{messagesCount, plural, one {There's {messagesCount} message in your inbox.} other {There're {messagesCount} messages in your inbox.}}"**:
    "{messagesCount, plural, one {V příchozí poště je {messagesCount} zpráva.} few {V příchozí poště jsou {messagesCount} zprávy. } other {V příchozí poště je {messagesCount} zpráv.}}", **"Last login on {lastLogin,date}."**:
    "Poslední přihlášení {lastLogin,date}",
}
```

让我们再次运行`[lingui extract](https://lingui.js.org/ref/cli.html#extract)`:

```
Extracting messages from source files…
Collecting all messages…
Writing message catalogs…
Messages extracted!

Catalog statistics:
┌────────────┬─────────────┬─────────┐
│ Language   │ Total count │ Missing │
├────────────┼─────────────┼─────────┤
│ cs         │     4       │   0     │
│ en (source)│     4       │   -     │
└────────────┴─────────────┴─────────┘

(use "lingui add-locale <locale>" to add more locales)
(use "lingui extract" to update catalogs with new messages)
(use "lingui compile" to compile catalogs for production)
```

就是这样！我们现在有了所有需要的翻译，我们可以进入最后一步。如你所见，我们可以随时运行`[lingui extract](https://lingui.js.org/ref/cli.html#extract)`，它会将现有的翻译与任何新消息合并。当我们从源代码中删除消息时，它还会标记过时的消息。

# 步骤 5—加载翻译

下面是与其他 i18n 库有点不同的部分。在加载翻译后的消息目录之前，我们需要编译它们。编译将 ICU MessageFormat 中的消息转换成简单的 JavaScript 函数，该函数接受插值的值。

编译使库变得很小很快，因为我们不需要在产品包中包含消息解析器。

我们可以使用`[lingui compile](http://lingui compile)`手动编译消息目录:

```
Compiling message catalogs…
Done!
```

该命令在原`messages.json`旁边创建`messages.js`。值得一提的是，`messages.json`是为翻译人员准备的文件，而`messages.js`是加载到我们的应用程序中的。

如果你使用的是 webpack，你可以使用`[@lingui/loader](https://lingui.js.org/ref/loader.html)`来动态编译信息。

现在我们需要将目录加载到我们的应用程序中。让我们回到教程的开头:

```
*// index.js*
**import** React from 'react'
**import** { render } from 'react-dom'
**import** Inbox from './Inbox.js'
**import** catalogEn from './locale/en/messages.js'
**import** catalogCs from './locale/cs/messages.js'

**import** { I18nProvider } from '@lingui/react'**const** catalogs = { en: catalogEn, cs: catalogCs }**const** App = () => (
  <**I18nProvider** language="en" catalogs={catalogs}>
    <**Inbox** />
  </**I18nProvider**>
)

render(<**App** />, document.getElementById('app'))
```

为了简化这个冗长的教程，我们一次加载所有消息。prop 接受一个字典，其中键是可用的语言，值是编译的目录。

如果你对更复杂的用例感兴趣，看看这篇关于使用 webpack 动态加载消息目录的指南。

# 第六步——下一步是什么

还有一些工作要做，比如切换语言(取决于 React 使用的数据层)，但 i18n 的基础知识已经涵盖。

在 [React 中描述了许多常见的 i18n 模式——常见模式](https://lingui.js.org/tutorials/react-patterns.html)教程，如文本属性的翻译或在 React 上下文之外使用 [LinguiJS](https://github.com/lingui/js-lingui) (例如在 redux-saga 中)。

查看全面的[文档](https://lingui.js.org/)以获取其他教程、指南和参考资料。

# 今后

下一个版本的 LinguiJS 将在不久的将来推出。巴别塔插件将被许多框架支持的巴别塔宏所取代。例如，我已经用即将发布的 create-react-app 2.0 测试了开发版本，效果非常好。

不要误会我的意思，你现在可以在 CRA 上使用 [LinguiJS](https://github.com/lingui/js-lingui) ，但是没有插件和无缝语法，它只是另一个 i18n lib(虽然占用空间更小)。

即将推出的版本也会小一点。现在核心库是 2kb gzipped，React 组件增加了 3kb gzipped。总的来说，这比全功能的 i18n lib 的[情感](https://github.com/emotion-js/emotion)少两倍。

我也在研究开箱即用的代码分割，但这需要更多的研究。

如果你有兴趣，试试看。加入我们的 [gitter](https://gitter.im/lingui/js-lingui) 或者在 twitter 上分享你的经历。我很好奇你的意见或建议。