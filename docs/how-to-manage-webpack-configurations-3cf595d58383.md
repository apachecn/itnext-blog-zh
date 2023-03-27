# 如何管理 Webpack 配置

> 原文：<https://itnext.io/how-to-manage-webpack-configurations-3cf595d58383?source=collection_archive---------2----------------------->

![](img/78e39883de38d5b2a61e7cef68e98e89.png)

配置 Webpack 非常容易，但是如果有许多环境和需求，它可能会变得更加困难。最近我在不同的项目中加入了 Webpack，我想和你分享一个简单的配置方法。我确信有许多不同的方式来配置 Webpack，也许更好，但我只打算比较其中的两种:

*   一个文件来统治他们所有人
*   扩展配置

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fhow-to-manage-webpack-configurations-3cf595d58383)

# 一个文件来统治他们所有人

当您开始配置 Webpack 时，您将会这样做。**一个配置文件统治一切**。这包括只有一个名为 *webpack.config.js* 的文件，所有内容都写在那里。

大概是这样的:

```
// Example file: webpack.config.js
const path = require(“path”);

module.exports = process.env.production ?
  getProdConfig() :
  getDevConfig();

/**
 * Creates webpack configuration for production.
 */
function getProdConfig() {
  return {
    entry: './foo.js',
    // Add your prod config here: Minify, remove logs, etc.
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: 'foo.bundle.js'
    }
  };
}

/**
 * Creates webpack configuration for development.
 */
function getDevConfig() {
  return {
    entry: './foo.js',
    // Add your dev config here (devserver, sourcemaps, etc.)
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: 'foo.bundle.js'
    }
  };
}
```

这对于小项目很有用，但是你的文件会变得越来越大，有一天你会发现这个结构不清晰也不可维护。在这一天到来之前，我建议你把你的配置分成不同的文件，就像我下面解释的那样。

# 扩展配置文件

这种结构非常简单且适应性强。**从一个名为 *webpack.base.config.js* 的文件开始，然后根据需要扩展它。**

这里有一个基本配置的示例:

```
const path = require(‘path’)

// This is the base configuration used on every environment
// It contains general information like entry point, output,
// common transformations, etc.
module.exports = {
  entry: './foo.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'foo.bundle.js'
  }
}
```

让我们假设你需要建立一个生产版本(缩小，没有源地图，没有开发服务器等。).没问题。有一个名为 [webpack-merge](https://www.npmjs.com/package/webpack-merge) 的包可以简化这种情况。这是怎么回事？很简单，只需创建另一个名为 *webpack.prod.config.js 的文件，然后*使用 webpack-merge 扩展您的基本配置，并覆盖您想要的任何内容。让我们看一个例子:

```
const merge = require(“webpack-merge”);
const baseWebpackCfg = require(“./webpack.base.config”);

// We are using webpack-merge to extend the base configuration
// with specific information needed to build a version for
// production. We will add here minification, log removal, etc.
module.exports = merge(baseWebpackCfg, {
  plugins: [
    new MinifyPlugin()
  ]
})
```

只有一点:配置插件时要小心，因为它的属性不会被合并，而是被覆盖。

webpack-merge 允许不同的选项来合并您的配置文件。我用了最简单的一个，但看看它的网站，找到更多的选项来满足你的需求。

# 结论

使用 [webpack-merge](https://www.npmjs.com/package/webpack-merge) 将会使你的 webpack 配置文件有条理、可读和可维护，但是如果你的项目刚刚开始，只使用一个文件，并在将来根据你的需要修改它。

# 信用

*   Iker Urteaga 在 [Unsplash](https://unsplash.com/search/photos/lego?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的特写照片