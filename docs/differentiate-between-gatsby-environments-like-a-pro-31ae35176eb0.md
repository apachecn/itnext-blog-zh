# 像专家一样区分盖茨比环境

> 原文：<https://itnext.io/differentiate-between-gatsby-environments-like-a-pro-31ae35176eb0?source=collection_archive---------2----------------------->

![](img/c5ef00721e58dac5975e8bd27cb6a6c1.png)

你一直在试图找出为什么你的代码的改变是不可见的，你打开了成千上万的标签页，然后突然想到…你打开了生产 URL。

我曾经经历过，浪费大量的时间和生产力来解决这个问题是非常容易的，结果发现你甚至没有注意到你的本地开发环境。所以，我有一个很好的建议，可以帮助确保这种事情不会再发生。

Gatsby 允许你为一个 favicon 指定一个图像路径，也可以为一个站点清单指定背景颜色和主题颜色。它可以通过`gatsby-plugin-manifest`插件获得。

您可以按常规方式将其添加到您的`gatsby-config`中，它看起来可能类似于:

```
module.exports = {
  plugins: [
    {
      resolve: `gatsby-plugin-manifest`,
      options: {
        name: `GatsbyJS`,
        short_name: `GatsbyJS`,
        start_url: `/`,
        background_color: `#f7f0eb`,
        theme_color: `#a2466c`,
        // Enables "Add to Homescreen" prompt and disables browser UI (including back button)
        // see https://developers.google.com/web/fundamentals/web-app-manifest/#display
        display: `standalone`,
        icon: `src/images/icon.png`, // This path is relative to the root of the site.
      },
    },
  ],
}
```

这是样板文件，如果你使用任何关于[www.gatsbyjs.org](https://www.gatsbyjs.org)的入门或教程，这将成为默认文件——对于停止那些讨厌的差异化问题不是很有用。

我们将编写两个小函数，它们将接受`NODE_ENV`的参数，并决定应该使用什么颜色和图像。然后我们将使用它们来代替上面的十六进制颜色值和图像路径。

```
const faviconSelector = (env) => {
    return env === 'production' 
      ? 'src/images/icon.png' 
      : env === 'staging' 
        ? 'src/images/icon-staging.png' 
        : 'src/images/icon-development.png'
}const envColourSelector = (env) => {
    return env === 'production' 
      ? '#7b41c1' // Purple
      : env === 'staging' 
        ? '#405ac1' // Blue
        : '#c13f5e' // Red}
```

我倾向于确保我的颜色突出，不要太相似，这也值得让你的本地开发环境更激进(反转)或红色阴影，以真正提醒你，你正在处理你的应用程序/网站的开发状态。

我们现在将这些函数放在我们的配置中:

```
**const { NODE_ENV } = process.env**module.exports = {
  plugins: [
    {
      resolve: `gatsby-plugin-manifest`,
      options: {
        name: `GatsbyJS`,
        short_name: `GatsbyJS`,
        start_url: `/`,
        background_color: **envColourSelector(NODE_ENV)**,
        theme_color: **envColourSelector(NODE_ENV)**,
        // Enables "Add to Homescreen" prompt and disables browser UI (including back button)
        // see https://developers.google.com/web/fundamentals/web-app-manifest/#display
        display: `standalone`,
        icon: **faviconSelector(NODE_ENV)**, // This path is relative to the root of the site.
      },
    },
  ],
}
```

您可以通过更改`NODE_ENV`的值并刷新您的浏览器选项卡来测试您的新配置。

`NODE_ENV=staging gatsby develop`或`NODE_ENV=production gatsby build && gatsby serve`或`gatsby develop`

有什么发展小技巧吗？把它们写在评论里吧，我总是在寻找新的方法来提高我的工作效率。