# 用 helmfile 设置 Kubernetes 集群

> 原文：<https://itnext.io/setup-your-kubernetes-cluster-with-helmfile-809828bc0a9f?source=collection_archive---------3----------------------->

![](img/912719ceb5e40be0e242f1b580f2b5f4.png)

所以，你有一个 Kubernetes 集群启动并运行。控制平面处于活动状态，工作节点已准备好处理负载。我们要继续部署我们的业务应用程序吗？在此之前，我们可能需要做一点准备。作为任何有自尊的企业，我们希望能够监控我们部署的应用程序，进行备份和自动扩展，并能够在出现问题时搜索日志。在实际部署应用程序之前，这些只是我们想要解决的一些需求。

幸运的是，有很多工具可以解决这些基础设施需求。此外，这些基础设施应用中的大多数都有相关的[舵图](https://github.com/helm/charts/tree/master/stable)。这些只是几个例子:

*   **普罗米修斯** / **格拉夫纳**进行监控
*   **测井用流利位**
*   **velero** 用于备份
*   **nginx 入口** **控制器**，用于负载均衡
*   **CI/CD 用三角帆**
*   **外部 dns** 用于与 dns 提供商同步 k8s 服务
*   **集群自动缩放器**

诸如此类…

问题是:**我们如何在 Kubernetes 集群**中部署它们？如果更多的 Kubernetes 集群涌现出来，我们如何以可靠、可维护和自动化的方式安装这些基础设施应用程序？

我们探索了几种方法来做这件事。有一把[伞舵图](https://stackoverflow.com/questions/51984976/how-to-build-umbrella-chart-with-exact-dependency-helm-chart-version-in-requirem)
2。[使用 Terraform 头盔提供者](https://github.com/terraform-providers/terraform-provider-helm)
3。使用 [helmfile](https://github.com/roboll/helmfile) 命令行界面

在这篇文章中，我们将探讨[**helm file**](https://github.com/roboll/helmfile)**，**，我们已经成功地将其用于基础架构应用部署。

# Helmfile

正如有人提到的， [helmfile 就像是给你掌舵的一个舵](https://medium.com/@naseem_60378/helmfile-its-like-a-helm-for-your-helm-74a908581599)！它允许我们部署舵图表，正如我们将在下面的章节中看到的。

# 基本的 helmfile 结构

下面我们将看到的是一个受 [**Cloud Posse 的 GitHub repo**](https://github.com/cloudposse/helmfiles) 启发的结构。我们有一个主 helmfile，它包含了我们想要部署的发布列表(helm charts)。

**helmfile.yaml**

```
---
# Ordered list of releases.
helmfiles:
  - "commons/repos.yaml"
  - "releases/nginx-ingress.yaml"
  - "releases/kube2iam.yaml"
  - "releases/cluster-autoscaler.yaml"
  - "releases/dashboard.yaml"
  - "releases/external-dns.yaml"
  - "releases/kube-state-metrics.yaml"
  - "releases/prometheus.yaml"
  - "releases/thanos.yaml"
  - "releases/fluent_bit.yaml"
  - "releases/spinnaker.yaml"
  - "releases/velero.yaml"
```

每个发布文件(基本上是一个子 helmfile)看起来像这样:

**发布/ngins-ingress.yaml**

```
---
bases:
  - ../commons/environments.yaml
---
releases:
- name: "nginx-ingress"
  namespace: "nginx-ingress"
  labels:
    chart: "nginx-ingress"
    repo: "stable"
    component: "balancing"
  chart: "stable/nginx-ingress"
  version: {{ .Environment.Values.helm.nginx_ingress.version }}
  wait: true
  installed: {{ .Environment.Values.helm | getOrNil "nginx_ingress.enabled" | default true }}
  - name: "controller.metrics.enabled"
    value: {{ .Environment.Values.helm.nginx_ingress.metrics_enabled }}
```

如果你看上面，你会看到我们正在模板化 [nginx-ingress](https://github.com/helm/charts/tree/master/stable/nginx-ingress) 图的舵部署。实际值(如图表版本、启用的指标)来自环境文件:

**commons/environments . YAML**

```
environments:
  default:
    values:
      - ../auto-generated.yaml
```

其中 **auto-generated.yaml** 是一个简单的 yaml 文件，包含以下值:

```
helm:
  nginx_ingress:
   installed: true
   version: 1.17.1
   metrics_enabled: true velero:
    installed: true
    s3_bucket: my-s3-bucket
    ...
```

注意，在上面的例子中，我们使用了一个 env(名为 *default* ，但是你可以有更多的 env(例如*dev*/*stage*/*prod*)并且在它们之间切换。每个环境都有自己的一组值，允许您进一步定制部署，我们将在下一节中看到。

有了这些，我们就可以开始在我们的 Kubernetes 集群中部署舵图了:

```
$ helmfile sync
```

这将开始在 Kubernetes 集群中部署基础架构应用程序。作为额外的奖励，**图表将按照主 helmfile** 中提供的顺序安装。如果您的图表依赖于另一个要先安装的图表(例如 kube2iam 之后的外部 dns ),这将非常有用。

# 释放 helmfile 的模板化能力

最有趣的 *helmfile* 特性之一是能够使用**模板化舵图表值**(这是*舵*所缺乏的特性)。你在 helmfile 中看到的一切都可以被模板化。让我们以[集群自动缩放器](https://github.com/helm/charts/tree/master/stable/cluster-autoscaler)为例。假设我们想在两个 Kubernetes 集群中部署这个图表:一个位于 AWS 中，另一个位于 Azure 中。因为集群自动缩放器与云 API 挂钩，所以我们需要根据云提供商定制图表值。例如，AWS 需要一个 IAM 角色，而 Azure 需要一个*azureClientId*/*azureClientSecret*。让我们看看如何用 *helmfile* 实现这个行为。

请注意，您也可以在[GitHub helm file-examples repo](https://github.com/costimuraru/helmfile-examples/tree/master/templatization)中找到以下示例。

**cluster-auto scaler . YAML**

```
---
bases:
  - envs/environments.yaml
---
releases:
- name: "cluster-autoscaler"
  namespace: "cluster-autoscaler"
  labels:
    chart: "cluster-autoscaler"
    repo: "stable"
    component: "autoscaler"
  chart: "stable/cluster-autoscaler"
  version: {{ .Environment.Values.helm.autoscaler.version }}
  wait: true
  installed: {{ .Environment.Values | getOrNil "autoscaler.enabled" | default true }}
  set:
    - name: rbac.create
      value: true{{ if eq .Environment.Values.helm.autoscaler.cloud "aws" }}
    - name: "cloudProvider"
      value: "aws"
    - name: "autoDiscovery.clusterName"
      value: {{ .Environment.Values.helm.autoscaler.clusterName }}
    - name: awsRegion
      value: {{ .Environment.Values.helm.autoscaler.aws.region }}
    - name: "podAnnotations.iam\\.amazonaws\\.com\\/role"
      value: {{ .Environment.Values.helm.autoscaler.aws.arn }}{{ else if eq .Environment.Values.helm.autoscaler.cloud "azure" }}
    - name: "cloudProvider"
      value: "azure"
    - name: azureClientID
      value: {{ .Environment.Values.helm.autoscaler.azure.clientId }}
    - name: azureClientSecret
      value: {{ .Environment.Values.helm.autoscaler.azure.clientSecret }}
{{ end }}
```

正如您在上面看到的，我们正在为集群自动缩放图表模板化 helm 部署。基于所选的环境，我们最终会得到 AWS 或 Azure 的特定值。

**envs/environments.yaml**

```
environments:
  aws:
    values:
      - aws-env.yaml
  azure:
    values:
      - azure-env.yaml
```

**env/aws-env.yaml**

```
helm:
  autoscaler:
   cloud: aws
   version: 0.14.2
   clusterName: experiments-k8s-cluster
   aws:
     arn: arn:aws:iam::00000000000:role/experiments-k8s-cluster-autoscaler
     region: us-east-1
```

**env/azure-env.yaml**

```
helm:
  autoscaler:
    cloud: azure
    version: 0.14.2
    azure:
      clientId: "secret-value-here"
      clientSecret: "secret-value-here"
```

现在，为了选择所需的云提供商，我们可以运行:

```
helmfile --environment aws sync
# or 
helmfile --environment azure sync
```

## 将整个值文件模板化

您可以更进一步，通过使用. gotmpl 文件，对整个值文件进行模板化。

```
releases:
  - name: "velero"
    chart: "stable/velero"
    values:
      - velero-values.yaml.gotmpl
```

你可以在[GitHub helm file-examples repo](https://github.com/costimuraru/helmfile-examples/tree/master/gotmpl)上看到一个例子。

## 对 repos 文件使用模板化

我们可以定义一个舵回购列表，从中获取舵图表。稳定的库是一个明显的选择，但是我们也可以添加私有的 helm 库。有趣的是，我们可以使用模板化来提供凭证。

**commons/repos.yaml**

```
---
repositories:
  # Stable repo of official helm charts
  - name: "stable"
    url: "[https://kubernetes-charts.storage.googleapis.com](https://kubernetes-charts.storage.googleapis.com)"
  # Incubator repo for helm charts
  - name: "incubator"
    url: "[https://kubernetes-charts-incubator.storage.googleapis.com](https://kubernetes-charts-incubator.storage.googleapis.com)" # Private repository, with credentials coming from the env file
  - name: "my-private-helm-repo"
    url: "[https://my-repo.com](https://my-repo.com)"
    username: {{ .Environment.Values.artifactory.username }}
    password: {{ .Environment.Values.artifactory.password }}
```

为了保护你的凭证， *helmfile* 支持注入环境秘密值。更多详情，请点击[这里](https://github.com/roboll/helmfile#environment-secrets)。

# 无瓷砖

关于 helm，困扰我们的一个问题是，它需要在 Kubernetes 集群中安装 Tiller 守护进程(在进行任何 helm chart 部署之前)。提醒你一下，helm 有两个部分:一个客户端(helm)和一个服务器(tiller)。除了[安全问题](https://engineering.bitnami.com/articles/helm-security.html)之外，为了设置角色、服务帐户和进行实际的 Tiller 安装，也增加了复杂性。

好消息是 *helmfile* 可以设置为在**无平铺模式**下运行。启用后，helmfile 将使用[舵柄](https://github.com/rimusz/helm-tiller)插件，该插件将基本上在本地运行舵柄。不再需要在 Kubernetes 集群中安装 *tiller* 守护进程。如果您想知道当多人试图使用这种无 tiler 方法安装图表时会发生什么，那么您不应该担心，因为掌舵状态信息仍然存储在 Kubernetes 中，作为所选名称空间中的秘密。

```
helmDefaults:
  tillerNamespace: helm
  tillerless: true
```

在*无舵*模式下运行时，可以这样列出已安装的图表:

```
helm tiller run helm -- helm list
```

如果你想了解更多关于这个插件的信息，这里有一篇来自主要开发者的非常酷的文章。我们也期待着 [**helm3**](https://github.com/helm/helm/releases) 的发布，这将彻底摆脱蒂勒。

# 结论

Helmfile 完成了它的工作，而且做得很好。它有强大的社区支持，模板化的力量只是*哇*。如果你还没有，那就试一试:[https://github.com/roboll/helmfile](https://github.com/roboll/helmfile)