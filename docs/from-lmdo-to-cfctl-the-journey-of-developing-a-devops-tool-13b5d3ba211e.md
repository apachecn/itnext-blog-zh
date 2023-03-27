# 从 lmdo 到 cf CTL——开发 DevOps 工具的旅程

> 原文：<https://itnext.io/from-lmdo-to-cfctl-the-journey-of-developing-a-devops-tool-13b5d3ba211e?source=collection_archive---------4----------------------->

![](img/7c8fb2404658dbbe457c68499a68cbf7.png)

迈克尔·克里斯滕森在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# 序言

2016 年，我在一家初创公司工作，我是这家公司的技术创始人。作为一个小型技术团队的领导者和多面手，我必须管理的技术和工具数量多得令人应接不暇。当时，我们工具链中的大多数技术都非常前沿。只是为了描述一下，我们对 SPA 和移动应用程序使用 react/react native；后端是微服务，完全建立在使用 AWS Lambda 的无服务器基础设施上，外加一大堆其他服务，如 API gateway、SQS、SNS、RDS、Cognito 和 CloudFormation。环顾四周，没有人设置与我们相似的堆栈。

在技术采用生命周期[1]中作为早期采用者的一个不利之处是，你不会找到一个好的生态系统来立即为技术服务。我们发现的大多数开源工具要么非常固执己见，要么专注于狭隘的用例。如果我们采用这些工具，我们将会有很多变通方法，同时还必须熟悉每一个只部分适合我们使用的方法。让新员工加入团队变得更具挑战性。

在这种情况下，我花了一周时间用 Python 写了第一个版本的 lmdo，因为这是团队的语言选择。它通过集中工具链解决了团队的紧迫问题，通过将 CloudFormation、Lambda 函数和 API gateway 等服务打包在一起，提供了一个管理微服务堆栈的简单点。结合 CICD 自动化和 scrum，团队的 SDLC(软件开发生命周期)是流畅和敏捷的。团队中的任何开发人员，无论他/她是前端还是后端开发人员，都可以轻松地构建他们自己的测试堆栈。对于新团队成员来说，只需要掌握一种工具。

由于我们已经完全控制了工具链，在此期间，基于团队的需求，更多的功能被快速地添加到了 [lmdo](https://github.com/liangrog/lmdo) 中。一年多一点以后，AWS 终于发布了他们的第一个 SAM(无服务器应用模型)版本，虽然支持更多的语言，但是有着非常相似的思想和功能。

很快，技术领域发生了翻天覆地的变化。许多正处于成熟阶段。有大量的工具可供选择。许多工具基本上都是抽象的。通过在顶层引入另一层，最终用户将能够仅用元级别的知识来执行和期望结果。尽管如此，它也有副作用，如失去技术技能的细节水平。以 [Terraform](https://www.terraform.io/) 为例，它很容易使用许多奇妙的功能，支持多云(有警告)。然而，它使用自己的模板系统。我怀疑它的许多用户在没有从头开始学习的情况下，知道如何编写一个等价的 CloudFormations 或 Google Cloud Deployment Manager。所有新的 AWS 服务或功能都自然地提供给它自己的 CloudFormation，但人们可能必须等待 Terraform 版本更新才能使用它。

不久前，我受命创建并管理一些 AWS 堆栈。考虑到堆栈的大小和相对简单的工作，我自然想使用 [lmdo](https://github.com/liangrog/lmdo) 。但是经过几次反复之后，我意识到它并不是 job 的最佳工具。一是我只需要使用它的 CloudFormation 特性，其他特性都不在范围内。其次 [lmdo](https://github.com/liangrog/lmdo) 的模板管理不适合我们当前的 DevOps 流程。似乎需要一种新工具。我提出了这些要求:

*   逆向供应商无关性:人们可以很容易地去除对工具的依赖性，并返回使用 awscli 或控制台来创建相同的堆栈。
*   没有持久状态:应该实时查询堆栈状态，而不是依赖于磁盘上的状态管理。
*   依赖嵌套堆栈自动上传。
*   堆栈输出自动注入到其他堆栈。
*   加密工具，便于对机密进行加密，以便将其存储在存储库中。
*   超越常规的配置:提供最大的灵活性。
*   管理 CloudFormation 堆栈生命周期。
*   参数文件的 YAML 格式。

经过大量的研究，很明显我需要自己写下来。这些年来，Go 已经成为我编写命令行工具的首选语言，因为我有多年使用它的经验，而且它提供了很多好处，比如多平台兼容性、作为一个二进制文件轻松分发。这个工具自然会用 Go 来写。

你可以在 [github](https://github.com/liangrog/cfctl) 中找到很多代码和版本。我将在这里阐述模板文件夹结构设计。

在 [cfctl](https://github.com/liangrog/cfctl) 中，需要配置三个文件夹(默认`stacks.yaml`)，这三个文件夹下的任何内容都完全由用户决定，只要在引用时给出相对路径，就有最大的灵活性:

```
|- templates **(1)**
   |--- ...
|- project
   |--- env-values **(3)**
        |--- default
        |--- aws-acc-1
        |--- aws-acc-2
   |--- parameters **(2)** |--- ...
   |--- stacks.yaml
```

1.  模板文件夹:

根据经验，为了重用和隔离关注点:

*   用参数化的值编写一般的云信息
*   将大的堆栈分成较小的或嵌套的堆栈

遵循这些原则，自然会有通用模板和由多个通用模板组成的特定用途模板。

2.模板参数文件夹:

该文件夹包含堆栈模板的所有参数文件。参数文件是以 YAML 格式编写的，但是如果需要，可以用一个命令将其转换回 JSON。 [cfctl](https://github.com/liangrog/cfctl) 使用 Go 模板为参数值提供了一个简单的模板系统。它允许您使用环境文件夹中的值，并有一些有用的帮助函数，如从堆栈输出或系统环境变量中获取值。

3.特定于环境的参数值文件夹:

该文件夹包含参数文件中使用的所有值。这些值可以引用其他值。这些值也可以由用户加密，并由 [cfctl](https://github.com/liangrog/cfctl) 使用 ansible-vault 动态解密。组织该文件夹的典型方式是为每个 AWS 帐户创建子文件夹。唯一的约定是，如果您有一个名为“default”的子文件夹，那么无论在部署期间设置了什么目标环境，每次都会加载其中的所有值。

这里有两个额外的 go 模块，我也是为了方便 [cfctl](https://github.com/liangrog/cfctl) 而编写的。一个是图形数据结构([你可以在这里](https://github.com/liangrog/ds)查看)因为 Go 没有内置的图形。基本上，这个图形模块允许你建立一个图形，并把它存储在一个线程安全的存储器中。Khan 排序函数允许你对树进行排序并检测循环依赖。它提供接口以便你能容易地扩展模块，例如，使用你自己的存储，来存储你自己的顶点上的数据。

另一个模块是[金库](https://github.com/liangrog/vault)模块。它根据 ansible-vault 的规范[2]提供文件加密功能。在它的帮助下，你可以将秘密和模板代码一起安全地存储。 [cfctl](https://github.com/liangrog/cfctl) 将在运行过程中即时解密这些文件，而无需事先手动解密。

有趣的是，在 cfctl 的开发过程中，AWS 发布了他们的 [CloudFormer](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-cloudformer.html) 的测试版，该版本也针对许多 cfctl 试图解决的类似问题。然而，随着 [CDK](https://docs.aws.amazon.com/cdk/latest/guide/home.html) 的引入，这里的用例略有不同

[cfctl](https://github.com/liangrog/cfctl) 并不完美。还有很多地方可以改进。如果你感兴趣，请随时提供反馈。这些版本可以通过 [github](https://github.com/liangrog/cfctl/releases) 下载。

参考:

1.  [技术采用生命周期](https://en.wikipedia.org/wiki/Technology_adoption_life_cycle)
2.  [可旋转拱顶](https://docs.ansible.com/ansible/latest/user_guide/vault.html#vault-payload-format-1-1)

*免责声明:本文仅代表作者个人观点，绝不代表任何组织的观点。*