# 管理 Kubernetes 上的入口控制器:第 1 部分

> 原文：<https://itnext.io/managing-ingress-controllers-on-kubernetes-part-1-784fc42ba397?source=collection_archive---------4----------------------->

## 什么是入口控制器

![](img/50d629a4a504a1f32f627285001d760f.png)

克里斯·拜尔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

Kubernetes 将部署在集群上的服务与集群外部的世界隔离开来。它这样做是为了保护服务免受不请自来的关注。然而，更多的时候，我们需要一个或多个部署的服务，以便在集群之外使用。Kubernetes 为实现这一点提供了许多机制，但它提供的最灵活的技术可能是 Ingress 提供的技术。

## 为什么不使用负载平衡器类型的服务？

部署一个[负载平衡器类型的服务](https://docs.giantswarm.io/guides/services-of-type-loadbalancer-and-multiple-ingress-controllers/)，而不是使用入口 API，其中一个[外部负载平衡器](https://blog.giantswarm.io/load-balancer-service-use-cases-on-aws/)(例如 AWS ELB)被自动创建并配置为将外部流量路由到代表服务的 pod。然而，这种方法的问题是，它提供了一个相当有限的解决方案，并且它为这种类型的每个服务创建了一个外部负载平衡器。从成本和管理开销的角度来看，创建大量外部的、特定于云的负载平衡器最终可能会成为一个昂贵的提议。相比之下，Kubernetes Ingress API 为配置 Kubernetes 集群的入口提供了很大的灵活性，入口控制器可以作为一个复制服务或单一外部负载平衡器背后的服务进行部署。

[![](img/c46c9bc5c32531dc4b9e02fbc12fd9f9.png)](https://info.giantswarm.io/guide-to-managing-ingress-controllers)

## 是什么让使用 Ingress 成为一个好的选择？

Ingress 提供第 7 层 HTTP 路由，使您能够使用单个域名为多个服务提供服务，跨服务副本负载平衡流量，终止 SSL，实现 SSL 直通，等等。

在 Kubernetes 中，入口概念被实现为一个 Kubernetes API 资源和一个特定于实现的控制器，称为入口控制器。在了解生态系统中的一些不同入口控制器之前，我们先简单介绍一下它们的功能。

## 入门:创建入口资源

入口资源用于定义路由规则，这些规则决定了外部客户机如何访问 Kubernetes 集群中运行的服务。这些规则允许基于 URL 请求的主机和路径进行路由。

下面这个简单的例子显示了一个发往主机`acme.com`的入口流量规则，其中匹配路径`acme.com/foo`的请求被路由到端口`80`上名为`svc1`的后端服务，匹配路径`acme.com/bar`的请求被路由到端口`80`上名为`svc2`的服务:

```
apiVersion: extensions/v1beta1kind: Ingressmetadata:name: acme-app-ingressspec: rules: - host: acme.com http: paths: - path: /foo backend: serviceName: svc1 servicePort: 80 - path: /bar backend: serviceName: svc2 servicePort: 80
```

*示例入口资源，您可以将它保存到一个文件中，并使用:* kubectl apply -f ingress.yml 将其应用到集群中

> [**阅读第 2 部分:*让它真正发生:各种入口控制器***](https://medium.com/@GiantSwarm/managing-ingress-controllers-on-kubernetes-part-2-36a64439e70a)

由 [Puja Abbassi](https://twitter.com/puja108) 撰写——开发者倡导者@ [巨型蜂群](https://giantswarm.io/)

[](https://twitter.com/puja108) [## puja Abbas si @ # kube con # kube Khan(@ puja 108)| Twitter

### Puja Abbassi 的最新推文@ #KubeCon #KubeKhan (@puja108)。开发者倡导者@ GiantSwarm & Researcher

twitter.com](https://twitter.com/puja108) 

准备好将您的云原生项目投入生产了吗？只需在 https://giantswarm.io/[申请免费试用](https://giantswarm.io/)