# 管理 Kubernetes 上的入口控制器:第 4 部分

> 原文：<https://itnext.io/managing-ingress-controllers-on-kubernetes-part-4-9f201296db28?source=collection_archive---------3----------------------->

## 部署多个入口控制器

![](img/63e104f66e256ca2718cffda6f95a8e4.png)

照片由[弗多斯·罗斯](https://unsplash.com/@firdoussross?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

您可能认为一旦使用 Kubernetes API 定义了入口对象，并且部署了入口控制器，那么定义入口的更多选项就完成了……事实并非如此。拥有一个以上的入口定义，甚至一个以上的入口控制器部署，这是完全可能的，也是经常需要的。可能有数不清的场景要求部署多个入口资源，但这里有几个:

我们已经看到，入口控制器的实现是不同的，可以基于不同的技术，具有不同的功能。您可能会选择首选的入口控制器实现，但需要针对特定服务后端的不同控制器实现的功能。一般入口可由优选的控制器来满足，特定的使用情况由具有独特能力的控制器来满足。

多个入口要求可能有细微的不同。例如，一些入口定义可能需要 TLS，而一些入口定义可能在前端负载平衡器中终止 TLS。因此，入口的语义在每种情况下都会不同，这可能需要创建不同的入口资源定义。

除了为面向公众的服务配置入口之外，我们还可能同时需要处理内部流量和组织私有流量的入口。也许我们需要将流量从一个 Kubernetes 集群路由到另一个集群，或者在非容器化服务和集群内部运行的服务之间路由。显然，在这种情况下，我们需要不同的入口对象和控制器部署，以便区分不同的流量类型。

我们满足这些不同场景的唯一方法是部署多个入口定义和/或不同的入口控制器部署。这提出了一些重要的问题——我们如何确保流量被路由到正确的后端服务？我们如何确保不同的入口控制器不会竞争相同的流量？

`kubernetes.io/ingress.class`注释确定哪个入口控制器将处理对该入口资源的请求。如果我们假设 Kubernetes 集群部署了 k8s-ingress-nginx 控制器和 Contour Ingress 控制器，那么为了确保由特定入口对象定义的流量由 Contour Ingress 控制器处理，它的定义必须包含注释`kubernetes.io/ingress.class: “contour".`

有时，部署相同入口控制器类型的两个不同配置版本来处理不同的需求可能是相关的。如果我们部署了两个 k8s- ingress-nginx 控制器，在入口对象中使用`kubernetes.io/ingress.class: “nginx"`定义是不明确的；它可以指两个控制器中的任何一个，并且会导致具有不可预测结果的控制器争用。为了避免这个问题，当使用命令行标志部署控制器时，入口控制器通常允许定义类名。

对于 k8s-ingress-nginx 控制器，这是`— ingress-class`，对于 Contour ingress 控制器，这是`ingress-class-name`，对于 Traefik Ingress 控制器，这是`kubernetes.ingressclass.`，因此，具有两个 k8s-ingress-nginx 控制器的集群可能有一个类别名称为 internal 的集合，用于处理来自集群内来源的流量，还有一个类别名称为`external`的集合，用于处理来自集群外的流量。

[![](img/c46c9bc5c32531dc4b9e02fbc12fd9f9.png)](https://info.giantswarm.io/guide-to-managing-ingress-controllers)

## 结论

Kubernetes Ingress API 为将流量从边缘路由到部署在 Kubernetes 集群上的服务提供了最灵活、最强大的方法。它的实现在固执己见方面取得了平衡；一方面，入口资源在本质上是非常规范的，但是控制器组件特意
留给第三方利用“同类最佳”反向代理技术来提供入口功能。

在本文中，我们考虑了三种流行的入口控制器，每一种都使用不同的底层反向代理技术，并有自己的侧重点和特定功能。还有更多入口控制器实现方式；[航海家号](https://github.com/appscode/voyager)、[孔](https://github.com/Kong/kubernetes-ingress-controller)、[船长](https://opensource.zalando.com/skipper/kubernetes/ingress-controller/)不一而足。虽然一些控制器实现比其他的更受欢迎，但是没有“最佳”控制器，或者事实上的标准。在决定使用哪种控制器实现时，根据手头的技术要求进行完整的评估并适当考虑非技术因素是有意义的，例如“内部是否熟悉底层反向代理技术？”，或者“有可能获得入口控制器的商业支持吗？”。

自最初实现以来，Ingress API 变化很小，尽管[声明意图](https://kubernetes.io/docs/concepts/services-networking/ingress/#future-work)将其开发到超出其有限的能力。这导致了特定于控制器的注释的激增，这些注释的唯一目的是为入口体验改进缺失的功能。Contour 入口控制器使用的 Ingres route CRD 是将讨论向前推进的一种尝试。但是，正如 Kubernetes Network SIG 在 2018 年 2 月进行的一项调查显示，人们希望有一个更具表现力的 API，但也希望入口定义可以在不同的控制器之间移植。也许就什么应该或不应该是 API 的一部分达成一致是绊脚石？[如果你有看法，一定要分享给社区。](https://kubernetes.io/community/)

由 [Puja Abbassi](https://twitter.com/puja108) 撰写——开发者倡导者@ [巨型群体](https://giantswarm.io/)

[](https://twitter.com/puja108) [## puja Abbas si @ # kube con # kube Khan(@ puja 108)| Twitter

### Puja Abbassi 的最新推文@ #KubeCon #KubeKhan (@puja108)。开发者倡导者@ GiantSwarm & Researcher

twitter.com](https://twitter.com/puja108) 

准备好将您的云原生项目投入生产了吗？只需在 https://giantswarm.io/[申请免费试用](https://giantswarm.io/)