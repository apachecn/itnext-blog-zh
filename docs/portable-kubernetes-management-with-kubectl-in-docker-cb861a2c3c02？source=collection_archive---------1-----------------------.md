# 用 Docker 容器中的 kubectl 进行可移植的 Kubernetes 管理

> 原文：<https://itnext.io/portable-kubernetes-management-with-kubectl-in-docker-cb861a2c3c02?source=collection_archive---------1----------------------->

![](img/b789dacdf045adcfa95ebcebfc784b99.png)

图片来自[皮克斯拜](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3021820)

说 Kubernetes 正在成为主流是一种保守的说法。事实上，它已经影响了现代分布式系统的设计和操作。通过将基础设施问题抽象化，我们能够将 Kubernetes 作为一个“构建平台的平台”或“云操作系统”来利用，这带来了许多明显的好处，但也带来了开发和运营挑战。

围绕 Kubernetes 的整个生态系统正在蓬勃发展，现在开源社区有大量令人惊叹的工具和项目，其中一些在 CNCF 的保护下，一些由爱好者或各种规模和影响力的公司设计。可以说，有很多事情正在发生，很难跟上这种不断变化的形势。

今天，我们将重点关注一个既具体又非常实用的领域:即 Kubernetes 对 kubectl 的管理。 [Kubectl](https://kubectl.docs.kubernetes.io/) 是一个命令行工具(CLI ),用于管理 Kubernetes 集群。Kubernetes 社区中一个众所周知的争论是:“kubectl 到底是怎么发音的？”。有很多观点，本页提供了一些最常见的观点，所以你可以自由选择，甚至创造你自己的观点。

当我们不忙着搞清楚世界饥饿或 kubectl 发音时，我们经常花大量时间手工制作我们运行 kubectl 的环境，我们创建别名，添加自动完成，安装插件和其他诊断和集群管理工具。如果我们能够随身携带这种诊断/管理环境，并对每个集群使用相同的工具、脚本、别名和快捷方式，会怎么样？

对我来说，用 kubectl 创建映像的动机是，我必须在没有[wsl 2](https://docs.microsoft.com/en-us/windows/wsl/install-on-server)(Lunix 的 Windows 子系统)的情况下，在 Azure 上运行的 Windows Server VM 中使用新的 Kubernetes 集群，我不想使用 cygwin 或 git-bash。我已经习惯了在 Ubuntu 上运行的环境，在一个图像中重建是一个快速解决这个挑战的方法。

在这篇博客的第二部分，我将解释如何在 Docker 容器中使用定制和个性化的 kubectl。您可以使用我现有的 kubectl 映像，或者克隆我的 [GitHub 库](https://github.com/Piotr1215/kubectl-container)，并根据您的喜好进行更改。毕竟，Kubernetes 是为管理容器而设计的，所以从容器中管理 Kubernetes 才是公平的！

# 使用 kubectl 创建和使用您自己的图像

这一章和下一章并不打算成为 Docker 教程，而是实践指导。如果你感兴趣的话，有很多关于 Docker 的资源。好的起点是 https://www.docker.com/的。

如果您想尝试创建自己定制的 kubectl 容器，下面是要遵循的步骤。你也可以克隆/分叉我的 GitHub 库【https://github.com/Piotr1215/kubectl-container。

你也可以直接从 Docker Hub 使用我的图片，如果你想测试它是如何工作的:[https://Hub . Docker . com/repository/Docker/piotrzan/ku bectl-comp](https://hub.docker.com/repository/docker/piotrzan/kubectl-comp)。

## 创建 Dockerfile 文件

创建一个新的 git 存储库，并向其中添加 Dockerfile。这是基于我的存储库的示例文件。

您可以将所有内容添加到一个 docker 文件中，我选择将配置部分拆分到一个 bootstrap.sh 中。配置脚本被复制到映像目录中，并作为运行步骤之一运行。另一种常见的做法是将供应脚本用作 Dockerfile 入口点。

## 构建和发布图像

一旦所有的程序和配置都准备好了，就构建映像并用 docker 用户名标记它，这样就可以发布它了。

```
**docker build --rm -f “Dockerfile” -t dockeruser/image-name “.”**
```

将图像发布到 docker 存储库中。

```
**docker publish dockeruser/image-name**
```

## 在任何环境下运行您的映像

当您需要使用新集群时，只需运行您的映像。注意，我们命名容器是为了更容易管理它，并将 *network 设置为“host”*，这将使 kubectl 能够通过它访问主机网络和集群。

```
**docker run --network=host --name=kubectl-host --rm -it piotrzan/kubectl-comp**
```

# 潜得更深

继续阅读，如果你想了解更多，理解我的形象以及我所做的设计选择。

## 图像中包含什么

*   具有 bash/zsh 完成功能的 kubectl 1 . 17 . 2 版
*   **zsh-zsh 外壳的自动建议**
*   **k9s** 出色的集群可观察性和基于终端的管理工具
*   流行工具: **curl，wget，git**
*   有用。bashrc/。zshrc 别名

```
# Instead of typing kubectl all the time, abbreviate it to just “k”
**alias k=kubectl**# Check what is running on the cluster
**alias kdump=’kubectl get all — all-namespaces’**# Display helpful info for creating k8s resources imperatively
**alias krun=’k run -h | grep “# “ -A2'**# Quickly spin up busybox pod for diagnostic purposes
**alias kdiag=’kubectl run -it — rm debug — image=busybox — restart=Never — sh’**
```

## 如何使用它

我的库包含了下面描述的所有脚本和命令，所以你可以直接使用它，而不是从这里复制。

使用映像最简单的方法是用下面的命令运行它。您可以选择运行任一 bash shell

```
**docker run --network=host --name=kubectl-host --rm -it piotrzan/kubectl-comp**
```

或者你喜欢 zsh shell，还有另外一个形象

```
**docker run --network=host --name=kubectl-host --rm -it piotrzan/kubectl-comp:zsh**
```

这将启动一个名为 *kubectl-host* 的新容器，并使您能够使用 kubectl，但是它不包含关于您的集群的信息。为了做到这一点，你需要确保。kube/config 文件在容器中可用。最简单的方法是以分离模式运行容器，复制文件并附加回容器，如下所示:

```
*# Run container with pass-through to local network***docker run -d — network=host — name=kubectl-host — rm -it piotrzan/kubectl-comp***# Generate raw config from kubectl on localhost and copy the config to the container***kubectl config view — raw > config****docker cp config kubectl-host:./root/.kube****docker attach kubectl-host**
```

或者，您可以使用`docker-compose`旋转容器并附加一个卷

## 扩展图像

假设您想要添加自己的别名或安装额外的软件等。Docker 通过使用`docker commit`命令实现简单的图像扩展。

1.  运行容器
2.  配置和定制
3.  打开另一个 shell 会话并运行:

```
docker commit CONTAINER_ID new-contianer-tag
```

在这之后，当你运行`docker images`时，你将会看到你的新图像和你所做的改变一起被列出。你可以把它发布到 Docker Hub 上，或者只是在修改后使用它。

## 结论

Kubernetes 和容器化的工作负载提供了极大的灵活性，并开辟了以前很难实现的新的可能性。我希望，通过创建一个专用的、完全可移植的和定制的 kubectl 映像的例子，我已经启发了您尝试用您最喜欢的命令行工具创建一个 Docker 映像。

我们最终创建了一个定制的、个性化的、完全可移植的 kubectl CLI 体验。无论如何，我的形象是唯一的，有很多伟大的想法。一个值得一提的图片是 Bitnami 为 kubectl 策划的图片，你可以在这里找到它:[https://hub.docker.com/r/bitnami/kubectl](https://hub.docker.com/r/bitnami/kubectl)

和往常一样，还有很多要说的，我们还没有谈到构建自动化 CI/CD 管道、缩小映像、围绕安全性实现最佳实践等等。其中一些话题非常复杂，以至于可以有自己的博客。

感谢您的阅读，下次博客再见！