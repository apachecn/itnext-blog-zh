# 使用 GitHub Actions、Helm 和 Kubernetes 的多拱门 Golang 应用程序的 CI/CD

> 原文：<https://itnext.io/ci-cd-for-a-multi-arch-go-application-using-github-actions-docker-buildx-helm-and-kubernetes-f415a42b2c82?source=collection_archive---------1----------------------->

![](img/1924cb10ef44300232d103b0bdda0313.png)

特斯拉公司的装配线。

*持续集成*和*持续部署* t 管道是自动化流程，是 de DevOps 生态系统的一部分，可以帮助您在软件的整个生命周期中提高软件的质量，并以更可预测和更灵活的方式进行生产部署。

在本文中，我们将为林挺配置 GitHub 操作，构建、测试并发布 Golang 应用程序的 Docker 映像。这样做之后，我们将创建一个 Helm 图表，将其推送到我们的 Helm 存储库，并通过使用另一个 GitHub 操作将其部署到我们的 Kubernetes 集群。

# 1.应用程序

我们将使用 Gotway，这是一个用 Golang 编写的轻量级 API 网关，它允许您将 HTTP 和 gRPC 服务暴露给互联网，并提供开箱即用的功能，如健康检查、服务发现和 Redis 中的缓存。

有关实现细节，请查看 GitHub 资源库:

[](https://github.com/gotway/gotway) [## 高速公路/高速公路

### 一个简单、轻量级、速度惊人的 API 网关🚀对 REST 和 gRPC 的 API 组合和动态路由支持…

github.com](https://github.com/gotway/gotway) 

# 2.集群

Gotway 的生产环境包括由 Rasperry Pi 构成的 [K3s](https://k3s.io/) 集群，具体来说:

*   2 个[树莓派 4 个](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/specifications/)
*   2 个[树莓派 3 个](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)

你可能知道，这些型号的 CPU 架构不同，型号 3 有`armhf`，型号 4 有`arm64`。考虑到这一点以及 Kubernetes 可以在任何节点(重新)调度您的 pods 的事实，我们将需要为这些架构以及`amd64`构建 Docker 映像，以便我们可以在我们的笔记本电脑上本地测试它们。

有关集群设置的更多详细信息，请参阅本文:

[](/deploying-a-microservice-oriented-application-to-kubernetes-from-zero-to-production-416a173a8505) [## 将面向微服务的应用部署到 Kubernetes，从零到生产

### 一步一步解释的实用指南

itnext.io](/deploying-a-microservice-oriented-application-to-kubernetes-from-zero-to-production-416a173a8505) 

# 3.连续累计

要设置 GitHub 操作，您只需要创建一个`[.github/workflows](https://github.com/gotway/gotway/tree/develop/.github/workflows)`目录，其中包含 YAML 格式的管道(工作流)定义。每个工作流都需要为其执行指定一系列步骤和一个触发事件，例如推送或拉取请求。

这些步骤称为动作，可以是:

*   您自己的 bash 脚本
*   [GitHub 提供的动作](https://github.com/actions)
*   [社区制定的行动](https://github.com/marketplace?type=actions)

例如，对于林挺工作流，我们使用 GitHub 提供的动作来设置 Go 环境和检查代码。然后，我们可以调用我们的 [Makefile](https://github.com/gotway/gotway/blob/develop/Makefile) 的`lint`目标:

林挺行动法典

构建工作流与前一个非常相似，但是在这种情况下，我们也将构建的二进制文件作为[工件](https://github.com/gotway/gotway/actions/runs/749117653)保存。这样，我们可以轻松地执行和测试存储库的所有提交:

构建我们的应用程序的行动

最后，测试工作流将运行测试并生成覆盖报告，该报告也将作为[工件](https://github.com/gotway/gotway/actions/runs/749117651)保存:

运行测试的操作

请务必注意，每当发生推送或拉取请求事件时，都会触发此工作流，这样我们可以确保:

*   我们存储库的代码遵循我们的风格指南。
*   可以构建存储库中的任何提交，因此可以部署到生产中。
*   应用程序按预期运行。

# 4.发布 Docker 图像

当一个版本的开发完成时，一个标签被推送到存储库，这触发了构建 Docker 映像并将它们推送到注册中心的发布操作。

为此，我们需要:

*   [QEMU](https://github.com/multiarch/qemu-user-static) :支持不同多架构容器的执行。
*   Docker Buildx :扩展 Docker 的 CLI 插件，允许同时为多个架构构建映像。

这两个工具都可以通过使用 Docker 提供的动作轻松设置。之后，我们可以执行我们的`release.sh`脚本:

将图像推送到 DockerHub 的操作

注意，我们将 GitHub secrets[和我们的凭证映射到环境变量，因此脚本可以访问注册表。如果你想查看细节，以下是完整的脚本:](https://docs.github.com/es/actions/reference/encrypted-secrets)

将图像推送到 DockerHub 的脚本

另一件重要的事情是图像的重量，我们在我们的 [Dockerfile](https://github.com/gotway/gotway/blob/develop/Dockerfile) 中使用多阶段构建，并指定 [alpine](https://hub.docker.com/_/alpine) 作为基础图像，以尽可能保持它的最小化。生成的图像可在 [DockerHub](https://hub.docker.com/repository/docker/gotwaygateway/gotway/tags?page=1&ordering=last_updated) 上获得。

# 5.创建舵图

Helm Charts 允许您使用模板来定义您的 Kubernetes 资源，这些模板可以根据您的版本需求进行参数化。要创建初始舵图，请运行:

```
$ helm create gotway
```

这将创建一组示例模板和一个包含图表信息的`Chart.yml`。这里我们也可以在`dependencies`部分添加子图表，在我们的例子中，我们将添加 Redis:

图表定义文件

还有一个名为`values.yml`的文件，它包含了你的版本的参数，将用于插入你的模板。基本上，您的 Kubernetes 资源将使用这些值来创建。

此外，子图表的值将在一个带有名称的键下指定，在我们的例子中是`redis.`

我们图表的参数值

为了使用这些值，我们需要遵循 [Helm 的模板语法](https://helm.sh/docs/chart_template_guide/)，基本上我们需要将它们放在花括号中，并通过`.Values`对象访问它们:

部署模板

# 6.在本地测试图表

很好，我们已经有了本地图表，但是，在将它部署到生产环境之前，我们如何测试它呢？

你脑海中的第一个答案可能是拥有一个开发集群。这在某些情况下没什么问题，但是配置集群的成本很高，而且需要一些时间。

一个更便宜更快捷的选择是使用[类](https://kind.sigs.k8s.io/)(Docker 中的 Kubernetes)。您只需要在您的机器上运行 Docker，就可以开始创建集群:

```
$ go get sigs.k8s.io/kind
$ kind create cluster gotway
```

就这样，您的集群在自己的 Docker 容器中启动并运行。现在，您可以通过执行以下命令来测试您的本地舵图:

```
$ helm install gotway charts/gotway
```

# 7.设置图表存储库

使用本地图表很好，但是从服务器集中并提供所有版本的图表更好。

或许托管你的图表的最佳选择是 Chartmuseum，这是 Helm 的一个图表库，可以索引你的图表，并允许你通过它的 REST API 管理它们。在我们的例子中，我们[扩展了官方图表 museum Helm Chart](https://github.com/gotway/charts/tree/main/charts/gotway-charts) ，这样我们就可以添加 Traefik 的`ingressRoute`并对官方图表的值进行版本化。

因此，我们只需要添加我们的存储库并安装我们想要的图表:

```
$ helm repo add gotway https://charts.gotway.duckdns.org
$ helm install gotway gotway/gotway
```

在公开您的存储库之后，您还可以在 [ArtifactHub](https://artifacthub.io/packages/search?repo=gotway) 中发布它，这是掌舵人的官方网站，用于探索和发现包。这样，你的包将被编入索引，并可以通过他们的搜索引擎访问。

# 8.释放舵图

我们可以通过它的 API 将我们的图表推送到 Chartmuseum，但是使用他们的 [helm push](https://github.com/chartmuseum/helm-push) 插件要简单得多:

```
$ helm plugin install [https://github.com/chartmuseum/helm-push.git](https://github.com/chartmuseum/helm-push.git)
$ helm repo add gotway [https://charts.gotway.duckdns.org](https://charts.gotway.duckdns.org)
$ helm push charts/gotway gotway
```

这将把您的图表打包成一个`.tgz`文件，并通过使用 Chartmuseum API 将其推送到存储库。值得注意的是，在此之前，我们需要更新依赖项，因为它们也将被打包。检查此脚本:

将图表推送到 Chartmuseum 实例的脚本

每当提交到`main`分支时，这个脚本将在 GitHub 动作中执行:

将图表推送到 Chartmuseum 实例的脚本

有关更多详细信息，请查看 GitHub 资源库:

[](https://github.com/gotway/charts) [## 高速公路/海图

### gotway Permalink 的 ARM 兼容舵图表无法加载最新提交信息。ARM 兼容的舵图表由…

github.com](https://github.com/gotway/charts) 

# 9.持续部署

最后但同样重要的是，现在我们的图表在存储库中可用了，我们可以用另一个 GitHub 操作将它们安装到我们的 Kubernetes 集群中。

我们需要做的第一件事是通过用我们的`~/.kube/config`文件的内容创建一个`KUBE_CONFIG`秘密来配置对集群的访问:

`~/.kube/config file example`

然后我们可以通过将它作为秘密传递给`azure/k8s-set-context@v1`动作来设置这个配置:

部署到 Kubernetes 的操作

最后，我们可以通过执行带有`install` 标志的`helm upgrade`来部署我们的图表，以便在它不存在的情况下创建发布:

部署到 Kubernetes 的脚本

就这样，我们的应用程序启动并运行，我们的徽章显示所有工作流都已成功完成:
[https://api.gotway.duckdns.org/api/health](https://api.gotway.duckdns.org/api/health)

表明我们 CI/CD 当前状态的徽章

# 10.包扎

如今，没有 CI/CD 是一个明显的症状，表明你并不真正关心你的开发团队的效率，也不关心你的产品的质量。

正如您在本文中所看到的，尽管配置 CI/CD 需要一些时间，但是在生产力上有一个实质性的回报，并且它为您的代码中的变更提供了一个安全网，为您的团队带来额外的信心和更好的开发体验。

黑客快乐！感谢阅读。

# 资源

[](https://github.com/gotway/gotway) [## 高速公路/高速公路

### 一个简单、轻量级、速度惊人的 API 网关🚀对 REST 和 gRPC 的 API 组合和动态路由支持…

github.com](https://github.com/gotway/gotway) [](https://github.com/gotway/charts) [## 高速公路/海图

### gotway Permalink 的 ARM 兼容舵图表无法加载最新提交信息。ARM 兼容的舵图表由…

github.com](https://github.com/gotway/charts) [](/deploying-a-microservice-oriented-application-to-kubernetes-from-zero-to-production-416a173a8505) [## 将面向微服务的应用部署到 Kubernetes，从零到生产

### 一步一步解释的实用指南

itnext.io](/deploying-a-microservice-oriented-application-to-kubernetes-from-zero-to-production-416a173a8505) 

*   [https://github.com/actions](https://github.com/actions)
*   [https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)
*   [https://github.com/multiarch/qemu-user-static](https://github.com/multiarch/qemu-user-static)
*   [https://docs.docker.com/buildx/working-with-buildx/](https://docs.docker.com/buildx/working-with-buildx/)
*   [https://helm.sh/](https://helm.sh/)
*   [https://k3s.io/](https://k3s.io/)
*   [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/)
*   [https://chartmuseum.com/](https://chartmuseum.com/)
*   [https://hub . docker . com/repository/docker/gotway gateway/gotway/tags？page=1 &订购=最后 _ 更新](https://hub.docker.com/repository/docker/gotwaygateway/gotway/tags?page=1&ordering=last_updated)
*   【https://artifacthub.io/packages/search?repo=gotway】