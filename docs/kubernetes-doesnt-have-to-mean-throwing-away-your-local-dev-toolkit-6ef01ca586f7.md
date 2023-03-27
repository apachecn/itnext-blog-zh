# Kubernetes 并不意味着扔掉您的本地开发工具包

> 原文：<https://itnext.io/kubernetes-doesnt-have-to-mean-throwing-away-your-local-dev-toolkit-6ef01ca586f7?source=collection_archive---------1----------------------->

![](img/04fd72ea74e1614dbecb244cada4bc3c.png)

## 使用您最喜欢的 IDE、调试器或其他在 Kubernetes 本地运行的工具

迁移到 Kubernetes 对开发团队来说可能意味着很多变化和挑战，但这不应该意味着您的开发人员不能使用他们喜欢的工具。改变工具会使开发人员更难高效工作。他们可以用自己熟悉的工具轻松解决的挑战在 Kubernetes 环境中可能无法实现。这意味着解决问题可能需要更长的时间，或者他们可能根本无法解决问题。

如果组织采用 Kubernetes 来加快速度，他们如何应对这些减缓他们的个体开发人员速度的变化呢？

# 调试 Kubernetes 服务

有大量的工具被用来调试传统的 web 应用程序。这些工具使开发人员更容易在一个看起来类似于生产环境的环境中识别错误和测试解决方案。

因为在使用 Kubernetes 时配置本地开发环境更具挑战性，所以使用本地工具也不那么简单。请考虑以下情况:

*   后端开发人员发现了他们服务中的一个问题/缺陷，但是他们无法在目标环境(生产、QA 等)之外重新创建它，他们需要使用基于 ide 的调试工具来诊断和修复问题
*   一个后端开发人员想在他们的 Kubernetes 集群中安装一个本地调试/诊断工具，但是由于潜在的安全风险，运营/平台/SRE/信息安全团队不允许他们这样做
*   一个后端开发人员想要从他们的本地机器 SSH/kubectl exec/port-forward 到一个远程集群，以便使用一个本地安装的工具来调试一个问题，但是运营团队已经禁用了这个选项来远程连接集群

对于单一的应用程序，大多数组织的开发人员都知道采取什么步骤来解决这些挑战，因为工具已经为这些环境创建和优化了。然而，使用 Kubernetes，过程没有很好地定义，开发人员倾向于浪费更多的时间来寻找解决方案。这当然意味着发布新功能的时间更少了。

# 网真+你在 Kubernetes 的本地工具

[Telepresence](https://www.getambassador.io/products/telepresence/) 是一款 OSS 工具，致力于改善单个 Kubernetes 开发者的工作流程。有了网真，开发人员可以像在 Kubernetes 集群中远程运行笔记本电脑一样进行开发。这样，他们可以保留现有的本地开发、构建和调试工作流。然后他们可以自由地使用本地工具(调试器、ide 等)。)来最大化他们的生产力。

这个演示展示了一个运行在 Kubernetes 中的 Java 微服务。由于远程呈现的强大功能，开发人员可以使用 VSCode 对他们截获到自己笔记本电脑上的服务进行更改。他们可以立即看到变化！

如果这是您在采用云原生技术时遇到的问题，我们很想听听您的故事以及您是如何解决的。请在推特上给我们留言 [@ambassadorlabs](https://www.twitter.com/ambassadorlabs) 或者[加入我们的 Slack 频道](http://d6e.co/slack)分享你的故事。

# 了解更多信息

*   看看[网真](https://www.getambassador.io/products/telepresence/)
*   了解更多有关 Kubernetes 微服务快速高效开发的信息
*   [今天就开始](https://www.getambassador.io/docs/latest/telepresence/quick-start/)