# 为 Microsoft Bot 框架构建 Kubernetes 部署管道—第 1 部分

> 原文：<https://itnext.io/building-a-kubernetes-deployment-pipeline-for-microsoft-bot-framework-part-1-ddbb9f6f1796?source=collection_archive---------5----------------------->

![](img/c4b3d2ae805b0eb592c4f4baf0137f80.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fbuilding-a-kubernetes-deployment-pipeline-for-microsoft-bot-framework-part-1-ddbb9f6f1796)

在我之前的[帖子](https://chatbotslife.com/writing-a-bot-using-microsoft-bot-framework-in-node-and-typescript-5d62878eb69d)中，我讨论了主题公园信息聊天机器人的开发。所有代码都可以在我的 GitHub 上找到:

*   主题公园机器人
*   [主题公园-基础设施](https://github.com/JonJam/themeparks-infrastructure)
*   主题公园-路易斯

到目前为止，这个机器人一直在我的机器上本地运行。下一步是托管它，以便人们可以真正开始与它聊天。

我可以用不同的方式来托管这个机器人，但是我决定用 Kubernetes。原因是我想学习 Kubernetes 已经有一段时间了，但只是没有一个合适的项目来冒险。

这篇文章和后续的[文章](https://medium.com/@jonjam/building-a-kubernetes-deployment-pipeline-for-microsoft-bot-framework-part-2-61b176c2bc26?source=linkShare-8ce1fa774ade-1520462555)将详细介绍我是如何为主题公园机器人建立一个端到端的构建和部署管道的。

第 1 部分详细介绍了在 Azure 中设置 Kubernetes 集群和支持基础设施。所有的代码都可以在我的 GitHub 上找到:[主题公园-基础设施](https://github.com/JonJam/themeparks-infrastructure)。

# 工具

在设置部署管道时，我使用了许多不同的工具，下面将逐一介绍。

## 蔚蓝的

Azure 是微软的云平台，我用它来托管机器人的所有基础设施。

我决定使用 Azure，因为我已经有了很多在 Azure 上构建和部署应用程序的现有知识。这意味着我可以更专注于学习 Kubernetes，而不是云平台的细节。

## Azure CLI

Azure CLI 是管理 Azure 资源的命令行工具。构建和部署脚本使用它向 Azure 进行身份验证并部署资源。

我选择 Azure CLI 是因为它是跨平台的，允许我在 Windows 机器上开发，同时也可以在 Travis CI 的 Linux 机器上工作。

## Azure 资源管理器模板

[Azure 资源管理器](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) [(ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) )模板允许你使用 JSON 定义 Azure 资源。

使用这意味着我可以将我的所有基础设施定义为代码并存储在源代码控制中，而且 Azure CLI 可以很好地与之集成。

## 特拉维斯·CI

Travis CI 是 GitHub 的一个持续集成工具，我用它来构建和部署我的 GitHub 项目。

我是看了这篇 [GitHub 博文](https://github.com/blog/2463-github-welcomes-all-ci-tools)才选择 Travis CI 的。这篇文章显示了 GitHub 上大多数使用 CI 的项目都使用了 Travis CI。我不反对。

## 舵

[Helm](https://docs.helm.sh/) 是 Kubernetes 的一个包管理器，我用它来:

*   为 Kubernetes 资源提供模板语言。这允许我在部署过程中轻松地更改容器图像的标记，同样，也可以传入秘密值。
*   定义机器人的依赖关系，如 [NGINX](https://www.nginx.com/resources/wiki/) 和 [Redis](https://redis.io/) 。这些是舵图，已经按照最佳实践进行了配置；这意味着我不需要对它们有深入的了解。这些依赖项会随着我的机器人自动部署。

## 库贝特尔

[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 是用于在 Kubernetes 上部署和管理应用程序的命令行工具。这是对赫尔姆的依赖。

# 定义 Azure 资源

以下 ARM 模板定义了用于托管和由 bot 使用的各个 Azure 资源；这些是:

*   [Azure Container Service(AKS)](https://azure.microsoft.com/en-us/services/container-service/)—托管的 Kubernetes 集群。
*   [应用洞察](https://azure.microsoft.com/en-us/services/application-insights/) —针对机器人的日志记录和分析。
*   [机器人通道注册](https://azure.microsoft.com/en-us/services/bot-service/)——允许通过脸书信使等通道与机器人通信的注册。
*   [路易斯](https://azure.microsoft.com/en-us/services/cognitive-services/language-understanding-intelligent-service/) —机器人用来理解用户信息意图的语言理解服务。
*   [存储帐户](https://azure.microsoft.com/en-us/services/storage/?v=16.50) —机器人的状态存储。

定义 Azure 资源的 ARM 模板

在创建 ARM 模板时，我想尽快知道我是否错误地定义了它。Azure CLI 提供了一种机制来验证 ARM 模板及其参数是否有效，而无需实际部署它。

所以我创建了这个脚本，可以作为构建的一部分:

验证 ARM 模板脚本

现在我有了验证模板的方法，我需要能够部署它。同样，使用 Azure CLI，我创建了以下内容:

部署 ARM 模板脚本

# 设置舵

创建 Azure 资源后的下一步是在新创建的 Kubernetes 集群中设置 Helm。

这需要在 Kubernetes 集群上安装 Tiller (Helm 的服务器组件)。根据[文档](https://docs.helm.sh/using_helm/#initialize-helm-and-install-tiller)，这可以使用以下命令完成:

```
helm init 
```

在运行之前，我需要设置[上下文](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)来与 Kubernetes 集群通信。这包含在以下脚本中:

在 Azure Kubernetes 集群中安装 helm

需要注意的一点是超控，这是[舵文档](https://docs.helm.sh/using_helm/#tiller-s-release-information)中的建议。默认情况下，由于遗留原因，Tiller 使用 [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) 来存储敏感的发布信息，实际上应该使用 [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) 。

# 我喜欢建筑组合在一起

在创建了所有需要的脚本和模板之后，我可以使用 Travis CI 将所有东西放在一起。然后，每当我做出更改时，更新的基础设施就会自动构建和部署。

我的. travis.yml 文件如下所示:

Travis CI 定义

这个 Travis CI 定义利用了[构建阶段](https://docs.travis-ci.com/user/build-stages/)(目前处于测试阶段)，它允许您清晰地划分构建/部署管道的各个阶段。您还会注意到，我使用了许多环境变量，这些变量是通过[存储库设置](https://docs.travis-ci.com/user/environment-variables#Defining-Variables-in-Repository-Settings)来设置的。

该定义执行以下操作:

*   在任何阶段之前，它会安装脚本运行所需的[依赖项](https://docs.travis-ci.com/user/installing-dependencies/)，即 Azure CLI、kubectl 和 helm。
*   运行测试阶段，验证 ARM 模板和指定的参数。
*   如果测试阶段通过了，并且构建工作脱离了主分支，那么它将运行部署阶段。这个阶段使用实验性的[脚本部署](https://docs.travis-ci.com/user/deployment/script/)来运行部署 ARM 模板的包装器脚本，并设置 Tiller。

包装器脚本如下所示，只调用我之前定义的脚本。

用于部署的包装脚本

这就是基础设施的设置！[第 2 部分](https://medium.com/@jonjam/building-a-kubernetes-deployment-pipeline-for-microsoft-bot-framework-part-2-61b176c2bc26?source=linkShare-8ce1fa774ade-1520462555)详细介绍了使用 Helm 构建和部署主题公园机器人。