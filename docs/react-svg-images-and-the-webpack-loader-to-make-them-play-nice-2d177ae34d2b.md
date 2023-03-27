# React、SVG 图像和 Webpack 加载器，使它们播放得更好

> 原文：<https://itnext.io/react-svg-images-and-the-webpack-loader-to-make-them-play-nice-2d177ae34d2b?source=collection_archive---------2----------------------->

![](img/c33acfdc90271d808043256fcb2f9c1a.png)

戈兰·艾沃斯在 [Unsplash](https://unsplash.com/search/photos/developer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

这不是一个新问题:让 SVG 用 React 正确渲染的困境，但这是一个我直到最近才不得不处理和克服的问题，我想分享我的经验，希望它能为其他人节省几个小时搜索 Github、StackOverflow 和 NPM 的时间。

在我目前的公司，我为两个独立的开发团队工作，他们都使用 JavaScript 微服务作为他们的 ui——一个使用 AngularJS，另一个使用 JavaScript MVC。所有的后端服务都是 Java Spring Boot 应用程序。

当不久前有消息称谷歌将停止支持 AngularJS 1.5(我们正在使用的版本)时，很明显我们可以:

1.  将系统升级到 [AngularJS 1.7](https://blog.angular.io/stable-angularjs-and-long-term-support-7e077635ee9c) (将继续提供 3 年以上的长期支持)，
2.  将我们的应用程序升级到 Angular 2，
3.  或者迁移到 ReactJS。

我和我的一个同事负责找出哪一个是最佳方向。这种探索是一个值得在自己的博客上发表的过程，但可以说，我最支持冒险进入 React，在我们研究了我们的选项并进行修补后，我们决定看看我们是否可以使用 React 工作进行一次小型的概念验证。

在开始我们的 React POC 大约 4 小时后，我们有了一个可以工作的应用程序，尽管不是很漂亮。我们添加了 Jest 和 Enzyme 单元测试、Cypress 端到端测试等等。但是有一件事困扰了我们将近两个星期:让 SVG 图形在 DOM 中正确呈现。

## 为什么选择 SVGs？有什么好处？为什么不直接做成 PNG 呢？

问得好。使用 SVG 的好处有很多，如果你不知道，SVG 代表[可缩放矢量图形](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics)。虽然教科书上的定义听起来很深奥:

> “S VG 是基于 [XML](https://en.wikipedia.org/wiki/XML) 的[矢量图像格式](https://en.wikipedia.org/wiki/Vector_image_format)，用于支持交互性和动画的[二维](https://en.wikipedia.org/wiki/Two-dimensional)图形”——维基百科，可缩放矢量图形

它们基本上取代了 JPEG、PNG、TIFF 或 GIF 等其他格式的传统图像。

以下是 SVG 优于其他图像格式的几个原因:

*   **无论屏幕大小、缩放级别或图像大小，都有清晰的分辨率** —因为 SVG 是基于矢量路径、形状和填充构建的，所以它们可以缩放到任何大小，而不会损失清晰度。另一方面，png 是基于像素的，不会一直看起来清晰明了(尤其是在视网膜显示设备上)。
*   **加载速度更快**——如果你在网站上使用高分辨率图像来迎合那些视网膜显示器，PNG 文件可能相当大，而且越大的文件渲染越慢(咄)。SVG 只是代码，这意味着文件更小(除非你加载的东西非常详细，有一百万个向量路径)。
*   **动画是可能的**——与静态的 PNG 文件不同——所见即所得，SVG 可以用 CSS 制作动画和样式。动画，不同的颜色，状态的变化(像悬停或点击)，所有这些事情都可以在 SVG 上完成。
*   **可访问性和搜索引擎优化** —谷歌将 SVGs 编入索引，这意味着你的网站可以获得更好的搜索引擎优化分数，对于访问你网站的有视觉障碍的人来说也更容易访问。

## 好，那么 SVG 就是了，现在如何在 React 中使用它们

在我的项目中，我正在考虑是否可以在 React 中重新创建一个现有 AngularJS 应用程序的简单屏幕。这个屏幕包含四个大的、可点击的卡片，将用户带到其他不同的屏幕，每个卡片都有一个漂亮的 SVG 文件，与单词相关联。

所以我从原始项目中复制了 SVG 文件，将它们移动到 React 中，将文件导入到将呈现它们的组件中，添加

`<Image source={require(‘../img/key.svg’)} />`

在 JSX，添加了 CSS 样式，等待 Webpack 进行神奇的解析，但…什么也没有。嗯，不是什么都没有，控制台中出现关于无法识别的文件类型的错误，但肯定没有按键图标显示在 UI 中。

因此，我开始寻找在 React 中正确呈现 SVG 所需的库。这是一个长达数天的过程。如果你不知道大约有 7，845，923 个 NPM 模块都声称可以让 React 很好地与 SVG 配合使用。其中一些可能有用，我尝试的前 10 个左右没有用。

## 然后… SVG 内联加载程序

然后，我找到了。一个名为 [svg-inline-loader](https://www.npmjs.com/package/svg-inline-loader) 的 NPM 模块，它有不错的文档，它有 3200 次周下载，它在 Github 上有 325 颗星，*它由构建 Webpack* 的核心团队开发和维护，它看起来很有前途。

所以我试了一下，把它保存到我的 dev dependencies 的`package.json`文件中。

```
npm install svg-inline-loader react-inlinesvg --save-dev
```

然后，将配置对象添加到`webpack.config.js`文件中:

```
{
    test**:** /\.svg**$**/,
    loader**:** 'svg-inline-loader'
}
```

我已经准备好了。对 Webpack 的一个简单更新，对 JSX 语法的一个小调整和…

```
import React, { Component } from 'react';
import KeyImage from '../img/key.svg'; 
import SVG from 'react-inlinesvg';class Card extends Component(){
return(
    <div>
      < /* other UI stuff here */ > 
      <SVG src={KeyImage} />
    </div>  
  )
}
```

BOOM — SVG 显示！

我还想补充一下，这个 NPM 模块有一些非常漂亮的额外功能:它可以从 SVG 中删除额外的标签信息(当它们是用 Sketch 或 Adobe 生成的时候，这种情况经常发生)，它可以添加类或 id 来防止 SVG 冲突。很好。

就这样。没有 CSS 变化，没有超级怪异的语法，什么都没有。只是 svg-inline-loader，Webpack 的一个额外的模块加载器，问题就解决了。

感谢阅读，我希望它有所帮助，掌声非常感谢！

**如果你喜欢读这篇文章，你可能也会喜欢我的其他博客:**

*   [为什么云配置服务器对良好的 CI/CD 渠道至关重要以及如何设置(第 1 部分)](https://medium.com/@paigen11/why-a-cloud-config-server-is-crucial-to-a-good-ci-cd-pipeline-and-how-to-set-it-up-pt-1-fa628a125776)
*   [JavaScript 数组方法让你成为更好的开发者](https://medium.com/@paigen11/javascript-array-methods-to-make-you-a-better-developer-4ce42052d54c)
*   [graph QL 到底是什么？](https://medium.com/@paigen11/what-is-graphql-really-76c48e720202)

**参考资料和更多资源:**

*   维基百科，可扩展矢量图形:[https://en.wikipedia.org/wiki/Scalable_Vector_Graphics](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics)
*   SVG 内嵌加载器:[https://www.npmjs.com/package/svg-inline-loader](https://www.npmjs.com/package/svg-inline-loader)
*   你应该使用 svgs 而不是 pngs 的 5 个理由:[https://watb . co . uk/5-你应该使用 SVGs 而不是 PNGs 的理由/](https://watb.co.uk/5-reasons-you-should-be-using-svgs-over-pngs/)