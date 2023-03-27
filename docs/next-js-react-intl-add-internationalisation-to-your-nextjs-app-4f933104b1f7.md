# 下一个 Js + React-Intl:在你的下一个应用中加入国际化

> 原文：<https://itnext.io/next-js-react-intl-add-internationalisation-to-your-nextjs-app-4f933104b1f7?source=collection_archive---------0----------------------->

![](img/62849cd3210a93b8b0c357f7c1628770.png)

实施国际化在过去是一个巨大的痛苦。Next 10 通过在内部处理一些必需的特性，简化了这个过程。

在这篇博客文章中，你将了解到

*   如何在你的下一个应用中集成 react-intl
*   如何配置 CLI 以自动提取消息
*   如何配置 Babbel 为您的消息自动生成 Id
*   如何使用 NextJs 10(及更高版本)中新的国际化特性

**先决条件:**

*   对 NextJS 的一般了解
*   对 react-intl 的一般理解

## 💻按照这篇文章安装所需的包

```
npm install react-intl && npm install --save-dev @formatjs/cli babel-plugin-formatjs
```

## 👏🏼使用 Next 的新本地化特性

从 Next 10 开始，框架将为您完成大部分的国际化工作。接下来会帮助你…

*   确定向用户显示的最佳语言(通过使用浏览器配置的语言或 Cookie)
*   使用子路径(example.de)或域路由(example.de)处理页面路由
*   在文档的全局< html >标签上设置`lang`属性

## ☄️:我们开始吧

要配置您的下一个应用程序使用国际化，请在您的项目根目录中创建或编辑您的`next.config.js`文件(即使您的项目是 TypeScript 项目，该文件也需要是 JavaScript 文件):

> **注意:**为了简单起见，我将在这篇博文中设置子路径路由。子路径路由会在你的 URL 后面添加一个 [UTS 地区标识符](https://unicode.org/reports/tr35/#Identifiers)，例如 example.com/de(你的应用程序的德语版本)或 example.com/en(你的应用程序的英语版本)。

```
// next.config.js
module.exports = {
  i18n: {
    locales: ['en-US', 'de-DE'],
    defaultLocale: 'en-US',
  },
}
```

这将告诉 Next 为您配置的语言环境处理子路径路由。

## 🚙创建区域设置文件夹和起始文件

React-intl 需要两个文件夹来管理你的语言文件。

1.  **一个包含您的翻译的文件夹(默认语言环境将由 CLI 生成):**您将使用该文件夹来创建&您想要翻译的编辑字符串。
2.  **编译语言文件的文件夹:** FormatJS 将使用您的翻译文件，并根据它们创建编译文件。React-intl 将使用这些文件来显示翻译后的字符串。

## 💻创建以下文件夹和文件:

1.  项目根目录下名为`content`的**文件夹**
2.  一个**文件夹里的**叫做`locales`里面的`content`
3.  一个**文件**在`locales`里面叫做`en.json`
4.  一个**文件**在`locales`里面被称为`de.json`
5.  一个叫做`compiled-locales`的**文件夹**里面的内容

在`en.json`和`de.json`内放置一个空的物体`{}`。空对象是有效的 JSON。

## 🤩配置 React-Intl

如果您以前使用过 react-intl，您会意识到设置 react-intl 需要在文档顶部提供一个提供者。转到您的`_app.tsx`文件并添加以下内容:

🎙我们在这里做了什么？

*   由于 NextJS 在内部处理国际化，它允许我们在应用程序的不同位置提取地区信息(例如 getStaticProps，getServerSideProps)。或者，我们可以使用`useRouter()`钩子来访问用户的当前语言环境。我们正在访问这个区域并从中提取前两个字母。于是，`en-US`(这是 locale)变成了`en`。我们使用前两个字母，因为我们希望向所有英语用户(美国、英国等)只提供我们应用程序的一种翻译。)
*   我们使用`messages`来检索正确的编译(！)翻译文件。我们通过使用`messages`属性将匹配的翻译文件传递给`IntlProvider`。
*   我们在`IntlProvider`上指定了一个`onError`道具，以抑制开发过程中恼人的错误消息。这些错误消息提醒您缺少翻译。如果您想查看错误，可以忽略此设置。

## 💻配置格式命令行界面

react-intl 的主要优势之一是它带有一个专用的 CLI。CLI 允许我们自动检索我们想要翻译的所有消息，为它们分配一个唯一的 Id，并将它们放入我们的翻译文件中。

**🎙是什么意思？**

如果我们通过使用`<FormatMessage .. /> or intl.formatMessage(...)`指定一个翻译字符串，我们可以运行一个 CLI 命令来获取新消息，并用新消息自动填充我们的默认语言环境文件。这意味着我们不必手动将每条新消息添加到我们的翻译文件中(至少对于我们的默认语言是这样)。

为了使用 FormatJS CLI，我们必须对其进行配置。在这篇文章的开始，我们已经安装了 CLI。现在，我们使用 package.json 文件配置定制命令。

在`package.json`文件的脚本部分，添加以下内容:

🎙我们在这里做什么？

*   `extract`命令将 CLI 配置为遍历文件夹`pages, components & sections`并找到所有定义的消息。它将对消息应用散列函数，并将输出作为本地文件中的一个键。React-intl 使用这种机制来检查消息是否发生了变化。我们定义所有找到的消息都应该被写到位于`content/locales/en.json`的`en.json`文件中。
*   `compile`命令将从`content/locales/*`中提取消息，并从我们的语言环境文件中创建编译后的消息。**节点:**确保您的文件夹路径不以“/”结尾，因为在这种情况下 CLI 将找不到您的文件。
*   `i18n`命令是一个快捷方式，我们可以在开发过程中使用它来提取和编译我们的翻译文件，并检查它们的正确性和完整性(您必须手动管理您的翻译或使用工具/服务)

你可以在这里 (FormatJS CLI 参考)阅读关于这些选项[的更多信息。](https://formatjs.io/docs/tooling/cli/)

## 🖐🏽在运行命令之前

为了配置我们的翻译，我们还需要做一步。到目前为止，我们已经配置了 react-intl 来为我们自动生成翻译 id。这是一个设计选择，因为我们也允许手动提供 id。

> 要手动提供 id，您可以这样指定消息:
> 
> <formattedmessage defaultmessage="Good Morning" class="ka ir">id="some id" / ></formattedmessage>

为了将这些自动生成的 id 附加到我们的消息中，我们必须使用定制的 [Babel](https://babeljs.io/) 配置来配置我们的构建管道。在您的项目根文件夹中，创建一个名为`.babelrc`的文件。这个文件是 Babel 的标准配置文件。将以下内容添加到文件中:

在这里，我们使用了我们在这篇博文开始时安装的`@babel-plugin-formatjs`。

> **注意:如果您已经运行了任何命令或更改了您的区域文件夹中的文件:**删除`content/compiled-locales`中所有编译的语言文件，并将`content/locales`中的文件恢复为空对象`{}`。这在更改巴别塔配置后是必要的。接下来，重启开发服务器(如果它正在运行的话)。

## 🤙🏽包装它

要创建和填充您的语言环境文件，请运行:

```
npm run i18n
```

您应该看到您的`content/locales/en.json`文件被填充了一个新的密钥。您还应该看到创建了一个包含键和值的`content/compiled-locales/en.json`文件。

## 💻添加翻译

添加一个新文件`content/locales/de.json`。从`content/locales/en.json`复制填充的字典并粘贴到新文件中，翻译字符串:

```
**// en.json**
{ "93G7sJ": "Good Morning"} **// de.json**{ "93G7sJ": "Guten Morgen"}
```

再次运行`npm run i18n`来创建一个编译好的德语翻译。

您现在应该看到一个名为`content/compiled-localed/de.json`的新文件被创建了。该文件应该包含一个属性。

## 🍎测试最终应用

成功设置后，您将能够导航到应用程序的本地化页面:

`localhost:3000/en`参见英文翻译

`localhost:3000/de`见德文译本

# 👏🏼无耻的插头

如果你喜欢这篇文章，一定要在 Medium 或 Twitter 上关注我。敬请关注更多内容。

*如果你有兴趣增加你在所有社交渠道的有机接触，请查看*[***https://gosquad . cc***](https://gosquad.cc)