# Kubernetes OWASP 十大:简介

> 原文：<https://itnext.io/kubernetes-owasp-top-10-intro-73943be7add2?source=collection_archive---------2----------------------->

最近， [OWASP 基金会](https://owasp.org)介绍了他们在采用和实施 Kubernetes 时的最新十大风险。这个列表是一个很好的切入点，可以确保在 Kubernetes 上部署工作负载时引入的风险可以从一开始就得到缓解，从而实现安全的设计架构。

本系列将贯穿前 10 大风险，更详细地介绍每个突出的风险，这些风险对您的应用程序和整体业务意味着什么，并提供一些模式和实践作为如何最好地减轻所呈现的风险的示例，从本简介和前 10 大风险的概述开始。

![](img/c6f3ffdfb817a3138a4eb08323dcc51f.png)

由[托马斯·博曼斯](https://unsplash.com/@thomasbormans?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# OWASP Kubernetes 十大

## 不安全的工作负载配置

默认情况下，Kubernetes 清单旨在确保在尽可能短的时间内启动并运行所需的最少配置，从而降低准入门槛。跟随许多在线教程，甚至许多开源项目和舵图，许多基础知识已经完成，跳过了一些重要的安全考虑。一旦事情启动并运行，安全性通常是事后的想法，因此这些控制可能会被忽略，并且在初始开发后不会返回。

示例:

*   [AKS 容器示例](https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/design-develop-containerized-apps/build-aspnet-core-applications-linux-containers-aks-kubernetes#deploy-webapiyml)
*   [EKS 样本部署](https://docs.aws.amazon.com/eks/latest/userguide/sample-deployment.html)
*   [空有证券的孔掌舵图](https://github.com/Kong/charts/blob/e9c77a52a7ef4f90128ebff4d0fef6600fa6eda7/charts/kong/values.yaml#L757)

一旦建立了安全基准，例如不应以 root 用户身份运行或不具有特权权限的 pod，使用集中策略管理可以实施基准并拒绝未能遵守基准的部署。

## 供应链漏洞

软件供应链漏洞是整个开发者生态系统的一大风险，Kubenetes 和 containers 对此也不例外。使用可能来自众多第三方来源的分层映像可能包括数百个第三方软件依赖项，每个依赖项都存在潜在风险。

有多个步骤可以降低容器化应用程序中的这种风险，例如，图像签名(确保容器图像来自预期的作者)，针对容器和代码使用漏洞扫描器，如 Snyk 或 Trivy 或 Clair 等开源项目可以融入安全软件设计生命周期，并限制仅使用已知或专业的小型基础图像，如 Scratch。

## 过度宽松的 RBAC 配置

在任何 Kubernetes 部署中创建的第一个帐户是集群管理员。这实际上是超级用户，可以不受限制地访问集群的所有组件。限制集群管理的使用是减少攻击面和集群完整性风险的第一步。

其他缓解措施包括:仅在需要时而非默认情况下挂载服务帐户令牌，尽可能在 ClusterRoleBindings 上使用 RoleBindings 来限制访问范围，以及审核外部包以确保它们不会授予比所需更多的权限。

## 缺乏集中的政策执行

Kubernetes 环境中可以采用的定制数量和密度为开发人员提供了极大的灵活性，可以充分利用容器化他们的应用程序。然而，这对于管理、审计和实施任何可能定义的安全设计模式来说会变得极其笨拙。

在 Kubernetes 集群中，有可能从 Docker Hub 等公共存储库中提取数百个不同的图像，有些图像比其他图像更安全。如果群集资源受损，攻击者还可能会下载易受攻击的映像来建立权限提升或持久性。使用开源项目，如云提供商提供的网关守护设备或中央安全策略(例如 Azure Defender for Containers 策略)，可以强制要求仅从受信任的或由组织管理的内部注册表下载图像。

使用 Gatekeeper 之类的工具利用 Kubernetes 准入控制器来确保资源不会部署到不符合策略的集群。这是非常可扩展的，可以应用于任何可能存在风险的错误配置。

## 记录和监控不足

Kubernetes 部署上的日志记录和监控不足或过多会导致无法了解集群的各个组件级别发生了什么，或者暴露了敏感信息。

*   Kubernetes 审计日志——记录针对 Kubernetes API 的操作，包括登录尝试。
*   容器日志—应用程序级日志记录，通常会暴露敏感信息。使用安全的日志传送方法有助于降低这种风险。
*   主机级日志—单个群集节点可以为其他日志提供更多上下文信息，对于调查至关重要。

## 被破坏的身份验证方法

Kubernetes 中有两种主要的认证类型

*   操作员身份验证—用户登录并针对 API 执行手动交互
*   服务帐户身份验证—针对 API 的自动化任务

在任何环境中，身份验证都是一个复杂的话题，Kubernetes 有自己的一些细微差别。通常，当用户第一次在集群中配置时，他将是集群管理员，并使用证书身份验证进行设置。但是，Kubernetes 没有证书撤销的概念，因此如果证书泄漏，没有简单的方法可以在不重新发布集群中的整个证书链的情况下拒绝访问。

使用众所周知的身份标准(如 OIDC ),可以将身份认证卸载给群集外的身份提供商，并允许使用 MFA 等附加控制来帮助降低风险。

![](img/c29e54d8b4aac506aa6abb7f4980d1eb.png)

照片由[飞:D](https://unsplash.com/@flyd2069?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

## 缺少网络分段控制

Kubernetes 网络的一个核心概念是，所有的 pod 都接收一个唯一的集群范围的 IP 地址，并且它们应该能够与任何主机上的任何其他 pod 通信，而不需要 NAT。这简化并允许向后兼容，但是在安全性方面，默认情况下在网络级别没有工作负载隔离。这带来了风险，如容器之间的横向移动，或在违规情况下访问内部 API。

根据工作负载和集群架构的类型，有多种创建网段的方式。

*   多集群—这并不总是可行的，尤其是在微服务架构中，但是在多租户情况下，利用多个集群作为网络边界可以隔离工作负载。
*   network policies——一种本地的 Kubernetes 类型，可以利用标签在 pod 和名称空间之间充当防火墙规则。但是，应该注意，为了能够实施网络策略，在大多数情况下需要使用 CNI(容器网络接口)插件，例如 Cilium 或 Project Calico。
*   服务网格—服务网格是 Kubernetes 网络的覆盖层，支持服务发现、mTLS 和安全规则等可扩展功能。

## 机密管理失败

Kubernetes 有一个本地秘密类型，这允许细粒度的访问控制，并确保只有需要访问适当秘密的实体才可以访问。Kubernetes 的秘密是以 base64 编码存储的，**而不是**加密，这意味着任何有权读取特定秘密的人都可以解码它，并以明文形式查看它。这意味着需要小心确保秘密保密。

Kubernetes 使用 etcd 作为其配置存储，并且能够对特定的对象类型进行加密。加密的使用意味着用户无意中获得了他们不应该看到的秘密，也需要有解密的能力。此外，应尽可能使用全磁盘加密对 etcd 实例进行静态加密，并注意确保备份时保持加密。

Kubernetes 利用两种方法将秘密类型注入到容器中，作为环境变量或作为文件挂载。这两种方法各有利弊，但是使用环境变量意味着存在应用程序崩溃的风险，会将环境变量的详细信息(包括机密)以明文形式发送到日志中。

理想情况下，机密将在第三方 secrets vault(如 Azure KeyVault 或 Hashicorp Vault)中进行管理，并在运行时尽可能地注入，以便机密在不使用时不会存储在磁盘上。

## 错误配置的集群组件

像 Kubernetes 对象一样，集群组件是高度可配置的，并且通过一些额外的努力可以变得更加安全。Kubernetes 由 4 种主要成分组成。

*   Kube-APIServer
*   Kube 代理
*   库伯莱
*   etcd

这些组件中的每一个都有由 CIS(互联网安全中心)设定的最佳配置实践。这些基准可以针对集群运行，评估每个组件并给出如何使其更加安全的建议。

## 过时和易受攻击的 Kubernetes 组件

像所有软件一样，Kubernetes 和外部集成组件容易受到攻击。补丁管理至关重要，不仅对 Kubernetes 和节点主机如此，对入口控制器和服务网格等其他附加组件也是如此，应在企业内使用的其他补丁管理程序中进行管理。

如你所见，有很多东西需要发掘和消化。在这个系列中，我将更详细地讨论每一部分，强调风险和减轻这些风险的方法。