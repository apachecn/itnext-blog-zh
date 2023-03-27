# 从 Cert-Manager v0.4 升级的基本指南

> 原文：<https://itnext.io/the-essential-guide-to-upgrading-from-cert-manager-v0-4-56d2a4ae32c2?source=collection_archive---------3----------------------->

![](img/0cdd8a5e3763f5cd8e94104a5ebd8923.png)

罗伯特·v·鲁杰罗在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# 什么是证书管理器？

Cert-manager 是由 [Jetstack](https://cert-manager.io/docs/) 提供的开源 Kubernetes-native 控制器。它可以为您颁发和管理群集中的所有证书，并在证书即将到期时取出 LetsEncrypt 证书并进行更新。Cert-manager 非常受欢迎，许多组织使用它来降低为加密内部点对点通信或为入口控制器公开的公共端点生成证书而颁发证书的复杂性。

当我在 2018 年夏天开始学习 Kubernetes 时，我第一次被介绍给 cert-manager。当时，我们使用最新版本之一的`v0.4.0`，在集群内发布和管理我们的证书。那年 9 月，我们研究了如何使用`v0.5.0`，它对如何使用 Helm 安装 cert-manager 进行了突破性的改变。由于 v0.4 版本仍然相当新，我们决定跳过 v0.5，以后再来看。

现在是 2020 年 2 月。后来有了新的工作和角色，我再次和我的老朋友 cert-manager 一起管理 Kubernetes 环境。我现在正在做的项目还在运行`v0.4.1`，比最新发布的`v0.13.0`落后多*英里。为什么我们还没有升级？这很大程度上是一句谚语“如果它没坏，就不要修理它。”还有，不能从 v0.4 跳到更高的版本；您必须按顺序进行，因为每个次要版本都有突破性的变化(尽管并不总是如此)。*

# 我为什么要升级？

回到 2018 年，我们注意到我们受到 LetsEncrypt 的速率限制。由于我们每天都要在沙盒环境中部署数十次，并且每次都要为多个域重新创建证书(我知道这很不礼貌)，我们认为我们的测试有点过于激进了。原来[这实际上是 cert-manager](https://blog.jetstack.io/blog/cert-manager-0.6/) 早期版本中的一个问题，从`v0.6.0`开始，每个版本都在逐步解决这个问题。

此外，cert-manager 的 Helm chart 的早期版本包括 CustomResourceDefinitions。CustomResourceDefinitions，或 CRDs，允许您扩展 Kubernetes 的 API 并创建自己的资源。例如，如果您为证书创建了一个资源定义，并将其捆绑到一个图表中，则 CRD 由 Helm 拥有和管理。如果您随后在该图表上运行`helm uninstall`，Helm 将删除 CRD，这将提示 Kubernetes API 删除与其相关联的所有对象。这意味着在旧版本的图表中，您的证书固有地绑定到 Cert-manager 的安装中，如果您卸载它，您的证书也会随之卸载。奇怪的是，如果您运行`helm upgrade`并且对其中一个 CRD 进行了更改，Helm 很可能不会考虑该更改，升级将会失败。James Munnelly (来自 Jetstack)在本期 GitHub 上总结了将 CRDs 从头盔图中移除的所有原因。

# 升级难吗？

升级说明似乎表明从`v0.4.1`到`v0.7.2`应该是一个安全的跳跃。任何 v0.7 以后的版本都需要额外的基础工作，我将在这篇文章的后面介绍。

![](img/c5146b444af0f302e44e036def8e93ee.png)

感觉我终于从深渊中浮出来了！安东尼·泰瑞尔在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

# 第 1 部分:从 v0.4 到 v0.7.2

正如我之前提到的，由于 Helm 对 CRD 的处理不当——cert-manager 严重依赖 CRD 来正常运行——jet stack 决定从 Helm chart 中完全删除他们的 CRD 模板。现在，在通过`kubectl apply -f`安装舵图之前，你必须手动安装它们*。同样，如果您想完全删除 cert-manager，您将必须卸载 Helm chart，然后运行`kubectl delete -f`来删除所有的 CRD。这是一个仔细阅读文档并向自动化脚本添加额外步骤的问题。*

另一点要提到的是，Jetstack 强烈建议您将 cert-manager *安装到它自己的名称空间*中，并对其进行标记，这样 cert-manager 就不会尝试验证它自己的自签名证书，否则会失败。你可以在这里阅读更多关于这个[的内容。](https://cert-manager-munnerz.readthedocs.io/en/stable/admin/resource-validation-webhook.html#tls-configuration)

下面是我们从 v0.4 升级到 v0.7.2 的步骤:

1.  备份证书，颁发者，群集颁发者
2.  从 kube-system 名称空间中删除旧版本的 cert-manager(如果您在那里安装了它)
3.  通过 kubectl 为证书管理器安装新的 CustomResourceDefinitions
4.  创建`cert-manager`名称空间并用`certmanager.k8s.io/disable-validation="true"`标记它
5.  将带有新舵图的新版本证书管理器安装到名为`cert-manager`的专用命名空间中
6.  恢复我们之前创建的备份

下面是第一步(1)的样子:

然后，我们可以删除旧版本的 cert-manager (2 ),并确保 Helm 也删除了与 cert-manager 关联的所有 CustomResourceDefinitions:

上述错误意味着 cert-manager 不再存在于我们的群集中。在升级之前，请检查以确保没有与以前版本的 cert-manager 关联的旧 ClusterRoleBindings、ClusterRoles 和 ServiceAccounts。这可能会与您即将安装的新版本冲突。运行以下命令以确保它们已被删除:

完成后，我们可以为版本`v0.7.2`应用新的 CRDs (3)(不再与图表捆绑在一起)，然后将所需版本的 cert-manager (4，5)安装到 cert-manager 名称空间中:

最后，我们将通过运行“ku bectl apply-f cert-manager-backup . YAML”从我们的备份(6)中恢复集群颁发者、颁发者和证书

现在，我们已经从一个极其过时的版本的头盔跳到了`v0.7.2`！**为了让整个过程对你来说更容易，我起草了一个脚本，应该可以一步到位:**

# 第 2 部分:从 v0.7.2 到 v0.12

这里是 Jetstack 提供的文档，它概括了每次升级所需的步骤。每个版本都有不同的需求，其中一些是可选的，后来变成了需求。因此，我不建议跳过多个版本，因为您可能会将自己置于以后环境崩溃的风险中:

从 v0.7 到 v0.8: [将 ACME 发行者和证书更新为新格式](https://cert-manager.io/docs/installation/upgrading/upgrading-0.7-0.8/)(不要求，但强烈建议)

从 v0.8 到 v0.9: [在应用升级之前删除证书管理器部署](https://cert-manager.io/docs/installation/upgrading/upgrading-0.8-0.9/)

*v0.9 到 v0.10: [删除 webhook 的发布者和证书资源](https://cert-manager.io/docs/installation/upgrading/upgrading-0.9-0.10/)(仅当使用静态清单时)*

*从 v0.10 到 v0.11:在将升级到 v0.11 *之前，必须升级 ACME 发行者和证书*。*此外，还有反映新 API 的注释更改。[所有这些都可以在这里找到。](https://cert-manager.io/docs/installation/upgrading/upgrading-0.10-0.11/)**

*v0.11 到 v0.12: [删除 webhook API 服务](https://cert-manager.io/docs/installation/upgrading/upgrading-0.11-0.12/)*

*从 v0.12 到 v0.3: [不需要特殊的升级步骤！](https://cert-manager.io/docs/installation/upgrading/upgrading-0.12-0.13/)*

# *进一步故障排除*

*从备份中恢复证书时，您可能会注意到以下错误:`spec.dnsNames in body must be of type array: "null"`。虽然我还不知道为什么会发生这种情况，但是解决它很容易。在我们之前创建的备份中，找到有问题的证书资源中的`dnsNames`属性。将其从空值更改为空数组:*

```
*apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: test-ca-crt
spec:
  secretName: test-ca-crt
  commonName: test-ca-certificate
  issuerRef:
    name: ca-issuer
    kind: Issuer
  dnsNames: <---change this to the following below
  dnsNames: []*
```

*这个 GitHub 用户很好地分析了如何根据模式验证证书，以及为什么这个特殊的案例会给你一个错误。*

*如果您想回滚到 cert-manager 的早期版本，您可能还会遇到无法删除`cert-manager`名称空间的问题。这可以通过运行`kubectl delete -f`删除您随图表安装的所有 CRD，并将 URL 传递给您安装的图表版本的 CRD 来解决。Jetstack 有一个正确删除 cert-manager 的指南，以及为什么会出现这种情况的解释[这里](https://cert-manager.io/docs/installation/uninstall/kubernetes/#namespace-stuck-in-terminating-state)。*