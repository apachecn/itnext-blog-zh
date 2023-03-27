# cert-manager:Kubernetes 的本地 x509 证书管理

> 原文：<https://itnext.io/cert-manager-native-x509-certificate-management-for-kubernetes-80ac478d314f?source=collection_archive---------2----------------------->

![](img/696af5858c5e58d18f9a0d26661c8300.png)

密切关注 Jetstack 的[开源项目](https://www.jetstack.io/open-source)的各位可能已经注意到，我们的新证书管理工具 [cert-manager](https://github.com/jetstack/cert-manager) 已经推出有一段时间了。事实上，我们现在在 GitHub 上有超过 1000 颗星星！

Cert-manager 是用于 Kubernetes 的通用 x509 证书管理工具。在当今的现代网络中，保护应用流量至关重要。cert-manager 旨在简化组织内证书的管理、颁发和更新。

[kube-lego](https://github.com/jetstack/kube-lego) ，我们的[原创](https://blog.jetstack.io/blog/kube-lego/)[Let ' s Encrypt](https://www.letsencrypt.org/)Kubernetes Ingress 资源证书供应工具，取得了巨大的成功。它使保护用户和集群入口点之间的流量变得简单。然而，随着时间的推移，仅仅围绕 Kubernetes 入口构建控制器的局限性变得显而易见。

通过围绕 Kubernetes[customresourceditions](https://kubernetes.io/docs/concepts/api-extension/custom-resources/)构建 cert-manager，我们已经能够极大地增加配置的灵活性、调试功能，并且支持比单独加密更广泛的 ca。

这篇文章高度概括了 cert-manager 的工作方式，并将重点介绍该项目的一些新特性和最新进展。

# 发行人和证书

在 x509 证书的真实世界中，ca(证书颁发机构)是一个信任点，负责以签名和受信任的 x509 证书的形式向各种客户端颁发身份。

让我们介绍一下 ACME 加密协议，但是，并不是所有的 ca 都支持这个协议。

为了支持许多不同类型的认证机构，我们在 Kubernetes API 中以“发行者”的形式引入了 CA 的概念。

cert-manager“Issuer”资源代表能够签署 x509 证书请求的实体。

今天，我们支持 ACME，CA(即存储在秘密资源中的简单签名密钥对)和从 v0.3.0 版本开始的 Hashicorp Vault！

下面是一个发行者资源的示例:

```
apiVersion: certmanager.k8s.io/v1alpha1 kind: ClusterIssuer metadata: name: letsencrypt-staging spec: acme: server: https://acme-staging-v02.api.letsencrypt.org/directory email: [[email protected]](https://blog.jetstack.io/cdn-cgi/l/email-protection) privateKeySecretRef: name: letsencrypt-private-key http01: {}
```

为了向这些发行者请求证书，我们还引入了证书资源。这些资源引用相应的发行者资源，以指示它们应该由哪个 CA 发行。

下面是一个证书资源的示例，使用前面显示的颁发者:

```
apiVersion: certmanager.k8s.io/v1alpha1 kind: Certificate metadata: name: example-com spec: secretName: example-com-tls dnsNames: - example.com - www.example.com issuerRef: name: letsencrypt-staging kind: ClusterIssuer acme: config: - http01: ingressClass: nginx domains: - example.com - [www.example.com](http://www.example.com)
```

关于发行人和证书的更多信息可以在我们的[文档](https://cert-manager.readthedocs.io/)中找到。

# 0.3 中的新功能

在过去的几周里，我们一直在测试 v0.3 的 alpha 候选版本，这是我们即将发布的新版本。

这个版本包含了新的特性和错误修复，本节描述了主要的特性。

# ACMEv2 和通配符证书

让我们加密最近发布的 ACME 协议 v2，除了其他改进之外，它现在支持通配符证书。这是一个期待已久并被要求的特性，直到最近才成为可能。

为了允许 cert-manager 用户请求和使用通配符证书，我们在 v0.3 版本中专门使用了 ACMEv2。

这允许用户像请求任何其他证书一样请求通配符证书——包括通过带注释的入口资源完全支持这样做(就像 kube-lego！).

这是一个很好的例子，说明了我们如何向 cert-manager 添加新的和更复杂的功能，同时继续支持从 kube-lego 采用的长期遗留行为。

如果您想开始尝试，请在我们的 GitHub 页面上查看最新的 v0.3 alpha 版本。通过将`*.domain.com`指定为您的 DNS 名称之一，您可以像请求任何其他证书一样请求通配符证书。参见 [ACME DNS 验证教程](http://docs.cert-manager.io/en/master/tutorials/acme/dns-validation.html)了解更多关于如何开始 DNS-01 挑战的信息。

# 保管库颁发者支持

经过大量的艰苦工作，对 [Hashicorp 金库](https://vaultproject.io/)的初始支持已经合并到 cert-manager 中！

这是另一个长期要求的特性，也是我们支持的发行者类型的一大补充。

特别感谢 [@vdesjardins](https://github.com/vdesjardins) ，他独自自发地承担了这项工作，并推动其通过审查流程，直到最后。

这允许最终用户创建一个“保险库”颁发者，它与单个保险库 PKI 后端角色配对。

最初，我们支持令牌认证和 Vault 的“AppRole”认证机制，以针对 Vault 服务器安全地授权 cert-manager。

Vault Issuer 的加入巩固了我们作为通用证书管理器的地位，并展示了在众多不同类型的 ca 之上提供单一清晰抽象的价值。

在文档中阅读更多关于[创建和使用保管库发行者](http://docs.cert-manager.io/en/latest/tutorials/index.html)的信息。

# 新文档网站

在任何项目中，文档化都是困难的。它需要为全新用户提供清晰的入职体验，同时为更高级的用户提供深入的细节。同时考虑到这些用户不同的技能水平(例如，有些用户可能对 Kubernetes 本身来说是全新的！).

我们有一些出色的社区贡献，包括用户指南和教程，解释如何开始使用 cert-manager。

到目前为止，我们一直使用存储在 GitHub 中的 markdown 来托管我们的文档。当我们开始更严格地对 cert-manager 进行版本控制，并发布 alpha/beta 候选版本时，这开始引起混乱。为了解决这个问题，也为了让用户更容易浏览和发现我们*所做的*的各种教程——我们已经将我们的文档转移到一个托管的 readthedocs 网站上。

你可以在 [readthedocs](https://cert-manager.readthedocs.io/) 上查看新文档——注意，如果你在寻找最新 0.3 版本的信息，我们在左下角也有一个“版本切换器”。

文档的每一页右上角都有一个“在 GitHub 中编辑”的链接，所以如果你发现了一个错误或者你有了一个新教程的想法，请进入并提交 PRs！

# 下一步是什么？

Cert-manager 还年轻，未来有很多计划！以下是我们路线图中的几个亮点:

# 定义证书策略

现在，任何能够创建证书资源的用户都可以从任何 ClusterIssuer 或同一名称空间内的颁发者请求任何种类的证书。

我们打算为管理员提供机制来定义谁可以获得证书的“策略”,和/或这些证书必须如何构造。这可能包括最短和最长持续时间，或者允许的 DNS 名称的有限集合。

通过在 Kubernetes 内部定义该策略，我们可以从组织内所有不同 ca 之间的通用策略控制级别中获益。这将帮助您的组织审计和管理谁可以做什么。

# 高级资源验证

Kubernetes 最近增加了对 ValidatingAdmissionWebhooks 的支持(以及它们的‘变异’对应物)。

这些可用于提供资源验证(例如，确保所有字段都设置为可接受的值)，以及资源的高级“变异”(例如，应用“默认值”)。

配置这些 webhooks 时的一个常见问题是，它们需要 x509 证书才能设置和运行。这个过程可能很麻烦，这正是 cert-manager 设计要解决的问题！

在 cert-manager 的未来版本中，我们将引入*我们自己的*验证 webhook，以便为开发者提供无效配置的预警。这将涉及一个新颖的“引导”过程，以便允许“自托管 webhooks”(即由 cert-manager 托管的为 cert-manager 供电的 webhooks)。

除此之外，我们将创建教程，解释我们对这些类型的 webhooks 的“推荐部署实践”,以及您如何利用 cert-manager 来处理保护它们的所有方面！

# 可插拔/进程外发行者

cert-manager 的一些企业用户有他们自己的 CA 流程，这是为他们的组织定制的新颖流程。

考虑到如此多的不同业务单位依赖 x509 证书平台，在短时间内将整个组织切换到新协议并不总是可行的。

为了缓解这些公司的过渡期，我们正在探索增加“基于 gRPC 的发行人”。这类似于 CNI(容器网络接口)和 CRI(容器运行时接口)。目标是提供一个通用的 gRPC 接口，任何人都可以实现它来签署和更新证书。

实现该接口的用户将立即受益于 cert-manager 已经实现的附加策略和续订功能。

# 参与其中

Cert-manager 发展迅速，围绕它的社区每天都在壮大。

我们**希望**看到新成员加入社区，并且相信如果我们想要项目继续下去，这是必要的。

如果你想参与进来，看看我们在 GitHub 上的发布版，或者在 [Kubernetes Slack](https://slack.k8s.io/) 上进入#cert-manager 并打个招呼！

想在社区项目中工作，比如日常工作中的 cert-manager？Jetstack 正在招聘软件工程师，包括远程(EU)职位。查看我们的[职业页面](https://www.jetstack.io/careers)了解更多关于职位和如何申请的信息。

*原载于*[*blog . jet stack . io*](https://blog.jetstack.io/blog/cert-manager)*作者*

![](img/e796741afed91a915871f29e8078d952.png)

James Munnelly，Jetstack 解决方案工程师

詹姆斯做软件工程师已经有 10 年了。他于 2015 年初加入 Kubernetes，在公共云和裸机环境中拥有近 2 年的生产经验。当不在 Kubernetes 上工作的时候，他就修补 Golang，并在其他开源项目上进行合作。