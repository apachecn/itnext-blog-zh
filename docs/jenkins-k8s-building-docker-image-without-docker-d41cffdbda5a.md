# 詹金斯+ k8s:不用 Docker 打造 Docker 形象

> 原文：<https://itnext.io/jenkins-k8s-building-docker-image-without-docker-d41cffdbda5a?source=collection_archive---------0----------------------->

正如 2020 年宣布的那样，Kubernetes 在 1.20 版中反对 Docker 作为容器运行时，它将从即将发布的 1.24 版中删除。如果您的 Jenkins pipelines 使用 Kubernetes 插件，则很可能 pipelines 使用底层 Docker 来构建图像，因此您可能会遇到问题。

![](img/438e55b559ccbf424dade21cca6b00fd.png)

如果您仍然希望使用 Kubernetes 插件来构建 Docker 映像，1.24 版本之后有哪些选项？

# 用户 Docker

最明显的一种方法是继续使用 docker 来构建图像。

是的，您仍然可以在 Kubernetes Worker 节点上安装 Docker，即使它没有被 kubelet 用作容器运行时。在这种情况下，不需要对管道进行任何更改。

但是，请记住，这仅适用于自我管理的集群，在这种集群中，您可以完全控制工作节点。

如果您不想使用 Docker 或者您正在使用托管的 Kubernetes，那么容器运行时的选择就超出了您的控制范围，该怎么办呢？

# 使用构建工具包

使用 [BuildKit](https://github.com/moby/buildkit) 将需要对管道 Jenkinsfile 做一些小的修改。

让我们来看一下所需的更改。

假设我们有以下渠道:

它是真实管道的简化版本，其中所有的测试、静态代码分析和标记步骤都被删除了。因此，只剩下两个阶段:构建 docker 映像并将其部署到 kubernetes 集群。

为了使用 BuildKit，你必须用`moby/buildkit:master`替换`docker:dind`容器。在这种情况下，不再需要安装`/var/run/docker.sock`。

但是，为了让 BuildKit 将您的映像推送到注册表，首先您需要为 Docker 配置文件创建一个新的 secret(如果您的注册表需要认证)。在下面的例子中，我们假设我们正在将图像推送到 Docker Hub，我们的凭证如下:用户名= `user`和密码= `password`。

首先，将用户名/密码对编码成 base64 字符串。为此，请运行以下命令:

`echo -n user:password | base64 -w 0`假设你在 Linux 上运行或者`echo -n user:password | base64`在 Mac 上运行。复制结果字符串(`dXNlcjpwYXNzd29yZA==`)并将其添加到以下 json 文件(config.json)的“auth”属性中:

运行以下命令将 config.json 文件的内容转换成 base64 字符串:`cat config.json | base64 -w 0`并将结果字符串添加到 kubernetes 秘密清单`secret.yaml`。我们假设您的 Jenkins Kubernetes 插件将 Jenkins 名称空间用于管道容器。

然后将其应用到 Kubernetes 集群:`kubectl apply -f secret.yaml`。

现在，我们准备修改 podTemplate。它应该如下所示:

注意，我们将容器从 docker 重命名为 buildkit，并且还删除了命令定义。

此外，我们将 Docker 的 config.json 中的一个秘密挂载到/root/中。docker 目录，因此它将作为/root/在 buildkit 容器中可用。docker/config.json。

接下来，我们用以下内容替换`Build Docker Image`阶段:

差不多就是这样。

# 使用 AWS ECR

最后一点，万一你用 AWS ECR。在这种情况下，您的 Docker config.json 文件会有所不同:

此配置文件没有机密，因此可以存储为 ConfigMap。

如果使用 ConfigMap，podTemplate 中的卷挂载应该从:`secretVolume(secretName: 'docker-config', mountPath: '/root/.docker')`更改为`configMapVolume(configMapName: 'docker-config', mountPath: '/root/.docker').`

然而，在这种情况下，您需要在路径中提供 ECR 登录助手，这样`Build Docker Image`阶段将如下所示:

有许多边缘情况，如代理或容器注册服务器后面的 K8S 集群使用自签名证书，需要稍微复杂一些的配置，我们将在后面讨论。