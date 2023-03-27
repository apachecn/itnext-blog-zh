# 使用 Tekton 构建云本机 CI/CD—构建定制任务

> 原文：<https://itnext.io/cloud-native-ci-cd-with-tekton-building-custom-tasks-663e63c1f4fb?source=collection_archive---------5----------------------->

## 了解如何使用 Tekton 管道在 Kubernetes 上为云原生 CI/CD 使用、构建和部署定制任务…

![](img/9fd1f7e12025b3380e74f8d248ee90f7.png)

在本文中，我们将继续我们在[上一篇文章](/cloud-native-ci-cd-with-tekton-laying-the-foundation-a377a1b59ac0)中停止的地方，在那篇文章中，我们部署了我们的 Tekton Pipelines 环境，我们将详细探讨如何找到、构建和定制 Tekton 任务，以便为我们的管道创建所有必要的构建模块。除此之外，我们还将研究如何维护和测试我们新构建的任务，同时使用所有最佳实践来创建可重用、可测试、结构良好且简单的任务。

如果您还没有这样做，那么请查阅上一篇文章，让您的 Tekton 开发环境启动并运行，这样您就可以按照本文中的示例进行操作了。

*注:本文使用的所有代码和资源都可以在* `[*tekton-kickstarter*](https://github.com/MartinHeinz/tekton-kickstarter)` [*资源库*](https://github.com/MartinHeinz/tekton-kickstarter) 中获得

# 什么是任务？

Tekton *任务*是管道的基本构件。任务是执行某个特定任务的一系列步骤。任务中的每个步骤都是任务窗格中的一个容器。将相关步骤的这种序列隔离到单个可重用的任务中，为 Tekton 提供了很大的通用性和灵活性。它们可以像运行单个`echo`命令一样简单，也可以像构建 Docker 然后通过映像摘要输出推入注册表一样复杂。

除任务外，*集群任务*也可用。它们与基本任务没有太大区别，因为它们只是集群范围的任务。这些对于执行基本操作的通用任务非常有用，比如克隆存储库或运行`kubectl`命令。使用 ClusterTasks 有助于避免代码重复，并有助于提高可重用性。但是要注意对集群任务的修改，因为对它们的任何更改都可能影响集群中所有其他名称空间中的许多其他管道。

当我们想要执行一个任务或集群任务时，我们创建 *TaskRun* 。在编程术语中，你也可以把一个任务看作一个类，把 TaskRun 看作它的实例。

如果上面的解释不够清楚，那么一个小例子可能会有所帮助。下面是我们可以创建的最简单的任务:

这个名为`echo`的简单任务确实做到了——它运行一个`ubuntu`容器，并向其中注入执行`echo 'Hello world!'`的脚本。现在我们有了一个任务，我们也可以运行它，或者换句话说，创建 TaskRun。我们可以为此创建一个 YAML 文件并应用它，或者我们也可以使用`tkn` CLI:

就这么简单！我们已经运行了我们的第一个任务，现在让我们转到一些更有用的东西，并探索已经存在的任务…

# 不要重新发明轮子

这篇文章是关于创建和定制 Tekton 任务的，但是我们不要试图在这里重新发明一个轮子。相反，让我们使用 Tekton 社区已经创建的内容。可以使用的现有任务的主要来源是 *Tekton 目录*。这是一个由 Tekton 维护者审查的可靠的、有组织的任务的存储库。除了 Tekton 目录存储库，您还可以使用 [Tekton Hub](https://hub.tekton.dev/) ，它列出了所有与目录相同的任务，但在导航视图上更容易一些。它还列出了每个任务的等级，这可能是一个有用的质量指标。

在这个目录中，您应该能够找到所有基本的东西，比如获取存储库(`git-clone`)、构建和推送 Docker 映像(`kaniko`或`buildah`)或发送 Slack 通知(`send-to-webhook-slack`)的任务。因此，在您决定构建自定义任务之前，请尝试检查目录中常见问题的现有解决方案。

如果您浏览了 Tekton 目录或 Tekton hub，您可能会注意到每个任务的安装只是一个单独的`kubectl apply -f ...`。这很简单，但是如果您依赖于其中的许多任务，并且希望在版本控制中跟踪它们，而不是复制粘贴它们的所有 YAML，那么您可以使用`[tekton-kickstarter](https://github.com/MartinHeinz/tekton-kickstarter)` [存储库](https://github.com/MartinHeinz/tekton-kickstarter)中的方便脚本，它将获取远程 YAML URL 的列表，例如:

通过调用`make catalog`，将它们应用到集群中。

# 布局

在我们开始定制任务之前，确定布局是个好主意，这将使它们易于导航、测试和部署。我们可以从 Tekton 目录存储库中获得一些灵感，并使用以下目录结构:

我们将所有任务存储在一个名为`tasks`的目录中。在这里，我们为每个任务创建一个目录，其中包含一个包含任务本身的 YAML 文件和一个包含测试所需资源的目录(`tests`)。这些将是在`run.yaml`中运行的任务，以及在`resources.yaml`中执行测试所需的任何额外资源。例如，可以是用于执行数据库备份任务的 PVC，或者用于执行应用程序扩展任务的部署。

我们在前面提到的结构中显示的另一个文件是`catalog.yaml`，它包含要从远程安装的任务列表。

为了方便起见(如果使用`tekton-kickstarter`)，所有这些都可以用一个命令安装，这个命令是`make deploy-tasks`，它遍历`tasks`目录并将所有任务应用到您的集群，同时忽略所有测试资源。

# 建造定制的

如果你在目录中找不到合适的工作任务，那么是时候自己写了。在文章的开头，我展示了非常简单的“Hello world”示例，但是 Tekton 任务可能会变得更加复杂，所以让我们来看一下我们可以利用的所有配置选项和功能。

让我们从简单的开始，介绍我们在构建任何任务时都需要的基础知识。这些是任务步骤的任务参数和脚本:

使用上面的`deploy`任务，我们可以执行 Kubernetes 部署的简单展示，方法是在`name`参数中为它提供部署的名称，并可选地提供它所在的`namespace`。我们传递给任务的这些参数在脚本执行前在`script`部分被展开。为了告诉 Tekton 扩展参数，我们使用了`$(params.name)`符号。这也可以用于 spec 的其他部分，而不仅仅是脚本。

现在，让我们仔细看看`script`部分——我们从 *shebang* 开始，以确保我们将使用`bash`。然而，这并不意味着你总是必须使用`bash`，就像你可以使用 Python 和`#!/usr/bin/env python`一样，这完全取决于你的偏好和所用图像中的可用内容。在 shebang 之后，我们还使用了`set -xe`,它告诉脚本回显每个正在执行的命令——你不必这样做，但这在调试过程中非常有帮助。

或者，如果您不需要整个脚本，而只需要一个命令，那么您可以用`command`替换`script`部分。这就是使用`kubectl wait`执行应用程序健康检查的简单任务的情况(注意:对于剩余的示例，省略任务体中明显的/不相关的部分):

这与 pods 中的`command`指令的工作方式是一样的，所以如果你有很多参数，它会变得冗长和难以阅读。出于这个原因，我更喜欢在几乎所有事情上使用`script`，因为它可读性更好，也更容易更新/更改。

任务中可能需要的另一个常见东西是某种类型的存储，您可以在其中写入数据，供任务中的后续步骤或管道中的其他任务使用。最常见的用例是[获取 git repo](https://github.com/tektoncd/catalog/tree/master/task/git-clone/0.2) 的地方。这种存储在 Tekton 中称为`workspace`，以下示例显示了使用`rmdir`装载和清除存储的任务:

上面的`clean`任务包括`workspace`部分，该部分定义了工作空间的名称和它应该被挂载的路径。为了使更新`mountPath`更容易，Tekton 提供了一个`$(workspaces.ws-name.path)`格式的变量，可以在脚本中使用它来引用路径。

因此，定义工作区非常简单，但是支持工作区的磁盘不会凭空出现。因此，当我们执行要求工作空间的任务时，我们还需要为它创建 PVC。为上述任务创建所需 PVC 的任务运行如下:

工作区非常通用，因此不仅可以用于在任务/管道运行期间存储一些临时数据，还可以用作长期存储，如下例所示:

该任务可以使用`pg_dump`实用程序执行 PostgreSQL 数据库备份。在本例中，脚本从参数中指定的主机和数据库中获取数据库数据，并将其传输到由 PVC 支持的工作区。

我潜入这个例子的另一个你会经常遇到的特性是从 *ConfigMaps* 或 *Secrets* 注入环境变量的能力。这在`env`部分完成。这与 Pods 的工作方式完全相同，因此您可以参考 API 的这一部分。

回到本例的主题——用于数据库备份的 PVC——考虑到我们希望该数据是持久的，我们不能使用前面所示的使用`volumeClaimTemplate`创建的 PVC，因为那样会在任务完成后被清除，所以我们需要单独创建 PVC，并以这种方式将其传递给任务:

这里我们用`persistentVolumeClaim`代替`volumeClaimTemplate`，我们指定现有 PVC 的名称，它也在上面的代码片段中定义。这个例子还假设在指定的主机上运行着 PostgreSQL 数据库——完整的代码包括 PostgreSQL 部署检验文件[这里是](https://github.com/MartinHeinz/tekton-kickstarter/tree/master/tasks/pg-dump/tests)。

类似于环境变量的注入，我们也可以使用工作区将整个配置图或秘密(或其中的一些密钥)作为一个文件注入。例如，当您希望将整个`.pem`证书从任务中的 Secret 或作为一个将 GitHub 存储库映射到应用程序名称的配置文件时，这可能很有用，如下例所示:

此任务获取存储库 URL，并使用它在文件中查找匹配的应用程序名称，该文件是上面显示的配置图的一部分。这是使用`yq`实用程序完成的，然后输出到一个名为`/tekton/results/...`的特殊目录中的文件。这个目录存储了任务的`results`，这是我们还没有提到的一个特性(和 YAML 部分)。

任务结果是任务可以输出的小块数据，随后可以被后续任务使用。要使用这些，必须在`results`部分指定结果变量的名称，然后向`/tekton/results/result-var-name`写一些东西。此外，正如您在上面的脚本中肯定注意到的，在将结果写入文件之前，我们使用`tr`从结果中去掉了换行符，这是因为结果应该是简单的输出——理想情况下只是一个单词——而不是一大块文本。如果您决定将更长的内容(多行)写入结果，如果不去除换行符，您可能会丢失部分值或只看到空字符串。

然后，为了使用任务——比如这个任务——使用来自配置映射的工作空间，我们必须以如下方式在`workspaces`部分指定配置映射:

最后我想展示的是边车容器的用法。这些并不常见，但是当您需要在任务执行期间运行一些服务(您的任务依赖于这些服务)时，它们会很有用。一个这样的服务可以是具有暴露套接字的 Docker 守护进程 sidecar。为了演示这一点，我们可以创建一个任务，使用名为 [Dive](https://github.com/wagoodman/dive) 的工具执行 Docker 映像效率分析:

正如您在这里看到的，`sidecars`部分与 Pod 规范中任何容器的定义都非常相似。除了我们在前面的例子中看到的常见内容，这里我们还指定了在任务步骤中 sidecar 和 container 之间共享的`volumes`——在本例中，其中一个是 Dive container 附加到的`dind-socket`中的 Docker 存储。

这涵盖了 Tekton 任务的大部分特性，但是有一点我到目前为止还没有提到，而你在阅读 Tekton 文档时肯定会遇到，那就是[*pipeline resource*](https://github.com/tektoncd/pipeline/blob/main/docs/resources.md)对象，它可以用作任务的输入或输出——例如 GitHub sources 作为输入，Docker image 作为输出。那么，为什么我还没有提到呢？PipelineResource 是 Tekton 的一部分，出于几个原因，我不喜欢使用它:

*   与我们目前使用的所有其他资源类型不同，它仍然处于 alpha 阶段
*   管线资源非常少。它大部分只是 Git、Pull 请求和图像资源。
*   很难对它们进行故障排除。

如果你需要更多的理由不使用它们(现在)，那么看看这里的文档部分。

在我们查看的所有这些示例中，我们看到了许多 YAML 部分、选项和功能，这显示了 Tekton 的灵活性，但这自然使其 API 规范非常复杂。`kubectl explain`不幸的是对探索 API 没有帮助，但是 API 规范可以在[文档](https://github.com/tektoncd/pipeline/blob/master/docs/api-spec.md)中找到，但是缺少。所以，如果你很难找到你能在 YAML 的哪个部分放什么，那么你最好的选择是依靠在[任务文档](https://github.com/tektoncd/pipeline/blob/master/docs/tasks.md#configuring-a-task)的开头列出的字段或者在这里的例子[，但是确保你在你的 Tekton 版本的正确分支上，否则你可能花费很长时间调试为什么你看起来正确的任务不能被 Tekton 控制器验证。](https://github.com/tektoncd/pipeline/tree/master/examples/v1beta1)

# 运行和测试

到目前为止，我们大部分时间只是谈论任务，并没有太多关于任务运行的内容。这是因为——在我看来——单独的任务运行最适合测试，而不是真正定期运行任务。为此，您应该使用管道，这将是下一篇文章的主题。

说到测试——当我们完成了任务的实现后，就该运行一些测试了。对于简单明了的测试，我推荐使用本文前面提到的布局。使用它应该有助于您以一种允许您独立于其目录之外的任何资源来测试它的方式来封装任务。

然后执行实际的测试，在`.../tests/resources.yaml`(如果有的话)中应用资源/依赖关系，然后在`.../tests/run.yaml`中应用实际的测试就足够了。测试实际上只是使用您的定制任务的一组任务运行，因此对于这种基本的测试方法，不需要任何设置/拆卸或额外的脚本——只需要`kubectl apply -f resources.yaml`和`kubectl apply -f run.yaml`。简单测试的例子可以在 [tekton-kickstarter](https://github.com/MartinHeinz/tekton-kickstarter/tree/master/tasks) 或 tekton 目录库中找到每个任务的目录。

不过，一般来说，对于这两个项目的存储库中的任何特定任务，您都可以运行以下命令来执行测试:

对我个人来说——当涉及到测试任务时——使用上面的基本测试方法进行验证和特别测试就足够了。然而，如果您最终创建了大量的定制任务，并且想要全力以赴，那么您可以采用 Tekton catalog 中的方法，并利用其存储库中的测试脚本。如果您决定走这条路线，并严格遵循布局和测试，您可能还想尝试将任务贡献给 Tekton 目录，以便整个社区可以从更多高质量的任务中受益。😉

至于你为此需要的脚本，你需要在你的代码中包含 Tekton 目录中的`[test](https://github.com/tektoncd/catalog/tree/master/test)` [目录](https://github.com/tektoncd/catalog/tree/master/test)以及 *Go* 依赖项(`vendor`目录)，然后在这里遵循文档[中的 E2E 测试指南。](https://github.com/tektoncd/catalog/blob/master/CONTRIBUTING.md#end-to-end-testing)

无论您选择基本的还是“全力以赴”的测试方法，都要确保您测试的不仅仅是任务和管道中的快乐路径，否则当它们被部署到*“野外”*时，您可能最终会看到许多 bug。

# 最佳实践

在您实现并测试了您的定制任务之后，最好回去确保您的任务遵循最佳开发实践，这将使它们在长期内更具可重用性和可维护性。

你能做的最简单的事情也能带来最大的好处——就是使用`yamllint`——YAML 文件的一个过磅器。这个技巧不仅适用于 Tekton 任务，也适用于所有的 YAML 文件，因为它们可能很难正确处理所有的缩进，但是特别重要的是，任务定义可能会变得很长很复杂，有许多级别的缩进，因此保持它们的可读性和有效性可以节省一些不必要的调试，并有助于保持它们的可维护性。你可以在我的资源库中找到一个我喜欢使用的定制`[.yamlint](https://github.com/MartinHeinz/tekton-kickstarter/blob/master/.yamllint)` [配置](https://github.com/MartinHeinz/tekton-kickstarter/blob/master/.yamllint)，但是你应该定制它以适合你的代码风格和格式。只要确保你不时地运行`yamllint`(最好是在 CI/CD 中)来保持事物的完整性和有效性。

至于实际的 Tekton 最佳实践——我可以给你一个很大的列表，但大部分都是 Tekton 维护者推荐的，所以我不会在这里复制粘贴，我只是给你指出相关的资源:

*   [任务创作建议](https://github.com/tektoncd/catalog/blob/master/recommendations.md)
*   [泰克顿设计原则](https://github.com/tektoncd/community/blob/main/design-principles.md)
*   [泰克顿投稿指南](https://github.com/tektoncd/catalog/blob/master/CONTRIBUTING.md#guidelines)

# 结束语

在本文中，我们看到了 Tekton 的灵活性和通用性，这种灵活性也带来了构建或测试任务的复杂性。因此，最好使用社区创建的现有任务，而不是自己重新发明一个轮子。然而，如果没有合适的任务可用，而您必须构建自己的任务，请确保为您的任务编写测试，并遵循上面提到的最佳实践，以保持您的任务的可维护性和可靠性。

在这篇文章之后，我们应该有足够的经验以及大量的个人定制任务，我们可以用它们来开始构建我们的管道。这正是我们在本系列的下一篇文章中要做的事情，我们将探索如何构建功能全面的管道来构建、部署、测试您的应用程序等等。

此外，如果您还没有这样做，请确保查看 [tekton-kickstarter 资源库](https://github.com/MartinHeinz/tekton-kickstarter)，在那里您可以找到本文中的所有任务和示例，以及您将在下一篇文章中看到的管道。😉

*本文最初发布于*[*martinheinz . dev*](https://martinheinz.dev/blog/47?utm_source=medium&utm_medium=referral&utm_campaign=blog_post_47)

[](/cloud-native-ci-cd-with-tekton-laying-the-foundation-a377a1b59ac0) [## 使用 Tekton 的云原生 CI/CD—奠定基础

### 是时候通过 Tekton Pipelines 在 Kubernetes 上开始您的云原生 CI/CD 之旅了…

itnext.io](/cloud-native-ci-cd-with-tekton-laying-the-foundation-a377a1b59ac0) [](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [## 用 seccomp 强化 Docker 和 Kubernetes

### 您的容器可能不像您想象的那样安全，但是 seccomp 配置文件可以帮助您解决这个问题…

itnext.io](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [](https://towardsdatascience.com/deploy-any-python-project-to-kubernetes-2c6ad4d41f14) [## 将任何 Python 项目部署到 Kubernetes

### 是时候深入 Kubernetes，使用这个成熟的项目模板将您的 Python 项目带到云中了！

towardsdatascience.com](https://towardsdatascience.com/deploy-any-python-project-to-kubernetes-2c6ad4d41f14)