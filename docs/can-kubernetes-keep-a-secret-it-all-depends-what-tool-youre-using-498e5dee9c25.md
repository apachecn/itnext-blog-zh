# Kubernetes 能保守秘密吗？这完全取决于你使用什么工具

> 原文：<https://itnext.io/can-kubernetes-keep-a-secret-it-all-depends-what-tool-youre-using-498e5dee9c25?source=collection_archive---------0----------------------->

![](img/de71935dc77c8e968d70e06c7028342f.png)

在 Soluto，我们的超级开发人员拥有完全的所有权:从编写代码到部署到监控。当我们转向 Kubernetes 时，我们希望保持我们的开发人员独立，并投入大量精力让他们能够快速创建服务(T3)。这一切都运转得很好——直到他们不得不处理凭证。

这个挑战引导我们建立了[Kamus](https://github.com/Soluto/kamus)——一个开源的、 [GitOps](https://www.weave.works/blog/gitops-operations-by-pull-request) 、零信任、针对 Kubernetes 应用的秘密解决方案。Kamus 允许您无缝地加密秘密值，并将它们提交给源代码控制。但是在深入研究 Kamus 如何工作之前，让我们快速回顾一下 Kubernetes 的本地秘密解决方案，以及我们为什么需要 Kamus。

你可能已经知道了，Kubernetes 有一个内置的秘密管理对象，名字超级惊人的“secret”。Kubernetes secret 是一个由 orchestrator 安全存储(例如静态加密)的简单对象，可以包含键值格式的任意数据。这里有一个库伯内特秘密的例子:

```
apiVersion: v1
data:
 super-secret: aHR0cDovL2pvYnMuc29sdXRvLmNvbS9hcHBseS95MDVYSXEvUHJvZHVjdGlvbi1FbmdpbmVlci1EZXZPcHM=
kind: Secret
metadata:
 name: sensitive-data
 namespace: default
type: Opaque
```

如您所见，我们有类型(秘密)和数据。该值是 base64 编码的，因此我们也可以像存储证书一样存储二进制数据。Kubernetes 通过让您简单地将秘密装载到您的容器上，或者作为 env var——不推荐——或者作为一个文件，使得使用秘密变得容易。

# 库伯内特平原秘密有什么问题？

为什么我们需要另一种解决方案？首先，Kubernetes 的秘密是 base64 编码，而不是*加密*。这意味着您不能将这些文件按原样提交到源代码控制中(这甚至是在文档中指定的[)。任何能够访问存储库的人都可以获取秘密并使用秘密值(除非这是一个您愿意接受的威胁，这在某些情况下是有效的)。因此，我们仍然需要一种简单的方法来存储这个秘密并将其上传到生产中，理想情况下不需要任何手动操作。](https://kubernetes.io/docs/concepts/configuration/secret/#security-properties)

这就引出了我的第二点:可用性。秘密对于服务的正常执行至关重要，其中一个秘密的更改会导致生产问题。您需要很好地了解这些秘密——谁更改了它们，何时更改了什么——以便您可以调查生产问题。这就是为什么我如此支持 GitOps，所有的事情都通过 git 进行审计。是的，Kubernetes 有审计机制，但它不像 git 那样简单。因此，当谈到可用性时，我们需要一种透明且易于遵循的方法来修改秘密。

第三，考虑你的服务如何消耗秘密。在 Kubernetes 中，你可以通过两种方式之一使用秘密:将秘密作为一个[环境变量](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)或者作为一个[卷](https://kubernetes.io/docs/concepts/storage/volumes/#secret)挂载。将秘密作为环境变量安装是最简单的方法，但是并不是每个人都喜欢将敏感数据存储在环境变量中(查看这个 [twitter 帖子](https://twitter.com/omerlh/status/1079088158929797121)以了解一些观点)。

因为这个原因，有些人(包括我自己)更喜欢把[秘密装裱成册](https://kubernetes.io/docs/concepts/storage/volumes/#secret)。这允许您像访问容器中的普通文件一样访问秘密内容。Kubernetes 为秘密数据中的每个密钥创建一个文件，并将解码值作为文件内容。当有一两个键时，这可能行得通，但是键越多，就越难——你需要从应用程序的代码中读取所有这些文件。这就是为什么我们通常最终会创建一个配置文件，对其进行 base64 编码，并将其存储为一个单独的秘密值。然而，这使得理解发生了什么变化变得更加困难。

最后，Kubernetes 拥有强大的 RBAC 支持，但它也有自己的缺点。秘密的现有权限是 *get* 和 *set* (以及与本讨论不太相关的其他权限)。秘密在静止时被加密，但是没有权限说“允许用户得到加密的秘密”。一旦用户被允许得到一个秘密，她将得到解密。理想情况下，我们希望有一个零信任系统，一旦你加密了一个秘密，就没有简单的方法来解密它。目前，使用 Kubernetes 没有简单的方法来实现这一点。

# Kubernetes 秘密的替代品

有一些很好的替代品，比如[密封秘密](https://github.com/bitnami-labs/sealed-secrets)和[头盔秘密](https://github.com/futuresimple/helm-secrets)。这两种解决方案都允许您创建可以提交给源代码控制的加密秘密，并在以后解密成常规的 Kubernetes 秘密(无论是在 CD 上还是在集群中)。但是这些解决方案也不是完美的。不支持一次性加密，因为为了修改秘密中的值，您必须解密其内容。我们之前所有的可用性问题仍然存在。

# 介绍 Kamus

一天，在制作[胶水](https://github.com/OWASP/glue)的时候，我使用了 Travis secrets [加密解决方案](https://docs.travis-ci.com/user/encryption-keys/)。如果您不熟悉 Travis 如何处理秘密，服务器会为每个存储库创建一个 RSA 密钥对，然后公开公钥。Devs 可以使用公钥来加密值，这些值以后只能由 Travis 解密。这让我想到:如果我为 Kubernetes 创建一个类似的解决方案会怎么样？如果每个部署都有一个特定的 RSA 密钥对，这样我就可以利用相同的机制，那会怎么样？结果……是卡姆斯。

让我们看看如何使用 Kamus，然后深入了解它的工作原理和解决方案的安全性。Kamus 为一个特定的[服务帐户](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)加密秘密，该帐户随后被一个 pod 用来认证 Kamus 并解密它们。因此，我们需要做的第一件事是创建一个服务帐户:

```
kubectl create sa dummy
```

现在我们可以使用 [kamus-cli](https://github.com/Soluto/kamus/tree/master/cli) 来加密这个服务帐户的超级秘密值:

```
kamus-cli encrypt --secret super-secret --service-account some-sa --namespace some-ns --kamus-url [https://kamus.my-company.com](https://kamus.my-company.com)
```

返回值将是加密值。我们现在可以安全地将它提交给 git，并且在运行时，使用 Kamus API 和 pod 的服务帐户的令牌来解密这个值。如果具有不同服务帐户的不同 pod 尝试解密该值，操作将会失败。

解密一个秘密很简单——只需使用 [Kamus init 容器](https://github.com/Soluto/kamus/tree/master/init-container)。init 容器将读取所有加密的秘密，使用服务帐户令牌用 Kamus 解密这些值，并将解密的值写入临时位置。要了解这一点，请尝试运行[示例应用程序](https://github.com/Soluto/kamus/tree/master/example)。

# 准备好试试了吗？

在过去的六个月里，我们 Soluto 一直在生产中使用 Kamus。这极大地改善了开发人员的体验，现在使用 secrets 是一项有趣而简单的任务。这也是我们决定开源 Kamus 的原因。开始使用它很简单——只需在 Kubernetes 集群上安装 Helm 即可:

```
helm repo add soluto [https://charts.soluto.io](https://charts.soluto.io)helm install kamus soluto/kamus
```

您可以查看我们的[安装指南](https://github.com/Soluto/kamus/blob/master/docs/install.md)进行生产级部署。

安全对我们很重要，这也是我们建造 Kamus 的原因。因为我们知道像 Kamus 这样的项目有多敏感，作为开源版本的一部分，我们也发布了一个[威胁模型](https://github.com/Soluto/kamus/tree/master/docs/features)。在这里，您可以找到我们检测到的所有不同威胁，以及 Kamus 如何减轻这些威胁。如果您发现了安全问题，我们很乐意听到—请查看 [security.md 文档](https://github.com/Soluto/kamus/blob/master/security.md)，了解如何向我们报告安全问题。

确保秘密安全至关重要，但并不总是那么简单。Kamus 通过提供一个安全的 GitOps 解决方案来解决这个问题。它让你告诉 Kubernetes 一个秘密，并确保没有其他人知道这个秘密。剩下的就看你的了——你会给 Kamus 一个机会吗？

*最初发表于* [*Soluto 工程博客*](https://blog.solutotlv.com/can-kubernetes-keep-a-secret?utm_source=medium) *。*