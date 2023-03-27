# 使用 vue.js、Python bottle 和 node.js Express 的新闻提要应用程序

> 原文：<https://itnext.io/news-feed-app-using-vue-js-python-bottle-and-node-js-express-e65c9cdadeb5?source=collection_archive---------8----------------------->

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fnews-feed-app-using-vue-js-python-bottle-and-node-js-express-e65c9cdadeb5)

在这里，我将解释我如何使用 vue.js 和 node express 创建了一个新闻订阅应用程序。用于 REST API 的 Python bottle 框架，从在线新闻网站获取新闻。我正在通过 web 报废获取 REST API 的数据。

![](img/00b2bda7fceba930f584a34d5e97c6df.png)

*写这篇文章的目的是展示我是如何让所有的东西一起工作的，而不是解释这些库是什么或者它们是如何工作的。*

所有将要开发的东西都可以在 GitHub 上找到: [NewsFeed](https://github.com/arunkrishna39/newsfeed)

![](img/aefb4ab29f4063c8119a6376793165a8.png)

照片由[克里斯里德](https://unsplash.com/@cdr6934?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

我们将从我如何使用 bottle 框架创建 REST API 开始。你可以参考这个奇妙的博客来了解更多关于如何使用 bottle 来做这件事的细节。

[](https://www.toptal.com/bottle/building-a-rest-api-with-bottle-framework) [## 用瓶子框架构建 Rest API

### REST APIs 一如既往地受欢迎。Bottle 框架是一个快速、轻量级的 Python web 框架，使构建…

www.toptal.com](https://www.toptal.com/bottle/building-a-rest-api-with-bottle-framework) 

首先安装所需的库。

我用过 bottle，还有另外两个库包括 **LXML** 和 **CSSSelect** 。您可以使用普通的 **pip** 命令来安装它。

上面的代码是核心文件，包括主要的端点和方法。我在这里定义了两条路线。一个是 default，另一个是/news，这是一个 get 请求，它调用**get _ latest _ from _ manorama()**函数来检索新闻。这个函数实际上向新闻门户发出请求，并使用从 lxml **解析**得到根元素。在接下来的 4 行中，我们通过 **cssselect** 过滤我们想要的实际数据，并将其返回到一个列表中。

[](https://pypi.python.org/pypi/cssselect) [## cssselect 1.0.3 : Python 包索引

### cssselect 解析 CSS3 选择器并将它们翻译成 XPath 1.0

pypi.python.org](https://pypi.python.org/pypi/cssselect) 

上面的代码是 Bottle REST API 的主要配置和入口文件，它将监听端口 3000。脚本的最后一行使用指定的服务器运行 Bottle。将新闻信息从 api 包导入到此，这样我们就可以访问我们在那里定义的所有端点。您可以执行此文件，服务器将启动并运行。

我已经使用 vue-cli 创建了 vue SPA 模板。可以使用下面的 npm 命令安装它。

```
npm install -g vue-cli
vue init webpack my-project
```

 [## 介绍手册

### 编辑描述

vuejs-templates.github.io](http://vuejs-templates.github.io/webpack/) 

上面的 vue 脚本将访问我们通过 axios 创建的 REST API，并将其添加到新闻列表中，作为数据公开。数据中提到的字段可以通过 vue 的声明性呈现跨模板部分访问。数据和 DOM 链接在一起，现在一切都是被动的。指令可以使用数组中的数据来显示项目列表，使用 v-bind 将值映射到属性。

# 摘要

现在可以自由运行应用程序并查看结果。正如介绍中所写的，这个项目可以在 GitHub 上获得: [NewsFeed](https://github.com/arunkrishna39/newsfeed)

如果你喜欢这篇文章，请在推特上[关注我，以获得下一篇文章的通知。](https://twitter.com/arunkrishna39)