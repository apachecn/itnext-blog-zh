# Jenkins + k8s + Buildkit:企业代理背后的生活

> 原文：<https://itnext.io/jenkins-k8s-buildkit-life-behind-the-corporate-proxy-cb052bd7f969?source=collection_archive---------2----------------------->

在[上一篇文章](/jenkins-k8s-building-docker-image-without-docker-d41cffdbda5a)中，我们讨论了当 Kubernetes 在 1.24 版中移除 Dockershim 后构建 Docker 映像时可用的选项。

但是，如果您的本地 Kubernetes 集群不能直接访问互联网，而是被迫通过代理服务器，那该怎么办呢？这种情况在企业界相当普遍。

此外，在这种设置中常见的是使用自托管 Docker 注册表，并且在某些情况下，内部自托管资源使用自签名证书或由本地创建的不受信任的证书颁发机构签名的证书。在这个设置中可以使用 Buildkit 吗？

答案是肯定的，这是可能的，但需要一点额外的努力来设置它。

![](img/438e55b559ccbf424dade21cca6b00fd.png)

# 自托管 Docker 注册表

让我们讨论每个场景，从自托管 Docker 注册中心开始。

如果自托管 Docker Registry 使用由公认可信的证书颁发机构签名的 SSL 证书，则不需要进行任何更改。

但是，如果它是由不受信任的 CA 签名的，或者它使用自签名证书，则必须将其添加到`moby/buildkit`映像中。因为我们在守护模式下使用 buildkit，所以必须在启动容器之前将它添加到映像中。

为此，您需要创建一个新的镜像扩展`moby/build`，将您的 CA 证书或自签名证书放入`/usr/local/share/ca-certificates/`目录(`moby/buildkit`基于`alpine`发行版)并运行`update-ca-certificates`命令。

例如，Dockerfile 将如下所示:

一旦您构建了它`docker build -t myorg/buildkit .`并将其推送到注册表`docker push myorg/buildkit`中，它就可以在管道中使用了。只需将 Pod 模板中的`moby/buildkit`替换为新建图像的名称即可:

如果您将图像推入需要认证才能访问图像的私有注册中心，您必须在 Jenkins 使用的名称空间中创建一个类型为`kubernetes.io/dockerconfigjson`的新秘密。

为此，首先用注册表凭证对以下 JSON 进行编码:

“auth”属性的内容是 base64 编码的`username:password` 字符串。

用 base64 编码对上述 JSON 进行编码，并将其作为`.dockerconfigjson`属性添加到以下 secret 中:

运行`kubectl apply -f secret.yaml`在名称空间中创建 secret，然后将其名称添加到 Pod 模板的`imagePullSecrets`数组属性中:

# 代理背后的 Kubernetes 集群

如果您的 Kubernetes 集群部署在代理之后，并且工作节点不能直接访问互联网来获取映像和/或获取依赖关系，则需要另外两个设置。

如果 Kubernetes 在代理之后，为了允许 BuildKit 提取图像，请向正在讨论的 pod 添加 HTTP_PROXY、HTTPS_PROXY 和 NO_PROXY 环境变量。在这种情况下，容器定义将如下所示:

为了允许 BuildKit 在构建映像时获取依赖项或任何其他网络资源，请在 build 命令中添加相同的环境变量。在这种情况下，构建命令将如下所示: