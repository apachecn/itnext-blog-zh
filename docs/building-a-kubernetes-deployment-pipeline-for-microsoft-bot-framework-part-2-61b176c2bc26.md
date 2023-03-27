# 为 Microsoft Bot 框架构建 Kubernetes 部署管道—第 2 部分

> 原文：<https://itnext.io/building-a-kubernetes-deployment-pipeline-for-microsoft-bot-framework-part-2-61b176c2bc26?source=collection_archive---------2----------------------->

![](img/1d244170a9b6b63b352becfc593c2311.png)

这是本系列的第二篇文章，讨论我如何为聊天机器人建立一个端到端的 Kubernetes 部署管道。如果你错过了第 1 部分，你可以点击这里查看。

第 2 部分详细介绍了主题公园聊天机器人的部署。所有的代码都可以在我的 GitHub 上找到: [themeparks-bot](https://github.com/JonJam/themeparks-bot) 。

# 工具

在我之前的帖子中，我介绍了我用来创建基础设施的工具；对于机器人，我重用了大部分工具，增加了一些东西:

## 码头枢纽

[Docker Hub](https://hub.docker.com/) 是一个云容器映像注册表。这用于存储从每个 Travis CI 构建中构建的 docker 映像，然后在 Kubernetes 中使用。

我选择了 Docker Hub，因为它对公共图像是免费的，并且与 Kubernetes 集成在一起。

## Docker CLI

Docker CLI 是用于管理 Docker 容器和图像的命令行工具。我用它来构建聊天机器人 docker 映像，并将其推送到 Docker Hub。

# 创建舵图

为了使用[舵](https://helm.sh/)进行部署，我必须创建一个[图表](https://docs.helm.sh/developing_charts/#charts)，这部分详细介绍了我是如何开始的，并显示了组成图表的各个组件。

## 入门指南

Helm CLI 提供了一个 [create](https://docs.helm.sh/helm/#helm-create) 命令来生成一个包含所需目录结构和文件的框架图。运行如下:

```
helm create themeparks-bot
```

该命令的输出是这样的文件夹结构:

```
themeparks-bot/
  |
  |- .helmignore   # Contains patterns to ignore when packaging Helm charts.
  |
  |- Chart.yaml    # Information about your chart
  |
  |- values.yaml   # The default values for your templates
  |
  |- charts/       # Charts that this chart depends on
  |
  |- templates/    # The template files
```

主工作区是 templates 文件夹，因为这是 Kubernetes 资源所在的位置。事实上，框架图实际上附带了示例[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)、[入口](https://kubernetes.io/docs/concepts/services-networking/ingress/)和[服务](https://kubernetes.io/docs/concepts/services-networking/service/)模板。

## 创建值文件

在创建模板之前，我首先修改了 [values.yaml](https://docs.helm.sh/chart_template_guide/#values-files) 文件。这包含模板中使用的属性的默认值。

价值观念

我在修改价值观文件时遵循了许多最佳实践(参见[文档](https://docs.helm.sh/chart_best_practices/#values))。一些关键因素是:

*   变量名应该使用大小写。
*   扁平结构比嵌套结构更好。
*   所有财产都应记录在案。

需要注意的一点是，我还没有为 secret 属性下的内容指定值；这是因为我在部署时传递它，因为它们是我不想在源代码控制中使用的值。

## 定义模板

使用骨架模板作为起点，我为聊天机器人修改并创建了额外的模板。

Helm 有许多关于开发模板的通用指南，例如:

*   模板文件应该遵循[图表名称]-[Kubernetes 资源类型]的命名约定。例如主题公园-机器人-部署。
*   一个模板文件应该只包含一个 Kubernetes 资源([源](https://docs.helm.sh/chart_best_practices/#structure-of-templates))。
*   所有资源都应该有标准的头盔标签([来源](https://docs.helm.sh/chart_best_practices/#standard-labels))。
*   include 函数应该在 template 函数之上使用( [source](https://docs.helm.sh/chart_template_guide/#the-include-function) )。

考虑到这些，这里是单独的模板。

**_ 助手**

[这是模板片段和助手](https://docs.helm.sh/chart_template_guide/#named-templates)的默认位置。这些助手在单独的 Kubernetes 资源模板中使用。

关于助手的指导包括:

*   为每个功能定义一个文档块([来源](https://docs.helm.sh/chart_template_guide/#declaring-and-using-templates-with-define-and-template))。
*   在每个函数前加上图表名称([源](https://docs.helm.sh/chart_template_guide/#declaring-and-using-templates-with-define-and-template))。

助手模板

**秘密**

这是第一个使用这些辅助函数和值来定义一个[秘密](https://kubernetes.io/docs/concepts/configuration/secret/)的例子。

秘密模板

**部署**

接下来是[deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)t，它依赖于之前定义的秘密；这就是我在注释部分引用它的原因。

我这样做的原因是，每当任何秘密值改变时，Kubernetes 将重新部署，选择任何新的值(关于这一点的更多信息可以在这里找到)。

部署模板

**服务**

现在我有了一个部署，我需要在集群中公开它，因此有了[服务](https://kubernetes.io/docs/concepts/services-networking/service/)的定义。

服务模板

**入口**

最后，最后一部分是 Ingres 的资源。该服务目前仅在集群内可用，外部不可用。

这是因为由于这个机器人是基于 expressjs 的，所以建议使用类似 NGINX 的代理来使它面向互联网。这正是这个入口定义的工作。

入口模板

## 指定依赖关系

正如我在定义模板一节中提到的，聊天机器人通过 [NGINX](https://www.nginx.com/) 向外界公开。此外，机器人使用 [Redis](https://redis.io/) 缓存主题公园数据。

Helm 不需要自己创建 Kubernetes 模板来部署 NGINX 和 Redis，它允许你将其他图表定义为与你的图表一起部署的[依赖关系](https://docs.helm.sh/developing_charts/#chart-dependencies)。

幸运的是，NGINX 和 Redis 已经有图表了。通过创建一个 requirements.yaml 文件，这些被包含在这个图表中。

图表相关性

## Chart.yaml

最后一个难题是实际的图表定义:

图表定义

# 构建和部署

创建了 Helm 图表后，我使用 Travis CI 将所有东西放在一起，构建并部署机器人到 Kubernetes。

我的. travis.yml 文件如下所示:

这个 Travis CI 定义利用了[构建阶段](https://docs.travis-ci.com/user/build-stages/)(目前处于测试阶段)，它允许您清晰地划分构建/部署管道的各个阶段。您还会注意到，我使用了许多环境变量，这些变量是通过[存储库设置](https://docs.travis-ci.com/user/environment-variables#Defining-Variables-in-Repository-Settings)来设置的。

该定义执行以下操作:

*   在任何阶段之前，它会安装脚本运行所需的[依赖项](https://docs.travis-ci.com/user/installing-dependencies/)，即 Azure CLI、kubectl 和 helm。
*   构建阶段执行以下任务:

1.  使用 yarn 安装依赖项。
2.  使用 yarn 构建机器人。
3.  舵轮图定义。
4.  如果构建在 master 上进行并且成功，那么它将构建一个 docker 映像并将其发布到 Docker Hub。

*   如果构建阶段已经过去，并且构建正在主分支之外进行，那么它将运行部署阶段。这个阶段使用实验性的[脚本部署](https://docs.travis-ci.com/user/deployment/script/)来运行包装器脚本，以将图表部署到 Kubernetes。

包装脚本如下所示:

用于部署的包装脚本

有了所有这些，主题公园聊天机器人就部署到 Kubernetes 上，并准备好供用户聊天。

我希望这篇由两部分组成的文章能够帮助那些希望开始使用 Kubernetes/Helm 并建立部署管道的人。