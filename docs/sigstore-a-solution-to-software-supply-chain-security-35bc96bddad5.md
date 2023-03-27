# Sigstore:软件供应链安全解决方案

> 原文：<https://itnext.io/sigstore-a-solution-to-software-supply-chain-security-35bc96bddad5?source=collection_archive---------2----------------------->

## 关于 sigstore 项目及其在软件供应链信任和安全中的作用，您需要了解的一切

![](img/41e89857a009476481f5de91510069dc.png)

[邵杰](https://unsplash.com/@neural_notworks?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍照

最近几个月，由于攻击的增加，供应链安全领域出现了许多噪音，其中值得注意的有*微软 Exchange Server* 、 *Colonial pipeline* 或*网络安全管理软件产品*漏洞。如果有合适的工具，这些攻击本来是可以避免的，但是找到合适的工具可能会很困难，因为这个领域很难导航，而且我们大多数人(开发人员)都不是安全专家。然而，最近宣布了一个新项目，可能会解决我们所有人的许多问题。它的名字是 *sigstore* ，在本文中，我们将看看它的作用，为什么我们需要它，以及它如何适应这个领域现有工具的环境。

# 西格-什么？

sigstore 是这个街区的新人。这是 CNCF 旗下的一个项目，在三月份被*“捐赠”给基金会。其宗旨是提供软件签名公益服务。这意味着它应该成为相当于*加密*的软件签名。然而，sigstore 不仅仅是一个工具或一个软件，它是一个旨在简化软件签名和透明性的项目集合。目前它的主要组件是`fulcio`、`rekor`和`cosign`(稍后会有更多细节)。*

*现在你可能会问*“为什么我们真的需要这个？”软件签名不是一个新问题，所以肯定已经有一些解决方案了，对吗？是的，但是签署软件和维护密钥是非常困难的，特别是对于非安全人员，现有工具如 PGP 的 UX 还有许多不足之处。这就是为什么我们需要类似 sigstore 的东西——一个易于使用的软件/工具集，用于签署软件工件。**

*此外，sigstore 的解决方案优于试图解决相同问题的 PGP 等工具还有几个原因。与其他工具不同，sigstore 不需要管理私钥。由于有了更好的 UX，你也不必了解安全标准的来龙去脉。sigstore 还使管理撤销变得更加简单，尽管如此，它仍然提供了软件签名所需的所有功能，即*完整性*、*不可否认性*和*认证*。*

*底线是 sigstore 的目标是使工件签名如此简单，以至于它可以默认和透明地完成，并且在所有注册中心和工件存储中无处不在。*

# *可供选择的事物*

*从上面来看，sigstore 似乎是可以解决所有问题的工具，但是如果你开始在谷歌上搜索，你会发现供应链安全领域有很多很棒的工具。然而，这些工具中的大多数并不完全服务于相同的目的，它们实际上是 sigstore 正在做的事情的补充。因此，让我们也回顾一下其余的景观，看看还有什么:*

*   *你会碰到的众多工具之一是 [*【更新框架】【TUF】*](https://theupdateframework.io/overview/)。它也是 CNCF 的一部分，其目的是专门保护为某些特定系统(如 YUM、PyPI)查找和下载补丁/更新的过程。这个系统适用于那些打算使用更新系统来发布的工件。*
*   *在谈论 TUF 时，提到*公证*也是有意义的，它是 TUF 规范的一个实现。它最显著地用于 *Docker 公证人*中，该公证人提供对发送到远程 Docker 注册处和从远程 Docker 注册处接收的数据使用数字签名的能力。你可以在这里阅读更多关于 *Docker 内容信任* [或者也可以尝试玩`docker trust`命令。如果你想实现类似的东西，那么你可以查看这篇文章来获得完整的演示。](https://docs.docker.com/engine/security/trust/)*
*   *另一个很棒的工具是。这个工具不仅仅是为工件签名，它还产生关于软件是如何产生的证明。本质上，验证流水线中的每个任务都按计划执行，从而保证最终产品不会被篡改。您可以将*作为 *Tekton 链条*的一部分整体使用*。*
*   *最后，我还想提到 [Trillian](https://transparency.dev/#trillian) ，它是一个防篡改日志，存储了准确、不可变且可验证的活动历史。例如，这种日志可用于向系统添加篡改检查、简化法规遵从性或跟踪文档修改。sigstore 还包括一个名为`rekor`的防篡改日志，这将在后面讨论。*
*   *我们还有很多可以谈的，但那需要一段时间。如果您想深入了解，请查看 [CNCF 景观页面](https://landscape.cncf.io/)，以及更具体的*安全与合规*(例如 OPA)和*密钥管理*(例如 SPIFFE 和 SPIRE)部分。*

*所有这些工具都各有利弊，可以组合和扩展以提供更强的安全性。关于这个的更多细节，你可以在 sigstore 的社区知识库中查看[文档](https://github.com/sigstore/community/blob/main/docs/zero-trust-supply-chains.pdf)(参见*进一步工作*部分)。*

# *成分*

*在深入研究 sigstore 的组件之前，我们首先需要了解签名过程的基础。基本步骤如下:*

1.  *生成短期代码签名凭据(密钥对)。*
2.  *用户通过 *OpenID Connect (OIDC)* 提供商(如 Google 或 GitHub)进行身份验证，以验证电子邮件地址的所有权和先前生成的密钥的所有权。*
3.  *如果身份验证成功，用户将收到代码签名证书。*
4.  *代码签名证书被发布到透明日志，以便其他人可以对其进行验证。*
5.  *用户使用代码签名证书及其密钥对对工件(如容器图像)进行签名。*
6.  *来自工件的签名被发布到透明日志。*
7.  *用于创建签名的短期代码签名凭据被销毁。*
8.  *可以发布已签名的工件(例如，在容器注册表上)。*

*在 sigstore 网站的[什么是 sigstore 中也可以找到对该过程的不同解释。第](https://sigstore.dev/what_is_sigstore/)节。*

*现在我们对它的工作原理有了更好的了解，让我们看看所有的组件。有几件事适用于所有这些应用程序，也就是说，它们可以(也应该)默认运行在云中(运行在 Kubernetes 上)。即使 sigstore 托管公益服务，您也可以自己托管这些组件中的任何一个(例如在防火墙后面)，并且您也不需要使用所有的服务，而可以只使用其中的一个，例如只使用透明日志服务器。*

*至于单个组件，目前有 3 个主要部分:*

*   *[cosign](https://github.com/sigstore/cosign) 是一个容器签名工具。它的职责是签署集装箱，并向 OCI 注册处发布信息。在与步骤 1、5、6 和 7 相匹配的上述过程中。*
*   *fulcio 是代码签名证书的根 CA。它的工作是发布代码签名证书，并将 OIDC 身份嵌入到代码签名证书中。从这个描述中，我们可以看到它在步骤 2、3、4 和 8 中执行这些任务。*
*   *[rekor](https://github.com/sigstore/rekor) 是透明日志。它是只附加的、不可变的分类帐，作为谁签署了什么的事实的透明来源。*

*现在，在实践中，上述工具和服务将以下列方式用于执行签名过程:*

*`cosign`生成一个临时密钥对，并向`fulcio`请求代码签名证书，然后要求您登录您选择的 OIDC 提供商。它使用身份验证来验证您是临时私钥的所有者。`cosign`然后将检索您想要签名的图像的容器图像清单，并将使用它先前生成的密钥生成签名。接下来，`cosign`将签名、证书和公钥上传到注册表。最后，它将信息发送给`rekor`，后者验证签名并将条目添加到透明日志中。这里，这个条目包括工件摘要、签名和公钥。此时，可以删除短暂的密钥对。*

*除了这些软件之外，还需要一个监控服务来检查透明度日志(`rekor`)是否有任何异常。这种异常的例子可能是，如果有人窃取了您的密码，并使用您的 OpenID 身份来签署和发布一个工件，这将从透明日志中清除。*

*最后，需要一种方法来——例如——说明谁是真正被信任来签署某个项目的工件/发布的维护人员。这可以例如通过使用开放策略代理(OPA)和通过在项目存储库中维护列表电子邮件(OpenID 身份)并允许仅在该列表中的人签署工件来完成。*

# *结束语*

*这种安全实践现在并不常见，在某些情况下确实被忽视了。因此，越多的人开始使用它，它就越有可能成为默认的过程和良好的实践。也就是说，在撰写本文时，sigstore 是一个非常年轻的项目，还没有准备好投入生产，但应该会在夏季结束时完成，所以很快您就可以将这些知识很好地利用起来，并帮助软件供应链变得更加安全。*

*综上所述，本文应该是关于供应链安全的初级读本，并让您对 sigstore 有一个大致的了解，在后续文章中，我们将通过实际例子详细讨论实际的签名过程。*

# *资源*

*   *[Sigstore 博客](https://blog.sigstore.dev)*
*   *[transparency.dev](https://transparency.dev/)*
*   *[零信任供应链](https://github.com/sigstore/community/blob/main/docs/zero-trust-supply-chains.pdf)*

**本文最初发布于*[*martinheinz . dev*](https://martinheinz.dev/blog/55?utm_source=medium&utm_medium=referral&utm_campaign=blog_post_55)*

*[](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [## 用 seccomp 强化 Docker 和 Kubernetes

### 您的容器可能不像您想象的那样安全，但是 seccomp 配置文件可以帮助您解决这个问题…

itnext.io](/hardening-docker-and-kubernetes-with-seccomp-a88b1b4e2111) [](https://towardsdatascience.com/analyzing-docker-image-security-ed5cf7e93751) [## 分析 Docker 图像安全性

### 码头集装箱远没有你想象的那么安全…

towardsdatascience.com](https://towardsdatascience.com/analyzing-docker-image-security-ed5cf7e93751) [](https://towardsdatascience.com/security-and-cryptography-mistakes-you-are-probably-doing-all-the-time-7407c332944f) [## 您可能一直在犯的安全和加密错误

### 让我们澄清一些关于加密和安全的常见误解…

towardsdatascience.com](https://towardsdatascience.com/security-and-cryptography-mistakes-you-are-probably-doing-all-the-time-7407c332944f)*