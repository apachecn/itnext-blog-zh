# 使用 Helm 部署到 Kubernetes 的最佳实践

> 原文：<https://itnext.io/best-practices-for-deploying-to-kubernetes-using-helm-73be1f3040d2?source=collection_archive---------0----------------------->

![](img/fefb0d8a73bf9c50b15497a56ff579c9.png)

这是一系列实用、安全和可重复的 Kubernetes 部署方法的一部分。这些帖子是建筑[*delivery bot*](http://deliverybot.github.io)*的一部分，是从 GitHub 部署到 Kubernetes 的缺失环节。*

在上一篇文章中，我们讨论了使用裸 Kubernetes 清单和 [Kustomize 来部署应用程序。Helm 是“Kubernetes 的包管理器”,因此已经成为管理应用程序部署的最常用工具之一。](https://medium.com/@colinjfw/kustomize-deploying-applications-using-the-new-kubernetes-templating-system-43f26706069)

然而，Helm 是一个包格式。不是将代码部署到 Kubernetes 的工作流。在这篇文章中，我想讨论如何在不同类型的工作流中最有效地使用 Helm，以便在 Kubernetes 上为您的基础设施提供简单而安全的部署。

Helm 是围绕图表构建的。基本上，这些是用 go 模板语言评估的 Yaml 文件集。图表允许我们为 Kubernetes 构建模块。我们可以在 Helm 中声明一组值，用户在部署一个图表来发布一个应用程序时提供这些值，这个图表抽象了向 Kubernetes 发布某些应用程序的复杂性。

# 基本舵图

```
~/charts/app
├── Chart.yaml
├── README.md
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── secret.yaml
│   └── service.yaml
└── values.yaml
```

掌舵图的基础包括图表元数据(“Chart.yaml”和“values.yaml”)以及构成主图表的模板。主图表配置包括图表的命名和版本控制。值文件是我们声明由图表模板内部读取的值的地方，这些值可以由图表的用户在外部声明。这是 Helm 作为一个抽象层，简化了一组可以扩展成大量 Kubernetes 的值。

## 部署、服务和入口

这是大多数专注于发布网络应用的掌舵图的核心。这三种资源是部署应用程序并能够从集群外部与应用程序对话的关键。非常简单:

*   部署发布您的代码。
*   服务在内部将流量路由到您的代码。
*   入口资源将进入集群的外部流量路由到您的代码。

这就是为什么这三种资源是发布网络化应用的支柱。让我们看一个 Helm 图表中服务资源的非常小的例子，以及模板化如何允许我们重新定义选项:

```
apiVersion: v1
kind: Service
metadata:
  name: {{ include "app.fullname" . }}
  labels:
{{ include "app.labels" . | indent 4 }}
spec:
  type: {{ .Values.service.type }}
```

这个基本服务资源示例展示了一些 go 模板工具，用于创建一个助手来指定资源的全名。创建此服务的相应值如下所示:

```
service:
  type: ClusterIP
  port: 80
```

这是一个简单的例子，说明了使用 Helm 抽象出底层资源的好处。

# 固执己见的发布工作流程

Helm 旨在消除社区在集群中安装软件的重复工作。在选择 Helm 发布代码时，您最终将采用两种主要的工作流:

1.  将第三方代码发布到集群中。这些代码通常是语义版本，以较慢的节奏发布，根据更新周期可能是每月或每年。
2.  持续交付到您的 Kubernetes 集群。这段代码经常每天发布，甚至比这更频繁。

第一种情况是 Helm 最初的设计目的。它要求图表实现语义版本控制，并使图表升级相对简单。为第三方软件包使用 Helm 的最有效方式是使用 [Terraform 提供者](https://www.terraform.io/docs/providers/helm/index.html)。这允许在您的集群中有一个稳定一致的发布过程。这通常作为代码工作流与基础设施的其余部分联系在一起。

这篇文章的主要目标是看看如何在连续交付中使用 Helm。开始时，有效地做到这一点似乎很简单，但要确保您的部署是有效的，需要记住一些事情。

使用 CLI 部署 Helm 相对简单。指定以下命令集，使用生产值文件中的值在默认命名空间中发布新图表:

```
helm upgrade --install release-name \
  --namespace default \
  --values ./production.yml
```

当我运行这个命令时，实际上会发生什么？在引擎盖下，Helm 有一个名为 Tiller 的服务器端实现。Helm 打开一个到 Kubernetes 集群的连接，并将 Helm 图表以及作为参数传入的值写入这个连接。服务器端组件呈现带有这些值的模板，并将它们应用到 Kubernetes 集群中。

这些是使用 Helm 运行部署的基础。固执己见的部分和您的工作流涉及到计算什么值作为参数传递以及何时运行 Helm upgrade 命令。让我们讨论一下在您的组织中实现这一目标的一些正确方法。

## 避免每个服务都有一个图表

使用掌舵图的一个好处是，您可以将整个组织或项目中可能一致的通用配置合并到一个图表中。在您想要部署的每个存储库中放置一个图表可能更简单，但是要投资拥有一组核心图表，用于整个组织的服务。

这可以加快标准化运行状况检查路径的速度，并保持跨资源的通用配置。此外，如果您想随着 Kubernetes 资源的成熟而采用新的 api 版本，您将能够从一个中心位置做到这一点。

舵图会变得非常冗长，实际上很难写好。您需要注意确保未定义的变量会出错，标签是一致的，并且不会有任何 Yaml 缩进错误。最好将舵图的创作交给那些有使用 Kubernetes 经验的人，并允许您的开发人员将它们作为模块使用。

## 分离生产和暂存配置值

推入图表中的配置值应该在不同的环境中是不同的。构建简单的可扩展图表，可以根据环境进行覆盖。

例如，一个好的工作流应该在每个环境中有不同的值文件，并且在配置上有特定的差异:

```
~/myapp
└── config
    ├── production.yml
    └── staging.yml
```

每个值文件的使用取决于部署环境。这与使用 if 语句或其他逻辑将配置放入舵图形成对比。Helm 是用来抽象出 Kubernetes 资源和价值的，文件是传递您的环境和应用程序特定信息的方式。

确保使用“require”语句和林挺工具，以确保在部署时没有未定义的值。如果未定义以下变量，实际上会导致相对严重的配置问题:

```
host: {{ .Values.hostPrefix }}.example.com
```

## 秘密管理

Kubernetes 秘密是管理应用程序秘密的最简单的方法之一。Helm 并没有试图以任何方式管理机密，你可能会陷入试图配置复杂的工作流程。秘密管理的一个简单的初步建议是:

1.  使用您的 CI/CD 提供商将机密存储在他们的仪表板中。
2.  在部署时将秘密值传递到图表值中。
3.  当这些秘密值改变时，使用[校验和来推出 pod](https://github.com/helm/helm/blob/master/docs/charts_tips_and_tricks.md#automatically-roll-deployments-when-configmaps-or-secrets-change)。

我们希望将一个新的秘密发布到集群中，以便在我们的部署管道中可审计和可见。我们也希望开发者能够访问这个。一如既往，想出适合你情况的最佳解决方案。

[Vault](https://www.vaultproject.io/) 也是一个神奇的秘密管理工具，它与 Kubernetes 深度集成，提高了安全性。它可能需要较长的设置时间，但它提供了一些高级功能，您的组织可以在以后利用这些功能。

## 编辑值

确保在使用 Helm CLI 时，只有 Helm 会更改 Kubernetes 清单中的值。例如，不要在 Helm 之外修改部署的副本，然后使用 Helm 更改值。如果你以前使用过 Kubernetes，你可能使用过`kubectl apply`工具。这是将基础设施应用到集群中的最简单的方法之一。Helm 没有使用与 Kubectl 相同的技术[。这意味着，如果您在 Helm 之外编辑基础设施，下次运行 Helm 命令时可能会出现问题。](https://github.com/helm/helm/issues/2070#issuecomment-284839395)

## 舵图储存库

Helm chart 存储库只是提供图表资源的 http 服务器。你可以在 github.com/helm/charts 的[看到官方策划的图表资源。图表存储库主要是为语义版本化的图表设计的。因此，图表存储库不能很好地处理并发更新。避免在 CI 或模式中使用图表存储库，比如为每次提交创建一个新的舵图。](https://github.com/helm/charts)

舵图应该是语义版本化的模块。它们代表部署应用程序的底层基础设施。可以把它们想象成一个库，用来抽象掉 Kubernetes 的复杂性。将图表存储在整个组织都可以访问的存储库中，并允许开发人员使用组织中的最佳实践来轻松使用这些图表进行部署。

在组织中分发图表的一个简单方法是使用 S3 甚至 GitHub 版本作为图表的存储。一个简单的模式是拥有一个公共的`myorg/charts`存储库，其中包含您的团队策划和构建的所有图表，可以安装到 Kubernetes 中。

## 握紧你的舵柄

如果您对 Kubernetes 集群有严格的访问控制，您可能会被 Tiller(Helm 的服务器端组件)在 Kubernetes 集群中作为超级用户运行的问题所困扰。有一些解决方法，但是需要一些更复杂的配置来解决。对于大多数团队，默认配置看起来像是当您连接到 Helm 时，您拥有对集群的管理员访问权限。

解决这个问题的一个常见方法是在不同的名称空间中创建不同的 Tiller 实例。如果您有一个非常大的 Kubernetes 集群，这可能是一种分割特定团队的方式，以便只访问一个 Tiller。Helm 第 3 版在这方面有更多的更新，实际上只在客户端应用资源。

# 摘要

新的 Kubernetes 部署面临的最大挑战之一，是涉水通过所有的最佳实践和意见，让您的部署管道第一次就设置正确。Helm 部署有很多问题，您需要正确导航到设置，但一旦设置完成，就可以提供灵活性和强大的功能来有效地连接您的部署。

*请继续关注 Kubernetes 系列的下一个实用而安全的部署。我们将与 GitHub 讨论自动化工具，将所有这些最佳实践包装在代码中。*