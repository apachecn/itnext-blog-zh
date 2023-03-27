# 构建 Kubernetes 聚合 API 服务器

> 原文：<https://itnext.io/our-journey-in-building-a-kubernetes-aggregated-api-server-29a4f9c1de22?source=collection_archive---------3----------------------->

Kubernetes 有两个主要的扩展机制——自定义资源定义(CRD)和聚合 API 服务器。一个聚合 API 服务器是一个 REST API 服务器，它运行在集群上，可以使用 kubectl 访问。它将客户端身份验证委托给主 API 服务器。

在这里，我们分享我们在构建一个聚合 API 服务器来解决一个特定问题的实验。

*问题:*

我们正在开发一个[发现工具](https://github.com/cloud-ark/kubeprovenance)，帮助发现关于 Kubernetes 集群的动态信息。这种信息的一个例子是 Kubernetes 对象的动态组合。在 Kubernetes 中，有由其他资源组成的顶级资源。例如，一个部署由一个复制集组成，而复制集又由一个或多个 pod 组成。今天，对于给定父资源，找出整个子资源树并不简单。

在构建这个工具时，我们的主要设计约束是不应该要求用户使用新的 CLI 工具来查找这些信息。他们应该可以直接使用 kubectl。例如，用户应该可以使用以下命令来获取部署类型的所有对象的合成信息:

```
kubectl
get --raw /apis/../../namespaces/default/deployments/*/compositions
```

在这方面，我们认为我们的问题类似于 Kubernetes 中的 [metrics API 服务器。特别是，查询到“../deployments/*/compositions”看起来类似于在 metrics API 服务器中使用定制指标。](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/metrics-server.md)

所以我们决定构建一个类似于[定制度量](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/custom-metrics-api.md)服务器的东西。

*达成解决方案:*

*方法 1:* 我们从 [apiserver-builder 存储库](https://github.com/kubernetes-incubator/apiserver-builder)开始。它有大量的文档，并很好地概述了主 API 服务器和聚合 API 服务器之间的身份验证和授权是如何工作的。它还支持创建子资源，这正是我们所寻找的。不幸的是，使用这个库只能在您第一次创建的类型上创建子资源。如果我们想要创建“compositions”子资源，我们必须首先为它的父资源创建新的种类——在上面的例子中是“deployments”。

这不是我们想要的！我们想在现有的种类上添加子资源。所以使用 apiserver-builder 是行不通的。

*方法 2:* 接下来，我们决定尝试一下 [custom-metrics-apiserver](https://github.com/kubernetes-incubator/custom-metrics-apiserver) ，因为它似乎使用子资源来定义现有 Kubernetes 种类上的自定义指标。不幸的是，它[没有足够的文档](https://github.com/kubernetes-incubator/custom-metrics-apiserver/issues/10)。我们仍然试着使用它，但是那没有得到太多。

*方法 3:* 在前两种方法失败后，我们决定尝试 [sample-apiserver](https://github.com/kubernetes/sample-apiserver) 。它像记载的那样工作。然而，对于我们的用例来说，它有两个问题。首先，它只演示了如何添加将由聚合 API 服务器处理的新的顶级资源/种类。这不是我们感兴趣的。其次，它没有展示如何添加定制的子资源。不过，这个服务器有一个好处。它在工作！我们现在有了可以作为起点的东西。剩下的工作(我们也是这样认为的)就是弄清楚如何转换 sample-apiserver，以便它能够支持现有 Kubernetes 种类上的“组合”子资源。

*方法 4:* 我们首先想到的是从 sample-apiserver 开始，然后一点一点地将 custom-metrics-apiserver 移植到其中。我们也是从这条路开始的。但是很快我们遇到了一个问题。原来 sample-apiserver 和 custom-metrics-apiserver 使用了两个 Kubernetes 子包的不同版本(apimachinery 和 apiserver ),并且这些版本是不兼容的。我们试图通过使用别名导入名称导入一个包来避免这个问题。然而，这不起作用，因为别名导入包中的导入仍然指向与别名包不兼容的非别名子包名称。深入思考这个问题，我们意识到除非 Kubernetes 为其包实现 Golang 即将发布的[语义导入版本](https://blog.golang.org/versioning-proposal)，否则不可能在一个项目中使用同一个包的两个不兼容版本。

*解决方案:*于是我们又回到了谈判桌前。感觉像是从 custom-metrics-apiserver 中得到一些我们可以使用的东西。事实证明确实有！为了注册定制的子资源端点，它使用 go-restful 库来创建 REST 路由，然后[将它们添加到 GenericAPIServer](https://github.com/kubernetes-incubator/custom-metrics-apiserver/blob/102f5e88afde0d31116b5498e0cf939da91cb6ba/pkg/apiserver/cmapis.go#L59) 中定义的 GoRestfulContainer。那是我们的线索。

那时，我们想出完整的解决方案只是时间问题。你可以在我们的 [kubediscovery github 资源库](https://github.com/cloud-ark/kubeprovenance)中找到它。使用它来创建您自己的带有自定义子资源的聚合 API 服务器。更好的是，使用它来找出集群中各种 Kubernetes 的动态组成。

[www.cloudark.io](https://cloudark.io/)