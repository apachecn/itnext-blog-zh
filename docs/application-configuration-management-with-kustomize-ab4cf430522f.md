# 使用 Kustomize 进行应用程序配置管理

> 原文：<https://itnext.io/application-configuration-management-with-kustomize-ab4cf430522f?source=collection_archive---------6----------------------->

[![](img/592533fc61bb0c1bc53d12c99b1f384b.png)](https://www.giantswarm.io/)

在上一篇文章中，我们探讨了赫尔姆的优点，这一次我们将把注意力转向[的 Kustomize](https://kustomize.io/) 。

在我们揭开 Kustomize 的盖子，看看它能为我们做什么之前，让我们花点时间看看它是从哪里来的。Kustomize 是谷歌在 2018 年年中[宣布](https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/)的一个开源项目，主要是受到 Kubernetes 缺乏可信的[声明式应用管理](https://docs.google.com/document/d/1cLPGweVEYrVqQvBLJg6sxV-TrE5Rm2MNOBA_cxZP2WU/edit)解决方案的启发。它避开了 Helm 用来呈现资源的模板方法，而是专注于修补和覆盖现有的配置。

# kustomize——基本原理

Kustomize 与 Helm 分享了很多灵感；那就是寻求为应用程序提供配置，但是能够定制配置以适合特定的目的或环境。它旨在为定制应用程序和那些被归类为普通现成(COTS)应用程序的应用程序提供这一功能。

它与 Helm 方法的不同之处在于它坚持使用 YAML 进行定制定义，而不是一种深奥的模板语言。打赌使用 Kubernetes 的 DevOps 人员已经熟悉了它的 API 资源和用于定义它们的 YAML 语法。至少在理论上，熟悉有助于无痛苦的采用。

Kustomize 可以向大量不同的资源(例如，标签或注释)添加一个公共字段，修改现有字段的值(例如，部署副本的数量)，并部分修补作为“基本”配置提供的资源。它甚至可以从规范的源定义中生成 ConfigMap 和 Secret 资源配置，稍后将详细介绍。

# Kustomize 是如何工作的？

Kustomize 的工作方式是从一组现有定义和 kustomize . YAML 文件中定义的新配置构建定制的资源定义。

让我们解释一下这是如何工作的。

# 调用 Kustomize

首先，要调用定制配置的构建，有两种方法可用。

1.一个独立的 Kustomize 二进制文件可以与其“构建”或“创建”子命令一起使用。

2.或者，更方便的是，从 Kubernetes v1.14 开始，Kustomize 可以作为 Kubernetes 本地 kubectl CLI 的一个组成部分被调用。

管理员和 CI/CD 工具每天都在使用 kubectl CLI，因此 Kustomize 功能可供每个人使用。然而，请注意，在撰写本文时，Kustomize 的嵌入式版本远远落后于独立版本。在这个延迟问题解决之前，我们建议使用独立的 Kustomize 二进制文件，而不是嵌入式版本。

至少可以说，Kustomize 在 kubectl 中的嵌入是[有争议的](https://groups.google.com/d/msg/kubernetes-sig-architecture/lw8AJXGrEW8/Y3BiTYMFFQAJ)。它的行为就像一个插件，但是绕过了 kubectl [插件机制](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/#writing-kubectl-plugins)，这使得它的可用性是可选的。这为 Kustomize 提供了比 Helm 或其他竞争配置管理解决方案更低的采用门槛。这些往往需要在使用前[安装](https://helm.sh/docs/intro/install/)。鉴于 Kustomize 和 Kubernetes 源自 Google，一些人不难推断 Kustomize 是社区中最有影响力的人青睐的首选方法。这是真是假现在已经不重要了，因为 Kustomize 功能是 kubectl 二进制文件的公认部分。

在处理现有的 Kubernetes 集群时，要查看 kustomize 将通过定制资源生成什么，可以使用以下命令，而不是使用 Kustomize 命令:

```
$ kubectl kustomize <directory>
```

该目录包含 kustomization.yaml 文件和其他资源定义，呈现的内容被发送到 STDOUT 流。如果我们需要将定制的应用程序资源定义应用到集群，以下命令可以实现这一点:

```
$ kubectl apply -k <directory>
```

这就是生成定制资源定义的机制，现在让我们看看定制是如何定义的。

# 定义资源自定义

作为命令调用的一部分指定的目录必须包含 kustomization.yaml 文件。这个文件告诉 Kustomize 如何渲染资源。它将列出将成为定制主题的资源，以及构成定制的任何转换和添加。

```
apiVersion: kustomize.config.k8s.io/v1beta1 kind: Kustomization resources: - deployment.yaml - service.yaml namespace: my-app-ns commonLabels: app.kubernetes.io/name: my-app
```

在上面的示例中，部署和服务资源(在其同名文件中定义)将使用“my-app-ns”名称空间定义以及“app.kubernetes.io/name”标签和“my-app”值进行定制。

所以，Kustomize 允许我们定制基本资源定义，但是我们如何处理多个定制场景而不重复呢？例如，我们如何对开发、试运行和生产环境进行细微的定制？

```
├── base │ ├── deployment.yaml │ ├── kustomization.yaml │ └── service.yaml └── overlays ├── dev │ ├── kustomization.yaml │ └── patch.yaml ├── prod │ ├── kustomization.yaml │ └── patch.yaml └── staging ├── kustomization.yaml └── patch.yaml
```

为了实现这一点，Kustomize 使用一个“基本”配置，可以使用附加 kustomization.yaml 文件中的定义对其进行进一步定制。“覆盖/暂存”目录中 kustomization.yaml 文件的内容如下所示:

```
apiVersion: kustomize.config.k8s.io/v1beta1 kind: Kustomization commonLabels: environment: staging bases: - ../../base/ patchesStrategicMerge: - patch.yaml
```

它引用“基本”目录中的原始定制，然后根据其内容应用进一步的定制。在这种情况下，它添加了另一个公共标签和一个在' patch.yaml '中定义的配置补丁。

这种定义通用配置和覆盖以支持相似但不同场景的技术非常强大。这与我们使用 Helm 模板所能达到的效果没有什么不同。但是，它是用纯 YAML 实现的，不需要定义复杂的参数化模板。此外，按照作者的定义，原始资源文件保持不变。这有助于从依赖的上游应用程序配置定义开始工作。

# 生成配置

Kustomize 的名字非常适合它的用途，使用覆盖层修补配置正是您所期望的。也许你不会想到 Kustomize 也有能力从头生成 Kubernetes API 资源。嗯，可以，但是因为资源有限，而且理由很充分。

Kustomize 可以从文字定义或规范的源文件(环境或常规)生成配置映射和秘密。通常，配置映射和机密是为工作负载强制创建的(使用“kubectl create”)，因此通过在 kustomization.yaml 文件中定义它们的生成，Kustomize 提供了一种更具声明性的方法。

```
apiVersion: kustomize.config.k8s.io/v1beta1 kind: Kustomization configMapGenerator: - name: my-app-config > files: - config.json <snip>
```

这里,' kustomization.yaml 文件根据本地文件' config.json '的内容定义了一个名为' my-app-config '的配置映射。在发出“kubectl apply -k”命令时，将会创建 ConfigMap，并且假设它的使用是在 Pod 模板规范中定义的，则内容随后将可供工作负载使用。

但是，如果配置数据或密码更新了，会发生什么呢？我们如何让工作量认识到变化？这是 Kubernetes 中的一个常见问题，虽然新内容可以通过卷挂载获得，但应用程序可能没有意识到这一点。除非它重新启动。

Kustomize 试图通过创建带有后缀名称的配置映射和机密来解决这种边缘情况。后缀是对象内容的散列，因此每次内容改变时，替换对象的名称也会改变。如果规范数据源被更改，一个“kubectl apply -k”将生成一个具有不同名称的新配置映射或秘密，并且 Pod 模板规范也将得到更新以反映新的对象名称。此外，由于相关的控制器协调循环，它还将导致工作负载更新，这将导致创建新的工作负载来访问修改后的内容。

这个资源生成特性将 Kustomize 扩展到了定制资源定义的纯领域之外，并有助于解决一个其他类似解决方案难以解决的问题。

[![](img/06c9a5f8a32b850e1daf3f3f76556446.png)](https://www.giantswarm.io/on-demand-webinar-kubernetes-for-software-architects?utm_campaign=Blog%20CTA%20Conversion&utm_source=Kubernetes%20for%20Software%20Architects_Blog&utm_medium=Blog%20CTA&utm_term=Kubernetes%20for%20Software%20Architects)

# Kustomize 和 Helm 一起

Kustomize 对于所有者创作的、定制的配置资源非常有用，但是如果您需要使用 COTS 应用程序，会发生什么情况呢？

您不拥有或维护基本资源定义。这些应用程序通常被打包成舵图，而不是库斯托米化配置，并且包含模板定义，而不是纯粹的 YAML。

Kustomize 和 Helm 能一起工作吗，或者 Kustomize 能以某种方式消耗 Helm 图表吗？

简单的回答是肯定的。

Helm 可以使用“helm template”命令和合适的 [values.yaml](https://helm.sh/docs/chart_template_guide/values_files/) 文件强制呈现资源定义。呈现的内容可以重定向到本地文件，Kustomize 可以将这些文件作为基本资源定义。但是，如果 COTS 应用程序不断发展，并且这些变化非常重要，不容忽视，会发生什么呢？

Kustomize 项目鼓励 fork/modify/rebase 工作流，其中上游配置最初被分叉到 Git 存储库。分叉的资源使用“helm template”呈现，根据定义的定制进行修改，并使用 Kustomize 应用。对上游资源的更改可以通过“git rebase”定期同步。

# 结论

在应用程序配置管理方面，Kustomize 是 Helm 的一个非常可靠的替代方案。从它在 [GitHub repo](https://github.com/kubernetes-sigs/kustomize) 上的明星数量、贡献者数量和活跃程度来看，它也非常受欢迎。

> ***但它并没有成为 Kubernetes 中应用配置管理的新灵丹妙药。也许，它从来没有承诺过。***

虽然对于定制应用来说，这是一个很好的解决方案，但对于 COTS 应用来说，它仍然隐含着对更流行的 Helm chart 解决方案的依赖。像 Helm 这样打包和分发应用程序的工具的吸引力并不难估计 Docker 的成功就是这一事实的证明。Kustomize 当然可以成为 Helm 的一个有价值的辅助工具，但它不太可能取代 Helm 成为管理应用程序配置管理的事实工具。最终，当 Helm 兑现其用 [Lua 脚本](https://sweetcode.io/a-first-look-at-the-helm-3-plan/)取代模板的承诺时，它也可能会失去一些流通性。只有时间能证明一切。

由[Puja Abbas si](https://twitter.com/puja108)——开发者倡议@ [巨型虫群](https://giantswarm.io/)撰写