# 在 Kubernetes 中应用零信任的两种快速方法

> 原文：<https://itnext.io/two-quick-ways-to-apply-zero-trust-to-kubernetes-79764dd420bf?source=collection_archive---------2----------------------->

如今，零信任风靡一时，但考虑到现代应用程序在整个云(或多个云)中的成熟性，它离大肆宣传并迅速成为现实还差得很远。传统上，敏感数据位于一个数据中心(您的城堡)内，外围安全的[层](https://en.wikipedia.org/wiki/Perimeter_security)(您的护城河)用于保护数据的进出。然而，当今的世界更加复杂，因此安全模式已经适应了新的模式。攻击者总是试图(并成功地)在门口获得一个“立足点”，但零信任确保他们只是呆在大厅里！

![](img/08a07f5f85fde0416ad0fb128e0f33d6.png)

传统安全是围绕数据城堡的护城河，零信任更像是围绕小城堡的小护城河

# 今天的多重云

对于一个组织来说，将销售数据保存在 SaaS CRM 中，将客户数据保存在公共云中(使用类似 Kubernetes 的东西)，将内部数据保存在本地数据中心(使用裸机)并不罕见。更有甚者，访问这些数据的员工和客户遍布世界各地，不再连接到办公室的局域网，而且一如既往，黑客不断试图进入。零信任非常直观，因为数据会在不同的信任区域中传输，您必须在每一步都验证其完整性和真实性。现在我们已经有了零信任的心态，让我们把它应用到 Kubernetes 上吧！

# 1.网络策略

开箱即用，Kubernetes 提供了一些可怕的，但不安全的默认设置。所有 pod 都可以与其他 pod 对话，并且所有 pod 都可以与互联网对话。这是一个很好的开始，但最终你会想把事情锁定下来。[网络政策](https://kubernetes.io/docs/concepts/services-networking/network-policies/)正是这么做的。我们将从真正的零信任开始——默认的拒绝所有策略，不允许任何东西与任何东西对话。

```
// Default Deny All Network PolicyapiVersion: networking.k8s.io/v1 
kind: NetworkPolicy 
metadata:   
  name: **default-deny-all** 
spec:   
  podSelector: {}   
  policyTypes:   
  - Ingress   
  - Egress
```

这个策略对我来说确实巩固了零信任的概念，这里的策略是默认拒绝一切，然后慢慢地添加更多的许可策略来覆盖最初的`default-deny-all`策略。在信息安全社区中，我们称之为最小特权原则。与许多新的安全趋势一样，零信任依赖于这一久经考验的原则。下面是我们如何覆盖`default-deny-all`策略:

```
// Allow Access from App A to App BapiVersion**:** networking.k8s.io/v1
kind:NetworkPolicy
metadata:
  name: access-my-app
spec:
  podSelector:
    **matchLabels:
      app: app-1**
  ingress:
  - from:
    - podSelector:
        **matchLabels:
          app: app-2** egress:
  - to:
    - podSelector:
        **matchLabels:
          app: app-2**
```

起初看起来可能有点混乱，但像 Kubernetes 的其他部分一样，*标签是这里的关键，*和`matchLabels`是我们希望连接的点。从入口和出口的角度来看，这个策略本质上是将带有一个标签的 pod 连接到下一个标签(我已经用粗体显示了)。在这种情况下，标签为`app: app-1`的 Pod 只能接收流量(入口)并向标签为`app: app-2`的 Pod 发送流量(出口)。

更直观地说——入口部分允许 **app-1 ← app-2**

*出口*部分允许 **app-1 → app-2**

总的来说，这为我们提供了 **app-1 ← → app-2**

网络策略是锁定您的 Kubernetes 集群开箱即用的完美工具——当它们成为强大的[持续集成/持续部署管道](https://www.f5.com/solutions/automate-dev-ops-and-secops-deployments-with-ci-cd-pipeline-integrations)的一部分时，其功能极其强大(这是开发人员的梦想！)

# 2.**工作负载标识的 SPIFFE**

信任的一个重要方面是身份，随之而来的是了解他们所说的东西(身份验证)。 [SPIFFE](https://spiffe.io/) ( **安全生产*面向所有人的身份*框架**)是一个开放标准，它试图为节点、容器或其他任何运行数据工作负载的东西赋予身份。最终目标是，每次您的数据跨您的集群移动到另一个云或其他方向时，都会断言身份(有时我们称这些方向为信任壁垒)。这听起来可能真的是理论上的胡言乱语(比如为什么我的电脑需要一个身份？)，所以让我把它归结为:**当我们知道两个事物的身份时，我们可以颁发证书并创建加密的 TLS 连接**。TLS 连接在传输过程中为我们提供了加密，并帮助我们安全地跨越信任壁垒。

SPIFFE 实际上是 [Istio's mutual TLS](https://istio.io/latest/docs/concepts/security/) 背后的魔法，它有一个美好的未来([和一个伟大的社区](https://spiffe.io/community/)！).从技术层面来看，Istio 的 Citadel(现在是`istiod`)在每个 worker 节点上创建和管理证书和密钥。因此，当 pod 通过 sidecar 网络代理(Istio 中的 Envoy)相互通信时，连接是加密的和可信的。我们在这里真正获得的是跨我们的 pod 的零信任——一个网络请求可能看起来好像来自一个已知的位置，但 mutual TLS 告诉我们它*实际上*是一个已知的位置，并且还阻止窃听者监听通信。Istio 和其他服务网状网通常提供现成的相互 TLS，但是这会增加运营开销。在接下来的几年里，我认为这将变得越来越容易。

# 摘要

总的来说，我发现零信任是安全中的一个概念，是实现最小特权原则旧模式的一种新方法。网络策略和 SPIFFE 是通过实际实施进入零信任思维和架构的好方法。很容易假设在您的集群中运行的 pods 没有受到危害，但是由于安全性完全是关于深度防御的，这是两个增加深度的好方法。感谢你的阅读，如果你喜欢阅读这篇文章，请在 Medium 上关注我！