# 在 Kubernetes 中使用 Rendertron 进行 SPA SEO

> 原文：<https://itnext.io/using-rendertron-in-kubernetes-for-spa-seo-39055567c745?source=collection_archive---------4----------------------->

我在 Kubernetes 演示中使用的一个应用程序是 [http://inkl.in](http://inkl.in) —这是一个以太坊区块链可视化工具，它使用前端(React)，与 API(节点)对话，与位于 Azure 私有 VNET 内的虚拟机上的 MongoDB 对话。你可以在 https://github.com/justindavies/inklin 的 GitHub [上找到代码，但是今天我想使用 Rendertron 来解决基于组件的 Javascript 应用的搜索引擎优化的固有问题。](https://github.com/justindavies/inklin)

SPA(单页应用程序)的问题是网络爬虫不太擅长 Javascript，尤其不擅长插入 Javascript 以获得合适的内容用于索引。

有许多“修复”，最突出的是服务器端渲染(SSR)。我在那里找到了大量的文档，但我只是一个懒惰的人，想要简单一点的东西(加上我在 webpack 上的技能水平是不存在的)。

随着像 Puppeteer(无头 Chrome)这样的东西的出现，我开始用它来代理对我的应用程序的请求，然后将其转换成搜索引擎的 HTML，以及像脸书和 Twitter 分享 Inkl.in 这样的东西

在这方面工作了一段时间后，我发现了 Rendertron，这是这类事情的惯常做法。看看 Sam Li 是怎么说的吧，这是解决问题的好方法。

Rendertron 本质上是一个请求代理，你不希望客户端执行 Javascript 在这种情况下，机器人和社交分享刮刀。为此，我们需要查看用户代理，看看是否需要将我们的请求路由到 Rendertron 进行预渲染，或者只是将请求直接传递给 React 应用程序。

因为我大部分时间都在 Kubernetes，而且我为微软工作，所以在 Google 上托管这个对我来说有点麻烦，我想让这个在 Kube 上工作。

Rendertron 不再附带 docker 文件，所以让我们继续创建一个

我们还需要把它放到 Kubernetes 上，所以让我们得到一个部署定义

和服务定义(我喜欢将它们分开)

现在我们的集群上已经有了 Rendertron，我们需要编辑 React 应用程序的 NGINX 配置。只是为了说明我是如何做的，这是构建我的 React 应用程序的 docker 文件

我们需要用 Rendertron goodness 替换默认配置。我们需要做的是感知 UA，然后仅当它是 Rendertron 的 Bot 或 Scraper 时才进行代理

基于非 Kube 部署的 Gists，有几件事让我抓狂。我必须将解析器设置为 kube-dns 实例，因为我对 proxy_pass 指令使用了一个变量(thankyou StackOverflow！)，并确保我在服务中使用了 FQDN，而不是简称。

在此之后，我可以看到，通过将我在 Chrome 中的 UA 更改为 GoogleBot，来自 GoogleBot 的请求被正确路由！