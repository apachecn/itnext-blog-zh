# 测试运营商 SDK 并为 Kubernetes 制定预取机制

> 原文：<https://itnext.io/testing-the-operator-sdk-and-making-a-prefetch-mechanism-for-kubernetes-7508577efdd7?source=collection_archive---------6----------------------->

![](img/08700f6bc6220ca5c9a91ad430b554bf.png)

## 介绍

在本文中，我们将探索如何使用 Operator SDK 创建一个可以预取我们的映像(从我们的部署到所有节点)的操作器，您可能想知道为什么要这样做？主要的想法是提前获取图像，这样当 pod 实际上需要在给定的节点上开始运行时，您就不必再调用它们了，这样可以加快速度，而且这也是一个有趣的练习。

如果你读过文章[用 kubebuilder 和 kind aka kubernetes operators](https://techsquad.rocks/blog/cloud_native_applications_with_kubebuilder_and_kind_aka_kubernetes_operators/) 云原生应用，你会注意到命令彼此之间非常相似，因为现在 operator-sdk 使用 kubebuilder，你可以在这里阅读更多。

这篇文章的来源是[这里](https://github.com/kainlite/kubernetes-prefetch-operator/)

**先决条件**

*   [运营商 SDK](https://sdk.operatorframework.io/docs/installation/install-operator-sdk/)
*   [开始](https://golang.org/dl/)
*   [种类](https://github.com/kubernetes-sigs/kind)
*   [码头工人](https://hub.docker.com/?overlay=onboarding)
*   [草薙](https://github.com/kubernetes-sigs/kustomize)

## 创建我们的本地集群

多集群的种类配置

这是本地多节点设置所必需的配置:`kind create cluster --config kind.yaml`

你可以在这里找到文件

**创建集群**

我们将需要一个集群来运行和测试我们的操作符，所以 kind 非常简单，并且足够轻量，可以在任何地方运行。

## 创建我们的操作员

在这里，我们引导我们的 go 项目，即 kubernetes 操作符

## 创建我们的 API

这个对象将保存给定图像的所有重要信息，我们首先需要修改的文件在:`controllers/*_controller.go`和`api/v1/*_types.go`

## 构建和推送(docker 图像)

使用项目助手基本构建和推送操作员映像

## 部署

既然我们已经将项目构建到 docker 映像中并存储在 dockerhub 中，那么我们可以安装我们的 CRD，然后部署操作符

**部署操作员**

然后我们可以部署我们的操作员

检查我们的吊舱正在运行

到目前为止，一切都很好，但我们的操作员目前什么都不做，所以让我们删除一些代码，让它做我们想要的…

## 我们的准则

我们使用的很多东西都是生成的，但是我们需要给它一些特定的权限和行为给我们的操作者，这样当我们在 kubernetes 中创建一个对象时，它就会做我们想要做的事情

**我们的清单**

这将是我们将用来告诉我们的操作员我们想要预取映像的部署的清单

nginx 部署示例

这个 nginx 部署将用于验证是否在所有节点中获取了图像。我们实际上不需要这样做，但是这样很容易确保如果标签不存在，就不会调度一个 pod:`kubectl label nodes kind-worker3 nginx-schedulable="true"`

**我们的实际逻辑(这让我窃笑了这么多自举只是为了到这里，但是想象一下你必须自己做所有这些只是为了得到足够的东西让一个 pod 以受控的方式运行)**

这是事情实际发生的地方，首先我们更新我们的规范:

然后我们可以放一些代码，稍后我会在代码中添加更多的注释来解释每件事的作用:

基本上，我们所做的是设置一个计时器，在每个节点中创建一个 pod，以强制它获取部署(我们按标签筛选)需要或将要使用的映像，通过这样做，如果节点已经有了映像，则不会发生任何情况，它将在下次运行时被删除，但是如果映像不在那里，它将被获取，因此如果发生任何情况，pod 需要实际安排在那里，它将不需要下载所有内容，因此它应该相对更快。

我们应该在集群中看到什么

确保`kubectl apply -f config/samples/k8s_nginx_deployment.yaml`和`kubectl apply -f config/samples/cache_v1_prefetch.yaml`

## 清理

要从集群中清理运营商，您可以这样做，同时记住清理您的集群或您正在使用的任何东西，如果它在云中，以避免意外的账单

**成交单据**

如果你想了解更多关于这个项目的信息，请务必查看链接，我希望你喜欢它，在 [twitter](https://twitter.com/kainlite) 或 [github](https://github.com/kainlite) 上再见！

这篇文章的来源是[这里](https://github.com/kainlite/kubernetes-prefetch-operator/)

# 正误表

如果您发现任何错误或有任何建议，请给我发消息，以便解决问题。

此外，您可以在这里查看源代码和[生成代码](https://github.com/kainlite/kainlite.github.io)和[源代码](https://github.com/kainlite/blog)的变化

*原载于 2020 年 11 月 1 日*[*https://tech squad . rocks*](https://techsquad.rocks/blog/testing_the_operator_sdk_and_making_a_prefetch_mechanism_for_kubernetes/)*。*