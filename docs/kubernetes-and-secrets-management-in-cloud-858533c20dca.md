# Kubernetes 和云中的秘密管理

> 原文：<https://itnext.io/kubernetes-and-secrets-management-in-cloud-858533c20dca?source=collection_archive---------1----------------------->

许多生产系统的运作都离不开秘密。意外泄露机密是应该妥善解决的最大风险之一。开发人员应该尽力保护应用程序的秘密。

![](img/01964014502ec0c5c3131c74bfb78442.png)

一旦公司转向微服务架构，多个服务需要访问不同的秘密才能正常工作，问题就变得更加棘手。这就带来了一个新的挑战:如何分发、管理、监控和轮换应用程序机密，避免意外暴露？

## 库伯内特的秘密

Kubernetes 提供了一个名为 Secret 的对象，可以用来存储应用程序敏感数据，比如密码、SSH 密钥、API 密钥、令牌等等。Kubernetes Secret 可以作为环境变量或作为文件装入到 Pod 容器中。使用 Kubernetes Secrets 允许我们从应用程序部署中抽象出敏感数据和配置。

例如，可以用命令`kubectl create secret`创建一个 Kubernetes 秘密:

或者 Kubernetes `db-credentials.yaml`文件，描述了同样的秘密:

请注意，在 Kubernetes Secret 中存储敏感数据并不能保证它的安全。默认情况下，Kubernetes Secrets 中的所有数据都存储为用`base64`编码的明文。

从版本 1.13 开始，Kubernetes 支持[加密静态的秘密数据](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)，使用内置或外部加密提供者的`EncryptionConfiguration`对象。

当前支持的加密提供程序列表:

[内置提供者](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#providers) : `aescbc`、`aesgsm`、`secretbox`

[KMS 供应商](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/):

*   [谷歌 KMS 提供商](https://cloud.google.com/kubernetes-engine/docs/how-to/encrypting-secrets)
*   [AWS KMS 提供商](https://github.com/kubernetes-sigs/aws-encryption-provider)
*   [蔚蓝 KMS 提供商](https://github.com/Azure/kubernetes-kms)

但是，默认情况下不会强制静态机密加密。即使启用，它也是不够的，不能被认为是一个完整的机密管理解决方案。

完整的机密管理解决方案还必须支持:秘密分发、轮换、细粒度访问控制、审计日志、使用监控、版本控制、高度加密的存储、方便的 API 和客户端 SDK/s，可能还有一些其他有用的功能。

## 云秘密管理

多家云供应商提供秘密管理服务，作为其云平台产品的一部分，帮助您保护访问应用程序、服务和 API 所需的秘密。使用这些服务消除了以明文形式硬编码敏感信息的需要，并开发了自主开发的机密管理生命周期。机密管理服务使您能够使用细粒度的权限和审核来控制对应用程序机密的访问。

## 将 Kubernetes 与秘密管理服务相集成

通常，任何应用程序都可以使用特定于供应商的 SDK/API 来访问存储在机密管理服务中的机密。通常，这需要修改应用程序代码或使用某种引导脚本或 [Init 容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)，使用客户端 CLI 工具或 web APIs。

## 使用`secrets-init`工具让它变得简单

今天，我们发布了 [doitintl/secrets-init](https://github.com/doitintl/secrets-init) -开源工具(Apache License 2.0)，简化了云原生机密管理服务与在云管理或自我管理的 Kubernetes 集群上运行的容器化工作负载的集成。

从本质上来说，`secrets-init`是一个极简的 **init** 系统，设计为在容器环境中作为`PID 1`运行，类似于 [dumb-init](https://github.com/Yelp/dumb-init) ，并提供与多个云原生秘密管理服务的流畅集成，例如:

*   [AWS 秘密管理器](https://aws.amazon.com/secrets-manager/)
*   [AWS 系统管理器参数存储](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
*   [谷歌秘密管理器(测试版)](https://cloud.google.com/secret-manager/docs/)

## 为什么 Docker 容器中需要一个 init 系统？

请[阅读 Yelp *dumb-init* 回购解释](https://github.com/Yelp/dumb-init/blob/v1.2.0/README.md#why-you-need-an-init-system)

总结:

*   正确的信号转发
*   孤儿僵尸收割

## `secrets-init`做什么

`secrets-init`作为`PID 1`运行，就像一个简单的 init 系统充当`ENTRYPOINT`或第一个容器命令，它负责启动一个子进程，将所有系统信号代理给一个以该子进程为根的会话。这是 init 进程的本质。另一方面，`secrets-init`也不加修改地传递*几乎*所有的环境变量，用来自秘密管理服务的值替换特殊的*秘密变量*。

## 与 Docker 集成

`secrets-init`是一个静态编译的二进制文件(没有外部依赖)，可以很容易地嵌入到任何 Docker 映像中。下载`secrets-init`二进制并将其用作 Docker 容器`ENTRYPOINT`。

例如:

## 与 Kubernetes 集成

为了在不修改 Docker 映像的情况下将`secrets-init`与 Kubernetes 对象(Pod/Deployment/Job/etc)一起使用，可以考虑通过 [initContainer](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) 将`secrets-init`注入到目标 Pod 中。您可以使用[doit int/secrets-init](https://hub.docker.com/r/doitintl/secrets-init)Docker 映像或创建自己的映像。将`secrets-init`二进制文件从 init 容器复制到一个公共共享卷，并将 Pod `command`改为作为第一个命令运行`secrets-init`。

## 与 AWS Secrets Manager 集成

要开始将`secrets-init`与 AWS Secrets Manager 一起使用，用户应该将 AWS Secrets ARN 作为环境变量值。`secrets-init`将使用指定的 ARN 将任何环境值解析为引用的秘密值。

例如，在 Kubernetes 作业中使用`secrets-init`:

## 与 AWS 系统管理器参数存储集成

可以使用 AWS Systems Manager 参数存储将应用程序参数存储为明文或加密(一种秘密)。

与前面的例子一样，用户可以将 AWS 参数存储 ARN 作为环境变量值。`secrets-init`将使用指定的 ARN 将任何环境值解析为参考参数值。

AWS 系统管理器参数存储格式示例:

为了从 AWS 机密管理器和参数存储中解析 AWS 机密，`secrets-init`应在有权访问所需机密的 IAM 角色下运行。

这可以通过将 IAM 角色分配给 Kubernetes Pod 或 ECS 任务来实现。参见[介绍服务账户的细粒度 IAM 角色](https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/) EKS 博客文章。

可以将 IAM 角色分配给 EC2 实例，容器在 EC2 实例中运行，但是这种方法不太安全，不推荐使用。

## 与谷歌秘密管理集成

谷歌云最近发布了一项新的管理云中秘密的服务:[谷歌秘密管理器](https://cloud.google.com/secret-manager)

用户可以输入 Google secret name，使用`secrets-init`可识别的前缀(`gcp:secretmanager:`)，后面的 secret name ( `projects/{PROJECT_ID}/secrets/{SECRET_NAME}`或`projects/{PROJECT_ID}/secrets/{SECRET_NAME}/versions/{VERSION}`)作为环境变量值。`secrets-init`将使用指定的名称将任何环境值解析为引用的秘密值。

为了从 Google Secret Manager 解析 Google secrets，`secrets-init`应该在 IAM 角色下运行，该角色有权访问所需的机密。

这可以通过将 IAM 角色分配给具有[工作负载标识](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)的 Kubernetes Pod 来实现。可以将 IAM 角色分配给运行容器的 GCE 实例，但是这个选项不太安全。

## (15–03–2020)更新:Kubernetes 入场网钩

[kube-secrets-init](https://github.com/doitintl/kube-secrets-init) 项目实现了一个 Kubernetes[admission web hook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)，它通过显式(环境变量)或隐式(Kubernetes `Secrets`和`ConfigMaps`)引用云秘密将`initContainer`注入到任何 Pod 中。

## 有用的链接

*   [Kubernetes GKE 工作量标识](https://blog.doit-intl.com/kubernetes-gke-workload-identity-75fa197ff6bf) DoiT 博文
*   [doit intl/secrets-init](https://github.com/doitintl/secrets-init)GitHub 知识库
*   [kube-secrets-init](https://github.com/doitintl/kube-secrets-init)Kubernetes 接纳 webhook GitHub 知识库

## 摘要

我希望，这篇文章对你有用。我期待您的评论和任何问题。

想要更多的故事？查看我们在 [Medium](http://blog.doit-intl.com/) 上的博客，或者[在 Twitter 上关注阿列克谢](https://twitter.com/alexeiled)。