# Jenkins 教程—第 6 部分—管道选项

> 原文：<https://itnext.io/jenkins-tutorial-part-6-pipeline-options-5ccd05035aaf?source=collection_archive---------2----------------------->

![](img/c518273176f996c36d381a98cf53ea9a.png)

在 Jenkins 教程系列的这一部分中，我们将深入研究 Jenkins 管道选项。使用这些选项，您可以配置管道并更改一些管道行为。建议你先看前面的文章再继续看。你可以在下面的 GitHub repo 中找到所有发表的文章。

如果你对这个系列教程感兴趣，**之星**凝视这个回购:

[](https://github.com/ssbostan/jenkins-tutorial) [## GitHub——ssbo stan/jenkins——教程:Jenkins 参考和管道代码示例

### 如果你觉得有用，就看星星。Jenkins 系列教程的参考和示例库。版权所有 2021…

github.com](https://github.com/ssbostan/jenkins-tutorial) 

# Jenkins 管道选项:

Jenkins CI 中有几个渠道选项。有些是原生选项，有些是由于安装插件而添加的。我打算解释 Jenkins 堆栈中可用的选项。您可以在 Kubernetes 和 Docker 环境中部署这个堆栈。以下存储库是部署 Jenkins 完整堆栈所需的一切。

[](https://github.com/ssbostan/jenkins-stack-kubernetes) [## GitHub-ssbo stan/jenkins-stack-kubernetes:在 Kubernetes 上部署 Jenkins 的脚本和清单

### 注意:此存储库是“DevOps with Saeid”类的一部分。部署 Jenkins 的脚本和清单…

github.com](https://github.com/ssbostan/jenkins-stack-kubernetes) [](https://github.com/ssbostan/jenkins-stack-docker) [## GitHub-ssbo stan/Jenkins-stack-Docker:Docker-compose 版本的 jenkins-stack-kubernetes

### Permalink 无法加载最新的提交信息。docker-撰写版本的詹金斯-栈-kubernetes。要查看…

github.com](https://github.com/ssbostan/jenkins-stack-docker) 

# 如何配置管道选项:

管道选项设置在管道的`options`块内。

这里有一个例子:

那么我们来介绍一下可用的选项及其作品。

在上面的例子中显示的 **buildDiscarder** ，用于删除旧的构建和工件。您可以根据构建和工件的日期和数量来设置旋转器。看上面的例子。

**checkoutToSubdirectory** 用于将`checkout scm`命令的目录更改为管道工作区的子目录。

```
checkoutToSubdirectory("mycheckout")
```

**copyArtifactPermission** 用于定义允许他们复制该作业工件的项目列表。您可以从它的[插件](https://plugins.jenkins.io/copyartifact)(复制工件插件)文档中了解更多关于复制工件的信息。

```
copyArtifactPermission("job1,myjob,testjob5")
```

**disableConcurrentBuilds**用于禁用并发构建。

```
disableConcurrentBuilds()
```

**禁用恢复**如果 Jenkins 的控制器重启，则禁用构建恢复功能。控制器正是詹金斯的主人。

```
disableResume()
```

**耐久性提示**用于覆盖作业的速度/耐久性选项。您可以使作业在控制器停机后可恢复(这会消耗更多时间和更多磁盘写入)，或者使作业更快(在计划外停机的情况下无法恢复)此选项是高级的。

*   **最大 _ 生存能力**:最大耐久力但最慢。
*   **SURVIVABLE_NONATOMIC** :耐用性和更快速度的结合。
*   **性能 _ 优化**:像兔子一样奔跑。

在**性能优化**的情况下，Jenkins 控制器应正常关闭，以保存正在运行的流水线。

```
durabilityHint("PERFORMANCE_OPTIMIZED")
```

**newContainerPerStage** 与 **docker** 和 **dockerfile** 顶层代理配合使用。运行作业阶段的正常行为是所有阶段都在一个容器中运行，但是使用此选项，每个阶段都在同一节点中的单独容器中运行。

**parallelsAlwaysFailFast**用于失效保护所有并行级。如果任何并行阶段失败，其他阶段也会中止，并且构建状态将设置为失败。这是一个高级选项。

```
parallelsAlwaysFailFast()
```

**保存隐藏文件**用于保存隐藏的文件。在构建作业的过程中，可能会创建一些文件，在构建结束时会删除这些文件。在某些情况下，您需要从特定阶段重新开始构建。在这种情况下，您需要在之前的构建过程中创建的临时文件。Stash 就是为了实现这个需求而来的。

以后的文章会深入讲解。

```
preserveStashes()
```

**quietPeriod** 用来指在特定的一段时间内相当长的一段时间。当一个作业被触发时，一个新的构建被放入队列，但是在这个选项设置的一段时间内，Jenkins 不会启动它。

```
quietPeriod(30)
```

**如有异常，重试**N 次管道本体。

```
retry(3)
```

**skipDefaultCheckout** 用于在输入新代理时停止自动结帐。您必须使用`checkout scm`命令手动检查。

```
skipDefaultCheckout(true)
```

如果构建状态变得不稳定，则跳过更多阶段。它用于停止整个管道。

```
skipStagesAfterUnstable()
```

**超时**用于指定构建超时。默认情况下，构件没有任何超时限制。超时后，抛出**flowtruptedexception**，流水线中止。

```
timeout(time: 300, unit: "SECONDS")
```

# 最后的话:

詹金斯有很多选择。其中一些是本地选项，许多是插件添加的。在本文中，我试图解释其中的一些，但是我打算在以后的文章中深入研究每一个。如果您有任何问题，请在教程 GitHub repo 上提交问题。

关注我的 LinkedIn[https://www.linkedin.com/in/ssbostan](https://www.linkedin.com/in/ssbostan)