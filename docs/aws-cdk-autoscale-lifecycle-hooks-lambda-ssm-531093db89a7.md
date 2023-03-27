# AWS CDK +自动缩放+生命周期挂钩+λ+SSM =💪

> 原文：<https://itnext.io/aws-cdk-autoscale-lifecycle-hooks-lambda-ssm-531093db89a7?source=collection_archive---------3----------------------->

![](img/a1ca44791af847ce26c4139c25e8d85b.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [Daniel Páscoa](https://unsplash.com/@dpascoa?utm_source=medium&utm_medium=referral) 拍照

Kubernetes 有时太重或太贵，无法解决所有问题，或者对于开发团队来说是一个太大的“提升和转移”解决方案。但是，当解决方案迁移到云并经常遇到工作负载增加时，由于成本或利用率的原因，解决方案仍然需要能够动态扩展。并非每个应用程序或解决方案都能自动或轻松地横向扩展。有些情况下，当解决方案的某个组件扩大或缩小时，需要采取措施；例如，如果 worker 组件向上或向下扩展，您可能需要通过配置更新让控制节点知道这些变化。

让我们来看看 Amazon Web Services 上的一种方法，在这种方法中，自动伸缩组(ASG)可以使用生命周期挂钩来触发实例启动或终止期间的操作。在此细分中，我们将利用 AWS 简单通知服务(SNS ),它将提供与 ASG 生命周期相关的消息 AWS Lambda，其中无服务器功能将监听 SNS 生命周期主题，以便对更改做出反应 AWS Systems Manager，它将用于调用目标端点上的 shell 命令，以便显示这些挂钩如何影响各种系统上的更改。一个重要的注意事项是，系统管理器代理(SMS)默认情况下只安装在 Amazon Linux 镜像上。您可能需要更新您的 Amazon 机器镜像，以包括[该项目的安装和设置](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-manual-agent-install.html)。

当我们浏览这些概念时，我们将看到的示例存储库利用了亚马逊云开发工具包(CDK)。CDK 允许您使用许多流行的代码语言中的一种，以便定义可用于可重复部署的云形成模板。这个基本示例设置了一个 VPC 和一个自动缩放组。

## 循序渐进

让我们来看一个基于 CDK 的示例项目，它将:
1。创建虚拟专用网络(VPC)

2.创建自动缩放组

3.创建社交网络话题

4.注册 Lambda 函数并订阅 SNS 主题

5.创建 Lambda 函数

以下示例 GitHub 存储库中包含了这五个不同的项目:

[](https://github.com/chambridge/cdk-autoscale-lambda-ssm-action) [## GitHub-cham bridge/CDK-auto scale-lambda-SSM-action:这个示例项目展示了一个 AWS CDK 部署…

### 该示例项目展示了一个自动缩放组的 AWS CDK 部署，该自动缩放组具有一个将触发 Lambda…

github.com](https://github.com/chambridge/cdk-autoscale-lambda-ssm-action) 

## 创造一个 VPC

有许多 VPC 基本设置的例子。我们将把注意力集中在`net.py`文件上。下面您可以看到一个简单的虚拟专用网络的创建。

## 创建自动缩放组

下一段代码`systems.py`提供了一个示例 autoscale 组，我们将使用它来跟踪生命周期的变化，以实现自动化。

## 创建社交网络话题

现在我们有了自动缩放组，我们可以创建 SNS 主题，并让它侦听来自自动缩放组的生命周期事件。

## 注册一个 Lambda 函数并订阅 SNS 主题

有了 SNS 主题，我们可以定义一个 Lambda 函数来监听自动缩放组生命周期事件发出的通知。

## Lambda 函数背后的代码

让我们也看看 Lambda 函数的代码是什么样子的。在这种情况下，我们将使用 Systems Manager API 进行一个简单的调用，将一个文件写入文件系统，但是根据您的具体使用情况，您可以进行许多更复杂的更改。

# 摘要

![](img/d8066ba852029560947b87913baf2e11.png)

由[卡尔·阿布伊德](https://unsplash.com/@spongzy?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

有了上面描述的方法，您就有了一套强大的工具来构建一个动态的、反应式的部署。您可以利用更多云原生功能的优势，减轻现有解决方案的重量，减少对现有解决方案的重新设计。我们已经通过基础设施即代码实现了所有这些步骤，利用了 CDK 的力量，并简单地在一些额外的 AWS 服务上进行分层，从而开放了一系列功能。