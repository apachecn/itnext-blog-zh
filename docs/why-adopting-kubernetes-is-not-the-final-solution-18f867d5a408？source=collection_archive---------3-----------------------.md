# 为什么采用 Kubernetes 不是解决方案

> 原文：<https://itnext.io/why-adopting-kubernetes-is-not-the-final-solution-18f867d5a408?source=collection_archive---------3----------------------->

采用 Kubernetes 只能是在组织范围内推广以获得真正 Kubernetes 好处的第一步。

![](img/b7fbc4bcdd9401127b0027bb7f2bc651.png)

库伯内特无处不在。如果你听说它的众多令人印象深刻的收养数字中的一个，你很容易产生这种感觉。这是否意味着 Kubernetes 已经到了它的“最后阶段”并且已经把它的所有好处带给了使用它的公司[？](https://devspace.cloud/blog/2019/10/31/advantages-and-disadvantages-of-kubernetes#advantages-of-kubernetes)

# Kubernetes 的采用只是第一步

要回答这个问题，我们有必要仔细研究一下收养的实际含义。术语“Kubernetes 采用”是相当模糊的，可以有广泛的含义，从在整个组织中采用 Kubernetes，在其上运行所有工作负载，到只是让一名工程师在测试环境中使用它。

如果你仔细看看收养统计，你会发现收养通常仍然非常有限。例如，Gartner 发现不到 [30%的公司在生产中使用容器化的应用程序，不到 5%的企业应用程序在容器环境中运行](https://www.gartner.com/en/newsroom/press-releases/2020-06-25-gartner-forecasts-strong-revenue-growth-for-global-co)。在 VMWare 的“2020 年 Kubernetes 状况”调查中可以找到 Kubernetes 使用仍处于早期阶段的进一步证据，该调查指出，57%的公司运营的 Kubernetes 集群少于 10 个。

虽然这些数字预计将在未来几年内大幅增长，但这也意味着今天，已经“采用”Kubernetes 的公司还不能靠后享受其全部好处。仅仅通过在 Kubernetes 上运行小而不重要的实验来实现下一个花哨的技术，除了乍一看像一家现代科技公司之外，不会带来任何好处。

然而，人们很快就会发现，如果这种 Kubernetes 的采用就此结束，那么它将毫无用处，开发者甚至会感到失望。正如[最新的堆栈溢出调查](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-platforms-wanted5)所发现的那样，许多开发者实际上喜欢 Kubernetes，并希望了解更多。

**因此，采用 Kubernetes 只是有意义地使用 Kubernetes 的第一步。**

# 你需要真正的库伯内特扩散

要获得 Kubernetes 的全部优势并让您的开发人员乐于使用它，您实际需要的是真正的 Kubernetes 扩散，这意味着整个组织范围的推广和对 Kubernetes 的访问。

虽然 Kubernetes 仍然是一项相当新的技术，但毫无疑问，它已经足够成熟，最初的采用步骤可以被视为回答是否使用 Kubernetes 这个问题的评估期。鉴于 Kubernetes 得到了主要参与者和云提供商的广泛认可，它已经成为一项主导技术，你可以相当肯定地说，它是一个面向未来的解决方案。

现在，作为下一步，您可以关注 Kubernetes 扩散，这带来了一些新的要求和挑战。具体来说，要在组织内广泛使用 Kubernetes 并使您的 Kubernetes 采用达到更高水平，需要回答两个核心问题:

# 1.如何让 Kubernetes 面向所有人

第一个基本问题是如何让每个人都能接触到 Kubernetes。解决这个问题至关重要，因为没有 Kubernetes 访问，就不可能使用它。这就是为什么如果您计划为您的公司进一步推广 Kubernetes，这个问题应该首先得到解决，并且具有高优先级。

## 首次采用的解决方案:

如果您处于早期采用和试验阶段，回答这个问题相对简单。您可以只为公共云中的托管 Kubernetes 服务付费，这可能是每月数百美元的集群管理费和计算资源，或者让工程师自己设置他们的 Kubernetes 集群。设置可以在公共云、私有云中进行，甚至可以在工程师的本地计算机上进行。由于这些采用者是 Kubernetes 专家(或者应该成为专家)，并且通常还拥有云环境的管理员权限，所以在早期采用阶段，所有这些都不是大问题。

## 扩散的挑战:

在扩散阶段变得更加复杂。现在，不仅一些拥有云环境管理权限的 Kubernetes 专家需要 Kubernetes，而且您组织中的每个工程师都需要 Kubernetes。为了有效地实现这一点，您需要一种自动化的方式来提供 Kubernetes，甚至向非专家提供，这就排除了本地 Kubernetes 解决方案(没有自动化，单独设置)和简单管理的 Kubernetes 产品(普通工程师不应该有管理员权限)。

## 扩散解决方案:

这意味着您需要为您的工程师提供一个易于使用的内部 Kubernetes 平台，让他们能够按需创建工作环境。同时，这个自助服务平台需要强制执行使用限制，以防止过高的云计算成本。

事实上，一些行业领导者和创新公司，如 [Spotify](https://www.youtube.com/watch?v=vLrxOhZ6Wrg) 和 [Datadog](https://static.sched.com/hosted_files/kccnceu20/a8/Toolchains%20v3.pdf) (第 25 页)已经为他们的工程团队建立了这样一个内部平台，这就表明这样一个内部平台是可行的。

然而，从头开始开发这样一个平台可能会非常昂贵，而且不容易，因此在现有解决方案的基础上构建可能是有意义的。例如， [kiosk](https://github.com/kiosk-sh/kiosk) 就是为这种用例而设计的，可以用作内部 Kubernetes 平台的底层多租户扩展。

如果这仍然太费力，您还可以使用一个现成的解决方案，它包括所有的自助服务、用户隔离和用户管理功能。 [loft](https://loft.sh/) 就是这样一个解决方案，它可以将任何常规的 Kubernetes 集群转变为您团队的自助式内部 Kubernetes 平台。在这种情况下，您所要做的就是将 loft 安装到您的 Kubernetes 集群中，该集群应该作为内部平台。要尝试一下，可以按照 [loft 快速入门](https://loft.sh/docs/quickstart)来。

# 2.如何让 Kubernetes 为每个人所用

在你想出如何让每个人都能访问 Kubernetes 之后，你需要让工程师能够有效地使用它。这个问题的解决方案是必不可少的，因为 Kubernetes 可能非常复杂，并且不是每个工程师都是(或者应该成为)Kubernetes 专家，但是您仍然希望通过引入 Kubernetes 来确保提高开发效率。

## 首次采用的解决方案:

同样，在最初的采用阶段，当只有少数工程师使用和探索 Kubernetes 时，这个挑战并不是一个真正的问题。他们通常要么是 Kubernetes 专家，要么可以通过与它合作成为专家。在 Kubernetes 中学习和尝试东西实际上是他们工作的一部分，在探索阶段，时间和效率不是那么重要。

## 扩散的挑战:

更广泛采用的情况，即扩散阶段，是非常不同的。在这里，“普通”工程师应该与 Kubernetes 一起工作。然而，他们的工作不是完美地学习和掌握 Kubernetes，而是尽可能高效地完成他们的常规任务。因此，Kubernetes 的使用应该尽可能简单、高效和快速，这样它就可以支持而不会干扰他们的实际工作。

## 扩散解决方案:

你需要为你的工程师提供涵盖他们整个工作流程的解决方案。首先要以简单的方式获得 Kubernetes 访问权限(参见[问题 1](https://loft.sh/blog/why-adopting-kubernetes-is-not-the-solution/#1-how-to-make-kubernetes-available-for-everyone) )。然后，工程师需要能够使用 Kubernetes 进行开发。为此，已经有一些流行的开源工具促进和规范了开发工作流，如 [DevSpace](https://github.com/devspace-cloud/devspace) (它也有一个可选的 [loft integration](https://github.com/loft-sh/loft-devspace-plugin) ，因此创建一个 Kubernetes 工作环境和后续的开发可以用同一个工具来完成) [Skaffold](https://github.com/GoogleContainerTools/skaffold) 或 [Tilt](https://github.com/tilt-dev/tilt) 。

最后，工程师需要能够轻松地部署到 Kubernetes，这可以使用相同的工具或专门的 CI/CD 工具来完成，如 [Jenkins](https://www.jenkins.io/) 、 [Circle CI](https://circleci.com/) 或 [Travis CI](https://travis-ci.org/) 。

在整个工程设计过程中，每个步骤都要有完整的文档记录，并且工具要尽可能地预先配置和设置，这样工程师就可以简单地遵循标准化的指导方针，从而快速提高工作效率。

由于不可避免地会出现问题(有些问题实际上是可取的，因为这样可以在 bug 进入生产系统之前发现它们)，所以鼓励和促进不断学习如何使用这些工具和 Kubernetes 也很重要。(尽管如此，普通工程师不应该成为 k8s 专家，而应该知道如何处理“标准问题”并理解基本的 Kubernetes 概念。)

# 你收养了 Kubernetes 后应该做什么

总而言之，让我们假设你已经成功地在小范围内采用了 Kubernetes，并且现在想要在你的组织中促进扩散。您的下一步应该是:

1.  通过内部 Kubernetes 平台，使 Kubernetes 在一个简化的流程中变得容易获得。要么像 Spotify 一样从头开始构建，基于现有的解决方案，如 [kiosk](https://github.com/kiosk-sh/kiosk) 或直接从现成的解决方案开始，如 [loft](https://loft.sh/) 。
2.  准备在您的组织中推广。为开发和部署工作流选择工具，编写内部文档，并尽可能多地设置/预配置工具。
3.  逐步在您的组织中推广 Kubernetes。也许可以从培训和入职会议开始。最好从几个工程师或一个团队开始，看看向 Kubernetes 的转变是如何为他们工作的，这样你就可以在下一个团队中改进它。
4.  持续改进您的系统、工作流程和沟通。听取工程师的反馈，继续教育和帮助解决问题。您还应该监控扩散进度，并关注云计算成本。

# 结论

Kubernetes 现在已经接触到了大多数公司，但还没有接触到普通人。虽然最初以实验的形式采用是很好的第一步，但这并不是获得 Kubernetes 真正好处的最终目的地。这种采用还需要继续，所以 Kubernetes 也覆盖了大多数工程师。为了在不损失工作效率的情况下完成这一步，您需要为工程师提供一个内部的自助式 Kubernetes 平台和各种工作流工具，使他们能够随时获得并轻松使用 Kubernetes。

林赛·亨伍德在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

*最初发布于*[*https://loft . sh*](https://loft.sh/blog/why-adopting-kubernetes-is-not-the-solution/)*。*