# SEO 友好的温泉

> 原文：<https://itnext.io/seo-friendly-spas-d3c461a56217?source=collection_archive---------0----------------------->

![](img/fa89c0e732c36be52e22df38e9d1a438.png)

SEO 是为你的网站获得有机流量的关键。

搜索引擎优化的增强对于增加营销网站、博客和其他应用的流量非常有用。但是越来越多的这些网站/应用程序是使用单页应用程序(spa)构建的，或者是完全在客户端浏览器中呈现的网站。

搜索引擎优化(SEO)是一门学科，专注于使网站在 Google、Bing 和 Yahoo 等搜索引擎的有机(非付费)搜索结果中出现得更高。它还发展到包括 Twitter、脸书和 Instagram 等社交媒体。然而，SEO 包含的不仅仅是制作搜索友好的 URL 和关键字，并将其添加到这些平台上。它包括一切，从你如何链接到你的网站，到网页上的文字本身，以及你在头部包含的元标签。事实上，整本书都是关于这个主题的，公司的存在只是为了帮助你进行 SEO。记住这一点，大多数 SEO 特定的主题将不在这里讨论。相反，我将主要关注使用 JavaScript 在 SPA 中可以做的事情，以使您的网站更加 SEO 友好。Google & Bing 的网站管理员指南提供了一些关于 SEO 的简单建议。因此，在继续之前，请查看一下这些资源。

谷歌网站管理员指南:

[https://support.google.com/webmasters/answer/35769?hl=en](https://support.google.com/webmasters/answer/35769?hl=en)

Bing 的网站管理员指南:

[https://www . bing . com/web master/help/web master-guidelines-30 FBA 23 a](https://www.bing.com/webmaster/help/webmaster-guidelines-30fba23a)

## 单页应用和搜索引擎优化

spa 和传统网站不一样。在互联网的早期，网站只是一个由大学、政府机构或企业的计算机提供的 HTML 文件。它们很容易搜索，因为所有内容都直接嵌入在文件中，就像在文字处理器中阅读文档一样。它有一个清晰的层次结构，也很像一个文档。网站使用的文档对象模型(DOM)甚至反映了这一点。例如，`<h1>`到`<h6>`标签是标题，用于文档的开始部分。一个`<p>`标签是一个段落，通常在一个`<section>`内有多个包含内容的段落。一个`<table>`用于表格数据。如果你需要更多关于 HTML 本身的信息，请阅读 MDN 的[HTML 简介](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML)。关于 DOM 如何使用 JavaScript 与它交互的更多信息，请阅读 MDN 的对 DOM 的介绍。

服务器端呈现是后来添加的，作为在提供给客户端之前将内容加载到页面中的一种方式。这使得人们可以写博客和新闻网站，并最终开始创建应用程序来满足各种业务需求。随着应用程序变得越来越复杂，人们不得不开始使用 HTML 来表达不太像文档的东西。这就产生了对 SEO 的需求，以便让搜索引擎在这些网站和应用程序中找到内容。

然而，随着 JavaScript 框架变得越来越流行，网站发生了更大的变化。无限滚动、通过 Ajax 和其他 XHR 库加载 HTML 后异步加载数据，以及客户端呈现 DOM 元素等功能使得搜索引擎很难抓取页面。SPAs 甚至可以更新整个页面，而无需向服务器发出另一个请求，甚至无需更改 URL。

现在水疗已经很普遍，搜索引擎知道如何消费它们。谷歌创建了 Angular.js，微软采用了它，脸书也用 React.js 做了同样的事情。他们还重新设计了他们的搜索功能，以便能够完全加载和抓取水疗中心。但这并不意味着你可以简单地建立一个并让它出现在搜索引擎中。无需深入了解 SEO 如何工作的基础知识，这里有一些事情你可以做，以确保你的 SPA 将与搜索引擎一起工作。

## 漂亮的网址

许多客户端框架使用散列(包含`#`的 URL，如`http(s)://www.example.com/#about`)和散列值(`http(s)://www.example.com/#!key=value`)来处理它们的路由。在功能上，SPA 的行为方式与普通网站相同。使用 JavaScript 路由器或其他方式，URL 会发生变化，内容也会更新。即使你可能认为抓取你的网站的机器人可能不在乎你是否使用散列，使用你的网站的用户将会看到一个看起来更干净更友好的 URL。

此外，一些爬行器，如 Google 的，将带有 hashbangs `#!`的 URL 解释为一个指示器，指示存在一个替代的传统 URL，在加载时提供相同的页面状态。出于这个原因，建议通过使用像 PushState 这样的技术，只使用哈希`http(s)://www.example.com/#/pagename`或 HTML5 风格的无哈希路由，如`http(s)://www.example.com/pagename`。PushState 是一种通过 JavaScript 改变浏览器中显示的 URL 而无需重新加载页面的方式。它这样改变历史对象:`window.history.pushState(data, "Page Title", "/new-url")`。你最终得到一个漂亮的网址。

另一个常用的 URL 技巧是查询字符串。将查询字符串用于类别、搜索词和其他参数是完全可以接受的。它可以与上面提到的任何 URL 操作技术结合使用。搜索引擎可以区分出`http(s)://www.example.com/?category=people`和`http(s)://www.example.com/?category=cars`，尤其是如果每个 URL 都被定义并鼓励在你的站点地图中被索引。

## 网站地图

搜索引擎使用网站地图来更好地理解你的网站内容。你可能并不总是需要一个。例如，如果你的网站很小，并且链接良好。然而，如果你的网站很大或者链接很差，很有可能网络爬虫会忽略新的或者最近更新的页面。

但是，如果你的网站有一个很大的内容页面的档案，这些内容页面是孤立的或者相互之间没有很好的链接，那么网站地图可能是一个很好的策略。例如，如果您的站点页面不通过链接自然地相互引用，您可以在站点地图中列出它们，以确保网络爬虫不会忽略这些页面。此外，如果您的网站是新的，很少有外部链接，网络爬虫可能找不到它。或者如果您使用元数据，如视频或图像内容。爬网程序通常会从一个页面跟踪到另一个页面，如果没有其他网站链接到您的页面或内容，可能无法发现它们。在这种情况下，网站地图是有益的。

要让一个机器人使用你的站点地图，你必须创建并托管一个名为`sitemap.xml`的文件。这里不讨论 sitemap.xml 的内容和结构。在[了解更多关于站点地图-搜索控制台帮助](https://www.xml-sitemaps.com/)的信息。

如果您的站点是静态托管的，您可以简单地将 sitemap.xml 放在根级别。如果您使用的是 Express 之类的服务器端框架，您可以通过这种方式托管文件:

```
app.get('/sitemap.xml', (req, res, next) => {
    res.sendFile('public/sitemap.xml')
})
```

根据您的框架，还有其他方法来托管文件。例如，使用 Express，你可以使用`express.static`提供`public`文件夹中的所有内容。更多信息参见[在 Express](https://expressjs.com/en/starter/static-files.html) 中提供静态文件。

```
app.use(express.static(__dirname + '/public'))
```

为了告诉机器人你的站点地图文件，你需要在一个名为`robots.txt`的文件中引用它:

```
// robots.txt
Sitemap: /sitemap.xml
```

确保您也在根位置或者在像 Express 这样的框架中提供这个文件:

```
app.get('/robots.txt, (req, res, next) => {
    res.sendFile('public/robots.txt')
})
```

或者，如果你正在使用`express.static`，只要把它放在你的`/public`文件夹中。有很多方法可以做到这一点，这取决于服务器(Apache，Nginx，Express 等。).

僵尸工具会一直查看这个文件的根 URL。确保在`http(s)://www.example.com/robots.txt`供应。

此外，还有一些工具可以为您动态呈现您的站点地图。Node 的一个流行选择是 NPM 包[站点地图](https://www.npmjs.com/package/sitemap)。

# 贮藏

另一个提高搜索引擎优化的简单方法是加快页面加载速度。现代的网络爬虫关心加载时间，并且将具有相似内容的较快的页面优先于较慢的页面。由于 JavaScript 框架在页面和所有资源下载完成后才呈现(有时甚至等待 Ajax/XHR 调用完成)，因此它们通常比预先呈现的服务器端页面要慢。我推荐使用谷歌的 PageSpeed Insights 来测试你的页面。

加快页面加载速度的一种方法是缓存内容，尤其是当内容不经常变化时。这在 Express 中很容易处理。

如果你把你的文件存放在一个静态文件夹中，你可以简单地使用 Express.static:

```
const cacheTime = 86400000*10 // 10 daysapp.use(express.static(__dirname + '/public', { maxAge: cacheTime }))
```

或者，如果您正在使用`sendFile`，您可以在那里设置缓存:

```
app.get('/robots.txt, { maxAge: cacheTime }, (req, res) => {
    res.sendFile('public/robots.txt')
})
```

如果您将文件托管在 CDN 上，您将需要在那里配置缓存规则。纯静态页面推荐这样，可以试试亚马逊 CloudFront，CloudFlare 或者其他 cdn。但是，如果您正在使用 Node，这可能是一个非常简单且可伸缩的解决方案。我建议在[Express 4 . x——API 参考](https://expressjs.com/en/api.html)查看更多规则。

最后，使用 gzip 或 deflate 来压缩你的内容可以大大加快速度。在 express 中，这很容易通过使用`compression`来实现。

```
$ npm install compressionconst compression = require('compression')
const express = require('express')const app = express()// compress all responses
app.use(compression())// add all routes
```

参见[文档](https://github.com/expressjs/compression)进行配置。

## 预渲染

有一些工具可以在将页面发送给客户端之前对其进行预渲染。这也加快了加载速度，提高了你的搜索引擎优化和整体用户体验。使用这种方法，您可以在部署应用程序之前运行应用程序，捕获页面输出并用捕获的输出替换您的 HTML 文件。通常，这是通过诸如 PhantomJS 之类的无头浏览器(没有图形用户界面的 web 浏览器)来实现的。

预渲染是一个很好的选择，因为没有额外的服务器负载，因此比服务器端渲染更快更便宜。这也是一个更简单的产品设置，并允许您编写更简单的应用程序代码(不需要同构代码)。因此，它不容易出错，可以更容易地缓存更长时间。此外，它不需要 Node.js 生产服务器。

预渲染并不总是一个好的选择。例如，对于显示不断变化的数据(这些数据必须在加载时动态加载)的页面，或者对于具有特定于用户的内容的页面，它就不太适用。通常，满足这些要求的页面对于预渲染来说不太重要。只有您希望快速提供的常用页面才应该预先呈现。否则，服务器端呈现(SSR)可能是更好的选择。(有关 SSR 的更多信息，请参见下一节。)

下面是一个如何使用 Webpack 的`prerender-spa-plugin`预渲染应用程序的例子。一个常用的 JS bundler，它有许多其他使用插件的功能。根据您的需要，您可以找到许多与其他语言、bundlers 或框架兼容的其他工具。

```
$ npm install --save-dev prerender-spa-plugin
```

预渲染插件创建一个 PhantomJS 实例并运行应用程序。然后，它拍摄 DOM 的快照，并将快照输出到 Webpack 构建文件夹中的 HTML 文件。它为每条路线重复这一过程，所以如果你有许多页面，构建应用程序可能需要相当长的时间。

下面是一个使用预渲染插件的简单 Webpack 配置的简单示例。

```
// webpack.conf.js 
const path = require('path')
const PrerenderSpaPlugin = require('prerender-spa-plugin')

module.exports = {
  // ... 
  plugins: [
    new PrerenderSpaPlugin(
      // Absolute path to compiled SPA 
      path.join(__dirname, './public'),
      // List of routes to prerender 
      [ '/', '/about', '/contact' ]
    )
  ]
}
```

在这个例子中，创建了一个 pre-render 插件的新实例，您让它知道输出到哪个文件夹，以及在生成 HTML 文件时您希望 PhantomJS 浏览器访问的路径列表。这是设置插件时仅有的两个必需选项。你也可以选择传递一个更高级的配置给插件。

```
// webpack.conf.js 
const path = require('path')
const PrerenderSpaPlugin = require('prerender-spa-plugin')

module.exports = {

  // ... 

  plugins: [
    new PrerenderSpaPlugin(
      // Absolute path to compiled SPA 
      path.join(__dirname, './public'),
      // List of routes to prerender 
      [ '/', '/about', '/contact' ],
      // (OPTIONAL)
      {
          // options go here
      }
    )
  ]
}
```

除非您依赖异步呈现的内容，比如在 Ajax/XHR 请求之后，否则这些选项都不是必需的。在捕获页面内容之前，已经执行了所有同步脚本。以下每个选项都是可选`options`对象的一部分。

如果您想等到页面中触发特定的 JavaScript 事件:

```
captureAfterDocumentEvent: 'custom-post-render-event'
```

然后在您的 JavaScript 文件中分派一个事件:

```
document.dispatchEvent(new Event('custom-post-render-event'))
```

您可以等待，直到用 document.querySelector 检测到特定的 HTML 元素。

```
captureAfterElementExists: '#content'
```

或者，您也可以等到脚本执行完毕几毫秒后再执行。需要注意的是，当依赖于网络通信或其他具有高度可变定时的操作时，这可能会产生不可靠的结果。

```
lang: javascript
captureAfterTime: 5000
```

如果你喜欢，你甚至可以组合策略。例如，如果您有时只想等待一个事件触发，您可以通过组合使用`captureAfterTime`和`captureAfterDocumentEvent`来创建一个超时。当组合策略时，页面内容将在第一个触发的策略之后被捕获。

您可以简单地忽略 JavaScript 错误，而不是大声疾呼失败(默认)。

```
lang: javascript
ignoreJSErrors: true
```

要更改索引文件的路径，而不是静态根目录中的默认 index.html:

```
lang: javascript
indexPath: path.resolve('/public/path/to/index.html')
```

因为 PhantomJS 偶尔会遇到间歇性问题，所以默认情况下，插件会自动重试 10 次页面捕获。如果你愿意，你可以提高或降低这个限制。

```
lang: javascript
maxAttempts: 10
```

现在，您必须防止幻像从传递给它的 URL 导航出去，并防止加载嵌入的 iframes(例如 Disqus 和 Soundcloud embeds)，这对 SEO 来说并不理想，可能会引入 JavaScript 错误。

```
lang: javascript
navigationLocked: true
```

下面的选项公开了幻像的配置选项，在极少数情况下，您需要为特定系统或应用程序进行特殊设置。

```
lang: javascript
// http://phantomjs.org/api/command-line.html#command-line-options 
phantomOptions: '--disk-cache=true',// http://phantomjs.org/api/webpage/property/settings.html 
phantomPageSettings: {
  loadImages: true
},// http://phantomjs.org/api/webpage/property/viewport-size.html 
phantomPageViewportSize: {
  width: 1280,
  height: 800
}
```

现在，您可以在预渲染后手动转换每个页面的 HTML，例如，在无法通过路由解决方案处理的边缘情况下设置页面标题和元数据。

该函数的上下文参数包含两个属性:

*   `html` -预渲染后得到的 HTML
*   `route` -当前正在处理的路线(例如“/”、“关于”或“/联系”)

无论返回什么，都将打印到预渲染文件中。

```
lang: javascript
postProcessHtml: context => {
    const titles = {
        '/': 'Home',
        '/about': 'Our Story',
         '/contact': 'Contact Us'
     }
     return context.html.replace(
         /<title>[^<]*<\/title>/i,
         '<title>' + titles[context.route] + '</title>'
     )
}
```

使用这个插件将允许你使用你所使用的 JS 库创建简单的 HTML 页面。例如，React、Vue、Angular、Riot 或任何其他，它们可以使用 Express 或您喜欢的任何服务器托管在您的服务器上的目录中。

请注意，预渲染插件仅适用于使用 HTML5 历史 API (PushState)的路由策略。没有散列(bang)URL 将使用这种方法。所以确保你设置了你的 JS 路由器来使用像`http(s)://www.example.com/contact`这样的 URL。此外，当 DOM 准备好的时候，您应该总是加载您的 SPA，方法是将您的标签放在 DOM 事件`DOMContentLoaded`中。

但是请记住，如果您计划运行动态变化的页面，这些页面是数据密集型的，并且根据数据不断变化，那么这不是一个好的选择。例如，如果您想要基于用户信息动态生成数据。相反，服务器端呈现对于这样的站点来说是一个更可行的选择。

## 服务器端渲染

上述方法最适合用于只需渲染一次而非动态的应用程序。例如，营销网站、产品主页或销售应用程序。

有关使用 JavaScript 框架进行服务器端渲染的更多信息，请查看 [ReactDomServer](https://reactjs.org/docs/react-dom-server.html) 和 [Vue 服务器端渲染](https://vuejs.org/v2/guide/ssr.html)。此外，现在大多数其他 JS 框架都有 SSR 解决方案。比如我最喜欢的两个 SSR 框架是 [NextJS](https://nextjs.org/) 和 [GatsbyJS](https://www.gatsbyjs.org/) 。此外，为 Express 使用 SSR [模板引擎总是有效的，这也是我们为 Aurelius](https://expressjs.com/en/guide/using-template-engines.html) 主页所做的。我们目前使用的是快速的 [MarkoJS，它易于使用并且速度非常快。](https://markojs.com/docs/express/)

请随时发表评论，让我知道你对这个水疗 SEO 指南的看法。如果有任何你想看到的补充或问题，请告诉我，我会尽力帮助你。如果我得到足够的反馈，我可能会专门跟进 SSR。