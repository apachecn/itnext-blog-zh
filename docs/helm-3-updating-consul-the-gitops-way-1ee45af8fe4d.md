# 头盔 3-更新执政官，吉托普斯之路

> 原文：<https://itnext.io/helm-3-updating-consul-the-gitops-way-1ee45af8fe4d?source=collection_archive---------4----------------------->

![](img/aa0cde89b779f091bda119d2e175867f.png)

如果你读过我以前的文章，你就会知道我是赫尔姆的忠实粉丝。Kubernetes 的“包管理器”通常被称为是一个强大的工具，它可以做的远不止是生产 Kubernetes 元数据。因为 Helm 是将您的工作负载打包交付给 Kubernetes 的引擎，为什么不将其他运营流程作为 Helm 升级的一部分加入其中呢？

从 GitOps 的角度考虑 CI/CD 流程，通常会采用多个阶段。以下是作为 mill GitOps CI/CD 管道运行的一部分完成的操作的一般列表(给出或采取一些步骤):

## 海峡群岛

*   将代码/数据库更改脚本/配置更改签入源代码控制。
*   构建代码
*   打包代码

## 激光唱片

*   进行环境变量或配置更新
*   应用一些数据库更改
*   将打包的代码部署到环境 X 中

环境变量和配置更改通常(但不总是)与新功能的发布相关联，数据库更改也是如此。当然，有时例行的密码更新和/或数据库更改是独立于软件发布进行的，但现在让我们保持简单。

此外，您的代码和配置*应该*存储在一个源代码控制系统中，比如 Git、Subversion 或 Mercurial。有很多原因可以解释为什么这被认为是最佳实践，但是这是另一篇文章的主题。请记住不要在 Git 中存储用户名、密码或其他敏感信息，这是另一篇文章的主题。由于 Helm 图表通常也存储在 Git 中，通常带有针对不同环境配置的特定分支或文件夹，因此我们可以利用 Helm 的强大功能将所有这些配置更改应用到我们的环境中。

# 领事是什么？

关于[领事](https://www.consul.io/)，我就不赘述了，不过，领事是 Hashicorp 这样描述的:

> Consul 是一个服务网格解决方案，提供具有服务发现、配置和分段功能的全功能控制平面。这些功能可以根据需要单独使用，也可以一起使用来构建一个完整的服务网格。

为了集成到 Helm 3 中，我们将重点介绍 Consul 的配置特性，称为 Consul KV store，Hashicorp 对它的描述如下:

> **KV Store** :应用程序可以将 Consul 的层次化键/值存储用于任何目的，包括动态配置、特性标记、协调、领导者选举等等。简单的 HTTP API 使其易于使用。

因此，Consul 的 KV store 是一个允许我们通过 REST API 改变配置的工具。这是我们所有应用程序获取配置数据的中心位置。通过这样做，我们的应用程序代码保持通用，我们在每个环境中都有一个中心位置来查看配置参数，并且我们可以在构建管道中更新我们的配置，因为我们将它存储在源代码控制中。

# 我们如何更新领事？

如前所述，Consul 有一个 REST API，允许我们通过许多工具更新它的配置。为了这篇文章的目的，也为了保持内容的简洁，我们将使用[gon sul](https://github.com/miniclip/gonsul)by*99*。

Gonsul 是一个实用程序，它从源代码控制系统(在本例中是 Git)中获取任意文件夹，并对存储在 Git 中的配置和指定的 Consul KV 存储路径中的配置进行区分。任何新添加的条目被添加，现有条目被变异，删除的条目可选地通过 Gonsul 二进制的标志被删除(见- *允许-删除*)。

Gonsul 的伟大之处在于，它既可以从 Git repo 中检查配置文件，也可以向它提供文件列表，它将对目录进行操作。

在我与 Consul 的真实用例中，我选择在我的 Helm chart Git 存储库中保留每个环境的 Consul 配置，并将这些配置文件提供给 Gonsul，而不需要 it 检查它们

# 头盔 3——把所有的东西放在一起

总而言之，我们需要以下设置:

*   一个 Git 存储库，包含我们的舵图，以及每个环境的配置文件夹。下面我们看到两个图表，一个用于生产，一个用于暂存。这些图表目录中的每一个都包含配置文件的子目录，这些配置文件包含为 Consul 设置的 KV store 条目。请注意，您的存储库不必看起来像这样，并且根据您的项目，可能会呈现完全不同的结构。我们也有一个*咨询-更新-配置图*。这个 ConfigMap 只是将 config 文件夹中的文件映射到我们的作业对象稍后将使用的 ConfigMap 中。关于如何创建这样的配置图的更多细节，请参见本文。

```
**+--helmcharts
   |
   +--config_charts
      |
      +--staging
         |
         +--templates
            |
            +--consul-updater-configmap.yaml
            +--...
         +--config
            |     
            +--config1.yaml
            +--config2.json
            +--config3.yaml
         +--values.yaml
         +--Chart.yaml   
      +--production
         |
         +--templates
            |
            +--consul-updater-configmap.yaml
            +--...
         +--config
            |     
            +--config1.yaml
            +--config2.json
            +--config3.yaml
         +--values.yaml
         +--Chart.yaml**
```

上面我们看到两个图表，一个用于生产，一个用于暂存。这些图表目录中的每一个都包含配置文件的子目录，这些配置文件包含为 Consul 设置的 KV store 条目。请注意，您的存储库不必看起来像这样，并且根据您的项目，可能会呈现完全不同的结构。

*   安装了 Gonsul 二进制文件的容器。我选择构建自己的容器，因为我通常这样做是为了保持环境的一致性，但是您可以只使用 *mincliposs* 图像:

```
**docker pull minicliposs/gonsul:latest**
```

*   将所有配置文件映射到 Gonsul 容器的配置映射。
*   最后，我们将利用 Kubernetes 作业来运行 Gonsul 作业:

```
**apiVersion: v1
kind: Job
metadata:
  name: consul-updater
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install, post-upgrade, post-rollback
    "helm.sh/hook-delete-policy": hook-succeeded, hook-failed, before-hook-creation
  labels:
    app: consul-updater    
spec:
  template:
    metadata:
      name: consul-updater
    spec:
      restartPolicy: Never
      volumes:
        - name: consul-config
          configMap:
            name: consul-updater-configmap
      containers:
        - name: gonsul
          image: minicliposs/gonsul:latest
          volumeMounts:
            {{ range $path, $bytes := .Files.Glob ( printf "config/**" ) }}
            {{ $name := base $path }}
            - name: consul-config
              mountPath: {{ printf "/path/to/files/%s/%s" (index (regexSplit "config" (dir $path) -1) 1) $name | indent 2 }}
              subPath: {{- sha256sum (printf "%s/%s" (index (regexSplit "config" (dir $path) -1) 1 ) $name ) | indent 2 }}
            {{ end }}
          command:
            - /bin/bash
            - -c
            - /usr/local/bin/gonsul --repo-root=/path/to/files/ --consul-url=**[**http://consul:8500**](http://consul:8500) **--allow-deletes=skip --input-ext=json,txt,yaml**
```

这份工作有一些需要注意的地方。让我们来分解一下:

1.  这项工作使用了舵钩。这些挂钩将在升级或安装过程的不同阶段运行。这些阶段是使用*helm.sh/hook*注释定义的
2.  我们定义一个删除策略。赫尔姆离开后，我们不希望这份工作继续存在
3.  在 VolumeMounts 部分有一些复杂的逻辑。这允许我们将文件目录映射到容器中。关于这方面的更多细节，我有[一篇文章解释为什么你会想这么做。](/helm-3-mapping-a-directory-of-files-into-a-container-ed6c54372df8)
4.  最后，我们有命令。这是运行 Gonsul 工具的入口点。你也可以选择将它添加到 docker 文件中，但是我喜欢将它添加到舵图中，因为在这里进行微调通常比构建另一个图像来测试旗帜变化更简单。通过指定一个 *repo-root* ，Gonsul 将不会尝试获取 Git 存储库，但是如果它在您的管道中工作得更好，您也可以选择这样做。

# 优势和总结

使用 Helm 更新 Consul 有一些显著的优势，即:

1.  Helm 管理您的整个升级操作，包括配置更新。
2.  由于 Helm 管理升级，如果您使用海图博物馆，配置将被捆绑到 Helm 海图中并被加工。这支持下面的 3 和 4。
3.  如果在配置升级过程中出现问题，Helm 可以设置为将您的部署回滚到上一个稳定状态
4.  **如果升级成功，但在生产中出现问题，Helm 回滚操作也将回滚您的配置(如果已经用*后回滚* Helm hook)**

第四点可能是选择这条路线的首要原因。试图找出哪一组配置参数与以前的版本相关联是一件非常浪费时间的事情，尤其是当您面临恢复一个损坏的环境(比如生产环境)的压力时。通过将配置及其发布和回滚过程联系在一起，可以获得回滚到已知状态的信心和自动化程度。

通过使用 Helm 来实现基本 Kubernetes 部署以外的自动化，可以将更多的责任转移到 Helm 升级过程中，并且在 CI 管道中需要的定制脚本更少。总而言之，这有助于我晚上睡得更好，希望其他人也能从这种方法中受益。