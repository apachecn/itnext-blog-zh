# Docker & Makefile | X-Ops —共享基础架构代码部分

> 原文：<https://itnext.io/docker-makefile-x-ops-sharing-infra-as-code-parts-ea6fa0d22946?source=collection_archive---------0----------------------->

![](img/dfa4f3c1be0640ee97b09021643a9a1a.png)

[甄虎](https://unsplash.com/@zhenhu2424?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

在我们作为 X-Ops(即 DevOps、CloudOps、GitOps、SysOps 等角色)的生活中，我们专注于构建基础架构即代码。我们经常使用 Docker 来构建和测试，提交代码并从存储库中提取代码，当所有测试都通过时，基础架构就可以完全自动化地投入生产。

现在的一个大机会是扩展和整合我们的 X-Ops 团队。候选名单上的两个目标是:1.低门槛(最好是零门槛)，有助于吸引更多同事加入；2 .易于遵循的构建和测试习惯，更快地将零件投入生产并提高重复利用率。

在本文中，我将重点介绍几个简单的技术实践——即应用 Docker、Makefile 和正确使用 UIDs 根据我的经验，这些实践让我们更接近这些目标。

# 使用 Makefile

Docker 为我们带来了快速有效地构建系统的自动化，我们甚至可以使用 docker-compose 来运行服务。

然而，在大多数生产环境中，都有一个单独的容器平台。我们在本地构建、测试和运行的容器是一个更大环境的一部分。一个经常出现的问题是，当你在一个团队或世界范围内共享 docker 代码部分时，你还需要解释你的容器需要如何运行。

就我个人而言，我希望我所有的项目都像这样运作:

```
git pull && make test && make build && make deploy
```

以这种格式使用一行程序是非常强大的。你可以很容易地让一个同事带着你的产品(组件)去兜风，而不需要他/她费力地阅读一大堆自述文件。要么成功，要么失败。

输入好的旧 Makefile。一个有 40 年历史的经典，仍然太年轻而不能退休，现在正在自动化项目中重生(这方面的简短阅读很好:[为懒惰的开发人员制作文件](https://localheinz.com/blog/2018/01/24/makefile-for-lazy-developers/))。

这是我版本的 Makefile 文件,与 Docker 配合得很好:

如果您不熟悉 Makefile，这可能看起来有点吓人。在你开始之前，你需要 Docker。Docker 安装说明在这里: [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce) ， [Mac](https://docs.docker.com/docker-for-mac/install/) 或者 [Windows](https://docs.docker.com/docker-for-windows/install/) 。我还假设，你也知道如何打开一个外壳，安装一个 git 客户端，并可以运行 make。推荐 Linux 发行版，但是我也在 Mac 上测试过。如果您已经设置好了，可以尝试以下命令:

```
# download our files
git clone [https://github.com/LINKIT-Group/dockerbuild](https://github.com/LINKIT-Group/dockerbuild)# enter directory
cd dockerbuild# build, test and run a command
make build test shell cmd="whoami"# my favorite for container exploration
make shell# shell-target is the default (first item), so this also works: 
make cmd="whoami"
make cmd="ls /"# force a rebuild, test and cleanup
make rebuild test clean
```

注意我没有在任何地方使用“&&”字符？您可以使用单个 make 命令和一个目标列表(例如，构建、测试、清理)，一旦遇到错误就停止。

在 Makefile 中，您可以定制在特定目标上运行的一行程序。就我个人而言，我总是喜欢在运行语句中添加“— rm”，以防止以一串废弃的容器结束。如果你仔细阅读 Makefile，你会看到更多有趣的东西可以尝试。

您应该想要定制测试目标，并使测试成为标准实践。当您的构建部分被集成到 CI/CD 管道中时(最终，它会)，存在“make test”是非常有用的。

从这里开始，添加部署目标只需要一步，将容器推到 Docker-hub 或类似的存储库。持续部署是真正令人兴奋的地方。不幸的是，这也需要更多的写作空间，这就是为什么我需要把它留到后续文章(或两篇)中。

在我们转向部署之前，我们需要再了解一个基本的配置实践。注意到 Makefile 中的 HOST_USER 和 HOST_UID 变量了吗？这些在运行时被填充，并被重新导出供 docker-compose 读取。这让我想到了关于 uid 的下一章。

# **处理 uid**

当没有配置 UserID (UID)时，Docker 将容器默认为用户 root。当您开始处理生产系统时，正确配置用户是必需的。你可以在[码头工人最佳实践清单](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)上读到它，K8s 的这篇文章也很好地涵盖了它。

当您在 Mac 上本地运行测试时，root 被映射到您自己的用户，所以一切正常。生产平台应该配置为用户隔离，但这并不总是默认的。例如，如果您在 Linux 系统上运行您的测试(没有重新映射)，您很可能会遇到 Docker 生成的文件属于 root 的问题。

当一个系统上有多个用户时，您也会遇到问题。即使您不共享一个系统，运行并行测试也需要分离。此外，您如何传递凭证(例如，通过以~/的形式链接卷)。ssh 和~/。aws)没有正确的 UID 设置？后者是处理基础架构部署时的常见模式。

如果你开始成为一个更密集的 docker-consumer，你的团队会成长，事情会变得复杂，最终你会想要或者需要在你所有的项目中嵌入 UID 分离。

幸运的是，早期配置 UID(相对)容易。虽然我花了一些时间进行良好的设置，但我现在有了一个模板(假设使用了 Makefile)可以自动完成所有工作，几乎不费吹灰之力。

下面是一个可以使用的 **docker-compose** 文件的副本:

一些神奇的东西就在这些变量语句中:

```
${HOST_USER:-nodummy}
${HOST_UID:-4000}
```

这将从运行时复制变量，如果不存在，则分别默认为“nodummy”和“4000”。如果你不喜欢默认值，就这样做:

```
${HOST_USER:?You forgot to set HOST_USER in .env!}
${HOST_UID:?You forgot to set HOST_UID in .env!}
```

请注意“HOST_”前缀。我避免直接使用用户和 UID。这些变量不保证在运行时可用。用户通常在 shell 中可用，但是 UID 通常是 Docker 不会选择的环境变量。拥有一个单独的命名方案可以防止意外，并允许灵活地配置自动化管道。

**Dockerfile** 看起来是这样的:

在这里，我们建立了一个小的(它只有 10MB，微服务 FTW！)添加了 bash 的 Alpine container，用于娱乐和练习。我们应用所谓的[分阶段构建](https://docs.docker.com/develop/develop-images/multistage-build/)的概念来保持基础映像(可重用的构建组件)与用户映像(为特定运行准备的映像)分离。

使用 Makefile 时，所有的变量都是自动设置的，如前一章所述。这个 Dockerfile 在没有 Makefile 的情况下也能很好地工作，但是用户仍然可以在他们自己的运行时或者一个单独的 [env-file](https://docs.docker.com/compose/env-file/) 中配置变量。就我个人而言，我只是使用 Makefiles，因为我喜欢零工作量。

最后一件事。如果您处于开发模式，您可能会遇到某种障碍，需要您以 root 用户身份进行故障诊断。只需键入:

```
make shell user=root
```

# 包装它

我希望这篇文章对你有用。完整代码可在 [github](https://github.com/LINKIT-Group/dockerbuild) 获得。

现在，当我们构建一个新的服务时，我们可以简单地通过复制/粘贴这些文件(Makefile、Dockerfile 和 docker-compose.yml)来启动我们的项目，在需要的地方进行修改，并立即启动和运行我们的新服务。

使用 Docker+Makefile 过程不仅使您的部分可移植，而且对您的团队(或整个世界)来说也是可重用的，并且允许附带测试规则的运输。欢迎来到 X-Ops 的世界！

编码愉快，下一篇文章再见！