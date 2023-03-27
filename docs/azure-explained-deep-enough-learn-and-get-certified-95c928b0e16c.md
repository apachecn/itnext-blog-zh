# Azure 解释得够深刻:学习并获得认证

> 原文：<https://itnext.io/azure-explained-deep-enough-learn-and-get-certified-95c928b0e16c?source=collection_archive---------0----------------------->

![](img/b2875287d77e54703e5f4e8efbdfc229.png)

图片由来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=2001090) 的 [Wynn Pointaux](https://pixabay.com/users/wynpnt-868761/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=2001090) 提供

## 基础知识和虚拟机

## 介绍

这是 Azure 迷你系列的第一部 **Azure 解释得够深刻了**

第 1 部分:Azure 解释得足够深入:学习并获得认证

[第 2 部分:Azure 解释得足够深入:容器](https://medium.com/codex/azure-explained-deep-enough-containers-a516af1caab1)

[Part3: Azure 解释得够深刻:Azure PaaS](https://piotrzan.medium.com/azure-explained-deep-enough-azure-paas-321a0f16bd57)

[Part4: Azure 解释够深:Azure DevOps](https://piotrzan.medium.com/azure-explained-deep-enough-azure-devops-210629b5480e)

## 介绍

这是博客系列的第一部分，面向开发人员、有技术头脑的产品所有者以及其他希望扩展 Azure 知识的敏捷软件开发角色。我们将探索 Azure，一步一步地学习。每个部分将展示一个特定的 Azure 主题，提供优秀学习资源的链接，并带你更接近理解和掌握云计算概念。最终目标是特别熟悉云和 Azure，给你实用的平衡的知识，你可以用它来提高你的技能，推进你的职业生涯，或者只是享受学习新东西的乐趣。

## 学习材料

除了博客，每个主题在 https://github.com/ilearnazuretoday 的[组织中都有一个配套的 GitHub 知识库。例如，您可以在](https://github.com/ilearnazuretoday) [create-vm](https://github.com/ilearnazuretoday/create-vm) 存储库中找到这个博客的源代码和附加脚本。

## 证书

云证书可以显著增加你获得高薪工作的机会，可以提高你作为技能衡量标准的可信度。现在在家办公直接参加考试是极其容易的！还有，微软最近宣布[云证书可以免费续费](https://docs.microsoft.com/en-us/learn/certifications/renew-your-microsoft-certification)，这真是个好消息。如果你是 Azure 的新手，从 [AZ-900](https://docs.microsoft.com/en-us/learn/certifications/exams/az-900) 开始。获得这个证书证明你对云计算有扎实的基础知识，特别是 Azure。即使对于非开发人员来说，这也是一个很好的开始。

如果你是一名开发人员，你可以瞄准 [AZ-204:为微软 Azure 开发解决方案](https://docs.microsoft.com/en-us/learn/certifications/exams/az-204)，这将为你在 Azure 上开发软件打下坚实的基础。这是一次具有挑战性的考试，但值得。

最后，还有更高级的认证，像[微软认证:Azure 解决方案架构师专家](https://docs.microsoft.com/en-us/learn/certifications/azure-solutions-architect/)，[微软认证:DevOps 工程师专家](https://docs.microsoft.com/en-us/learn/certifications/devops-engineer/)。

> 我已经写了关于获得 Azure 认证的博客，所以你可以查看这里的[和这里的](https://piotrzan.medium.com/8-tips-to-prepare-for-az-300-exam-cadff5532394)和。

## 资源

有很多真正有用的资源，帮助你学习 Azure 和云技术，并变得更好。以下是我最喜欢的 Azure 学习资源列表:

还有一些:)

让我们换个方式，用更实际的方式探索 Azure。我们将创建一个虚拟机，并在其上上传一个网页。

一个小小的免责声明，我们在这里使用 Azure 资源的方式纯粹是说明性的，在真实的场景中，有更好的方式来运行网页。我们将在以后的博客中探讨其中的一些。

## 在 Azure 中创建虚拟机

在本练习中，您将学习如何:

*   创建 Azure 虚拟机
*   公开虚拟机上的端口
*   运行一个示例 web 应用程序
*   从互联网访问 web 应用程序
*   删除虚拟机和附带的资源

## 先决条件

要继续练习，需要满足以下先决条件:

*   Azure 订阅。很容易[创建一个 12 个月的免费服务](https://azure.microsoft.com/en-us/free/)
*   bash 的基本理解

这个练习是自定进度的，你将在 Azure 上创建资源，请记住按照说明删除资源，以避免意外费用。

继续这个练习最简单的方法是在创建一个 Azure 账户后登录到你的 Azure 账户，并在 [Azure 云外壳](https://docs.microsoft.com/en-us/azure/cloud-shell/overview)中克隆这个库。在练习中，我们将把命令复制并粘贴到 Azure cloud shell 中。

> *提示我们现在不打算这么做，但是您也可以使用* `*az interactive*` *模式来完成命令*

## 自动化

出于学习目的，请一步一步遵循以下说明，但如果您想以包括 web 应用程序在内的全自动方式创建虚拟机，请使用 [create-vm-script](https://github.com/ilearnazuretoday/create-vm/blob/main/create-vm.sh) 。

> *你可以通过运行* `*az account list-locations -o table*` *轻松看到所有可用的 Azure 位置。使用第二列*名称*为脚本提供位置。*

## 设置变量

我们将用于创建和管理虚拟机的设置变量。通过检查回显输出来确保它们是正确创建的。Azure 中的资源组和虚拟机名称在您的订阅中必须是唯一的，此外，虚拟机 DNS 标签必须是全局唯一的。请确保选择符合上述要求的名称。

```
group='your group name'
vm='your vm name'echo $group, $vm
```

## 创建 Linux ubuntu 虚拟机

这里我们用 Ubuntu 映像创建了一个简单的 Linux VM。Azure 将为我们创建一个虚拟机用户，我们称之为 *azureuser* ，并允许使用现有的 SSH 密钥“sshing”到虚拟机

> 在虚拟机上保留一个打开的 SSH 端口被认为是不安全的，应该在生产场景中避免。在这里，我们这样做只是为了演示。

```
az vm create \
  --resource-group $group \
  --name $vm \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

看看 Azure 门户网站，你的虚拟机创建好了吗？在搜索框中键入您的资源组名称。

## 为 web 流量打开端口 8000

为了允许外部流量进入我们的虚拟机，我们需要在 Azure 中打开一个端口，这是通过在虚拟机中打开一个外部端口来完成的。

```
az vm open-port --port 8000 --resource-group $group --name $vmip=$(az vm list-ip-addresses -g $group -n $vm --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" -o tsv)echo 'Public is is:' $ip
```

## SSH 到机器

创建一个新的 shell 会话并 ssh 到机器中

ssh azureuser@IP

## 创建简单的 web 应用程序

在这里，我们创建了一个极简主义者的 HTML 页面，并启动了一个简单的 python 服务器，通过开放的 8000 端口为我们的 HTML 提供服务。

> *Python3 http.server 默认在 8000 端口服务流量。此外，默认情况下，服务器将从启动服务器的根位置提供 index.html 内容。*

```
mkdir webapp
cd webapp
echo "Hallo this is simple web app" > index.html
python3 -m http.server
```

## 从浏览器或 wget 访问网站

切换回之前的会话，并通过 wget 或浏览器调用网页。

> *如果您忘记了虚拟机的 ip 地址，请使用* `*echo $ip*`

## 浏览器

IP 地址:8000

## Wget

`wget -O- [http://$ip:8000](/$ip)`

> *看看服务器是如何通过 GET 动词*服务 http 页面的

## 清理资源

在从 Azure 门户手动删除 VM 之前，请浏览它。您也可以通过命令*az group delete-name resource group*将其删除

## 结论

云计算，有 Azure、AWS、GCP、Linode、Digital Ocean、阿里巴巴、IBM Cloud、甲骨文 Cloud 或任何其他云提供商，是开发和运营软件的新的现代方式。我们已经开始探索 Azure，创建(并删除)了一个虚拟机。Azure 并不困难，你很快就会对云计算有一个坚实的理解，甚至可能在这个过程中获得一些证书。

请继续关注下一部分，我们将探索 Azure PaaS 和容器。