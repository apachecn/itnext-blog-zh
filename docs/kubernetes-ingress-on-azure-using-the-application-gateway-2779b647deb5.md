# 使用应用程序网关在 Azure 上导入 Kubernetes

> 原文：<https://itnext.io/kubernetes-ingress-on-azure-using-the-application-gateway-2779b647deb5?source=collection_archive---------0----------------------->

## 如何在单个主机上公开多个服务

![](img/9db4833d6ca2d4f4529b16c4be9caf81.png)

拉斯·金勒在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

将一个应用程序部署在由多个微服务组成的 Kubernetes 集群上，您可能希望公开其中的一些微服务，以便可以通过互联网进行访问。虽然这显然是为了您的 web 应用程序服务，但是您可能还想公开一些额外的 API。

在 Kubernetes 的世界里，任何与你的微服务的连接都是通过使用*服务*资源来完成的。使用 Kubernetes *服务*资源的类型*负载平衡器*利用底层云提供商创建特定于云提供商的负载平衡器，用于通过外部 IP 公开微服务。这种方法的问题是，每个微服务将暴露在一个单独的 IP 地址下。

让它们暴露在同一个主机下会方便得多，同时有不同的路径到达专用微服务，对吗？

本文展示了如何使用 Azure 上的 Kubernetes 集群和 Azure 应用程序网关来实现这一点。

# 介绍

微服务可以使用 Kubernetes *服务*资源在 Kubernetes 内部和外部公开。到目前为止，一切顺利。但是如前所述，如果我们想在集群之外公开它们，使用类型为 *LoadBalancer* 的服务资源，我们最终会为每个微服务提供不同的 IP。这不是我们想要的，相反，我们想让它们暴露在一个和主机使用不同的路径。

这就是 Kubernetes *Ingress* 资源派上用场的地方。把入口想象成 Kubernetes 服务之上的一层。它是访问我们微服务的流量的单一入口，微服务根据指定的规则将流量路由到不同的 Kubernetes *服务*。

Kubernetes*Ingress*resource 的概念就像一个抽象概念。为了使用 Kubernetes *入口*，你必须安装一个特殊的*入口控制器*。Kubernetes *Ingress* 抽象有很多不同的实现。Nginx 和 Traefik Ingress 是其中两个在 Kubernetes 和开源社区中非常受欢迎的工具。

当然，我们还有云提供商，你可以使用负载平衡器和网关等资源作为 Kubernetes *入口*。在这篇文章中，我们关注的是 *Azure 应用网关入口* (AGIC)。

# 先决条件

以下是完成整个过程所需的一些先决条件:

*   Azure CLI 已安装
*   获取您拥有全局管理员角色的 Azure 订阅
*   kubectl 已安装
*   jq 已安装

# 准备 Azure 环境

在创建应用程序网关和 AKS 集群之前，我们从准备 Azure 环境开始。

## 创建新的资源组

我们要做的第一件事是在 **eastus** 区域创建一个名为 **k8srg** 的新资源组。因此，您首先必须登录您的 Azure 帐户。以下命令将打开一个带有单点登录站点的浏览器，您可以在其中输入凭据。

```
az login
```

成功登录后，CLI 将列出您帐户的所有可用订阅。选择您的订阅。

```
az account set --subscription <subscription id>
```

现在我们可以实际上我们的新资源组。

```
az group create —-name k8srg --location eastus
```

## 添加 AKS 预览扩展

当我使用 Azure 应用网关测试 Kubernetes *Ingress* 时，这个功能还在预览版。为了在 Azure CLI 中使用 AKS 的预览功能，我们必须添加相应的扩展。

```
az extension add --name aks-preview
```

您可以通过调用以下命令来检查是否成功添加了扩展。

```
az extension list
```

最后一个命令还显示了扩展的版本。如果有新版本的扩展，您可以使用以下命令更新扩展:

```
az extension update --name aks-preview
```

## 注册提供商

接下来，我们必须注册允许 Kubernetes *入口*连接到应用程序网关的提供者。对于 Azure 订阅，该提供商只需注册一次。但是首先，我们需要扩展*微软。ContainerService* 提供者具有将入口连接到应用程序网关的功能。

提供商扩展过程可能需要一些时间。对我来说，大约需要 15 分钟。无论如何，您可以使用下面的命令检查提供者扩展的状态。

最后一个命令将显示当前状态，它首先应该等于*注册*，如果提供者扩展成功，则等于*注册*。

最后，我们需要注册*微软。ContainerService* 提供者(如果它已经注册到您的订阅，它将被刷新)。

```
az provider register --namespace Microsoft.ContainerService
```

## 创建新的虚拟网络

当使用*应用网关 Kubernetes Ingress* 时，每当您想要公开一个微服务时，在应用网关内部会创建一个指向特定微服务的新路由。为了使连接工作，应用程序网关和 Kubernetes 必须在同一个 Azure Vnet 中。否则，连接将无法建立，您最终会在应用程序网关端得到不健康的后端探测。

因此，首先我们创建一个新的 Vnet，并在该 Vnet 内创建两个子网，分别用于应用程序网关和 Kubernetes。

# 创建 Azure 应用网关

正如我们将在本文后面看到的，根据 URL 路径将流量路由到不同的微服务可以通过 Kubernetes 清单进行配置。然而，Azure 应用程序网关的创建必须提前完成。以下命令创建一个强制公共 IP，并将其分配给一个新创建的应用程序网关。

# 创建 AKS 实例

现在我们可以开始创建一个简单配置的 Kubernetes 集群了。

列出的命令将在 **eastus** 地区创建一个名为 **k8srg** 的资源组，并在其中创建一个 Kubernetes 集群。Kubernetes 集群被命名为 **myk8s** ，包含 1 个 worker 节点。最后一个命令将使您的`kubectl`连接到新创建的 **myk8s** Kubernetes 集群。

在之前的步骤中，我们为 Azure 订阅注册了提供者，这支持将应用程序网关用作 Kubernetes *入口*的特性。现在我们需要为我们的 Kubernetes 集群启用该特性，并指定应该使用的应用程序网关。

请注意，最后一个命令会将应用程序网关入口控制器所需的所有组件安装到 Kubernetes 集群中。在我撰写本文时，这种安装 AGIC 的方法还处于预览阶段，不推荐用于生产用例。或者，您可以“手动”安装组件，这稍微复杂一点。

# 测试 AppGw 入口

如果在此之前一切顺利，我们就可以开始测试我们的安装了。因此，我们可以利用 ASP。Net 应用程序，它通过我们的应用程序网关暴露自己。继续运行下面的命令。

您应该能够通过应用程序网关访问应用程序。打开浏览器，导航到应用网关的公共 IP。您应该会看到一个. NET 核心应用程序的基本欢迎页面。

# 公开多个服务

现在终于到了使用不同的路由在同一台主机上公开多个服务的时候了。因此，我们利用了我在 Dockerhub 上发布的一个小小的 Node.js 应用程序。这个应用程序是一个非常简单的服务器，它返回一个由环境变量指定的消息。

让我们看看部署清单是什么样子的，它公开了主机上某个路由下的服务。

文件的前两部分相对来说并不引人注意。请注意最后一节，它配置了如何公开服务。在第 53 行中，我们定义了应该使用哪个入口实现，第 59 行定义了应该公开服务的路由。一个有趣的方面是第 54 行，我们说后端的前缀是“/”。这是必需的，因为示例应用程序只有一个端点“/”。如果没有第 54 行，对*主机名/服务 1* 的请求将被路由到不存在的*示例应用程序/服务 1* 。

此时，您可以复制文件内容，创建您的文件并应用它。我为您简化了这一过程，并创建了一个 GitHub repo，其中包含两个部署清单，每个清单公开了不同路径上的测试应用程序，而每个应用程序返回不同的响应。所以你只需要运行以下要点的命令。

如果一切顺利，我们的 Azure 应用程序网关上应该有两条路由指向每个服务。我们将首先通过 Azure 门户或使用以下命令获取应用程序网关的 IP 来测试它们:

现在我们可以简单地使用 curl 来检查主机上的两条服务路由。

`curl $pubIP/service1`应返回“来自服务 1 的问候”,相应地`curl $pubIP/service2`应返回“来自服务 2 的问候”。

# 解决纷争

由于在公开我们的微服务的过程中包含多个组件，如果某些东西不能按预期工作，您将不得不查看不同的地方。

*   首先，您应该检查所有 pod 是否都在运行。最重要的是名称空间 kube-system 中的*ingress-app GW-deployment-**，而*等于随机字符串。该 pod 负责触发应用程序网关上的相应更新。
*   检查之后，如果代表您的微服务的 pod 运行正常。此外，检查指向您的微服务的服务是否已创建并正在运行。不要忘记仔细检查它们是否指向正确的端口。
*   如果 pod 和*服务*运行正常，检查*入口*是否也已创建，并且它们分别指向正确的*服务*。另外，检查入口定义中的*后端路径前缀*是否设置正确。
*   最后但同样重要的是，如果它仍然不能正常工作，您可以检查 kube-system 名称空间中的*ingress-app GW-deployment-**的日志，这可能会给你一些提示，如果有什么问题的话。如果所涉及的组件有一些瑕疵，删除该 pod 也会有所帮助。

# 结论

关于所示的在一个主机下公开多个服务的解决方案，最重要的结论是，您不应该盲目地遵循所提供的解决方案。而是想想自己真正的需求是什么。虽然应用程序网关是公开服务的一种方式，但也有其他技术可以用作 Kubernetes *入口*，从而实现在一台主机下公开多个服务的目标。不要忘记仪表定律:“如果你有一把锤子，所有东西看起来都像钉子”。如果您确实需要应用程序网关的高级特性，或者如果您已经有了一个应用程序网关，那么就去使用它吧。

但是我鼓励你在这里务实一点。我在一个客户项目中的个人经历表明，我并不真的需要一个全功能的应用程序网关。此外，在重新部署微服务时，我在应用网关方面遇到了几个关于证书处理和零停机部署的问题，但这是另一回事了。也许我会写另一篇关于它的文章。作为一个试探，我选择 Traefik 作为那个客户项目的 Kubernetes *入口*。一定会有另一篇文章展示我是如何做到的。

我希望你喜欢阅读我的文章，并且你已经成功地公开了你的服务。我很想听听你在 AGIC 的经历。

# 进一步阅读

*   [https://github . com/Azure/application-gateway-kubernetes-ingress](https://github.com/Azure/application-gateway-kubernetes-ingress)
*   [https://docs . Microsoft . com/de-de/azure/aks/load-balancer-standard](https://docs.microsoft.com/de-de/azure/aks/load-balancer-standard)
*   [https://docs . Microsoft . com/de-de/azure/application-gateway/tutorial-ingress-controller-add-on-new](https://docs.microsoft.com/de-de/azure/application-gateway/tutorial-ingress-controller-add-on-new)