# 寻找终极 CMS

> 原文：<https://itnext.io/in-search-of-the-ultimate-cms-98e45e8ca6ae?source=collection_archive---------2----------------------->

![](img/f42c23724d9a2091a343bc0800539a86.png)

*甘道夫带领寻找终极 CMS (* [*图片来源*](https://www.intofilm.org/intofilm-production/scaledcropped/970x546https%3A/s3-eu-west-1.amazonaws.com/images.cdn.filmclub.org/film__3930-the-lord-of-the-rings-the-fellowship-of-the-ring--hi_res-a207bd11.jpg/film__3930-the-lord-of-the-rings-the-fellowship-of-the-ring--hi_res-a207bd11.jpg) *)*

最近，我在考虑改造 www.rhythmandbinary.com，并有可能扩展到不同的平台。有很多选择，我想为我正在做的事情选择一个正确的。在我的研究过程中，我也能够开发我的专业网站 [andrewevans.dev](https://www.andrewevans.dev) (稍后会有更多的介绍)。这一切被证明是一次很好的学习经历，我想写下来。在这篇文章中，我将谈谈我使用最新 CMS 技术的经历，以及我选择使用我今天所拥有的技术的原因。

> 顺便说一下，在这篇文章中，我只关注 CMS 选项的一小部分。有很多种选择，我鼓励你在做出选择之前看看这篇文章以外的内容。

# 搜索

如果你用谷歌搜索“CMS 工具”，你会看到一系列不同的网站和平台。首先，术语“CMS”代表“内容管理系统”CMSs 通常用于管理企业客户以及博客等网站的内容。我在这篇文章的标题中加入了 CMS，因为我特别回顾了为博客网站服务的 CMS 平台。CMSs 的常见例子包括[WordPress.com](https://wordpress.com/)、[德鲁帕尔](https://www.drupal.org/)和[朱姆拉](https://www.joomla.org/)。

较大的 CMS 平台提供诸如分析、RSS 提要和其他维护和交互数据的方式。CMS 系统非常强大，因为它们提供了易于使用的工具，对技术人员和非技术人员都适用。有了 CMS 平台，您可以非常快速地启动并运行。CMS 平台通常提供:

*   出版能力
*   主办；主持
*   内容索引
*   邮件列表(关注者)
*   RSS 源
*   分析学
*   更多

然而，挑战在于有大量不同的选择。它们都提供类似的服务，但这取决于你喜欢什么和你想要什么样的体验。

在我的搜索中(对于这篇文章来说)，我主要关注以下几点:

我还想提一下，有许多由个别工程师开发的创新解决方案。我的朋友 Tim Deschryver 在 https://timdeschryver.dev/的 T2 有一个很棒的网站，提供了一个很好的例子。我在寻找更大的平台，因为我已经和 WordPress.com 的 T4 有了一个很好的网站。我喜欢平台系统的易用性，并希望查看我的所有选项，看看有什么可用的。我还对一个自定义选项非常感兴趣，因为它能够完全控制我的内容。在接下来的部分中，我将介绍几种平台的优缺点，并以总结我为什么选择我所做的来代替竞争来结束。

# 带有 Netlify CMS 的 GatsbyJS

![](img/629f910f37628760394496fd0ade9c93.png)

*(* [*)图片来源*](https://css-tricks.com/wp-content/uploads/2018/05/gatsby-netlify.jpg) *)*

如果您想生成和管理自己的内容，带有 Netlify CMS 的 GatsbyJS 是一个非常酷的选择。

[Netlify CMS](https://www.netlifycms.org/) 是一个开源工具，它(当[与 Netlify 的身份服务](https://www.netlify.com/docs/identity/)结合时)在生成和检索内容方面为博客做了大量的跑腿工作。当您使用 Netlify CMS 时，您可以在您的站点中使用一个提供 GUI 和编辑器的小部件。

[GatsbyJS](https://www.gatsbyjs.org/) 是一个基于 [ReactJS](https://reactjs.org/) 的静态站点创建器，它使用 [GraphQL](https://graphql.org/) 来梳理 GitHub 中站点的内容，并在你编写的模板中呈现出来。

当你用 GatsbyJS 和 Netlify CMS 创建一个站点时，你的所有内容都存储在你的 GitHub repo 中。Netlify 在您的 repo 中使用 OAuth 密钥来协调安全性，并将内容推送到 repo。您使用 Netlify CMS 小部件来生成内容，当您保存它时，它会被直接推送到 repo。如果你用 Netlify 托管它，这将导致你的站点有一个新的构建。除此之外， [Netlify 还为开发者提供免费账户](https://www.netlify.com/pricing/)！如果你想拥有一个博客，在这里你可以控制系统的所有活动部分，这一切都是技术和服务的完美结合。

> 为了更好地为自己设置这个，请查看这里的[教程](https://www.netlifycms.org/docs/gatsby/)。

正如我说过的，这个解决方案最酷的部分是它全部由您控制。Netlify CMS 实用程序是[的一个开源项目](https://github.com/netlify/netlify-cms)，实际上支持多个提供者(不仅仅是 Netlify)。另外，我没有提到 GatsbyJS 也使用 GraphQL 解析 repo 的内容。如果您想使用这两种技术，这对 GraphQL 和 ReactJS 开发来说都是一个极好的学习机会。

尽管有这些很酷的特性，但是，我没有选择这个选项有几个原因。首先，我浏览并设置了几个不同的网站，但我觉得我的 WordPress 网站的外观和感觉仍然更好。第二，原来工作量很大！我以前没有使用过 GraphQL，也不太熟悉 ReactJS。学习 [JSX 语法](https://reactjs.org/docs/introducing-jsx.html)和 ReactJS 中样式组件的来龙去脉肯定有一个学习曲线。我不得不说，在被 ReactJS 充分击败后，学习经验非常扎实。这次经历让我很好地了解了 GraphQL 和 ReactJS 的强大功能。

此外，我还意识到我真的很喜欢我的 WordPress 网站的易用性。我将在后面介绍更多关于 WordPress 的内容，但是用 Netlify CMS 回顾 GatsbyJS 向我展示了 CMS 工具实际上为您做了多少事情。然而，如果你是初学者或者想了解它们是如何工作的，我仍然建议你看看这两种技术。

# 中等

![](img/6af686f60f6e92fca5896486a8b732d1.png)

*(* [*图像来源*](https://atevans85.files.wordpress.com/2019/06/c8539-1l0zf9ap8xoinvbm78sijba.png) *)*

所以我实际上已经是一个频繁的媒体投稿人了。如果你查看我的[个人资料页面](https://medium.com/@andrew_evans)，你会看到我对 Angular-In-Depth 和 ITNext 的贡献。这些也是我个人帖子之外的。

我不介意中等，其实真的很喜欢界面。内容很容易创建，而“社交媒体”工具使得分享内容变得很容易。

我没有选择 medium 的原因是:( 1)目前付费墙存在很多[的混乱,( 2)他们不再允许](https://help.medium.com/hc/en-us/articles/360018834314-Stories-that-are-part-of-the-metered-paywall)[拥有自定义域名](https://help.medium.com/hc/en-us/articles/115003053487-Custom-Domains-service-deprecation)。

# 工兵

![](img/59ec0e056708d177a096a380d7c85fdb.png)

所以我不得不称赞我的朋友蒂姆·德施里弗让我对工蜂产生了兴趣。正如我在第一部分提到的，Tim 实际上在 https://timdeschryver.dev/用 Sapper 维护了一个个人博客网站，这真的很酷！他在构建自定义内容生成方面做得很好，甚至创建了自己的 RSS 提要。当我向 Tim 询问不同平台时，他向我提到了 Sapper，在编写了几个示例应用程序后，我印象非常深刻。

关于[工兵](https://sapper.svelte.dev/)的一点历史…..它基本上是一个尽可能利用服务器端渲染的框架。目标是开发高性能的应用程序，这些应用程序在运行它们的机器上使用有限的资源。这对不具备传统计算机处理能力的移动设备尤其有利。随着越来越多的东西进入云应用程序，这一点尤其有用。下载包的大小和浏览器的处理要求变得至关重要。

Sapper 实际上也是一个由 [SvelteJS](https://svelte.dev/) 驱动的框架。这两个框架都来自《纽约时报》的 T4·里奇·哈里斯。SvelteJS 是利用服务器端渲染来提高浏览器性能的一种尝试。Sapper 构建在 SvelteJS 之上，作为一种更容易搭建应用程序的方式，并利用 SvelteJS 提供的 serger 端渲染能力。两个主要的 Sapper 和 SvelteJS 文档站点都非常可靠。SvelteJS 甚至有一个[基础教程](https://svelte.dev/tutorial/basics)，向您展示一些可靠的代码示例。

Rich Harris 也有一些 YouTube 视频和这篇介绍性的博客文章，更详细地介绍了这项技术。

我喜欢 Sapper 的地方在于它极大地简化了开发体验。你建立“苗条”的文件，就像老式的 html。文件中内置了`<style>`和`<script>`来编写任何必要的脚本或样式。您也可以通过直接导入其他库来使用它们。当您使用 SvelteJS 构建应用程序时，您也有一个“静态”文件夹，您可以在其中存储您想要显示的图像和其他文件。这非常简单明了，并且使开发体验变得既简单又有趣。

> Sapper 和 SvelteJS 都是你可以单独写整篇文章的主题。请继续关注未来的帖子，它将更深入地探讨这两个问题。

尽管使用了服务器端渲染，SvelteJS 仍然支持使用 [fetch](https://sapper.svelte.dev/docs#this_fetch) 进行 rest 调用。还有多个生命周期挂钩，您可以利用它们来处理诸如“onload”等事件。

我非常喜欢 Sapper，以至于我用它建立了自己的专业网站。我甚至还创建了一个开源的帖子聚合器功能，用来收集我在不同地方发表的博客帖子。要查看我的帖子聚合器，请点击这里的 GitHub repo。

在构建了 andrewevans.dev 之后，我将注意力转向了 WordPress 和移动我的内容的潜力。我最终决定继续使用 WordPress，下一节将解释原因。

# 解决方案

很多时候，人们面临着选择，要么**去尝试一些新的、令人兴奋的东西**要么**保持现有的状态**。我非常支持为了更光明的未来而冒险，但在这种情况下，我决定留在原地。

我在这个博客网站上使用 WordPress 已经一年了。转换的过程需要我将内容转移到一个新的提供商，更新任何链接的帖子等等。此外，在考虑了其他平台之后，我意识到我对 WordPress 相当满意。WordPress 提供了我所需要的一切，我搜索的过程让我欣赏到了那些我认为理所当然的功能。订阅 WordPress.com(不是。org)为我提供了以下信息:

1.  主办；主持
2.  网站备份
3.  网站弹性
4.  博客网站的所有基本 CMS 需求
5.  RSS 源
6.  分析学
7.  还有更多！

如果我转向其他解决方案，我将不得不构建提供我刚才列出的所有这些项目的基础架构。**这会变成一件非常令人头疼的事情。**

然而，我必须说，我提到的其他技术仍然是有价值的路线。我不得不说，WordPress 为我工作，转换的成本对我的主站点来说太大了。

我还想指出的是，我认为 WordPress 作为“非技术”解决方案受到了不良的评价。然而，我不得不说，作为一个非常技术化的人，WordPress 让我的生活变得很简单。我知道如果可以说我完全建立了自己的网站会很酷，但是我意识到我对我所拥有的很满意。我列出的任何技术都是值得选择的。正如我在介绍中所说，这完全取决于你想做什么。

# 结束语

![](img/2865241f04ec3d7d3cc63421a465d8c7.png)

*甘道夫高高兴兴带着他的 CMS 选择退役(* [*图片来源*](https://media.tenor.com/images/22d9c96e28e15498c68c25ac2814be21/tenor.png) *)*

我对 CMS 解决方案的搜索让我走了很多路。在这个过程中我学到了很多东西。搜索相当累人，但它让我对现有的技术有了一个坚实的了解。此外，我获得了对技术的尊重，直到现在我才真正能够进入这些技术。

> “一切都应该尽可能简单**，**但不能简单**。”-爱因斯坦**

**这次搜索也让我体会到保持事情简单的价值。作为技术人员，我们经常试图找到最佳解决方案。这可能会让我们陷入本可以避免的兔子洞和深海潜水。我从这次经历中得到的最大收获之一是**简单就是好**。这完全取决于你想要什么，以及你对什么感到舒服。我们都有很多事情要做，有时候欣赏简单的事情真的很好。我选择的 CMS 解决方案不一定是最好的，但是我对我的选择很满意。**

**希望你喜欢这篇文章。请随意发表评论，并查看上面的一些链接。感谢阅读！**

***原载于 2019 年 6 月 9 日*[*https://rhythmandbinary.com*](https://rhythmandbinary.com/post/In_Search_of_the_Ultimate_CMS)*。***