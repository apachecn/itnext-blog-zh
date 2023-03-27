# AWS: CloudTrail 概述以及与 CloudWatch 和 Opsgenie 的集成

> 原文：<https://itnext.io/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie-1efa192bc2fa?source=collection_archive---------5----------------------->

![](img/4db53edc2d31ceaf0cc5efaa8b7bac60.png)

AWS CloudTrail 是用于审计 AWS 帐户事件的服务，默认情况下处于启用状态。

它保存用户、IAM 角色或 AWS 服务通过 AWS 控制台、AWS CLI 或 AWS SDK 完成的所有操作。

CloudTrail 会写每个 API 调用的信息，登录系统，服务事件，是 AWS 账号安全不可或缺的工具。

此类事件将存储 90 天，但您可以配置一个*跟踪*，它将把选定的事件存储到 AWS S3 桶中，并可以将它们发送到 AWS CloudWatch，在 CloudWatch 中，我们可以配置警报来接收通知。

这些踪迹可以有两种类型:应用于所有 AWS 区域的全局踪迹，以及仅用于一个选定区域的局部踪迹。参见【CloudTrail 在地区和全球的表现如何？

因此，在这篇文章中，我们将进一步了解 AWS CloudTrail 及其特性，将配置导出到 CloudWatch，并将通过 Opsgenie 创建警报和通知到 Slack。

*   [AWS CloudTrail 事件](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#AWS_CloudTrail_Events)
*   [管理事件](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Management_events)
*   [数据事件](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Data_events)
*   [洞见事件](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Insights_events)
*   [CloudTrail 步道](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#CloudTrail_trail)
*   [创建 CloudTrail 踪迹](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Create_a_CloudTrail_trail)
*   [CloudTrail 和 CloudWatch 集成](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#CloudTrail_and_CloudWatch_integration)
*   [创建一个 AWS CloudWatch 自定义指标](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Create_an_AWS_CloudWatch_custom_metric)
*   [Opsgenie 和 AWS 简单通知服务](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Opsgenie_and_AWS_Simple_Notifications_Service)
*   [创建一个 AWS CloudWatch 警报](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Create_an_AWS_CloudWatch_Alarm)
*   [来自 AWS SNS 的 Opsgenie 和自定义字段](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/#Opsgenie_and_custom_fields_from_AWS_SNS)

# AWS CloudTrail 事件

参见[什么是 CloudTrail 事件？](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html#cloudtrail-concepts-events)和[小道日志管理事件](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-events-with-cloudtrail.html)。

CloudTrail 中的*事件*是 AWS 帐户中任何活动的记录。事件可以分为三种:*管理事件*，*数据事件*，*洞察事件*。

## 管理事件

管理事件是在 AWS 资源上执行的任何操作，也称为*控制平面操作，*例如:

*   安全操作(例如，`AttachRolePolicy` API 调用)
*   新资源创建(例如，`CreateDefaultVpc` API 调用)
*   网络配置更新(例如，`CreateSubnet` API 调用)
*   记录配置更改(例如，`CreateTrail` API 调用)

此外，它还包括非 API 调用，如登录 AWS 管理控制台，请参见 CloudTrail 捕获的[非 API 事件](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-non-api-events.html)。

同样，这些事件可以是关于*读*或*写*的 API 调用。读操作是只请求关于 AWS 资源的信息的操作(例如，`DescribeSubnets`)，而写任意修改请求(例如，`TerminateInstances`，参见[读写事件](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-events-with-cloudtrail.html#read-write-events-mgmt)。

## 数据事件

参见[数据事件](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html#logging-data-events)。

这些是关于具体 AWS 资源内操作的信息，也称为*数据平面操作*，例如:

*   S3 桶操作如`GetObject`、`DeleteObject,`、`PutObject`
*   AWS Lambda 函数调用
*   亚马逊 DynamoDB 操作如`PutItem`、`DeleteItem`、и `UpdateItem`

默认情况下不会记录，创建踪迹时必须启用。

## 洞察事件

CloudTrail Insights 将监视异常，如果发现异常，将创建 CloudTrail 事件。

这种异常可能是对 AWS S3 存储桶的异常数量的删除 API 调用，或者像`AuthorizeSecurityGroupIngress`这样的过高(或过低)调用。

# 云迹步道

如上所述， *trail* 可以存储超过 90 天的事件，并且可以与 CloudTrail 和 CloudWatch 集成。此外，它可以收集所有地区的数据，而 CloudTrail Dashboard 只显示当前地区的信息。

## 创建 CloudTrail 踪迹

转到*轨迹*，点击*创建轨迹*按钮:

![](img/87c8f8784b70c688da9ef67d4627be6c.png)

设置它的名字，选择*创建新的 S3 桶*，指定它的名字，现在禁用它的数据加密:

![](img/1f4637718cc70b0af8260f1c5098b7ab.png)

要配置警报，请启用向 AWS CloudWatch 日志发送事件:

![](img/0e3652039453222bbba8d2d064fb29c9.png)

离开*管理事件*并启用*洞察*。

对于*管理事件*启用读写，对于*洞察事件*启用 *API 调用速率*:

![](img/b9cdcf2e4ab990cadec0700ec2c425b8.png)

检查 CloudWatch 日志中的数据:

![](img/aed46c43bbbc83c537ca2996bfbf8e66.png)

现在，我们可以从这些事件中创建 CloudWatch 指标并配置警报。

# CloudTrail 和 CloudWatch 集成

接下来，让我们基于 CloudTrail 事件创建一个新的 CLoudWatch 指标，然后将配置一个警报，该警报将向 AWS 简单通知服务(SNS)发送通知，该服务又将通知转发给 Opsgenie，我们将从那里接收消息到 Slack。

有一个问题，哪些赛事是我们真正需要观看的。这可以通过“ *cloudtrail security alerts* ”查询进行谷歌搜索，该查询将提供两个有趣的资料——Splunk 中的[使用 cloudtrail 和 GuardDuty 搜索威胁，以及](https://www.chrisfarris.com/post/reinforce-threat-hunting/)[在 AWS](https://www.gorillastack.com/blog/real-time-events/important-aws-cloudtrail-security-events-tracking/) 中监控安全的关键 CloudTrail 事件，或者只查看[使用 AWS CloudFormation 模板创建 CloudWatch 警报](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/use-cloudformation-template-to-create-cloudwatch-alarms.html)中的示例。

为了便于使用，我们将使用以下内容:

*   在我们办公室之外登录 AWS 控制台(作为这里的办公室 IP 示例—8 . 8 . 8):`{ ($.eventName = ConsoleLogin) && ($.sourceIPAddress != "8.8.8.8") }`
*   登录 AWS 控制台的根用户:`{ ($.eventName = ConsoleLogin) && ($.userIdentity.type = "Root") }`
*   出现错误的 AWS 控制台登录:`{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failed authentication") }`
*   未经授权的操作:`{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }`
*   例如，我们团队使用的最大类型是 c5.4xlarge : `{ ($.eventName = RunInstances) && (($.requestParameters.instanceType = *.8xlarge) || ($.requestParameters.instanceType = *.4xlarge)) }`
*   新 IAM 用户创建:`{ $.eventName="CreateUser" }`

参见[过滤器和模式语法](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html)。

首先，让我们创建一个关于任何 AWS 控制台登录的简单警报

首先，看看事件中的登录是什么样子的:登录到您的帐户，等待 10-15 分钟，根据*事件名称* == `ConsoleLogin`进行过滤:

![](img/f03fc916ad6952536cc1718388c9be83.png)

## 创建 AWS CloudWatch 自定义指标

回到上面的日志组创建，在*度量过滤器*选项卡上点击*创建度量过滤器*。

在*过滤模式*中使用`{ ($.eventName = ConsoleLogin) }`:

![](img/405c13b90c7cc91c9c79318eceb8afa0.png)

填充其字段:

![](img/d22eab0b49116d2ad63a97c4e3eb751d.png)

检查图表上的指标:

![](img/1baee3bae349a0913d252d365c9e7e8c.png)

## Opsgenie 和 AWS 简单通知服务

转到 AWS SNS，使用*standard*类型创建一个新的 SNS 主题:

![](img/76121f39baea28401d088f395a34a989.png)

在 Opsginie *整合列表*中找到*即将发布的亚马逊社交网站*:

![](img/0960e7d31124d3e18db3d09ab16eee83.png)

保存集成，保存 API 键:

![](img/e4f19ee4a140787c27c8dab0291cac61.png)

回到 SNS，点击*创建订阅*，选择 *HTTPS* ，设置 Opsgenie 提供的 URL 和 API key:

![](img/29d702e72065dea5bc0d0799c8cf02ea.png)

收到订阅已确认的通知:

![](img/1ac308b579669bfe72718552e34e602c.png)

## 创建 AWS CloudWatch 警报

转到*报警*，点击*创建报警*:

![](img/12003581ca2eb26cfd62757827d5dad5.png)

选择一个指标:

![](img/1449d9e46d94906c3af4178410d14c74.png)![](img/a6934da37a68d3767123e72134587078.png)

描述警报的条件:—大于或等于 1 个事件(每次登录时发出警报):

![](img/1a1a4cde66dc9655ec05ac8733fa25db.png)

配置向上面创建的 SNS 主题发送通知:

![](img/d105f0c1a753006be6417ff4b4ee8bc9.png)

设置提醒的名称，并保存它:

![](img/355197e74de642b437ffaa8a85969d8b.png)

几分钟后检查它的状态:它将变成*ок*而不是*数据不足*(或者如果有登录，则变成*报警*):

![](img/7fba93f9518bf519901cb976b395b09e.png)

登录 AWS 控制台，等待 10-15 分钟的 CloudTrail 事件，您将触发警报:

![](img/48d48ad63f3249575880547dedf24070.png)

以及空闲时的警报:

![](img/8d692b049f60f7ff1baa4591362ce5ce.png)

## 来自 AWS SNS 的 Opsgenie 和自定义字段

我想做的是过滤 Slack 的消息文本，只显示自定义字段。

这可以通过[操作通用字符串函数](https://support.atlassian.com/opsgenie/docs/string-processing-methods-in-opsgenie-integrations/)来完成。

进入*传入 SNS* 整合，点击*高级*:

![](img/28219d70acdc29161160c0ddbcaba7e4.png)

在*描述*字段中添加`Message.extract()`调用:

```
{{Message.extract(/AlarmDescription":"(.+)","AWSAccountId"/)}}

AWS account ID: {{Message.extract(/AWSAccountId":"(.+)","NewStateValue"/)}}

AWS region: {{Message.extract(/Region":"(.+)","AlarmArn"/)}}
```

![](img/16f20f1409671811d4a4f29b55ab1af1.png)

现在，当 Opsgenie 将从 AWS SNS 接收 JSON 中的消息时，它将只从那里获取三个字段— `AlarmDescription`、`AWSAccountId`和`Region`，因此，我们将在 Slack alarm 的消息中看到下一种形式:

![](img/d63dcb5fe0eb1e09b85380c430f1e4b8.png)

完成了。

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/aws-cloudtrail-overview-and-integration-with-cloudwatch-and-opsgenie/) *。*