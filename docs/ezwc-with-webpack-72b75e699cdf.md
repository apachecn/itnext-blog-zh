# 带 Webpack 的 EZWC

> 原文：<https://itnext.io/ezwc-with-webpack-72b75e699cdf?source=collection_archive---------11----------------------->

![](img/6224bd6e885835f460ed72df9ea610a2.png)

自从写了介绍 [EZWC](https://github.com/pynklynn/ezwc) 的[帖子](/ezwc-for-native-web-components-e574a6f39de)后，我有机会研究为 Webpack 写一个 [EZWC 加载器](https://github.com/pynklynn/ezwc-loader)。众所周知，Webpack 是构建前端项目最流行的工具之一。我在自己的网站上使用它，它也被像 Vue 这样的框架使用。

编写加载程序实际上比我预期的要简单得多。作为我写的第一篇文章，完全有可能会更有效率，所以请随意提出问题或提出建议。

# 使用 EZWC Webpack 加载器

使用加载器非常简单。从安装开始:

然后更新 Webpack 配置:

*(* ***注:*** *可能有一种方法可以用加载器自动处理文件类型，而不需要更新 Webpack 配置。如果有并且你知道怎么做，请让我知道！)*

一旦这两个步骤完成，只需导入。JavaScript 文件中的 ezwc 文件就像任何使用加载器的文件一样:

如果你想尝试一下 EZWC，希望这个加载器能让它变得更简单！