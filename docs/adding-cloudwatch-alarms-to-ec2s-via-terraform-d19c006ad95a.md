# 通过 Terraform 向 EC2s 添加 Cloudwatch 警报

> 原文：<https://itnext.io/adding-cloudwatch-alarms-to-ec2s-via-terraform-d19c006ad95a?source=collection_archive---------3----------------------->

![](img/3e756d337a5f4be0cf27cf1718561929.png)

我想将 Cloudwatch 状态检查警报添加到我的 ec2 实例中，并使用 terraform 来实现这一点。以下是方法。

基本上。下面的 tf 示例在与我的 terraform 工作空间相关联的 VPC 中查找任何正在运行的 ec2 实例。它[将 InstanceIDs 与](https://www.terraform.io/docs/configuration/functions/zipmap.html)[公共或私有 IP](https://www.terraform.io/docs/providers/aws/d/instances.html)进行 zipmaps (下面的例子使用“private_ip”，但也可以很容易地替换为“public_ip”)。我还将警报动作设置为触发 lambda 的 SNS，但这不是这篇文章的内容。

```
locals {
  vpc_env = local.vpc_env_lookup[terraform.workspace]vpc_env_lookup = {
    Production = [
      "vpc-xxxxxxxxxxxxxxxx1",
      "vpc-xxxxxxxxxxxxxxxx2"
    ]
    Staging = [
      "vpc-xxxxxxxxxxxxxxxx1"
    ]
    Acceptance = [
      "vpc-xxxxxxxxxxxxxxxx1"
    ]
  }actions = local.actions_lookup[terraform.workspace]actions_lookup = {
    Prod  = [aws_sns_topic.ProductionCloudWatchLambda.arn]
    Stage = [aws_sns_topic.StagingCloudWatchLambda.arn]
    Test  = [aws_sns_topic.AcceptanceCloudWatchLambda.arn]
  }# Associate IDs with IPs (Key is InstanceID, Value is Private_IP)
  all_ec2s = zipmap(data.aws_instances.all_ec2.ids, data.aws_instances.all_ec2.private_ips)
}data "aws_instances" "all_ec2" {
  instance_state_names = ["running"]filter {
    name   = "vpc-id"
    values = local.vpc_env
  }
}resource "aws_cloudwatch_metric_alarm" "instance_statuscheck" {
  for_each = local.all_ec2s alarm_name          = join(local.delim, compact([local.envname_prefix, "StatusCheck: ${each.value}"]))
    comparison_operator = "GreaterThanOrEqualToThreshold"
    evaluation_periods  = "2"
    metric_name         = "StatusCheckFailed"
    namespace           = "AWS/EC2"
    period              = "300"
    statistic           = "Maximum"
    threshold           = "1.0"
    alarm_description   = "EC2 Status Check"
    alarm_actions       = local.actions
    ok_actions          = local.actions
    dimensions          = { InstanceId = each.key }
}
```

在上面的例子中，警报名称使用了本地 envname 前缀，这是我在 terraform repo 中的其他地方建立的。最终结果是一个类似这样的警报名称: **Staging-StatusCheck: IP。OF.MY.EC2**

希望这对您有所帮助！