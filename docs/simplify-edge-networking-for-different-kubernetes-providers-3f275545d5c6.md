# 为不同的 Kubernetes 提供商简化边缘网络

> 原文：<https://itnext.io/simplify-edge-networking-for-different-kubernetes-providers-3f275545d5c6?source=collection_archive---------5----------------------->

Kubernetes 为编排容器提供了高级接口，使得开发、部署和维护云原生应用程序变得更加容易。虽然这一切都是开放和免费使用的，但将 Kubernetes 接口与物理基础设施捆绑在一起可能会很复杂。

通过与数百名客户和社区成员的合作，我们意识到工程师需要一种简单而廉价的方法来试验运行在不同基础设施上的 Kubernetes。这就是我们创建 K8s 初始化器的原因。

![](img/191e58804daa7fcc23e357edd87d4650.png)

# 特定于平台的工具和配置

各种公司每天都在通过提供托管的 Kubernetes 平台来帮助简化这一过程。这些平台致力于将这些 Kubernetes 接口与其他工具联系起来，并帮助成千上万的组织轻松采用 Kubernetes。

虽然这对于简化 Kubernetes 的采用非常好，但是它在 Kubernetes 接口的实现方式上引入了碎片。例如，Kubernetes 提供了 **type: LoadBalancer** 服务接口来提供负载平衡器，但是平台决定如何实现它。这意味着根据平台的不同，您可能会得到不同类型的负载平衡器。

# 分裂:选择和复杂性

平台能够实现自己的[定制资源定义](https://www.getambassador.io/learn/kubernetes-glossary/custom-resource-definition/)，扩展现成可用的 API 集，从而扩展了这种碎片化。平台提供商真诚地这样做，试图使 Kubernetes 更容易使用，但它可能会产生负面影响，使您更难构建平台不可知的应用程序。

这个生态系统可以提供很多东西，但不幸的是，Kubernetes 公开的接口的平台实现可能很难轻松开始并利用不同提供商所提供的东西。

# K8s 初始化器:自动化平台无关网络

由于不同的 Kubernetes 平台提供不同的服务和优势，构建一个平台无关的应用程序可以帮助您尝试每个平台所提供的功能。由于边缘操作通常依赖于平台特定的配置，因此很难轻松替换。

[Kubernetes 初始化器](https://app.getambassador.io)通过自动引导各种 Kubernetes 平台的边缘网络来帮助简化这一过程。这使得您只需担心构建运行在任何 Kubernetes 平台上的应用程序，并让初始化器来负责为它们获取流量。