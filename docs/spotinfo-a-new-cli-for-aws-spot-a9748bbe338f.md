# Spot info——自动气象站 Spot 的新命令行界面

> 原文：<https://itnext.io/spotinfo-a-new-cli-for-aws-spot-a9748bbe338f?source=collection_archive---------1----------------------->

## 从命令行浏览 AWS spot 实例

## TL；速度三角形定位法(dead reckoning)

`spotinfo`是一个命令行工具，可以用来跨多个 AWS 区域探索 AWS Spot 实例。

![](img/b7fe89ac4912c1b587288ba3a2ea0d66.png)

## 介绍

使用 [Amazon EC2 Spot](https://aws.amazon.com/ec2/spot/) 实例是降低 EC2 随需应变实例成本的一个很好的方法，最多可降低 90%。无论何时，如果您的工作负载能够在虚拟机中断后继续运行，或者在不影响业务用例的情况下暂停并恢复运行，那么选择现货定价模式是一个显而易见的选择。

中断率越低，Spot 实例运行的时间就越长。

Amazon 提供了一个优秀的 web 界面 [AWS Spot Instance Advisor](https://aws.amazon.com/ec2/spot/instance-advisor/) 来探索可用的 Spot 实例，并确定中断机会最少的 Spot 实例池。您还可以查看通过按需费率节省的费用。您还可以查看通过按需费率节省的费用。然后，您应该使用这些度量来选择适当的 Spot 实例。

虽然 **AWS Spot Instance Advisor** 是一个有价值的工具，但使用它的数据进行脚本编写和自动化并不容易，而且一些用例需要太多的点击。

## Spotinfo 工具

这就是我创建`spotinfo`工具的原因。这是一个易于使用的命令行工具(在 Apache 2.0 许可下开源)，允许您在终端中探索 AWS Spot 实例，并使用它提供的 Spot 数据进行脚本编写和自动化。

在引擎盖下，`spotinfo`使用 AWS 提供的两个公共数据源:

1.  **AWS Spot 实例顾问** [数据馈送](https://spot-bid-advisor.s3.amazonaws.com/spot-advisor-data.json)
2.  AWS 现货定价[数据馈送](http://spot-price.s3.amazonaws.com/spot.js)

## 特征

`spotinfo`允许您访问与在 **AWS Spot Instance Advisor** 中看到的信息相同的信息，但是是从命令行访问，并且可用于脚本和自动化用例。此外，该工具还提供了一些**AWS Spot Instance Advisor**web 界面所没有的有用功能。

## 高级过滤

第一个特点是*高级过滤*。您可以通过以下方式过滤定点实例:

*   vCPU—CPU 核心的最小数量
*   内存 GiB —最小内存大小
*   操作系统— Linux 或 Windows
*   区域—一个或多个 AWS 区域(或`all` AWS 区域)
*   节省(与按需相比)
*   中断频率
*   小时费率(以`USD/hour`为单位)

按实例类型过滤时，支持[正则表达式](https://github.com/google/re2/wiki/Syntax)。这可以帮助您创建高级查询。

示例:使用正则表达式过滤

列出(以文本形式)在`us-west-2`(俄勒冈州)地区所有可用的由 [Graviton2](https://aws.amazon.com/ec2/graviton/) 处理器支持的 EC2 Spot 实例，至少有八个 CPU 内核，按现货价格对结果进行排序。

## 现货价格可见性

使用 **AWS Spot Instance Advisor** ，您可以看到相对于按需 EC2 实例价格的折扣。但是为了找出你将要支付的实际价格，你必须访问不同的 [AWS 现货定价](https://aws.amazon.com/ec2/spot/pricing/)网页，并再次搜索特定的实例类型。

`spotinfo`节省你的时间，可以显示现货价格和其他信息。如果你愿意，你也可以根据现货价格进行筛选和排序。

## 灵活的输出格式

在命令行中处理数据以及从脚本和自动化中访问数据需要灵活的输出格式。`spotinfo`可以返回多种格式的结果:人性化的格式，如`table`和普通的`text`，自动化友好的:`json`、`csv`，或者只是一个保存的数字。选择任何具体用例所需的格式。

## 跨多个区域比较斑点

关于 **AWS Spot 实例顾问**的一个恼人的事情是无法跨多个 AWS 区域比较 EC2 spot 实例。只有一个区域视图可用，或者您需要打开多个浏览器选项卡并不断地在它们之间切换，以便跨多个 AWS 区域比较 spot 实例。

`spotinfo`可以帮助您跨多个 AWS 区域比较 spot 实例。您所需要做的就是传递一个`--region`命令行标志，并且您可以多次使用这个标志。

另一个选项是传递一个特殊的`all`值(带有`--region=all`标志)来查看所有可用 AWS 区域中的 spot 实例。

示例:浏览`t4g.small` Spot 实例

探索`t4g.small` spot 实例类型的可用性以及在**所有** AWS 区域中该实例类型可用的比率。

## 网络弹性

虽然`spotinfo`使用公共 AWS 数据源，但它也在工具中嵌入了相同的数据。因此，如果数据馈送由于任何原因(没有连接、服务不可用或其他原因)不可用，那么`spotinfo`仍然能够返回相同的结果。

## 摘要

我希望`spotinfo`能够成为探索 AWS EC2 spot 实例的有用工具。我期待着您的评论和任何问题。

我邀请你为[阿列克谢领导的/spotinfo](https://github.com/alexei-led/spotinfo) GitHub 项目贡献(问题、功能、拉请求)。

*附言*:如果你喜欢`spotinfo`工具，可以考虑给 GitHub [项目](https://github.com/alexei-led/spotinfo)一个⭐️

***感谢阅读！*** *要保持联系，请关注我们的* [*DoiT 工程博客*](https://blog.doit-intl.com/) *，* [*DoiT Linkedin 频道*](https://www.linkedin.com/company/doitintl/) *，以及* [*DoiT Twitter 频道*](https://twitter.com/doitint) *。要探索职业机会，请访问 https://careers.doit-intl.com*[](https://careers.doit-intl.com/)**。**