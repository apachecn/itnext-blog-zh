# AWS IAM Identity Center 访问令牌以明文形式存储，这不是一个严重的漏洞

> 原文：<https://itnext.io/aws-iam-identity-center-access-tokens-are-stored-in-clear-text-and-no-thats-not-a-critical-68a48c1e398?source=collection_archive---------0----------------------->

每个星期，几乎毫无例外，我都会遇到一件让我困惑、娱乐或者最常见的激怒我的事情。我决定记录我的冒险经历。

上周，我谈到了一种特殊类型的网络钓鱼，[# AWS](https://www.linkedin.com/feed/hashtag/aws)[# IAM](https://www.linkedin.com/feed/hashtag/iam)Identity Center 的用户很容易受到这种网络钓鱼的攻击。今天，我们将再次谈论身份中心，但这一次是关于访问令牌的使用和一般访问令牌*(作者注:实际上，我们将在下周讨论这个问题)。*

**背景**

在过去的一周里，我读了[一篇文章](https://www.bleepingcomputer.com/news/security/microsoft-teams-stores-auth-tokens-as-cleartext-in-windows-linux-macs/)，不知何故它甚至出现在了 [slashdot](https://it.slashdot.org/story/22/09/14/1939242/microsoft-teams-stores-auth-tokens-as-cleartext-in-windows-linux-macs) 上，其特色是[访问令牌](https://www.rfc-editor.org/rfc/rfc6749#section-1.4)。这篇文章惊呼，微软团队的访问令牌可以明文读取！这个漏洞被 [#Vectra](https://www.linkedin.com/feed/hashtag/vectra) 披露为“严重的”，真正展示了当人们将每个发现都公布为“世界末日”时会发生什么。我很惊讶他们没有给它起个名字。

尽管这份报告具有耸人听闻的性质，但我有令人震惊的消息要与大家分享，你们的一些开发人员在他们的设备上明文存储了更敏感的秘密。最明显的例子就是 AWS secrets。长期 AWS 秘密访问密钥通常存储在 *~/中。AWS/配置*。许多公司可能认为他们是安全的，因为他们正在使用 SSO，但典型的工具，如 [aws-okta](https://github.com/segmentio/aws-okta) 或 [#Nike](https://www.linkedin.com/feed/hashtag/nike) 的 [gimme](https://github.com/Nike-Inc/gimme-aws-creds) 不仅不安全地处理 SAML 断言，而且经常将 okta 用户名和密码持久化到设备上(这要糟糕得多)。

**问题**

如果你一直关注我的帖子，你会知道 AWS IAM Identity Center(以前的 AWS-SSO)通过允许为 CLI 访问集成 [#OIDC](https://www.linkedin.com/feed/hashtag/oidc) 解决了对这种 hacky 解决方案的需求。在前一篇博客中，我讨论了 AWS CLI(或任何其他应用程序/用户)如何获取 AWS OIDC 访问令牌。我没有谈到的是这些是如何被亚马逊坚持下来的。**修卡，访问令牌以纯文本的形式保存到 *~/。AWS/SSO/cache/{ sha1 _ of _ start _ URL }。json* 。该死的，他们甚至还把起始网址和地区包括在内。**

问题是，这是个问题吗？我们在 [#vectra](https://www.linkedin.com/feed/hashtag/vectra) 的朋友会说是的，而且可能是一个超级挑剔的人。然而，对于大多数供应商来说，这种类型的问题通常被视为威胁模型之外的问题(正如他们所发现的)。原因是操作系统对用户(及其会话)的隔离做了某些假设。因此，如果我们有多个用户在一个共享帐户上使用团队或 AWS-CLI，这确实违反了那些假设(未言明或其他)。

同样，如果攻击者危害主机，到处都是访问令牌可能不是最大的问题。不管它们是如何存储的，攻击者都可以，例如，安装内核模块、注入进程空间、改变库加载顺序、读取内存或安装代理，等等——虽然操作系统在某些情况下可能会使这变得很难，但我不认为 macOS(或任何其他操作系统)会让正常的安全从业者对未经授权的访问感到舒服。见鬼，当我在 RIT 教书的时候，我曾经有一个任务，学生的任务是创建一个完全锁定的 SELinux 环境，并允许其他学生访问，这经常导致拒绝服务，偶尔导致完全妥协。

**解决方案**

因此，我们应该做些什么来保护访问令牌，因为它们代表了一个安全敏感的对象。T2:嗯，从某种意义上说，Vectra 是对的，如果应用程序利用操作系统级别的控制，就像 macOS 中的钥匙链一样，这将是最好的 T4。虽然这些不会阻止被入侵的主机泄露信息，但它可能会使攻击者更难——这是我们都赞成的权衡——但肯定不是“关键的”。

同样，RFC 也为访问令牌安全指定了一些便利的好东西。最重要的建议是“保护无记名令牌”，这显然可以包括您的本地设备，但随后的“更具体”建议都围绕使用 TLS，并且出于良好的原因，用户工作站之外通常是一个不太友好的地方。

冒着听起来像经济学家的风险，在某些时候需要做出假设。如果这些假设证明不是真的，调整你的模型，再试一次。这个想法被融入到威胁建模中。假设所有机器的持续完全损害可能会导致用户无法合理工作的控制，相反，大多数组织(和操作系统开发人员)试图防止机器的损害，并在损害发生时检测损害，以便可以遏制损害。

包含折衷可能包括撤销访问令牌，这涉及 RFC 中的另外两个建议:“颁发短期承载令牌”和“颁发限定范围的承载令牌”。两者都是 *SHOULD* 指令，但是用来限制泄露的密钥(*作者注意:下周我们将看到 AWS 没有遵循这些建议，这导致了一些令人困惑的会话生存期*)。这个想法相当简单，如果经常需要重新认证，至少在一个 [#zerotrust](https://www.linkedin.com/feed/hashtag/zerotrust) 的世界中，用户终端的安全性将在重新发布令牌之前被重新验证——从而导致可接受的风险。