# 为您的生产部署升级 Cert-Manager，无需停机

> 原文：<https://itnext.io/upgrade-cert-manager-for-your-production-deployment-without-downtime-ee5d32fabec8?source=collection_archive---------1----------------------->

![](img/cbe44513bae4adf260a4fc2cfeb3691b.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [Brian Scott](https://unsplash.com/@briangb?utm_source=medium&utm_medium=referral) 拍照

# 什么是证书管理器？

[Cert-Manager](https://cert-manager.io/docs/) 是 Kubernetes 和 OpenShift 最广泛采用的云本地证书管理解决方案之一。它简化了从各种来源获取、更新和使用证书的过程，包括著名的非营利认证机构“让我们加密”。为了方便证书管理过程，Cert-Manager 添加了许多相关对象，如证书、证书请求和发行者等。，作为定制资源类型，由 Kubernetes 集群中的 [CRDs(定制资源定义)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)定义。

将 Cert-Manager 升级到新版本时，通常需要更新这些 CRD 的定义。不幸的是，这不是一个简单的过程。当问题出现时，您通常必须完全卸载 Cert-Manager 来解决问题，这会导致生产部署中的停机事件。

在本文中，我们将讨论一些确保 Cert-Manager 顺利升级以避免生产部署停机的选项。

# 推荐的安装/设置选项

许多选项可用于在您的集群上部署全新的 Cert-Manager 安装。然而，在升级或出现问题时，并不是每个选项都有很大帮助。如果您要部署一个新的安装，您可能要考虑我推荐的安装/设置选项，以便将来升级和维护。您还将从以下几节中了解更多关于其背后的原因。

*   [使用](https://cert-manager.io/docs/installation/helm/) `[kubectl](https://cert-manager.io/docs/installation/helm/)` [手动安装所需的 CRD，并使用 Helm](https://cert-manager.io/docs/installation/helm/)
    安装 Cert-Manager-确保`--enable-certificate-owner-ref`标志未设置为真。该标志的默认值为 false。然而，如果你在你的 helm 配置中设置它为 true，当相关的 Cert-Manager 自定义`Certificate`资源被删除时，包含 X.509 证书的`Secret`对象将被自动删除。这可能会导致重新安装过程中不必要的停机。
*   [手动创建 Cert-Manager 自定义](https://cert-manager.io/docs/usage/certificate/) `[Certificate](https://cert-manager.io/docs/usage/certificate/)` [](https://cert-manager.io/docs/usage/certificate/)资源，而不是通过添加 [ingress-shim](https://cert-manager.io/docs/usage/ingress/) 监视的注释来保护您的`Ingress`资源。
    - `Certificate`通过入口填充程序为入口创建的资源将具有指向入口资源的所有者引用。当您必须重新创建您的`Ingress`资源时，例如为了迁移到新的集群，此所有者引用将是不正确的。创建/管理没有入口填充的`Certificate`资源将省去修复这个潜在问题的麻烦。

# 备份证书管理器资源

为了避免停机，理想情况下，我们应该尝试升级现有的 Cert-Manager 安装。但是，在获得新版本的安装以正常工作之前，您总是有可能必须删除当前的安装。发生这种情况时，备份现有的 Cert-Manager 资源将非常有助于恢复现有的设置并避免停机。

要备份所有必要的 Cert-Manager 配置资源，您可以运行:

```
kubectl get --all-namespaces -oyaml issuer,clusterissuer,cert > backup.yaml
```

该命令将备份您可能在集群中创建的`Issuer`、`ClusterIssuer`和`Certificate`资源。

> 请注意，如果您使用 ingress-shim 自动创建`Certificate`资源，您应该从备份中排除`Certificate`资源。否则，`*Certificate*`资源的错误所有者引用将导致配置同步问题。即`Ingress`的任何更新都不会应用到`Certificate`。

要在没有`Certificate`资源的情况下进行备份，您可以改为运行以下命令:

```
kubectl get --all-namespaces -oyaml issuer,clusterissuer > backup.yaml
```

除了 Cert-Manager 配置资源之外，您还需要备份包含 X.509 证书的`Secret`资源:

```
kubectl -n [namespace] get secret [secret name] -oyaml > secret-bak.yaml 
```

这里，`[namespace]`是包含`Secret`的名称空间，`[secret name]`是`Secret`资源的名称。

除了包含已颁发的 X.509 证书的`Secret`资源之外，如果您正在将数据传输到新的集群，您可能还需要跨您已配置的颁发者引用的其他秘密资源进行复制。更多信息可在[这里](https://cert-manager.io/docs/tutorials/backup/#backup)找到。

# 如何使用您的备份

当事情出错时，如果`Secret`资源由于任何原因丢失，您应该总是首先恢复`Secret`资源。这应该在您尝试恢复任何`Issuer`、`ClusterIssuer`、`Certificate`或`Ingress`资源之前发生，以避免触发可能导致停机的不必要的证书重新颁发。

要恢复`Secret`资源，您可以运行:

```
kubectl apply -f secret-bak.yaml
```

要恢复证书管理器配置资源，您可以运行:

```
kubectl apply -f <(awk '!/^ *(resourceVersion|uid): [^ ]+$/' backup.yaml)
```

> `awk`命令将删除不需要恢复的`uid`和`resourceVersion`字段。

# 验证“- enable-certificate-owner-ref”标志

在升级您现有的 Cert-Manager 安装之前，您应该确保在您当前的部署中没有将`--enable-certificate-owner-ref`标志设置为 true。否则，当`Certificate`资源由于任何原因被删除时，包含已颁发的 X.509 证书的`Secret`资源也将被删除，这在大多数情况下会导致停机。

该标志的默认值为 false。但是，如果在您当前的部署中将该标志设置为 true，您应该在将 Cert-Manager 升级到较新版本之前，更改您的部署配置，将该标志设置回 false。

# 升级证书管理器资源 API 版本

如果需要将 Cert-Manager 升级到 1.6 版或更高版本，必须首先升级现有 Cert-Manager 资源的 API 版本。最简单的选择是使用`[cmctl](https://cert-manager.io/docs/usage/cmctl)`[CLI 工具。](https://cert-manager.io/docs/usage/cmctl)你可以从[发布页面](https://github.com/cert-manager/cert-manager/releases)为你的系统下载二进制文件。

安装完`cmctl` CLI 工具后，您可以运行以下命令来升级集群中现有的 Cert-Manager 资源 API 版本。

```
cmctl upgrade migrate-api-version --qps 5 --burst 10
```

`cmctl`还附带了[一个转换工具](https://cert-manager.io/docs/usage/cmctl/#convert)，允许你将保存的 Cert-Manager 资源清单文件(例如之前生成的备份文件)转换成较新的版本。

```
cmctl convert -f cert.yaml
```

# 升级 CRD 定义

升级现有 Cert-Manager 安装的第一步是升级已安装的 CRD 定义。您可以:

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/<version>/cert-manager.yaml
```

> 这里，<version>是你打算升级的版本。例如`v1.8.2`</version>

# 升级证书管理器

升级 CRD 定义后，您可以将 Cert-Manager helm 图表升级到您计划升级的版本。

```
helm upgrade --namespace <namespace> --version <version> <release_name> jetstack/cert-manager
```

> 这里，<namespace>是证书管理器部署名称空间。<version>是您计划升级到的版本。例如`1.8.2`(没有起始字母`v`)。<发布名称>是你的头盔部署发布名称。</version></namespace>

您可以使用`cmctl` CLI 来[验证升级后的安装](https://cert-manager.io/docs/installation/verify/#check-cert-manager-api):

```
$ cmctl check apiThe cert-manager API is ready
```

# 强制续订证书

您可以使用`cmctl` CLI 工具检查现有证书的状态:

```
cmctl status certificate [cert name]
```

如果您在升级后卡住了证书续订顺序，您可以使用`cmctl` CLI 工具强制续订证书:

```
cmctl renew [cert name]
```

# 当事情出错时

首先，您应该检查 Cert-Manager 控制器 pod 的日志，以消除任何明显的配置错误。例如，不正确的集群颁发者 [ACME](https://cert-manager.io/docs/configuration/acme/) 凭证。

```
kubectl -n [namespace] logs cert-manager-xxxxxxxxxx-xxxxx
```

> 这里，[名称空间]是证书管理器部署名称空间，而“证书管理器-xxxxxxxxxx-xxxxx”是其中一个证书管理器控制器盒的名称。

然而，您总是有可能无法确定当前安装的根本原因。发生这种情况时，您可能需要完全删除 Cert-Manger 安装，并在集群中重新安装。

# 删除证书管理器安装

要重新安装 Cert-Manager，您需要首先完全删除当前安装。只要不删除包含已颁发的 X.509 证书的`Secret`资源，这不一定会导致应用程序停机。这就是为什么我们需要确保您的安装的`--enable-certificate-owner-ref`标志没有设置为 true，以确保删除`Certificate`资源不会删除`Secret`资源。

要删除 cert-manager 安装，我们必须首先确保所有 Cert-Manager 资源都已删除。

```
kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces
```

删除所有这些资源后，您可以通过以下方式卸载 Cert-Manager:

```
helm --namespace <namespace> delete <release_name>
```

> 这里，<namespace>是证书管理器部署名称空间。<release_name>是 cert-manager helm chart 部署发布名称。</release_name></namespace>

如果您当前的安装不是使用 Helm 安装的，您也可以使用`kubectl` CLI 工具删除安装。

```
kubectl delete -f https://github.com/cert-manager/cert-manager/releases/download/vX.Y.Z/cert-manager.yaml
```

> 给你，vX。Y.Z 是您当前的安装版本。例如 v1.8.2

最后，我们需要删除安装的所有 CRD 定义:

```
kubectl delete -f https://github.com/cert-manager/cert-manager/releases/download/vX.Y.Z/cert-manager.crds.yaml
```

> 这里，` vX.Y.Z '是已安装版本的版本。例如 v1.8.2

# 重新安装证书管理器

一旦现有的 Cert-Manager 安装被完全删除，我们可以通过推荐的安装选项继续安装新版本的 Cert-Manager。以前生成的证书管理器配置资源备份可用于恢复以前的设置。

恢复所有资源后，我们可以使用前面介绍的`cmctl` CLI 工具来验证安装，并在必要时强制更新任何现有证书。

# 结论

Cert-Manager 是针对 Kubernetes 和 OpenShift 的优秀云原生证书管理解决方案。然而，升级现有的 Cert-Manager 安装并不简单。本文介绍了帮助平稳升级的一般过程。为了防止任何意外问题导致安装不正常，本文还讨论了如何在重新安装过程中避免停机。