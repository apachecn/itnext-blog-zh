# 为什么 Kubernetes 会赢？

> 原文：<https://itnext.io/why-kubernetes-is-winning-864c113eaf81?source=collection_archive---------3----------------------->

![](img/5ab1e69707a133e235d16193e731f4d0.png)

当我还在一份固定的工作中全职工作的时候，我就开始和木偶一起工作了。

Puppet 远非理想，但仍然是一个非常强大的工具——这是我第一次介绍“基础设施即代码”的想法。

这些年来，我越来越精通它的显式语法，并且可以做许多令人兴奋的事情，至少从我的角度来看是这样。

我对 Puppet 的语法和我可以用它做的事情很满意，然而回想起来，它的语法的多功能性并不是它流行的唯一特征。围绕它的生态系统——有许多你可以下载和重用的小型开源模块——是它成功的主要部分。

几年后，我对 Kubernetes 和它的开源工具生态系统(从现在开始我们称它们为插件)也有同样的感觉。

# 所以你有一个 Kubernetes 集群，现在呢？

在 [Opsfleet](https://opsfleet.com) ，我们帮助公司迁移到 Kubernetes。每次 Kubernetes 迁移都有或多或少相同的阶段——在所有 Docker 映像准备就绪后，我们通常会启动一个新的 Kubernetes 集群，并开始处理部署模板。

一旦我们有了一些这样的迁移，我意识到“香草”Kubernetes 不能满足当今公司的期望。

以下是一些我无法用简单的“是”来回答的客户问题！

“Kubernetes 支持自动缩放吗？我能否根据我的总体邮件处理率来调整我的员工？”

“金丝雀释放怎么样？Kubernetes 支持吗？”

“出于合规性原因，我们需要启用端到端加密。Kubernetes 能帮忙吗？”

“我们希望每个开发分支都有一个新的环境，Kubernetes 可以根据动态创建的入口资源分配公共 DNS 记录吗？”

所有这些问题的答案是**在 Kubernetes 中实现它们是可能的，但不是现成的**。

Kubernetes 提供了非常坚实的基础，这是真的！但是它的客户想要和需要的不仅仅是满足他们的开发和生产环境的需求。

# 插件生态系统

对于其他 Docker 调度平台，比如 ECS，你别无选择，只能卷起袖子自己编码。我已经做过几次了。使用 AWS 的 API 总是“一种乐趣”。

另一方面，Kubernetes 有一个更大的开源插件生态系统可以安装。**最难的部分是找到适合这项工作的工具**。

这里有一个流行插件的例子，其中一些是我们经常使用的。让我们来看看不属于 Kubernetes 核心的社区工具可以释放多少功能:

## 1.外部-dns +证书管理器

与此同时，至少从我的记忆中，Herkou、T2、GitLab 和 T4 开始为每个开发分支提供一个预览环境。

基本思路是，你不需要在房间里(或者懈怠)大喊:“谁用分期？我现在就拿。”相反，对于您推送到您的源代码控制系统的每一个新分支，都会自动为您创建一个新的环境。

Kubernetes 拥有我们可能需要动态创建的几乎所有东西，并在不再需要时销毁，一个独立的环境:名称空间、部署自动化、入口等等。

为了在动态环境场景中正确部署 web 应用程序，我们通常需要为环境中的每个前端 web 组件分配一个公共 DNS。

例如，如果我刚刚推出了一个修复小部件分支，并且为我创建了一个新环境来测试结果，我希望能够浏览到 fix-widget-app.company.com 来与应用程序进行交互。

最好我还想获得一个分配给我的动态端点的 SSL 证书。如果它不是自签名证书，我可以跳过接受恼人的浏览器警告——甚至更好！

这两个功能都不是 Kubernetes 的一部分，但是可以通过安装 [external-](https://github.com/kubernetes-incubator/external-dns) dns 和 [cert-manager](https://github.com/jetstack/cert-manager) 插件来相对快速地添加。我只需要用一些额外的注释来调整我的部署清单，我们就可以开始了。

## 2.Nginx/ALB 入口控制器

当你在 GKE 上运行一个新的 Kubernetes 集群时，它会预装一些有用的“插件”。这些内置的额外功能之一是与谷歌的 L7 负载平衡器紧密集成。我只需要指定一个“公共”服务或入口控制器，Kubernetes 就会自动为我提供一个新的负载平衡器。

然而，有时我不想提供一个新的云负载平衡器，或者我可能只是需要一个 Google 的负载平衡器目前不支持的功能。

对于这些情况，Kubernetes 支持各种带有插件的负载平衡器——大多数时候它是入口控制器的开源实现，例如在 [Nginx](https://github.com/kubernetes/ingress-nginx) 和 [ALB](https://github.com/coreos/alb-ingress-controller) 示例中。

我认为将所有特定于云的功能提取到单独的可安装模块中，而不是将代码添加到 Kubernetes 的核心中，这是一个不错的决定。这样，我可以在一个 Kubernetes 集群中使用多个入口控制器，并且能够从 Kubernetes 的主要组件中单独更新它们。

## 3.聚类自动缩放器

Kubernetes 中的自动缩放具有误导性！

当您阅读文档时，您会觉得自动增加或减少应用程序副本的数量就像添加一个简单的 HorizontalPodAutoscaler 资源一样简单。

然而，为了让新扩展的容器实际运行，Kubernetes 集群调度器应该能够将它们放入一个或多个可用节点中。

那么，如果因为所有节点都在努力工作而没有空闲点，会发生什么呢？(或者就当他们是吧！)你的 pod 将会以‘待定’状态挂起，直到某个不幸的 pod 死去并释放出它的空间。

为了让自动伸缩在 Kubernetes 中真正发挥作用，您需要考虑两次:一次在容器/pod 级别，一次在集群/节点级别。

HorizontalPodAutoscaler 资源负责容器/pod 级别，并将基于 CPU 或内存添加或删除容器副本。

然而，Cluster-autoscaler 插件负责集群/节点级别，并根据等待空闲点和即将被调度的 pod 的数量向集群添加节点或从集群中删除节点。

这是最关键的插件之一，通常需要配置来与将在集群上运行的应用程序保持一致。可自动缩放的可执行文件有超过 100 个配置开关的事实并没有使工作变得更容易！

## 4.舵

大多数 Kubernetes 用户会将他们的应用程序部署到多个环境中。几乎可以肯定的是，在向用户推广新版本之前，您至少会有某种预生产环境来进行测试。

然而，Kubernetes CLI 只允许您应用静态清单。如果您想要更改一些参数，比如您托管的 MongoDB 数据库的端点，那么您需要将您的清单文件转换成模板，并编写脚本和自动化来填充这些值。

当我第一次发现 Kubernetes 时，Kubernetes 的 CLI 中缺乏模板支持实际上让我很困惑——为什么这是一个如此常见的场景，但 Kubernetes 的 CLI 不支持模板语言？我甚至在 Github 上发现了一些公开的[讨论，但似乎没有任何进展。](https://github.com/kubernetes/kubernetes/pull/18215)

在运行 bash 和 sed 以及其他一些工具几次迭代之后，我偶然发现了[头盔](https://github.com/kubernetes/helm)。

Helm 不仅仅是一种模板语言，但对我来说，它的主要特点是允许我在一个地方指定我想用 Kubernetes 管理的所有服务的模板——一个 Helm 图表。

有时，我还会创建一个更高级别的组装图，将所有微服务捆绑在一个版本中。在其他情况下，只需要一个简单的图表回购结构就可以了。

## 5.荣誉奖:Istio

有一些工具，每个人都在谈论，但我仍然没有机会认真尝试。 [Istio](https://github.com/istio/istio) 就是其中之一。

我们合作的公司还没有经历过 Istio 承诺要解决的问题。部分原因是，在 Kubernetes 迁移的早期阶段，他们仍然专注于确定“基础”。

无论如何，我决定提及 Istio 的原因是，在一些用例中，我确实看到人们开始认真看待 Istio。

其中一个使用案例是端到端加密，尤其是当法规遵从性对于许多公司来说是一个非常重要的优先事项时。有了 Istio，基础设施可以保证 Kubernetes 集群中服务之间的每次通信都将被加密。

我经常被问到的另一个特性是金丝雀发布——从你的新版本应用中获取一小部分实时流量的能力:金丝雀发布。Istio 承诺将该功能添加到 Kubernetes 中。

# 为什么是 Kubernetes？

根据我的经验，公司迁移到 Kubernetes 的最常见原因是让开发人员能够部署和维护他们的代码，并使添加新服务以支持不断增长的微服务架构变得容易。

并非每个人都有同样的问题——有时是一个不完整的部署脚本或一个复杂的配置管理解决方案引发了关于 Kubernetes 的最初对话。然而，大多数时候，潜在的动机几乎是一样的。

今天，我和一个潜在客户打了一个很不寻常的电话；

当我们在谈话中谈到我对这个项目的动机挖掘得更深时，我问道:“你为什么决定开始迁移到 Kubernetes？看起来你的团队中已经有很多自动化工作做得很好了”

这一次，我得到了一个非常有趣的回复:“我们想迁移**，因为生态系统**。我们不想从头开始开发 Kubernetes 社区已经提供的所有功能。”

我花了几秒钟思考这个问题——这是迁移到 Kubernetes 的正当理由吗？

【opsfleet.com】最初发表于[](https://opsfleet.com/why-kubernetes-is-winning/)**。**