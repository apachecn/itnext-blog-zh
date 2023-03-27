# 使用 Terraform 通过代码配置 AppDynamics

> 原文：<https://itnext.io/configuring-appdynamics-through-code-using-terraform-f499068b5d02?source=collection_archive---------1----------------------->

虽然更改用户界面的配置很容易，但长时间维护它却是一件麻烦的事情。如果您负责多个环境和应用程序的配置，这一点尤其正确。在本文中，我们将看到一个新的用于 AppDynamics 的 Terraform 提供程序，它使用配置作为代码来解决这个问题。

![](img/6272acbab083c6c43055f5143b5218d8.png)

# 为什么将配置作为代码

“as code”实践最近已经被广泛使用。在基础设施中，你可能认为它是代码，通常用于在 AWS 上构建应用程序。有各种各样的工具用于这一点，如地形，CDK，云形成和木偶，仅举几例。我们稍后将使用 Terraform，对此的介绍可以在[这里](https://www.terraform.io/intro/index.html)找到。

这种做法的替代方法是手动创建和更改内容。这可能是在服务器上运行命令或者改变用户界面(ui)中的东西。这使得复制东西变得困难，因为你必须记住并手动应用更改。

通过将我们的配置定义为代码，我们获得了能够重用它的好处。如果我们应用好的编程技术，我们可以将它模块化，并在许多地方使用它。例如，在生产前和生产环境中使用相同的配置，以及在类似的应用程序之间共享通用配置。

作为代码的配置允许我们保持我们的配置接近相关的代码库。这使得查找和编辑变得容易，而且还允许人们无需登录第三方应用程序就可以轻松查看配置。还可以将配置登记到版本控制中，允许它随着应用程序而改变，并且还具有审计和协作工具。如果您的公司符合 SOX，这可以用作一种控制。

# 什么是 AppDynamics

[AppDynamics](https://www.appdynamics.com/) (AppD)是一款应用性能监控(APM)工具。它为您的应用程序提供工具，因此您可以看到内部是如何工作的。这包括数据库查询、HTTP 请求等等。它为您提供了各种指标，如延迟、请求和错误，让您对一切按预期运行充满信心。可以设置警报，在事情没有按预期运行时通知您，以便有人进行干预和修复。

AppDynamics 的关键卖点之一是自动配置。在现实中，团队通常需要手动配置他们的应用程序，尤其是当它是一个路径中带有 id 的 rest 应用程序时。要配置应用程序，团队必须使用 AppD 提供的 UI，并且必须在每个环境中进行这种配置。

# 给我看看代码！

为了使用代码配置 AppDynamics，创建了一个 Terraform AppDynamics 提供程序。这可以在 [Terraform 社区注册表](https://registry.terraform.io/providers/HarryEMartland/appdynamics/latest/docs)和 [GitHub](https://github.com/HarryEMartland/terraform-provider-appdynamics) 中找到。下面是一个示例 Terraform 文件，它用事务配置了一个 restful 端点并设置了一个警报。该示例使用了 0.13 版中的新提供者语法，允许轻松集成社区提供者。AppDynamics 提供者的文档可以在[这里](https://registry.terraform.io/providers/HarryEMartland/appdynamics/latest/docs)找到。

这是相当多的代码，让我们来分解一下。前 27 行是定义变量，所以我们可以在以后重用它们。别担心，这个文件中的所有值都是假的。如果您在管道中运行它，变量值将被传入。更多信息参见[文件中的变量分配](https://www.terraform.io/docs/configuration/variables.html#assigning-values-to-root-module-variables)。

第一个主配置块从第 29 行开始，这创建了一个事务检测规则，告诉 AppDynamics 如何将请求分组在一起。在这个例子中，我们使用 regex 将 restful 用户端点分组在一起，以 id 结尾。默认情况下，AppD 会为每个用户 id 创建一个新的事务。

从第 42 行开始的下一个块创建了一个健康规则。此规则检查应用程序中的所有业务事务，并将它们与基线进行比较。健康规则用于定义什么对应用程序是健康的。

从第 56 行开始定义一个动作，当一个动作被触发时，它执行一些事情。在这种情况下，它会向一系列地址发送电子邮件。

最后，从第 65 行开始，定义了一个策略。这将健康规则和行动联系在一起。当健康规则更改时，会触发该操作。请注意，我们如何从健康规则和操作中引用属性，这样，如果它们发生变化，策略就会自动更新。在本例中，策略查看应用程序中的所有健康规则，并触发之前设置的操作。由于 AppDynamics API 对电子邮件操作的工作方式，我们必须使用电子邮件列表作为操作名称。

# 未来配置

terraform 提供程序目前仅限于单个指标健康规则。这主要是为了降低初始版本的复杂性，但是在一个规则中包含多个指标可能是一种反模式。它们不应该是单独的规则吗？

有一些资源尚未实现，如作用域和动作抑制。这些仍然可以在 UI 中创建，直到它们包含在未来的版本中。

如果您确实需要提供商缺少的功能，请在 GitHub 上提出问题，如果已经存在，请投票。更多用户的问题将被优先考虑，就像所有好的开源软件一样，拉请求是受欢迎的。

我希望这个 Terraform 提供者能够通过为 AppDynamics 提供一个 config as code 解决方案来帮助您保持工程实践的完整性。如果您有任何问题或建议，请随时提出 [GitHub 问题](https://github.com/HarryEMartland/terraform-provider-appdynamics/issues)。