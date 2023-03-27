# 改进 Vue.js 应用程序的最简单方法—第 1 部分

> 原文：<https://itnext.io/the-easiest-way-to-improve-your-vue-js-application-part-1-5b48d239f4c9?source=collection_archive---------7----------------------->

![](img/c0287d212219cbad53604fa805c2c38b.png)

[由 ADCI 解决](https://www.adcisolutions.com/?utm_source=medium&utm_medium=referral&utm_campaign=medium-prerendering-mainwebsite&utm_content=medium-prerendering)

**Vue.js** 是一个开发不同类型网络应用的伟大框架，然而，它有一个关于 **SEO** 的问题:一个应用的异步内容不能被搜索引擎机器人最优地抓取，所以这可能是你的业务的一个巨大问题。在本文中，我们将告诉您解决这个问题的最简单的方法之一，同时提高您的应用程序的 UX。是的，我们将讨论预渲染。

预呈现是预加载页面上的所有元素并生成单页应用程序的静态 HTML 的过程。

# 预渲染的优势

使用**预渲染**有以下好处:

1.  SEO。是的，最后，我们可以解决具有异步数据的站点的 SEO 问题:所有这样的数据将在爬虫将索引你的站点之前被预加载。
2.  加速页面内容加载。用户甚至可以在整个 JavaScript 加载之前访问您的应用程序，这大大提高了应用程序的 UX。没有人喜欢等很长时间。
3.  廉价优雅的退化。这一点遵循前一点。现在，即使是旧浏览器的用户或使用禁用 JS 或慢速设备的用户也可以访问您的应用程序。当然，他们不能完全与它互动，但他们可以访问重要的信息，而不是看到一个空白的页面。
4.  丰富的预览。所有唯一的 OpenGraph 元数据现在都被正确预取。在脸书和其他社交网站上分享你的网站，以扩大目标受众。

Vue 官方文档建议使用 prerender-spa-plugin 进行预渲染。通常，该插件执行以下步骤:

1.  它创建一个 Phantom JS 实例并运行应用程序。
2.  拍摄 DOM 的快照。
3.  将快照输出到我们的构建文件夹中的 HTML 文件。

好吧，让我们看看它在实践中是如何工作的。

# 练习:在 Vue.js 应用程序中进行预渲染

为了演示，我将使用我们上一篇关于单页面 Vue 应用程序的文章[中创建的应用程序。熟悉这个项目，并按照文章中的指南开始构建一个简单的 SPA，或者，如果您现在想继续，只需克隆 repo 并跳到该指南中的步骤 4:](https://www.adcisolutions.com/knowledge/how-build-single-page-application-spa-vuejs?utm_source=medium&utm_medium=referral&utm_campaign=medium-prerendering-spa-vue-article&utm_content=medium-prerendering)

现在安装插件:

之后，我们需要为正确的工作配置插件。让我们转到 webpack.conf.js 并添加以下设置:

这里我们指定一个目录，我们想从这个目录中获取一个页面进行预渲染。在我们的例子中，这是根目录。此外，我们应该添加一个我们想要预渲染的路线列表。出于演示目的，我将只选取前四篇帖子。

我们还应该考虑到我们是异步获取数据的，所以如果我们继续按照这种方法构建应用程序，我们将得到一个没有内容的 HTML。为了解决这个问题，插件提供了以下三个选项(查看文档以了解更多信息):

就这些了，伙计们！现在，如果我们构建应用程序，我们会得到一个预先呈现的 HTML。你可以在我们的作品集里用 Vue.js 看看真正的项目。

如果想管理元数据，可以使用 Vue 的一个可用插件，例如 vue-meta。例如，让我们为每个页面添加一个独特的标题。我们需要为此安装插件。

将其添加到 main.js 中

并将以下内容插入 src/components/Post.vue

这足以使应用程序的内容被索引，并且对于禁用 JavaScript 的用户也是可用的。现在搜索引擎和社交网络可以读取我们网站的元信息。此外，我们成功地减少了 FMP 的加载时间(第一次有意义的绘制)，浏览器不会等待 JavaScript 的完全加载，而是在 HTML 加载后立即开始渲染页面。因此，用户可以更早地从我们的网站接收信息。

# 当您不需要预渲染时

在构建步骤中使用简单的插件 prerender-spa-plugin，您可以轻松地改进搜索引擎优化，并为您的 Vue 应用程序创建更好的用户体验。不幸的是，很少有不适合使用预渲染的情况。

1.  特定于用户的内容:某些页面的内容，例如用户资料页面的内容，应该只为特定的用户显示。在构建步骤中，我们不知道谁会打开这个页面，所以我们不能预渲染它。无论如何，如果我们做到了，我们将向所有人公开用户的私人信息。
2.  经常变化的内容:如果你在聊天、在线交易应用程序或实时游戏中工作，内容必须总是相关的，使用预渲染可能是无效的，因为它会显示旧的内容，直到客户端 JS 接管最新的数据。作为一个潜在的解决方案，您可以将您的构建设置为在每次数据更改时重新预呈现，或者频繁地确保内容是相关的。但是对于服务器来说，这可能非常消耗资源。对于更新频率甚至超过每分钟一次的数据，您应该避免预呈现。
3.  大量的路由:预渲染成千上万的路由并不是一个好主意，因为这会花费大量的时间来构建您的应用程序。

在这些情况下，最好的方法是使用服务器端呈现，这将在以后的文章中介绍。

# 有用的链接

1.[Vue 官方文档](https://ssr.vuejs.org/en/#ssr-vs-prerendering)
2。[Prerender SPA 插件文档](https://github.com/chrisvfritz/prerender-spa-plugin)
3。Drupal 8 中的 RESTful Web 服务:快速入门指南。[如何创建一个无头的 Drupal 站点](https://www.adcisolutions.com/knowledge/how-create-headless-drupal-site?utm_source=adci-website&utm_medium=knowledge&utm_campaign=knowledge-spa-vue.js&utm_term=none&utm_content=useful-links-headless-drupal-site)
5。[单页应用和多页应用有什么区别？](https://www.adcisolutions.com/knowledge/whats-difference-between-single-page-application-and-multi-page-application)
6。[如何用 Vue.js 构建单页面应用(SPA)](https://www.adcisolutions.com/knowledge/how-build-single-page-application-spa-vuejs)

作者是 Dmitry Chuchin，ADCI 解决方案公司的前端开发人员。此外，他还努力通过教授开发新手和在教育活动中发表演讲来分享他的经验。此外，他喜欢滑雪和山地自行车。

*最初发布于*[*ADCI 解决方案网站。*](https://www.adcisolutions.com/knowledge/easiest-way-improve-your-vuejs-application-part-1?utm_source=medium&utm_medium=referral&utm_campaign=medium-prerendering-article&utm_content=medium-prerendering)

![](img/73bcb62bbfda355e913e5b86e2193e46.png)

**在社交网络上关注我们:** [Twitter](https://twitter.com/ADCISolutions) | [脸书](https://www.facebook.com/adcisolutions/) | [LinkedIn](https://www.linkedin.com/company/adci-solutions/)

[](https://codeburst.io/visual-regression-testing-with-backstopjs-bc749615191a) [## 用 BackstopJS 进行可视化回归测试

### 项目越大，每次变更后花在回归测试上的时间就越多。BackstopJS 是一种工具，它…

codeburst.io](https://codeburst.io/visual-regression-testing-with-backstopjs-bc749615191a)