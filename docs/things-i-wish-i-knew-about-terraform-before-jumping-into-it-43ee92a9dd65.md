# 在进入 Terraform 之前，我希望我知道一些事情

> 原文：<https://itnext.io/things-i-wish-i-knew-about-terraform-before-jumping-into-it-43ee92a9dd65?source=collection_archive---------0----------------------->

几周前，我写了我的 Terraform 之旅。你可以在这里阅读更多关于这个探索的内容:

[](https://medium.com/@hbarcelos/how-i-learnt-to-love-and-hate-terraform-in-the-past-few-weeks-db085d012882) [## 在过去的几个星期里，我是如何学会爱恨地球的

### 进入“基础设施即代码”世界的快乐/痛苦孤独之旅的故事

medium.com](https://medium.com/@hbarcelos/how-i-learnt-to-love-and-hate-terraform-in-the-past-few-weeks-db085d012882) 

现在我将试着总结一下我在这个过程中所学到的一些东西。

# 序

![](img/9e969ec7f89e7e605f1a2b808b404762.png)

具有讽刺意味的是，葡萄牙语对地球的翻译是“Terra”。

在真正开始探索这个基础设施即代码的世界之前，我听到了很多关于 Terraform 的废话。

> — Terraform 真的很简单，你只需要写一些代码，然后*bam！*，这是您的基础架构…
> 
> — Terraform 使用声明性语法，不会那么难…
> 
> — Terraform 与云无关，只需编写一次基础设施代码，就可以在 AWS、GoogleCloud、Azure、Bluemix 或其他平台上运行
> 
> — Terraform 可以防止沙汗以南非洲的儿童挨饿…

当然那都是扯淡！

不要误解我的意思，我仍然认为 Terraform 是一个很棒的工具，只要你进一步了解它的细节，但是学习曲线可能会非常陡峭，特别是如果你没有很好地理解底层提供者是如何工作的。

以下是我希望在开始探索之前就知道的一些事情。

# Terraform 还不成熟

![](img/d24876829e02fb5f923a6f950da89cb1.png)

唷，放松点…你还没长大到可以做这个！

Terraform 仍在积极开发中。通过访问 Github 上的[版本](https://github.com/hashicorp/terraform/releases)和[问题](https://github.com/hashicorp/terraform/issues)页面，你可以看到我对此有多认真。在我开始写这篇文章的时候，最新的“稳定”版本是`0.9.11`，仅在主回购中就有 742 个问题(每个主要的云提供商也有单独的回购)。在过去的几周里，我自己也打开了一些。

因为它的主要版本仍然是`0`，这意味着你可以期待重大的突破性变化，直到`v1`推出。事实上，当您试图开始使用它时，最令人沮丧的部分是，如果您在互联网上找到关于如何使用 Terraform 做 X 的酷文章，它很可能会过时，即使它只有几个月的历史。

在您开始将所有基础设施放入其中之前，请注意这一点。

# Terraform 不是云不可知的

![](img/6a3e7c91719cabf9246eae208a5f9e7a.png)

最终，我们仍然是一群云信徒…

我不记得我在哪里听到或读到这个，但这可能是关于 Terraform 的最广泛的假新闻。

虽然它确实提供了对多个[提供者](https://www.terraform.io/docs/providers/index.html)的支持——从全功能的提供者(如 AWS)到更具体的服务(如 Github)——但这并不意味着您可以用通用代码来描述您的基础设施，这些代码将被转换成特定于提供者的资源。

事实上，Terraform 拥有或多或少一对一映射到底层提供者资源的资源，通常还保留已知的术语。例如，一个 AWS 经典负载平衡器在 Terraform 中被命名为`aws_elb`，而在微软 Azure 上更接近的对等物被称为`azurerm_lb`。如您所料，每个资源的配置参数也会改变，因此它们是不可互换的。

所以，云平台迁移暂时还会继续吸，但是 Terraform 可以让这个任务不那么脆一点(对，就一点点)。

# Terraform 不会隐藏底层提供商的复杂性

如果你不了解 AWS 的工作原理，Terraform 不会让你的生活变得更轻松。事实上，这可能会使情况变得更糟，因为您必须同时处理 AWS 和 Terraform 的问题。

AWS 提供区域性和全球性服务。例如，EC2 是区域性的——这意味着北弗吉尼亚的自动伸缩小组与圣保罗的小组没有任何关系。因此，它们可以具有相同的名称(它们是 ASG 标识符)。IAM 则是全局的，这意味着当您定义一个角色时，它可以在任何地方使用。然后是 S3。S3 是一个混合体:虽然它有区域范围，但是它的名称空间是全局的，这意味着你不能有同名的桶，即使在不同的区域。

Terraform 在这种事情上帮不了你。当你运行`teraform plan`时，它会告诉你一切正常。然后你会愉快地进行`terraform apply`，而不是像你期望的那样平稳运行，它会在你脸上爆炸，告诉你你有多愚蠢。

# “避免代码重复”已经过时了

不，我不支持 RCP 设计模式(通过复制和粘贴重用)，但是，据我所知，用 Terraform 很难保持代码的通用性。

例如，如果您正在嵌套模块，要使最内层模块的参数可在外部配置，您必须将它提升到中间的每个模块:

```
# A/main.tf
variable "b" {}
variable "c" {}module A {
  source="/path/to/module/A" b="${var.b}"
  c="${var.c}"
}# ...
# B/main.tfvariable "b" {}
variable "c" {}module B {
  source="/path/to/module/B" var_c="${var.c}"
}# ...
# C/main.tfvariable "c" {}some_provider "some_resource" C {
  some_attribute="${var.c}"
}
```

你必须声明变量`c` 3 次和`b` 2 次才能让它工作。

有时候，你最好无耻地复制一个资源或模块，并对其进行一点点修改，然后尝试对其进行大范围的参数化，以使其更加通用。你以后会感谢我的😉。

如果你知道更好的方法，看在上帝、布达、奎师那、悟空等的份上。，请告诉我。

# Terraform 到处都是肮脏的黑客

[HCL](https://www.terraform.io/docs/configuration/syntax.html) (HashiCorp 配置语言)是 Terraform 使用的描述语言的名称(HashiCorp 是其背后的公司)。它有一些，比如说，解决某些问题的特殊方法。

![](img/322cd4bc5e0c3740dc8afdb037330c1f.png)

用 Terraform 加热披萨的感觉就像…

一个例子:假设您想要有条件地创建一个资源，如下所示:

```
variable "custom_sg" {
  description = "Custom security groups for the instance"
  default = ""
}resource "aws_security_group" "default_sg" {
  count = "${custom_sg == "" ? 1 : 0} # this is a bit odd, but ok# More params bellow... they are not relevant
}resource "aws_instace" "example" {
  securit_groups = ["${custom_sg != "" ? custom_sg : aws_security_group.default_sg.id}"]# More params bellow... they are not relevant
}
```

上面的代码应该可以工作吧？不对！

每当您在资源中使用`count`参数时，Terraform 将假定它是资源列表，即使唯一可能的值是`0`和`1`。所以你上面的代码就在你眼前崩溃了(好消息是它在`plan`阶段失败了)。

因为你的`aws_security_group.default_sg`是一个列表，你不能直接访问它的参数。相反，你必须指出清单上的单个项目。Terraform 让你使用类似于`resource.name.<n>.param`的语法来实现，其中`n`是一个数字。

> 所以我可以这样声明:

```
aws_security_group.default_sg.0.id
```

不错的尝试，但你不能！因为 HCL 的实现方式，它必须解析整个模板，即使值没有被使用。所以在你设置了`default_sg`的情况下，`aws_security_group.default`中没有项目可以被`0`引用，所以 Terraform 会再一次把你打趴在地，然后笑到你的哭娃娃脸上。

“官方”的解决方法是这样的:

```
resource "aws_instace" "example" {
  securit_groups = ["${custom_sg != "" ? custom_sg : join("", aws_security_group.default_sg.*.id)}"] # WTF dude? # More params bellow... they are not relevant
}
```

真的吗？这到底是从哪里来的？那个`*`表示列表中的所有元素。如果不存在，则返回一个空列表。那个`join`只是一个普通的`List -> String`函数。当列表为空时，它返回一个空字符串。当它有一个元素时，它返回我想要的参数。

有很多像这样肮脏的小黑客。找到它们的地方是[问题](https://github.com/hashicorp/terraform/issues)页面。

# 不要使用嵌套/内嵌资源

不，说真的…不要那样做…这让我很痛苦。

如果你不知道我在说什么，Terraform 允许你在它的“父”中定义一些资源，以及一个引用它的独立资源。

我建议这样做的原因是，至少对我们来说，一个常见的用例是需要扩展安全组、路由表和其他支持内嵌资源的资源。我们可以从模块定义之外做到这一点的唯一方法是使用指向模块内资源的独立资源。

我定义了一个 VPC 模块，它包含了 VPC 的基本定义。这使得在多个区域上创建 VPC 变得轻而易举。然而，我们的主要基础设施位于 N. Virginia，所以它的配置应该与其他基础设施有一点不同(尽管没有不同到足以证明代码复制的合理性)。

最初，我在我们的 VPC 模块定义中有这样的内容:

```
# ...
resource "aws_route_table" "public" {
  vpc_id = "${aws_vpc.cluster_vpc.id}" route {
    destination_cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.default_ig.id}"
  }
}
# ...
```

然后，我将通过重用模块来创建我们的 VPC:

```
module "us-east-1" {
  source = "../../modules/vpc" aws_region = "us-east-1"
  vpc_cidr_block = "10.0.0.0/16"
}module "us-east-2" {
  source = "../../modules/vpc" aws_region = "us-east-2"
  vpc_cidr_block = "10.1.0.0/16"
}
// ...
```

然而，对于`us-east-1`，我需要在这个地区的另一个 VPC 之间创建一个 VPC 对等。为了实现这一点，除了创建对等连接，我还需要修改路由表，添加一个条目来正确路由流量。

```
// ...
resource "aws_route" "peer_vpc" {
  route_table_id = "${module.us-east-1.public_route_table_id}"
  destination_cidr_block = "172.16.0.0"
  vpc_peering_connection_id = "pcx-xxxxxx"
}
// ...
```

这应该可以，对吧？

![](img/6528bbb25f05af3f0f4ab40d253532e2.png)

请不要把这个帖子政治化！

我在混合两种风格时观察到的行为是，如果独立资源不存在，它们将被创建。然而，一旦创建了，如果我再次运行`terraform apply` ，它们就会被删除。如果我再试一次，它们就会被创造出来，以此类推…

问题是你不能两者兼而有之。我不得不在 Github 上挖掘一些 Terraform 问题来了解这一点。

幸运的是，这在文档中已经很清楚了，例如在`aws_route_table`资源中提到的:

> **路由表和路由注意:** Terraform 目前提供独立的[路由资源](https://www.terraform.io/docs/providers/aws/r/route.html)和内嵌定义路由的路由表资源。目前，您不能将包含内嵌路由的路由表与任何路由资源一起使用。这样做将导致规则设置冲突，并将覆盖规则。

解决方案是将模块内的内联路由提取到它自己的资源:

```
// ...
resource "aws_route" "internet" {
  route_table_id = "${aws_route_table.public.id}"

  destination_cidr_block = "0.0.0.0/0"
  gateway_id = "${aws_internet_gateway.default_ig.id}"
}
// ...
```

这样我就可以在模块外修改路由表，而不会被吓到。

## 结论

与 Terraform 合作非常愉快。有时候我觉得活活烧死会更爽，但还是很爽。

从我开始到现在，我注意到文档的改进，这可能有助于使学习曲线不那么陡峭。

由于我还在学习，我可能有些地方做错了。因此，随着我发现新的/更好的模式，将来这篇文章可能还有改进的空间。

你喜欢你刚刚读的吗？你为什么不给我买一瓶啤酒呢？

![](img/4e203d744bdd682634f2998eb989464c.png)

继续收听更多…