# AWS: WAF WebACL 日志记录和 Logz.io 集成

> 原文：<https://itnext.io/aws-waf-webacl-logging-and-logz-io-integration-da2803cbb8da?source=collection_archive---------5----------------------->

![](img/4db53edc2d31ceaf0cc5efaa8b7bac60.png)

在第一篇文章中— [AWS: Web 应用防火墙概述、配置及其监控](https://rtfm.co.ua/en/aws-web-application-firewall-overview-configuration-and-its-monitoring/) —我们谈到了它的主要组件，为它创建了一个 WebACL 和规则，并进行了基本的监控。

此外，我们已经用 AWS Kinesis 配置了 WebACL 的日志集合，但现在是时候查看它们 Logz.io 了，因为 CloudWatch Logs 还不可用。

因此，在这篇文章中，我们将配置日志通过 AWS Kinesis 发送到 AWS S3 桶，然后将配置 Logz.io 从 S3 获取这些日志，并将谈论日志内容以及如何使用它们进行调试。

# **内容**

*   [AWS 晶圆日志](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#AWS_WAF_logs)
*   [AWS Kinesis](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#AWS_Kinesis)
*   [WAF ACL 记录](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#WAF_ACL_logging)
*   [Logz.io S3 桶](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#Logzio_S3_Bucket)
*   [Logz.io S3 配置](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#Logzio_S3_configuration)
*   [IAM 策略](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#IAM_policy)
*   [我的角色](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#IAM_role)
*   [晶圆日志—字段](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#WAF_logs_-_fields)
*   [编辑字段—从日志中排除数据](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#Redacted_fields_-_excluding_data_from_logs)
*   [Logz.io 报警](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/#Logzio_alerting)

# AWS 晶圆日志

除了 CloudWatch 指标，我们还可以为通过 WebACL 传递的所有请求启用日志。

AWS WAF 使用 AWS Kinesis 发送数据，使用 AWS S3 存储日志。

遗憾的是，WAF 和 Kinesis 无法使用 CloudWatch 日志(至少，在绞拧的那一刻，正如 AWS 技术支持告诉我的，计划启用它们)。

所以，让我们做以下事情:

1.  配置 Kinesis 消防站流
2.  配置 WebACL 以向流发送数据
3.  Kinesis 流将数据发送到 AWS S3 桶
4.  Logz.io 将使用它的连接器从这个 S3 获取日志

## AWS 室壁运动

> **注:**一个 AWS WAF 日志相当于一个 Kinesis 数据消防软管记录。如果您通常每秒收到 10，000 个请求，请在 Kinesis Data Firehose 中设置每秒 10，000 条记录的限制，以启用完整日志记录。

转到 AWS Kinesis，创建一个新的*数据消防软管传输流*，记住其名称必须以“ *aws-waf-logs-* 为前缀。

选择*直接放或者其他来源*，点击*下一个*:

![](img/8888d1556d996c69d8b84f07307a15c9.png)

在下一页跳过数据转换，点击*下一个*，然后对于*目的地*选择 *S3* ，并创建一个新的 S3 桶:

![](img/4ec71ded3f740df1588b4e05609456a5.png)

这里，我使用一个已经存在的桶。为避免混合不同 ACL 的日志，请添加前缀:

![](img/cff00033faadcb2945953a8da2b1229f.png)

在下一页，保留所有默认值。如果出现任何问题，Kinesis 将通过 CloudWatch 发送一条消息:

![](img/b7f3997f5e8ed2243558927d8ad83039.png)

要获得这些信息，请启用*错误记录*选项:

![](img/a04eadcd0924b470022edd08f0f7abbd.png)

检查设置，并创建流:

![](img/4eda12d56c6927523c865fa573e05e32.png)

等待几分钟以创建流:

![](img/5ac6618689e6c3e7777d6a8038673d05.png)

或者直接访问网络访问列表。

## WAF ACL 记录

转到 ACL，在*日志记录和指标*选项卡上点击*启用日志记录*:

![](img/a50689d0b2680e77da956e5be32d2018.png)

选择上面创建的流:

![](img/89ee3c0bf64ee42628e12d5aec5cf0ea.png)

在这里，我们还可以启用过滤器，这将在下面谈到它们。

# Logz.io S3 桶

这里的文献资料是[>>>](https://app.logz.io/#/dashboard/send-your-data/log-sources/s3-bucket)。

首先，我们需要创建一个 IAM 策略和一个角色，它将提供对包含日志的 S3 存储桶的访问。查看其文档[此处> > >](https://docs.logz.io/user-guide/give-aws-access-with-iam-roles/) 。

Logz.io 将通过这个角色来获取访问 bucket 的权限。

## Logz.io S3 配置

要向 [S3 添加铲斗，点击*上的*](https://app.logz.io/#/dashboard/send-your-data/log-sources/s3-bucket)添加铲斗:

![](img/4d4a2ea56362627e5f179b30ca0c7fff.png)

选择*用角色*认证:

![](img/6c7789d56c38e92d4145ff3838f4d11b.png)

指定桶的名称，点击*获取角色策略*:

![](img/fc7c78dce9d5b3ceee24a7046b53c742.png)

## IAM 策略

复制策略内容，转到 AWS IAM，创建新策略:

![](img/e5a06cea52e9a3b5f1af0b78a16a6bdd.png)![](img/0eed1d491450c3ff0358a73d59863de0.png)

从 Logz.io 添加 JSON:

![](img/1a2f74c0eea37c18cc932521e7983291.png)

保存策略:

![](img/0eae47c66ecd093c6e0101abe98fc499.png)

## IAM 角色

转到в AWS IAM 角色，创建一个新角色。

选择*另一个 AWS 账户*，从 Logz.io 页面指定账户 ID 和外部 ID:

![](img/cd5eb0b5a27985a4d9ae2459be0d24bc.png)

在*权限*页面上，附加我们在上面创建的策略:

![](img/2571d046de1b783a86f2cf0e02c0641e.png)

保存角色:

![](img/d05691b4542b7c99c54a972e869b1779.png)

在其*可信实体*中，我们可以看到帐户 ID，它将能够承担该角色。

复制角色的 ARN:

![](img/4daee76a191709dfe821af8a79ad8930.png)

返回 Logz.io，设置 ARN，并选择一个 AWS 区域。

在数据类型中选择*其他*:

![](img/5eb5ee6ab31567afa2dd1226b91c6124.png)

并将其值设置为 *awswaf* (此处参见所有类型[>>>](https://docs.logz.io/user-guide/log-shipping/built-in-log-types.html))，这样 ELK 将能够解析日志 JSON:

![](img/e370a9fa4a54a44a41f8962b7a29013b.png)

点击*保存*，水桶就连接到你的 Logz.io 账号了。

# WAF 日志-字段

现在，需要为 WebACL 附加一个 ALB，并等待流量查看 ACL 日志中的内容。

将 ACL 附加到入口— `[alb.ingress.kubernetes.io/wafv2-acl-arn](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.1/guide/ingress/annotations/#wafv2-acl-arn)`:

![](img/e047124b75a304feaac1974c2c298b5a.png)

几分钟后，查看 S3——一个带有我们前缀的新目录将被创建，它是由 Kinesis 添加的:

![](img/6aeda3a995dec7ca4c0a41dc86445f26.png)

第一个日志文件:

![](img/2f38d7ebd42f7e7e7a3273b7a6f5ae44.png)

同样，等待几分钟，以便日志将被传送到 Kibana。

ELK 将解析它的字段，我们可以在搜索时使用它们。

有两个主要使用的字段:`type: "awsfaw"`，用于查看所有日志，或者`webaclId`，用于仅选择特定的 WebACL 日志:

![](img/b8c30247f0c499e72d71658ff050c17a.png)

此外，值得在结果表中使用以下字段(参见[文档> > >](https://docs.aws.amazon.com/waf/latest/developerguide/logging.html#logging-fields) ):

*   `httpRequest.clientIp`:发起请求的 IP
*   `httpRequest.uri` : URI 用于请求
*   `httpRequest.args`:其论据
*   `terminatingRuleId`:WebACL 的规则，这是最后一个允许或阻止的规则
*   `ruleGroupList`:WebACL 的所有规则，针对请求进行检查

让我们看看在日志消息中到底能找到什么。

首先，让我们检查一个请求，该请求被允许访问我们的应用程序，即使用 ALLOW 操作:

![](img/35631a853dae0c2a6a18bf1dd2fea740.png)

这是来自 IP *99 的请求。***.***.101* 到 URI */***/126/like* ，按`ruleGroupList`链规定的规则检查，最后用 WebACL 的`Default_Action`允许。

此外，在`ruleGroupList`中，我们可以看到一个规则被应用于请求- `SignalNonBrowserUserAgent`，但是由于我们将该规则设置为*计数*操作，因此它被忽略:

![](img/62cb46c7ba1200d17f6749442c9900b4.png)

最后，在`terminatingRuleId`в中，我们看到`Default_Action`被应用，因为`SignalNonBrowserUserAgent`由于计数动作而被排除。

如果一个请求被阻塞，那么在`Default_Action`中，我们将看到一个应用了*阻塞*操作的规则:

![](img/0a075fd65deb47947937b0fa3a9115b9.png)

## 密文字段—从日志中排除数据

在日志中，我们可以获得一些我们不希望存在的数据，如 cookies 信息或身份验证令牌:

![](img/179c7ac05a434fe422f2cdbd6335182f.png)

要删除该数据，请转到日志设置，然后在*修订字段*中选择需要排除的字段。

在当前的例子中，它将是标题和来自那里的两个字段— `authorization`和`cookie`:

![](img/d7e9d1575c63dd1c93b3138c291a051b.png)

应用更改，现在您将看到它们的值被编辑为*:*

*![](img/81e6072826e649613dd1c8ef24b6949e.png)*

# *Logz.io 警报*

*此外，获得块的通知也是一个好主意。*

*让我们通过 [Logz.io 与 Opsgenie 的集成](https://support.atlassian.com/opsgenie/docs/integrate-opsgenie-with-logzio/)创建一个警报，Opsgenie 将向我们的 Slack 发送通知。*

*描述警报的条件，在*过滤器*中添加`action != ALLOW`:*

*![](img/0255273c770f70a59b2275e91b4b326f.png)*

*选择发送通知的渠道，以及您希望在消息中看到的字段:*

*![](img/cd039a11673fdde062a823cca44e31c6.png)*

*在空闲时等待警报:*

*![](img/65f971f065df6704f229914d1cf51b63.png)*

*完成了。*

**最初发布于* [*RTFM: Linux，DevOps，和系统管理*](https://rtfm.co.ua/en/aws-waf-webacl-logging-and-logz-io-integration/) *。**