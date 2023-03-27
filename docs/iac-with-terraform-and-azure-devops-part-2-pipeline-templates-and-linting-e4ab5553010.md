# 使用 Terraform 和 Azure DevOps 的 IaC，第 2 部分:管道模板和林挺

> 原文：<https://itnext.io/iac-with-terraform-and-azure-devops-part-2-pipeline-templates-and-linting-e4ab5553010?source=collection_archive---------5----------------------->

*这是关于 Terraform、Azure DevOps 和用代码管理基础设施的系列文章的第 2 部分。参见第一部分* [*此处*](/infrastructure-as-code-iac-with-terraform-azure-devops-f8cd022a3341) *。*

![](img/f3796fe95d11b46e343c2cbb25f36dd9.png)

照片由[克里斯蒂娜·加夫里拉](https://unsplash.com/@cristinagavrila?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

在本系列的上一篇文章中，我们介绍了在 Azure DevOps 中创建 YAML 管道来运行 Terraform 的基础知识。我们的目标是将我们的基础设施作为代码来管理，为了这个目标，我们希望我们的管道——部署所述代码——尽可能地简洁、可配置，并遵守 [DRY 原则](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。

# 模板变量

在本系列的第一篇文章中，我们创建了一个简单的管道，在顶部定义了变量。以这种方式定义的变量被认为是“全局的”，这意味着它们适用于管道中的每个阶段、工作和任务。如果这些变量在我们管道的每个阶段都是相同的，这将是很好的，但是如果我们想要部署到多个环境呢？如果我们想使用 Terraform 在一个管道中构建不同的环境呢？我们需要为正在部署的每个组件传递一组不同的 tfvars、远程状态和文件夹路径。到目前为止，我们有一个静态管道，它有一个至关重要的问题:我们将配置与管道定义本身混合在一起。管道必须尽可能简洁地与环境相适应。

让我们将我们在管道顶部定义的变量移动到一个名为`dev_vars.yml`的 YAML 文件中。我们将把它放在一个名为`vars`的文件夹中，就在保存管道 yaml 文件的目录下:

```
➜ tree -L 2
.
├── pipeline.yml
└── vars
    └── dev_vars.yml
```

在这个文件中，我们将把前面的所有变量声明为键值对:

现在，我们的变量保存在一个模板文件中，我们可以在需要它们的阶段直接调用它们，而不是在管道顶部硬编码它们:

这意味着我们可以为我们部署到的每个环境创建多个可变模板。我们甚至可以将变量分离到特定于帐户和特定于环境的文件中，然后使用多个`— template:`参数在管道中组合起来！现在，我们将坚持使用一个环境和一个变量模板，以保持简单。

# 模板化 Azure DevOps 管道任务

我们可以转换任务、工作和阶段——是的，甚至阶段！—转换成模板。模板可以被认为是类似的函数；它们接受参数(如果您选择定义它们)，然后用这些值执行定义的指令。我们将获取组成我们的`terraform plan`的三个任务，并通过执行以下操作将它们转换成一个模板:

1.  创建一个名为`templates` 的文件夹，并在这里创建一个名为`terraform_plan.yml`的模板文件。
2.  在地形计划阶段从`steps`中剪下所有东西并粘贴到这个文件中。

下面是我们的文件夹结构现在的样子:

```
➜ tree -L 2
.
├── pipeline.yml
├── templates
│   └── terraform_plan.yml
└── vars
    └── dev_vars.yml
```

以及我们的模板文件目前的样子:

为了使其像函数一样工作，我们将执行以下操作:

1.  定义在主管道中调用时模板将接受的参数
2.  通过使用`${{ parameters.parameterName }}`语法将使用`$(variableName)`语法的变量引用更改为参数

一旦这些都完成了，你的模板应该是这样的:

# 组合变量和阶段/作业/任务模板

我们现在有了模板化的变量和模板化的管道作业。让我们用新模板替换之前作业定义下的任务，并将我们在管道顶部定义的变量传递给模板。

太棒了。现在，如果 Terraform 需要与另一个组件一起运行或针对另一个环境运行，我们只需创建一个新的变量模板。就是这样。这里有一个例子，说明我们如何在同一个管道中对两个环境运行 Terraform，只需改变我们传递给作业的`template`: