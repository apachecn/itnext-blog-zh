# 使用 Terraform 应用命令

> 原文：<https://itnext.io/using-the-terraform-apply-command-a2b8413f6e77?source=collection_archive---------3----------------------->

在这篇文章中，我将解释`terraform apply`命令的用途、作用、可用选项以及何时运行它。

![](img/524513204f55aa4716022bd84b4c1948.png)

Nubelson Fernandes 在 [Unsplash](https://unsplash.com/s/photos/code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

## 什么是地形应用？

`terraform apply`命令执行`terraform plan.`中提出的动作，它用于部署您的基础设施。通常`apply`应该在`terraform init`和`terraform plan`之后运行。

## terraform 应用命令有什么作用？

如果在没有任何选项的情况下运行`apply`命令，它将首先运行`terraform plan`，要求用户确认计划的行动，然后在确认后执行这些更改。

`apply`命令也可用于先前从`terraform plan -out=<path to file>.`生成的平面文件

## 快速使用示例

`terraform apply` —根据配置文件创建或更新基础结构。默认情况下，将首先生成一个计划，并需要在应用之前进行审批。

`terraform apply -auto-approve` —应用更改，无需对计划交互键入“是”。在自动化 CI/CD 管道中很有用。

`terraform apply <planfilename>` —提供使用`terraform plan -out`命令生成的文件。如果提供，Terraform 将在没有任何确认提示的情况下采取计划中的措施。

`terraform apply -var="domainpassword=password123"` —传入一个变量值。

`terraform apply -var-file="varfile.tfvars"` —传入包含在文件中的变量。

`terraform apply -target=”module.appgw.0"` —仅将更改应用于目标资源。

`terraform apply -lock=false` —在地形应用操作期间，不要持有状态锁。如果其他工程师可能对同一工作空间运行并发命令，请谨慎使用。

`terraform apply -parallelism=<n>` —指定并行运行的操作数量。

## 多次运行 terraform apply 是否安全？

如果与当前 Terraform 状态相比，配置文件中没有变化，则不会对基础设施进行任何更改。Terraform 是一种声明性语言，所以多次运行`apply`命令是绝对安全的。

## 在自动化中运行 terraform 应用程序

在自动化中`terraform apply`可以在计划阶段后运行，传入计划输出文件。如果在应用阶段之前没有计划阶段(不推荐)，那么可以使用`terraform apply -auto-approve`选项。在一些 CI/CD 系统(如 Azure DevOps)中运行管道时，避免编码问题的另一个有用选项是`terraform apply -no-color`选项，因为构建代理无法正确处理彩色输出。

## 摘要

`terraform apply`是 Terraform 工作流中的核心命令，用于部署配置文件中描述的基础架构。

有关`terraform apply`命令的更多文档，请访问 Hashicorp 网站:

[](https://www.terraform.io/cli/commands/apply) [## 命令:应用|哈希公司的 Terraform

### 搜索 Terraform 文档动手操作:尝试 HashiCorp Learn 上的 Terraform: Get Started 集合。更多信息…

www.terraform.io](https://www.terraform.io/cli/commands/apply) 

想要更多的 Terraform 内容？点击这里查看我在 Terraform 上的其他文章。

干杯！🍻

[](https://www.buymeacoffee.com/jackwesleyroper) [## Jack Roper 正在 Azure、Azure DevOps、Terraform、Kubernetes 和 Cloud tech 上写博客！

### 希望我的博客能帮到你，你会喜欢它的内容！我真的很喜欢写技术内容和分享…

www.buymeacoffee.com](https://www.buymeacoffee.com/jackwesleyroper) 

*最初发表于*[*space lift . io*](https://spacelift.io/)*。*