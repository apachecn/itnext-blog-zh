# 金丝雀部署:太可爱了！

> 原文：<https://itnext.io/canary-deployments-so-much-to-love-1fcf0257ac74?source=collection_archive---------5----------------------->

![](img/62e2deeea301386634cab8aad235a470.png)

艾萨克·本赫瑟德在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

Canary 部署是一个很好的例子，说明了 Istio 和 App Mesh 等服务网格在微服务环境中实现的自动化和规模。服务网格在安全性、可观察性和流量转移方面有很多好处，所有这些都有助于更轻松地进行大规模部署。南北向(应用集群内外)和东西向(集群内的服务对服务)的易操作性，尤其是微服务架构。微服务架构是关于如何组织您的代码、人员和流程以提高速度。将您的代码快速投入生产，并从最终用户那里获得快速反馈。然而，您不希望牺牲良好的用户体验来换取这样的速度，这就是在您的部署中自动化金丝雀模式可以提供帮助的地方。无论您是已经完全投入到服务网格部署中，还是只需要足够的功能就可以开始部署，都有几种选择可供您选择。

高级 Canary 部署模式是，当将应用程序变更部署到生产中时，您慢慢地将一小部分请求转移到变更中，并验证它是否按预期处理这些请求。如果一切顺利，您可以不断增加请求流量的百分比，直到您的版本更改处理所有请求，此时您可以关闭该代码的先前版本。金丝雀部署模式帮助您控制生产变化的“爆炸半径”影响。为您的所有服务部署自动化 Canary 部署模式(理想情况下作为可配置的策略),使您的应用程序团队能够快速将增强功能投入生产，同时降低中断业务或提供糟糕客户体验的风险。

三个功能帮助团队自动化金丝雀模式部署:部署控制器、服务健康可观察性和请求流量转移。Kubernetes 提供了一个运行时控制器模式，它可以监视一个声明性的定制资源来启用一个金丝雀部署策略。 [Prometheus](https://prometheus.io/) 是一个很好的选择，用于捕获健康指标，帮助您监控您的版本变更是否按预期执行，以便您可以决定是否将更高比例的请求转移给它，或者由于太多错误而回滚。 [Envoy](https://www.envoyproxy.io/) 提供了一个出色的数据平面，既提供了流量转移，例如，10%的请求服务 v1，90%的请求服务 v2。Envoy 还提供了许多关于与服务的请求和响应交互的指标，这些指标可以输入 Prometheus 来帮助检查 Canary 服务的运行状况。

部署在 Kubernetes 上的服务网格当然会为您提供所有这些功能，甚至更多，尽管这不是唯一的选择。越来越多的基于 Envoy 的入口/网关为您提供自动化 Canary 部署所需的功能，而没有像 Istio 这样的全服务网格安装的所有额外复杂性/优势。如果您的 Kubernetes 部署规模较小或资源有限，比如只部署了几个微服务，那么这个更有限的堆栈会很有帮助。您可能还会发现使用这个较小的堆栈更容易开始/试验。

让我们把这个说得更具体些。Weaveworks 支持一个开源技术调用 [Flagger](https://github.com/weaveworks/flagger) ,通过让您在 Kubernetes 中为每个部署的 Kubernetes 服务定义一个金丝雀自定义资源(CRD ),支持自动金丝雀部署。Flagger 在服务网格上分层，以处理流量转移和指标收集，作为实现 Canary 部署的一部分。我以前在《[超级格洛拯救世界》中写过。](https://medium.com/solo-io/supergloo-to-the-rescue-making-it-easier-to-write-extensions-for-service-meshes-50313a401692)”以及在 [SuperGloo](https://supergloo.solo.io/) 上部署 Flagger 的完整工作代码示例。SuperGloo 是一个多服务网格编制器/管理器，可以安装和管理一个或多个服务网格，如 Istio、App Mesh、Linkerd 等，SuperGloo 帮助激发了微软在 2019 年 5 月在巴塞罗那 KubeCon 上宣布的 [SMI 互操作性标准](https://smi-spec.io/)。Flagger 最近增加了对 Ingress' like [Gloo](https://gloo.solo.io/) 的支持，这为那些确实需要服务网格的所有额外功能(和复杂性)的人提供了一个更轻量级的金丝雀部署选项。

Gloo 不是一个服务网格。这是一个基于特使的网关，是管理南北入口流量的绝佳选择。如果您需要的只是流量转移，它可以提供与 Istio 相同的流量转移功能，而没有额外的复杂性。例如，Knative Serving 项目添加了 [Gloo 作为 Istio](/knative-and-solo-io-gloo-2a877d456238) 的替代方案，原因完全相同——Knative Serving 只需要流量转移功能，就可以将其规模缩小到零功能。最近 [Jenkins X](https://jenkins-x.io) 项目[将 Knative Serving](https://jenkins-x.io/developing/knative/) 功能添加到 CI/CD 功能组合中，他们选择将 Knative Service 与 Gloo 结合使用，以保持其强大平台的轻便和敏捷。

使用 Gloo 安装和运行 Flagger 的说明与我为使用 SuperGloo 安装/管理 Istio(或 App Mesh)的 Flagger 提供的说明非常相似。区别在于安装 Gloo 和 SuperGloo，并配置 Flagger 使用 Gloo 作为它的“meshProvider”。完整的安装和运行脚本可以在[这里](https://docs.flagger.app/usage/gloo-progressive-delivery)找到。

当您继续发展您的微服务架构时，使用金丝雀模式自动化您的部署是获得速度和可控风险的好方法。像 Flagger、SuperGloo 和 Gloo 这样的开源项目有助于更容易地使用和消费足够的新技术，以获得立竿见影的效果，同时还可以在准备好的时候轻松采用更先进的技术。