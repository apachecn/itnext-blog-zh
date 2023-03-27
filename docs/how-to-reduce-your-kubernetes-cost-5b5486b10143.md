# 如何降低你的 Kubernetes 成本

> 原文：<https://itnext.io/how-to-reduce-your-kubernetes-cost-5b5486b10143?source=collection_archive---------1----------------------->

降低在更大规模和更多用户中运行 Kubernetes 的成本将很快成为许多公司的首要任务。

![](img/26642adbca608222f2f1c7cddfd45d2f.png)

运行 Kubernetes 可能非常昂贵，尤其是在效率低下的情况下。当公司刚刚开始在他们的组织中推出 Kubernetes 时，通常会出现这种情况，因为相同的配置和设置通常用于初始测试项目或小型应用程序。这样的初始未优化设置可能是正常的。然而，公司应该很快开始考虑成本，因为这可以节省很多钱，并防止不良行为的蔓延。

在这篇文章中，我将描述一些控制和降低 Kubernetes 成本的方法，这些方法可以应用于非常不同的 Kubernetes 用例，从开发和 CI/CD 到生产。

# 成本监控

有效降低 Kubernetes 成本的第一步是监控这些成本。这不仅能帮助你确定你最终节省了多少，还能让你了解什么是最重要的成本驱动因素。

对您计算成本的基本监控由公共云提供商提供，因为他们是最终向您收费的人。除了总成本之外，您还可以在他们的计费概览中看到到底是什么在运行，是什么推动了您的成本，例如，计算、存储和网络流量在您的成本中占多大比例。

然而，对云提供商的概述只能让您有一个基本的了解，对多租户 Kubernetes 集群的帮助有限，当然在私有云中是不可用的。因此，使用额外的工具来衡量 Kubernetes 的使用和成本通常是有意义的。这方面一些有用的工具有[普罗米修斯](https://github.com/prometheus/prometheus)、[库贝科斯特](https://kubecost.com/)和[雷普利克斯](https://www.replex.io/)。

一旦您建立了 Kubernetes 成本监控并确定了需要改进的地方，您就可以开始实际的成本优化了。

# 资源限制

节约成本的第一步是限制资源的使用。有效的资源限制确保 Kubernetes 系统的任何应用程序或用户都不会消耗过多的计算资源。因此，它们可以防止你遭遇不愉快的意外，比如成本突然增加。

特别是如果你让开发者直接访问 Kubernetes，例如以[自助式 Kubernetes 平台](https://loft.sh/blog/building-an-internal-kubernetes-platform/)的形式，限制是至关重要的，因为它们强制公平地共享可用资源，从而保持集群的总规模较小。如果没有限制，一个用户可能会消耗所有资源，使其他人无法工作，这将使他们总共需要更多的计算资源。

这不仅适用于 [Kubernetes 多租户](https://loft.sh/blog/kubernetes-multi-tenancy-best-practices-guide/)环境中的工程师，也适用于在同一个集群上运行的应用程序。在任何情况下，设置正确的限制当然是重要的:如果它们太低，工程师和应用程序不能正常工作，如果它们太高，它们大多是无用的。这里，前面提到的 Kubernetes 监控工具可以帮助您确定不同用例的正确限制。

为了实现资源限制，您可以使用[资源配额](https://kubernetes.io/docs/concepts/policy/resource-quotas/)和[限制范围](https://kubernetes.io/docs/concepts/policy/limit-range/)来配置它们。

# 自动缩放

既然您已经实施了成本监控并防止了意外的成本爆炸，您就可以专注于节约成本了。最好的方法是只为你真正需要的东西付费。为此，获取集群的大小、[虚拟集群(vClusters)](https://loft.sh/blog/introduction-into-virtual-clusters-in-kubernetes/) 、名称空间等非常重要。没错。

同样，监控工具对此很有帮助，但是手动观察和调整计算资源不是一个非常快速和灵活的解决方案。为了能够应对短期波动，您应该启用 Kubernetes 自动缩放。

这里，有两种类型的自动缩放:水平和垂直自动缩放。简而言之，水平自动缩放意味着如果负载高于或低于预定义的阈值，则添加和移除 pod。使用垂直自动缩放，可以调整各个单元的大小。

这两种类型的自动缩放都有助于根据实际需要自动调整可用的计算资源。然而，这一过程并不总是最佳的，因为它可能过于“保守”或不适用于所有用例，例如，可能有一些正在运行的东西(需要计算资源，因此不会自动缩减)不再被使用。因此，应该实施一些其他措施来进一步降低您的 Kubernetes 成本。

# 打折的计算资源

另一种节省计算成本的方法是使用打折的资源。当然，这只适用于提供这种资源的公共云中。然而，所有主要的云提供商对此都有一些选择:

[AWS Spot 实例](https://aws.amazon.com/ec2/spot/)、 [GCP 可抢占虚拟机](https://cloud.google.com/compute/docs/instances/preemptible)和 [Azure Spot 虚拟机](https://azure.microsoft.com/pricing/spot/)提供了大幅打折的计算资源，这些资源是云提供商的“过剩容量”。提供商以更低的价格出售这种计算能力，否则它将无法使用。然而，这也意味着这些计算资源的价格会根据需求而波动，并且如果价格超过了您的限制或者没有可用的容量，您的实例和应用程序可能会停止。为此，spot 实例通常是不经常需要的应用程序和工作负载的一个选项，例如，如果您想要运行一个计算密集型实验。然而，如果你能使用这种类型的折扣计算，你可以节省大量成本——AWS 声称你可能会获得“高达 90%的折扣”。

如果 spot 实例和可抢占的虚拟机不适合您的应用程序，例如，因为您总是需要无中断地运行它，您还可以在特定时间段内永久使用资源。通过承诺一年或三年的使用期，你可以获得很大的折扣——根据 AWS 的说法是 40%-60%。这样的长期折扣对于 [AWS](https://aws.amazon.com/ec2/pricing/reserved-instances/?nc1=h_ls) 和 [Azure 是“保留实例”](https://azure.microsoft.com/pricing/reserved-vm-instances/)，对于 [Google 是“承诺使用折扣”](https://cloud.google.com/docs/cuds)。如果您在较长时间内对计算资源有可预测的持续需求，这种折扣是有意义的。

总体而言，这取决于您的应用类型和计算需求，以及适用的折扣类型。在这里，也可以将这两种类型用于应用程序的不同部分或不同的用例。因此，即使所有公共云提供商的定价都非常复杂，但仔细看看那里的可用折扣肯定会有所回报，这样您就可以以更低的价格获得相同的计算资源。

# 休眠方式

在很多情况下，集群、vClusters 和命名空间会继续运行并产生成本，尽管不再需要它们。工程师在开发、测试或 CI/CD 期间使用 Kubernetes 时，这些用例经常(但不仅仅是)发生。

例如，如果一个开发人员正在云中使用一个 [Kubernetes 开发环境](https://loft.sh/blog/kubernetes-development-environments-comparison/)，他们只在工作时间需要这个环境。如果假设他们每周在 Kubernetes 环境下工作 40 个小时(不考虑会议或假期)，并且该环境一直在运行，那么通过在不需要时关闭该环境，您可以节省 75%以上的时间(一周有 168 个小时)。

开发人员当然可以在完成后自行关闭他们的环境，但这是一个很容易被遗忘或忽略的过程。因此，使用“[睡眠模式](https://loft.sh/docs/self-service/sleep-mode)将其自动化是有意义的，也就是说，一个自动化的过程可以缩减未使用的名称空间和虚拟集群。这确保了环境的状态被存储，并且当再次需要时，环境可以非常快速和自动地再次“唤醒”,以便工程师的工作流不被中断。

可以通过脚本或者使用内置睡眠模式的工具来实现这种睡眠模式，比如 [Loft](https://loft.sh) 。

> *关于睡眠模式的成本节约潜力的更详细的描述和计算，你应该看看我关于* [*为工程师节约 Kubernetes 成本*](https://loft.sh/blog/how-to-save-more-than-2-3-of-engineers-kubernetes-cost#inefficiency-2-unused-resources) *的文章。*

# 清除

与睡眠模式相关，为了减少暂时不需要的计算资源，您还应该不时地清理您的 Kubernetes 系统。特别是，如果您允许工程师[按需创建名称空间](https://loft.sh/blog/self-service-kubernetes-namespaces-are-a-game-changer/)或者如果您将 Kubernetes 用于 CI/CD，您可能会有许多未使用的名称空间，甚至仍然会花费您的钱的集群。

即使您有一个可以缩减计算资源的睡眠模式，睡眠模式也只适用于暂时未使用的计算资源，因此可以保留状态，包括存储和配置。但是，特别是为了运行 CI/CD 管道或测试，通常没有必要保留 Kubernetes 环境的状态，所以最好也删除它。

同样，这可以通过单独的脚本或使用 Loft 来完成，Loft 提供了开箱即用的[自动删除功能。作为一个额外的积极副作用，删除一些环境使管理员更容易监督系统，这也可以被视为一个间接的成本节约因素。](https://loft.sh/docs/self-service/sleep-mode)

# 集群共享

Kubernetes 节约成本的最后一个方法是减少集群的数量。这通常可以节省成本，因为在 [Kubernetes 多租户设置](https://loft.sh/blog/kubernetes-multi-tenancy-best-practices-guide/)中，控制平面和计算资源可以由多个用户或应用共享。如果使用收取集群管理费的公有云，比如 AWS 或者 Google Cloud，也可以省下这笔费用。最后，管理许多集群是一项复杂的任务，因此限制集群的数量也减轻了管理员的负担。

特别是在前期生产阶段，许多公司有太多的集群，有时甚至是针对每个单独的开发人员。这显然是低效的，因为在开发过程中很少需要整个集群，即使开发人员想要试验集群范围的配置，他们也可以使用[虚拟集群作为开发环境](https://loft.sh/blog/kubernetes-virtual-clusters-as-development-environments/)。因此，对于所有非生产用例，依靠一个多租户[内部 Kubernetes 平台](https://loft.sh/blog/building-an-internal-kubernetes-platform/)可以通过消除不必要的冗余来节省大量成本。

然而，同样在生产过程中，减少集群的数量对于提高资源利用率、减少冗余和避免集群管理费用的基本思想也是有意义的。当然，这并不意味着您应该只有一个集群。您应该更全面地评估哪些用户组和应用程序可以共享一个集群，以及专用集群何时有意义。

> *关于这个话题的更详细的讨论，也可以阅读我的关于* [*Kubernetes 通过减少集群数量节约成本*](https://medium.com/faun/kubernetes-cost-savings-by-reducing-the-number-of-clusters-336378680995) *的帖子。*

# 结论

我相信，在更大的规模上运行 Kubernetes 并拥有许多用户的成本将很快成为越来越多公司的一个问题，因为一些最初的低效率可以相对容易地消除，但仍然可以导致巨大的成本节约。

管理 Kubernetes 成本的第一步是获得一个概览并开始监控成本。然后，您应该实施限制以防止过度的计算资源消耗，这使得您的成本更可预测。为了降低成本，找到集群、虚拟集群和名称空间的正确大小是非常重要的，而自动缩放对此非常有帮助。如果公共云提供商是您的选择，您还应该看看他们的折扣选项。自动休眠模式和清理未使用的 Kubernetes 名称空间和 vClusters 是消除空闲资源的进一步措施。最后，您应该考虑减少集群的数量，因为这可以提高资源利用率并减少不必要的冗余，尤其是在生产前阶段。

如果你实施了所有这些措施，你的 Kubernetes 系统是非常成本优化的，因此比任何初始的最佳猜测设置都要便宜得多。然后，您应该准备好应对使用 Kubernetes 的下一个挑战，而不必担心其成本。

亚历山大·密尔斯在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

*最初发布于*[*https://loft . sh*](https://loft.sh/blog/reduce-kubernetes-cost/)*。*