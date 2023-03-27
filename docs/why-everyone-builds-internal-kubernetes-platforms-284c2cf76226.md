# 为什么每个人都建立内部 Kubernetes 平台

> 原文：<https://itnext.io/why-everyone-builds-internal-kubernetes-platforms-284c2cf76226?source=collection_archive---------1----------------------->

## 几个因素促使越来越多的公司为他们的工程师开发和采用内部 Kubernetes 平台。

![](img/0de7010096385e4e2d8a18c4bac2764c.png)

最近，你可能从许多不同的来源听说过“Kubernetes 内部平台”:KubeCon 讲座、博客文章，或者只是同事和朋友。即使这种平台并不总是被称为内部 Kubernetes 平台，但允许工程师在云环境中获得标准化和轻松的 Kubernetes 访问的解决方案现在似乎变得更加常见。

在这篇文章中，我将描述是什么驱使公司建立和采用这样的平台。为此，我将依靠三个公开描述的热门科技公司的例子， [Eventbrite](https://dev.to/ethanjjackson/why-eventbrite-runs-a-700-node-kube-cluster-just-for-development-2l9p) 、 [Spotify](https://www.youtube.com/watch?v=vLrxOhZ6Wrg) 和 [Datadog](https://youtu.be/4YanEWCAPlk?t=544) ，并分享我自己在开发名为 Loft 的开箱即用内部 Kubernetes 平台解决方案时的一些经验

# 内部 Kubernetes 平台

我已经在另一篇文章中定义了什么是内部 Kubernetes 平台。

简而言之，内部 Kubernetes 平台为工程师提供了一个直接访问 Kubernetes 开发环境的途径，他们可以在预生产阶段使用这个环境。这种平台通常允许工程师创建[自助服务名称空间](https://loft.sh/blog/self-service-kubernetes-namespaces-are-a-game-changer)，但是具有整个集群或[虚拟集群(vClusters)](https://loft.sh/blog/introduction-into-virtual-clusters-in-kubernetes/) 的解决方案也是可能的。

尽管术语“平台”可能意味着它只是一个基础设施组件，但内部 Kubernetes 平台通常也包含一些工具，这些工具使开发人员更容易使用和管理平台和一般的 [Kubernetes 开发工作流](https://loft.sh/blog/kubernetes-development-workflow-3-critical-steps/)，尤其是部署工作流，即使他们以前没有 Kubernetes 的经验。

因此，如果您听说过内部 Kubernetes 平台，您也会经常听说开发人员体验和相关工具，而不仅仅是技术基础设施决策。

# 是什么驱使公司建立这样的平台

因为它涵盖了很多方面，所以为工程师们构建一个内部的 Kubernetes 平台可能会很有挑战性。例如，Spotify 花了 2 年时间才使其内部平台普遍可用。考虑到为大型团队运行平台的大量工程工作和可观的云计算成本，问题就来了，为什么公司要进行这样的投资。这样做显然有很大的价值甚至压力:

## 1.对于本地环境来说，应用程序变得太大

因此，让我们通过查看 Eventbrite 的案例来探索内部 Kubernetes 平台的潜在原因:[在一次采访](https://dev.to/ethanjjackson/q-a-how-eventbrite-prioritizes-developer-productivity-33m0)中，Eventbrite 的 DevTools 团队的首席工程师 Remy DeWolf 解释了他们为什么从[本地开发转移到基于云的 Kubernetes 开发](https://medium.com/swlh/local-cluster-vs-remote-cluster-for-kubernetes-based-development-6efe2d9be202)的原因。像许多公司一样，Eventbrite 也是从建造一个整体开始的。随着时间的推移，这个整体变得越来越大，这就是为什么它被部分转化为微服务，并且新的服务也被添加为微服务。

因为所有这些都是在容器中运行的(至少对于开发来说是这样)，所以这在一开始不是问题。然而，随着软件的进一步发展，你最终会发现它太大了，以至于无法在使用本地 Kubernetes 解决方案(如 [Minikube](https://github.com/kubernetes/minikube) 或 [Docker Desktop](https://www.docker.com/products/docker-desktop) )的本地计算机上运行。

此时，您可以开始开发解决方案，例如，只运行应用程序的一些必要部分，或者本地运行一些部分，远程运行其他部分，以保留您现有的本地开发工作流。这可能是一个合适的方法，但由于大多数公司和系统都在不断发展，随着时间的推移，继续进行本地开发会变得更加困难。这将导致开发人员的生产力降低，因为开发人员越来越难以适应本地设置和运行缓慢的笔记本电脑。

为此，您面临着投资内部平台(和云资源的持续成本)和浪费工程时间之间的权衡。在这里，很清楚为什么许多公司从本地 Kubernetes 解决方案开始，但越来越多的公司现在达到了“临界点”,即值得迁移到基于云的 Kubernetes 环境。

因此，在 Eventbrite，他们也决定采用云环境，并构建了一个名为“yak”的内部工具，允许工程师部署和管理远程容器。因此，现在每个工程师在一个共享的 Kubernetes 集群中都有自己的名称空间，并且可以在其中轻松运行大约 50 个容器。

这样的演变不仅仅发生在 Eventbrite，拥有大约 800 名开发人员的 Datadog 似乎也做出了类似的决定。我也可以用我们 Loft 客户的经验来支持这一点:在某种程度上，他们的应用程序变得过于复杂，无法在本地运行，因此迁移到基于云的 Kubernetes 环境是必要的一步。有趣的是，这不仅适用于大型组织，如果他们开发一般资源密集型或复杂的应用程序，如 AI/ML 软件，使用较少的工程师也会出现这种情况。

**在许多情况下，应用程序在某些时候变得太大而无法在本地运行。然后，从本地开发环境转移到云中的 Kubernetes 开发环境通常是有意义的。**

## 2.将自主和专注结合起来

在他的 [KubeCon 演讲](https://www.youtube.com/watch?v=vLrxOhZ6Wrg)中，Spotify 的高级 SRE 温升豪没有明确分享他们为什么要建立一个内部的 Kubernetes 平台。然而，人们仍然可以获得有价值的见解，这可能是这个决定的驱动因素:

虽然他没有提到 Spotify 的工程师不可能进行本地开发，但考虑到他们有非常多的团队(280 多个)、工程师(1500 多个)和微服务(1200 多个)，并且他们之前已经有了非 Kubernetes 云解决方案，人们可以假设这一点。

不过，我认为温升豪提到了一些更有趣的方面:Spotify 采用了“团队运营”模式，这是团队完全自治和完全集中运营的诱人组合。这些团队独立地构建和操作他们自己的服务，但是他们可以依赖于由集中运营团队提供的工具和平台。

我认为这是一个聪明的解决方案，因为它为团队提供了自主工作和按需配置 Kubernetes 的选项(他们甚至可以按照自己希望的速度采用它)，但他们不必亲自关心和管理一切，而是使用一个公共平台来解决所有“标准”问题，否则这些问题将被重复解决。

从管理员(DevTool 团队)的角度来看，这种解决方案也很有吸引力，因为管理员不必单独与所有开发环境交互，这可能是一个大问题，例如，如果他们必须手动供应所有环境。相反，管理员可以专注于提供集群和出色的平台体验，并支持开发人员使用它，例如通过解决问题，这也更容易，因为可以在云中共享开发环境。

有了内部的 Kubernetes 平台，开发人员可以自由自主地工作，但同时不必关心可以更好地集中解决的运营任务。管理团队从中受益，因为他们可以专注于平台并支持其他工程师，但不必关心每个单独的环境。

## 3.简化 Kubernetes 工作流程

当开发人员开始使用 Kubernetes 时，可能会非常具有挑战性。使用内部平台可以使过渡到 Kubernetes 更加容易，因为一些任务可以简化，开发人员不必自己设置 Kubernetes。

例如，Spotify 开发了一个全面的教程和文档，使新开发人员能够从第一天开始轻松地使用内部平台和 Kubernetes。在 Loft，我们为开发人员(我们客户的内部 Kubernetes 平台的用户)提供一份[入职指南](https://loft.sh/docs/guides/onboarding)，向开发人员解释基本概念和所有标准任务。

这是可能的，因为工作环境总是以相同的标准化方式创建，由于不同的硬件、操作系统、系统配置等，这在本地计算机上是不可能的..这是 Eventbrite 也意识到的一个优势:他们能够减少“平均恢复时间”，即恢复干净状态的时间，如果开发人员陷入困境或犯了错误，这是必要的。因此，一个内部的 Kubernetes 平台也降低了 Kubernetes 的实验风险，使其对开发者更具吸引力。

总的来说，正如最近的[栈溢出开发者调查](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-platforms-wanted5)所发现的，许多开发者对 Kubernetes 感兴趣，并希望学习和使用它。有了内部平台，他们可以很容易地接触到 Kubernetes，然后一步步发展自己的技能。

在这里，附加的 Kubernetes 工具发挥了重要作用。显然，Spotify、Eventbrite 和 Datadog 已经开发了某种工具来使工程师们更容易使用 Kubernetes 的工作流程。至少 Spotify 的“tuggle”和 Eventbrite 的“yak”不仅作为平台管理工具，还支持部署过程，从而使开发人员对 Kubernetes 有更好的体验。在 Loft，我们也看到这对我们的客户至关重要，这就是为什么我们为 [DevSpace](https://github.com/devspace-cloud/devspace) 开发了一个直接的 [Loft 集成](https://github.com/loft-sh/loft-devspace-plugin)，这样工程师只需要一个工具就可以启动他们的工作环境，在其中开发，并部署到其中。

**内部 Kubernetes 平台也包含一些开发工具，有助于标准化 Kubernetes 工作流程，因此即使没有 Kubernetes 经验的工程师也可以使用它。这促进了平台的采用过程，并支持开发人员更多地了解 Kubernetes。**

## 4.节约成本

采用内部 Kubernetes 平台的另一个驱动因素是成本节约。一般来说，[共享集群更便宜](https://medium.com/faun/kubernetes-cost-savings-by-reducing-the-number-of-clusters-336378680995)，自助式命名空间平台允许这种集群共享。成本的节省是基于计算资源的更有效的自动扩展，以及资源和一些中心 Kubernetes 组件(如 API 服务器)的公共共享。因此，与其他基于云的环境相比，内部平台可以节省成本，例如开发人员/团队的单个集群。

除了这些直接的成本节约之外，当开发人员不再需要关心他们的 Kubernetes 环境，并且他们可以使用优化的工作流时，由于生产力的提高，您还可以节约间接的成本。因此，前面提到的内部平台的原因最终都有助于[降低 Kubernetes 的成本](https://loft.sh/blog/reduce-kubernetes-cost/)。

同样，我可以用我们客户的体验来支持这一点，这实现了显著的成本节约(也是由于成本节约功能，例如可以在通用平台中实施的[睡眠模式](https://loft.sh/docs/self-service/sleep-mode))。然而，正如温升豪在他的演讲中提到的，成本也是工程团队转向 Spotify 内部服务的一个重要因素。

内部 Kubernetes 平台是为工程师提供 Kubernetes 环境的极具成本效益的解决方案，因为计算资源可以得到充分利用，工程师可以更高效地工作。

# 提供内部 Kubernetes 平台的重要因素

现在，您可能想要开始建立自己的内部 Kubernetes 平台。我已经写了一篇关于如何建立内部 Kubernetes 平台的[指南](https://loft.sh/blog/building-an-internal-kubernetes-platform/)，你当然应该看看温升豪的演讲，了解一下 Spotify 的经验。

但是，在您开始之前，我想在此强调一些对您的平台的成功具有决定性的重要因素:

**提供出色的开发者体验:**即使你的平台是供内部使用的，你也有“客户”，他们是你组织中的开发者。为了确保他们会采纳并喜欢你的解决方案，你需要为他们提供一个良好的开发体验。在最好的情况下，这意味着他们可以只用一个工具做任何事情，包括创建环境，用它开发，最后部署到它。您还需要为开发人员准备大量的文档，这些文档的编写应该考虑到开发人员的体验(这对编写文档的管理员来说是一个挑战)。总的来说，正如温升豪所说，你的平台必须是开发者的“最佳选择”。

**衡量参与度，与用户沟通:**了解你的用户非常重要。因此，您必须不断地与开发人员交流，这通常很自然地发生，因为平台提供商支持开发人员使用它。此外，您还应该实施参与度措施，以查看您的平台是否被实际使用，并找出可能缺少的内容。

**衡量平台的成本和性能:**您还应该衡量您的平台是否工作可靠，并确保您可以快速修复问题。准备好备份和大量测试对此非常有帮助。在这里，你应该记住，即使这个平台“只”在内部使用，一旦每个人都使用它，它对你的整个工程部门的生产力变得至关重要。为了支持和理解平台的业务案例，衡量引入平台前后的成本也很重要。

**改善平台和开发体验:**从前面的因素可以看出，构建内部平台是一项持续的工作。因此，您必须不断更新和改进平台的稳定性和性能，以及开发人员的使用体验(也包括文档)。

**开发经验比技术细节更重要:**对于开发人员来说，最重要的是他们能够高效地使用平台。他们通常并不真正关心底层技术到底是如何工作的，或者在哪里运行。(Eventbrite 使用 EKS，从一个集群开始，现在运行几个集群，而 Spotify 依赖 GKE 上的许多集群，Datadog 只使用一个自我管理的集群。我相信这个选择对开发商来说并不是决定性的。)因此，提供平台的工程师团队应该以用户为中心。这意味着即使是看起来很小的改进也是值得投资的，这使得开发人员的生活更加轻松。

**购买可能比构建更便宜:**和大多数软件一样，自己构建并不总是最好的解决方案。相反，你可以简单地购买一个由专门团队持续开发的[内部 Kubernetes 平台解决方案，如 Loft](https://loft.sh/) ，这可能会使它在许多用例中更好、更便宜(尤其是没有非常特殊的需求)。即使您决定自己构建它，您也应该考虑集成现有的开源解决方案，例如 [kiosk](https://github.com/kiosk-sh/kiosk) 可以帮助您实现多租户，而像 [Skaffold](https://github.com/GoogleContainerTools/skaffold) 或 [DevSpace](https://github.com/devspace-cloud/devspace) 这样的开发工具可以用作开发和部署工具。

# 结论

内部的 Kubernetes 平台已经变得越来越普遍，即使它们并不总是这样叫。有了这样一个平台，开发人员可以直接访问云中的 Kubernetes 环境。

与 Kubernetes 一起工作并希望扩展其技术的软件团队应该预料到，当本地开发变得不可行或者至少效率低下时，他们会达到一个临界点。然后，迁移到云环境是有意义的，如果您的生产系统也运行在 Kubernetes 上，那么应该是 Kubernetes 环境。

考虑到构建这样一个平台所花费的时间和精力，他们应该尽早预见到他们可能需要一个内部开发平台，并开始不断地评估这种技术的引入，以便他们可以找到投资回报的正确时间点。

一旦做出了内部 Kubernetes 平台的决定，重点应该是开发人员的体验，以确保真正的采用和参与，因为只有这样才有可能实现这样一个平台的全部好处。

伊万·涅托在 [Unsplash](https://unsplash.com/s/photos/platforms?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

*最初发布于*[*https://loft . sh*](https://loft.sh/blog/why-internal-kubernetes-platform/)*。*