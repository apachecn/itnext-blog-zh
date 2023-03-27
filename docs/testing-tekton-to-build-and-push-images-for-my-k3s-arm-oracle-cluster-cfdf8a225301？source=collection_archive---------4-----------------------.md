# 测试 Tekton 为我的 K3S ARM Oracle 集群构建和推送映像

> 原文：<https://itnext.io/testing-tekton-to-build-and-push-images-for-my-k3s-arm-oracle-cluster-cfdf8a225301?source=collection_archive---------4----------------------->

![](img/0cc94176c15eca7675cebee0b8eeb916.png)

## 介绍

在本文中，我们将探讨如何部署和配置 tekton 来构建映像并将其推送到您的注册表，以便从您的集群中使用，我们还将在另一篇文章中了解如何部署这些映像。在这篇文章中，我想向您展示如何准备好使用映像，以及一个无需依赖外部因素的 CI 系统的便捷解决方案，在我的案例中，我在 docker 构建跨架构映像时遇到了问题，在设置 tekton 后，一切都变得更快、更简单，跨架构默认情况下很慢，但也不能如您所预期的那样 100%地工作。 通过使用这种方法，我们可以忘记架构，只构建我们运行的东西，它肯定更快，甚至你的一些节点已经有可用的映像，这意味着更少的带宽消耗
从长远来看也是如此。

下面列出了我们将要测试的项目的源代码和/或文档:

*   [tr](https://github.com/kainlite/tr) ，去看看吧，我的新博客就在那里:[https://tr . tech squad . rocks](https://tr.techsquad.rocks)你可以在`manifests`文件夹中查看这里使用的清单。

下面列出了我们将要测试的项目的源代码和/或文档:

*   [泰克顿](https://tekton.dev/docs/)
*   tekton-tri 公司
*   [kaniko](https://github.com/GoogleContainerTools/kaniko)

## 安装 tekton 管道和 tekton 触发器

为什么我们还需要泰克顿管道或泰克顿触发器？管道允许你按顺序运行多个任务并传递东西(这是 tekton 和任何 CI/CD 系统的基础)，然后我们需要做一些事情，例如当我们推送到我们的 git 库时，这就是 tekton-triggers 变得方便的时候，让我们对变化做出反应并触发一个构建或一些过程，拦截器是 tekton-triggers 的一部分，让我们说它给你使用事件的灵活性。

然后我们需要在本地安装`tkn`,并从 hub 配置一些包

在我的部署中，我使用了推荐用于任何“生产”部署的固定版本，你可以在这里看到自述文件。

## 让我们进入正题

**泰克顿管道**

好了，我们已经安装了 tekton 和 friends，可以开始工作了，但是现在呢？嗯，这有点棘手，需要一些清单才能开始，所以我将尝试解释每个文件发生了什么，以及我们为什么需要它们。

你也可以在 github 中看到这个文件 [01-pipeline.yaml](https://github.com/kainlite/tr/blob/master/manifests/tekton/pipelines/01-pipeline.yaml) ，基本上我们需要定义一个管道来定义步骤和将要发生的事情，这里我们正在克隆存储库，然后用 kaniko 构建它，然后将其推送到 docker 注册表，请注意，脚本是硬编码的，可以是动态的，但对我的用例来说并不是真正必要的。

你也可以在 github 中看到这个文件 [02-pipeline-run.yaml](https://github.com/kainlite/tr/blob/master/manifests/tekton/pipelines/02-pipeline-run.yaml) ，这基本上是运行我们定义的具有特定值的管道，我们将使用触发器中非常类似的东西来自动运行，当我们将提交推送到我们的 repo 时，docker secret 是一个常规的 dockercfg secret 挂载，因此我们可以推送到那个注册表。

有了所有这些，我们有了一个基本的管道，但我们需要手动触发或运行它，让我们为它添加必要的清单，以对 github 存储库中的更改做出反应

**tekton 触发器**

你也可以在 github 中看到这个文件 [01-rbac.yaml](https://github.com/kainlite/tr/blob/master/manifests/tekton/triggers/01-rbac.yaml) ，让我们给 tekton-triggers 一些权限

你也可以在 github 上看到这个文件 [02-eventlistener.yaml](https://github.com/kainlite/tr/blob/master/manifests/tekton/triggers/02-eventlistener.yaml) ，这是事情变得有点棘手的地方，理论上你不需要一个秘密来读取你的回购，如果它是公开的，但当我开始测试它时，它是私有的，然后它是公开的，如果你对这个 yaml 下面的秘密检查的格式感兴趣，但是它只“监听”我们回购中的事件，并使用我们的管道触发事件，我们仍然需要一个 webhook 和其他配置的入口，正如我们将在

这个秘密类似于下面描述的，用你生成的令牌替换`secretToken`这将用于 webhook 配置，所以把它保存在一个安全的地方，直到它在那里被配置。

你也可以在 github 上看到这个文件 [04-triggerbinding.yaml](https://github.com/kainlite/tr/blob/master/manifests/tekton/triggers/04-triggerbinding.yaml) ，当我们收到 webhook 时，我们可以从中获得一些信息，基本上我们感兴趣的是回购 URL 和提交 SHA。

你也可以在 github 中看到这个文件 [05-triggertemplate.yaml](https://github.com/kainlite/tr/blob/master/manifests/tekton/triggers/05-triggertemplate.yaml) ，这相当于我们手动运行的 pipelinerun，但是它使用触发器和模板来自动触发，因此有相似之处。

你也可以在 github 上看到这个文件 [06-ingress.yaml](https://github.com/kainlite/tr/blob/master/manifests/tekton/triggers/06-ingress.yaml) ，最后但并非最不重要的是 ingress 配置，没有它它不会工作，因为我们需要从 github 接收一个请求，要配置它，只需转到存储库上的设置，点击 webhooks 并使用你生成的秘密令牌创建一个新的，并将你的 URL 设为`https://subdomain.domain/hooks`，然后将 TLS 标记为 on，仅 push 和 active。

咻！这需要大量的工作，但相信我，这是值得的，现在您可以从您的集群中构建、推送和运行您的映像，无需外部或奇怪的 CI/CD 系统，一切都遵循 GitOps 模型，因为一切都可以从您的存储库中提交和应用，在我的情况下，我使用 ArgoCD 和 Kustomize 来应用一切，但这将是另一个章节。

我们已经准备好了事件监听器:

我们有管道，请注意它说失败，这是因为 ARM 有一个问题尚未解决，但实际上一切都按预期工作:

我们可以看到 pipelinerun 被触发，与之前描述的问题相同，请参见 github 问题的注释:

我们还可以看到为 tekton 创建的一些其他资源:

您还可以使用`kubectl`或`tkn`查看创建的窗格或日志:

我希望这对某些人有用，如果您的 CI/CD 系统有问题，请尝试 tekton，您会喜欢它，在我的特定情况下，我在 ARM 和构建它时遇到了许多问题，它很慢，有很多奇怪的错误，所有这些都通过构建我运行的映像而消失了，它更快，还利用了空闲的计算能力。

## 一些来源和已知问题

这篇文章深受这些文章的启发，并按照以下示例进行了配置和测试:

[](https://github.com/tektoncd/triggers/blob/main/docs/getting-started/README.md) [## 主 tektoncd/triggers 上的 triggers/README.md

### 下面的教程将引导您使用 Tekton 触发器来构建和部署 Docker 映像，以检测 GitHub…

github.com](https://github.com/tektoncd/triggers/blob/main/docs/getting-started/README.md)  [## 使用 Tekton 构建和推送映像

### 使用 Kaniko 和 Tekton 创建一个管道来获取源代码、构建和推送图像。本指南将向您展示如何…

tekton.dev](https://tekton.dev/docs/how-to-guides/kaniko-build-push/#full-code-samples) [](https://www.arthurkoziel.com/tutorial-tekton-triggers-with-github-integration/) [## 教程:Tekton 触发器与 GitHub 集成

### 在之前的博客文章中，我们已经使用 Tekton 管道建立了一个简单的管道来运行测试，构建 docker 映像…

www.arthurkoziel.com](https://www.arthurkoziel.com/tutorial-tekton-triggers-with-github-integration/) 

在 ARM 上运行会有一些问题，在其他架构上它只是工作，查看更多:

[](https://github.com/tektoncd/pipeline/issues/4247) [## fork/exec /busybox/mkdir: exec 格式错误问题#4247 tektoncd/pipeline

### 此时您不能执行该操作。您已使用另一个标签页或窗口登录。您已在另一个选项卡中注销，或者…

github.com](https://github.com/tektoncd/pipeline/issues/4247) [](https://github.com/tektoncd/pipeline/issues/5233) [## docker-build 任务失败，出现 exec 格式错误问题#5233 tektoncd/pipeline

### 预期行为能够构建 docker 映像实际行为无法运行 docker-2022/07/28 构建步骤失败…

github.com](https://github.com/tektoncd/pipeline/issues/5233) 

但是一切都应该正常工作。

# 正误表

如果您发现任何错误或有任何建议，请给我发消息，以便解决问题。

此外，您还可以在这里查看源代码以及[生成代码](https://github.com/kainlite/kainlite.github.io)和[源代码](https://github.com/kainlite/blog)的变化。

*原载于 2022 年 10 月 25 日*[*https://tech squad . rocks*](https://techsquad.rocks/blog/testing_tekton_to_build_and_push_images_for_my_k3s_arm_oracle_cluster/)*。*