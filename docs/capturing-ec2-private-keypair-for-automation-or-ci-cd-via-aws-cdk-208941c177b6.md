# 通过 AWS CDK 为自动化或 CI/CD 捕获 EC2 私有密钥对

> 原文：<https://itnext.io/capturing-ec2-private-keypair-for-automation-or-ci-cd-via-aws-cdk-208941c177b6?source=collection_archive---------2----------------------->

![](img/a23020bebbc5fbeab3ccdfe828a1c22e.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [olieman.eth](https://unsplash.com/@moneyphotos?utm_source=medium&utm_medium=referral) 拍摄的照片

您是否曾经遇到过这样的情况:您需要通过 SSH 访问一组 EC2 实例，但是您没有访问权限，因为原始所有者已经离开，并且相关的私有密钥不可用(因为初始设置后私有密钥不可访问)？或者，您可能想使用像 [Ansible](https://www.ansible.com/) 这样的工具，从一个专用的网络访问中对一组 EC2 实例运行自动化。也许您正在设置一个 CI/CD 管道，并且需要建立主节点和工作节点之间的通信。本文将使用基础设施作为代码，为这种设置介绍一种可重复的机制。

## AWS CDK

第二版[亚马逊网络服务云开发套件(CDK)](https://aws.amazon.com/cdk/) 于去年(2021 年)上市。CDK 允许开发者使用一些最流行的编码语言来开发[云形成模板](https://aws.amazon.com/cloudformation/resources/templates/)；AWS 的基础设施作为代码规范。编写云形成模板(CFT)可能是一项繁重的任务，但通过 CDK，开发人员可以使用 API 来生成、部署和销毁 CFT。本演练中显示的示例是用 Python 编写的，但可以转换为您选择的受支持语言。

![](img/5aaad47e04dd5fff93b18b412d928747.png)

由[absolute vision](https://unsplash.com/@freegraphictoday?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

## 循序渐进

让我们看一个基于 CDK 的项目的例子，它将:
1。创建虚拟专用网络(VPC)

2.生成用于访问 EC2 实例的密钥对，该 EC2 实例将通过 [AWS Lambda 函数](https://aws.amazon.com/lambda/)创建

3.将私钥存储在 [AWS 秘密管理器](https://aws.amazon.com/secrets-manager/)中

4.创建一个可以使用生成的密钥对访问的自动缩放组

5.创建一个 manager EC2 实例来捕获系统上的私钥，以便可以使用它来访问自动缩放组中的实例。

以下示例 GitHub 存储库中包含了这五个不同的项目:

[](https://github.com/chambridge/cdk-keypair-private-key-secrets-mgr) [## GitHub-cham bridge/CDK-key pair-private-key-secrets-mgr:原型显示了通过…

### 原型展示了通过 AWS CDK 创建具有私有密钥对访问的管理实例的云结构…

github.com](https://github.com/chambridge/cdk-keypair-private-key-secrets-mgr) 

## 创造一个 VPC

有许多 VPC 基本设置的例子。我们将把注意力集中在`net.py`文件上。下面您可以看到一个简单的虚拟专用网络的创建。

## 创建 Lambda 自定义资源

现在我们有了一个网络设置，我们可以创建将在 keypair 管理器可以访问的实例中使用的 keypair。下面的代码也在`net.py`文件中，创建了:

*   一个 IAM 角色，允许 Lambda 函数在 AWS Secrets Manager 中运行和存储私钥结果
*   安全小组
*   λ函数定义
*   自定义资源，使该功能能够在云形成部署的流程中被触发

## 密钥对生成和存储

现在让我们看看实际的 Lambda 函数，`ec2_keypair_init.py`文件，它将使用 Boto3 API 创建密钥对，并将生成的私钥存储在 AWS Secrets Manager 中。

## 创建密钥对管理的实例

创建了密钥对并将私钥存储在 AWS Secrets Manager 中后，我们可以继续使用密钥对部署 EC2 实例。下一段代码`systems.py`提供了一个示例自动缩放组，它利用了 Lambda 函数创建的 keypair。

## 设置密钥对管理器实例

接下来，我们将设置 keypair 管理器实例`manager.py`，并查看引导脚本。

这个管理器实例可以设置任何访问密钥对，并且它可以访问受管实例的私钥。管理器可以做到这一点，因为它有一个 IAM 角色，允许它从 AWS Secrets Manager 中读取，并且它有私钥的秘密名称。秘密名称被传递给引导文件，引导文件使用 AWS CLI 创建 PEM 文件，如下面的引导脚本`bootstrap_manager_node.sh`所示。

## 摘要

![](img/013254fb65fc26fc17cc64d07107a08a.png)

照片由[金伯利农民](https://unsplash.com/@kimberlyfarmer?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

希望这个故事已经启发了您 EC2 实例的密钥对管理方法。我们通过 CDK 和 Boto3 交互来创建 Lambda 函数，并创建和存储密钥对。我们查看了管理器的创建流程，以及它如何通过引导脚本获得私钥。所有这些步骤中最好的是在可重复的基础设施即代码中捕获的。