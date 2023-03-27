# 网络包及更少

> 原文：<https://itnext.io/webpack-and-less-a75e04aaf528?source=collection_archive---------1----------------------->

## 如何让他们友好相处

![](img/d9d85dbab2f04f3be4675e69a69bdcf3.png)

这是 Webpack，当他们不配合时就更少了。但是他们可以！艾伦·泰勒在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

所以我的一个朋友，Frode，问了我一个关于 Webpack 和 LESS 如何工作的问题，以及你是如何设置的。与其写一个一次性的答案，我想我应该在 Medium 上写一篇快速的文章——这样其他人也可以从我的答案中受益。

## 关于依赖性的一个注记

如果您想跟进，您需要创建一个新文件夹，运行`npm init -y`并安装以下依赖项:

```
npm install --save-dev webpack webpack-cli less less-loader css-loader style-loader mini-css-extract-plugin
```

在您阅读本文的过程中，将会解释这些依赖项。

# 如何开始

首先，让我们建立一个基本的`webpack.config.js`文件:

在这里，我们指定要加载文件`src/index.js`，并从那里创建一个依赖图。无论我们最终得到什么，我们都希望在`dist/bundle.js`中将其打包到一个大的 ol’文件中。如果你以前使用过 Webpack，你可能会看到这样的东西。

作为参考，让我们假设我们的`src/index.js`看起来像这样:

而我们的`src/my-styles.less`长这样:

现在我们已经准备好开始工作了！

# 使用 webpack 处理更少的文件

每当您导入一个文件时，就像我们在上面的`index.js`中所做的，webpack 通过一个“规则”数组列表来运行它，并检查是否设置了任何规则来处理这种类型的文件。在我们的例子中，我们还没有指定规则——所以默认情况下只支持 JavaScript 文件。

为了增加对少文件的支持，我们需要建立一个规则来处理这些文件。我们想告诉 webpack，每当需要导入一个 LESS 文件时，它应该通过 LESS 编译器运行它，解析任何相对 URL，然后以某种方式导出它。

这是它的样子:

由于种种原因，这个规则数组位于一个名为`module`的对象中。总之，每个规则都是一个具有`test`属性和`use`数组的对象，它们指定了加载器。

`test`属性是一个正则表达式，它根据文件名运行，以决定是否应该在当前正在处理的文件上运行该规则。在这种情况下，它检查文件是否以`.less`结尾。

`use`数组是一个所谓的加载器列表——或者转换——将在这个文件上运行。它们从右向左读取，并以菊花链方式处理文件。在我们的例子中，情况是这样的:

首先，运行`less-loader`。这将文件的内容传递给 LESS 编译器，后者返回编译后的 CSS。

第二，`css-loader`跑了。在这里，任何图像、字体等的相对 URL 都是为我们解析的。解释它的深度超出了本文的范围，但是如果您感兴趣的话，请看看它的[文档](https://github.com/webpack-contrib/css-loader)。

最后，运行`style-loader`。这个加载器接受传递给它的任何文本(我们编译的带有解析的 URL 的 CSS)，并将其推入一个`<style />`标签。这对于开发来说非常好，并且让我们在样式改变时更新我们的样式，而不用刷新页面(所谓的热重载)。

# 提取 CSS 文件

样式加载器对于开发来说很棒，但是对于生产来说并不理想。我们希望提取一个 CSS 文件，我们可以独立于 JavaScript 对其进行版本控制、缓存和服务，这样它的运行速度会更快！

为了确保在我们将东西投入生产时事情是不同的，我们需要某种标志来指定我们处于“为生产而构建”的模式。社区作为一个整体已经决定设置为`"production"`的`NODE_ENV`环境就是那个标志。

现在，如果我们正在为生产构建我们的应用程序，我们希望用一个将 CSS 提取到文件中的转换来切换我们的`style-loader`转换。为此，我们使用`mini-css-extract-plugin`。它完全按照罐头上说的做——提取你的 css。

要使用它，我们需要将它添加到我们用于`less`-文件的两个加载器中，并且我们需要将它作为一个“插件”添加。

插件就像超级加载器，因为它可以做大量加载器做不到的事情——比如写文件。webpack 文档[很好地解释了它们的概念，所以我不会在这里跳进去。](https://webpack.js.org/concepts/plugins/)

一旦添加了所有内容，我们就完成了最终的 webpack-config:

# 奖金:来源图！

源代码图对于调试编译后的代码非常有用。它们将已编译代码的特定部分映射到未编译代码的相关部分。

Webpack 让你制作源地图变得非常容易——你所需要做的就是添加`devtool`属性。你可以在[文档](https://webpack.js.org/configuration/devtool/)中读到如何设置它。现在打开你的开发工具，直接进入源代码！

# 奖金奖金:阅读来源！

我已经创建了一个样例 repo 来实现上面的代码。请浏览它，并克隆/安装它以查看它的运行。

您可以在此处找到代码:

[](https://github.com/selbekk/webpack-with-less-example) [## sel bekk/web pack-with-less-example

### 这是一个简单的 webpack 安装程序，文件较少

github.com](https://github.com/selbekk/webpack-with-less-example) 

# 最后的想法

一旦你知道怎么做，用 webpack 设置 LESS 并不太难，但是在你第一次这么做的时候就不是很直接了。

我希望这篇指南能对你有所帮助，并且你能愉快地探索 webpack 的奇迹。

感谢阅读这篇文章！我希望你喜欢它——如果你喜欢，请为我鼓掌以示谢意。最多可以给 50 个！

Kthxbai