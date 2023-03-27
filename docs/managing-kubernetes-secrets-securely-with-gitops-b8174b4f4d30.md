# 使用 GitOps 安全管理 Kubernetes 机密

> 原文：<https://itnext.io/managing-kubernetes-secrets-securely-with-gitops-b8174b4f4d30?source=collection_archive---------1----------------------->

![](img/06bb91d94fb73c7b0fc2cb6fb2624638.png)

你可以用 GitOps 管理秘密，你应该这样做。使用 GitOps，您可以安全地完成它，并且可以大规模地完成它，而且它是自动化的！

GitOps 只是一个不断演变的流行语景观中的最新流行语。它的核心是以一种表示当前状态的声明性方式管理资源，以便随时了解 git 中的情况，并将声明性状态解析到集群。

人们在 GitOps 上犯的最大错误是结构。存储库的结构至关重要。你如何选择在你的公司组织 GitOps 将决定它的成败。

一旦你解决了结构问题，人们面临的下一个最大问题是如何保护秘密。一般来说，它最终会损害不需要访问权限的人的利益，或者通过 Slack 或 1Password 传播共享的秘密。

[密封的秘密](https://github.com/bitnami-labs/sealed-secrets)是 Kubernetes 上解决秘密的一种流行方法，这是一种体面的解决方案，它依赖于 PKI 和共享公钥来加密，并在群集上安装私钥，但归根结底，**还有更好的方法。**

# 保护秘密的更好方法

[Mozilla 的 SOPS](https://github.com/mozilla/sops) 。SOPS 支持关键材料的多种来源，包括 AWS KMS、GCE、Vault、PGP 等。它提供了使用来自 AWS KMS 的外部密钥材料来加密和解密秘密的能力，但它也支持 Shamir 的秘密共享和密钥组，本质上允许关于需要什么密钥和需要解密多少密钥的策略。

由于 SOPS 使我们能够使用像 AWS KMS 这样的服务，我们可以控制谁有权加密和解密，但这也意味着我们可以利用 AWS 的实例配置文件，让机器做它们被设计为最好的事情，通过自动化使我们的生活更加轻松。

我想提请注意的最后一个特征是，一旦文件被加密，关于它如何被加密的所有元数据都是文件本身的**部分**，不需要第三方服务、数据库、隐藏目录或二级文件来知道如何解密文件。

**最好的消息是，Flux 对 sop 有一流的支持。**

# 结构

在开始时，我说过适当地构建您的存储库是至关重要的。我不是自己意识到这一点的。感谢[史蒂文·韦德](https://medium.com/u/ce76492ff8b8?source=post_page-----b8174b4f4d30--------------------------------)与我合作并分享他一路走来学到的经验。

结构化有两个部分。存储库中内容的布局以及要使用的存储库数量。我个人用 3。一个用于定义集群运行所需的每个基础资源，一个用于处理工程团队负责的产品的所有发布，一个用于机密。我只使用专用秘密储存库的标准操作程序。

使用 Flux 和 SOPS 为我们提供了前所未有的灵活性。

## 存储形式

```
|-- .sops.yaml
|-- secrets
    |-- dev (environment/cluster)
        |-- database.yaml
    |-- stg (environment/cluster)
        |-- database.yaml
    |-- prd (environment/cluster)
        |-- database.yaml
```

这个文件布局给了我们使用 [SOPS 创建规则](https://github.com/mozilla/sops#id19)来控制文件如何加密的基础。我们可以使用不同的密钥、密钥组和阈值。它还允许我们告诉 Flux 在克隆和解析需要在群集上的内容时只关心特定的路径。这里有一个例子，但它不是一个工作。

```
creation_rules:
- path_regex: secrets/dev/.*
  encrypted_regex: "^(data|stringData)$"
  kms:
    arn: kms-arn
    role: ksm-role
- path_regex: secrets/stg/.*
  encrypted_regex: "^(data|stringData)$"
  kms:
    arn: kms-arn
    role: ksm-role
- path_regex: secrets/prd/.*
  encrypted_regex: "^(data|stringData)$"
  kms:
    arn: kms-arn
    role: ksm-role
```

# 缩放比例

使用这种设置，我们几乎可以无限扩展这种设置。它只受存储在存储库中的数据量和为其提供服务的 git 服务器的限制。秘密通常不会频繁改变，但它们确实会发生变化，这就是像 [Reloader](https://github.com/stakater/Reloader) 这样的额外工具的用武之地，但那是另一篇文章的内容。(感谢史蒂文·韦德

# 设置 sop

既然我们已经讨论了布局和设计的原则，我们可以把它付诸行动了。利用 sop 有多种方法，但我将只介绍其中一种。将 AWS KMS 与多个键和一个 AWS 实例概要一起使用，Flux 可以利用这个概要。

## 设置 AWS

在现实世界中，我会建议使用多个 AWS 帐户，并假设角色权限，但对于像这样的文章来说，这可能会变得不必要的复杂，所以我们将假设单个 AWS 帐户，但仍然使用 3 个密钥。

```
aws kms create-key
```

确保你收集了 ARNs 以备后用。

接下来，我们需要用创建规则创建我们的`.sops.yaml`文件。在现实世界中，您希望阈值为 2 或更多，并且您的键至少比阈值高 N+1。

让我们从创建一个基本目录开始，用下面的内容创建`.sops.yaml`文件，确保用有效的 ARNs 替换掉 ARNs。

```
creation_rules:
- path_regex: secrets/dev/.*
  encrypted_regex: "^(data|stringData)$"
  shamir_threshold: 2
  key_groups:
    - kms:
        - arn: arn:aws:kms:us-west-2:000000000000:key/b5d44bf0-7ec5-49a9-b404-bc4d8b4036fb
    - kms:
        - arn: arn:aws:kms:us-west-2:000000000000:key/16d44186-2393-40d9-90e1-9a2f92fd5863
    - kms:
        - arn: arn:aws:kms:us-west-2:000000000000:key/2120d2c1-a89e-4aeb-844f-f17ae2abd210
```

这个创建规则声明所有在`secrets/dev`目录中的文件都将使用 3 个不同的密钥加密，解密阈值为 2。

接下来创建您的目录`secrets/dev`并创建一个包含以下内容的`example.yaml`文件

```
apiVersion: v1
kind: Secret
metadata:
  name: example
type: Opaque
data:
  username: bXktdXNlcm5hbWU=
  password: bXktcGFzc3dvcmQ=
```

要加密此文件，请运行以下命令

```
sops -e -i secrets/dev/example.yaml
```

`-e`用于加密，`-i`用于文件的就地修改。如果您想保持文件不变并看到加密的输出，只需省略`-i`参数。

现在您可以将这个设置提交到您的存储库中。

# AWS 实例配置文件

这个系统的好处在于 Flux 可以简单地利用 AWS 实例概要文件，通过 STS 来承担角色，并获得短期密钥，以便能够在集群本身上运行时解密秘密。

要做到这一点，您需要设置一个角色，该角色根据您使用的密钥类型拥有加密/解密和可能的生成权限，并设置您的 [AWS 实例来使用概要文件](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)。

完成此设置后，只需使用正确的 CLI 选项将 Flux 安装到集群中，然后高枕无忧，让 Flux 保持集群最新状态。

# 设置流量

只有几件事你需要做，以确保通量运行正常。您需要启用 sop，这取决于您的部署方法，可能是配置选项或 CLI 标志`--sops`。您还需要指示您的 Flux 实例只关心默认分支中的特定路径。`--git-path`需要被设置为 secrets 下的一个目录，比如`--git-path=secrets/dev`

# 结论

使用这种方法可以让您更好地控制密钥材料和谁可以访问。它允许您允许 AWS 处理提供密钥的困难部分，以通过自动化和短期秘密来访问要解密的密钥。您可以将所有秘密保存在一个地方，以便于管理和轮换。