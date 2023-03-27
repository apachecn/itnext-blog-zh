# AWS 中的容错 OpenVPN

> 原文：<https://itnext.io/fault-tolerant-openvpn-in-aws-a47e0d17ca85?source=collection_archive---------2----------------------->

![](img/0bbffe7377073dd210551974e62c5d51.png)

**采用 RDS MySql 后端部署的 OpenVPN 可以承受 AWS AZ 崩溃。**

我确信你们中的许多人已经在使用 OpenVPN 安全地访问你们的 VPCs。我也确信，和我一样，你也试图找出一种方法，以高度可用或容错的方式部署**支持的**[AWS market place ami](https://aws.amazon.com/marketplace/search/results?page=1&filters=vendor_id&vendor_id=aac3a8a3-2823-483c-b5aa-60022894b89d&searchTerms=openvpn)。

我必须说，不幸的是，AWS 没有为 EC2 服务提供实时迁移，而事实上，其他云提供商，如 Azure 和谷歌云平台提供了这种服务。简而言之，该特性允许机器在一个机架中自由地发生故障，并自动地连同它们的状态一起转移到不同的机架或数据中心(尽力而为)。

虽然我同意这种特性会给一些工作负载带来潜在的不确定性，但我们仍然有大量像 OpenVPN 这样的宠物服务器应用程序，它们与某种类型的网络或存储状态紧密绑定。

> 更新:正如[卢克·扬布拉德](https://medium.com/@LukeYoungblood?source=responses---------0----------------)在下面的评论中提到的，AWS 提供了一个 [CloudWatch 事件](https://aws.amazon.com/blogs/aws/new-auto-recovery-for-amazon-ec2/)你可以监听并恢复 EC2 附件。

AWS 中一个典型的解决方法是将应用程序启动编码为一个**启动配置**，并在一个多 AZ **自动伸缩组**的一个实例上运行服务器。为了使这项技术适用于 Marketplace OpenVPN AMI，我们需要确保设置满足以下要求:

1.  能够将 OpenVPN 的配置数据库卸载到多 AZ RDS 实例。
2.  能够在第一次启动时自动更新 OpenVPN 的 DNS 记录以解析到一个全新的弹性 IP。

幸运的是，这都可以通过 AWS 实例 UserData 和 IAM 权限来实现。以下脚本用几个亮点描述了我们当前的实现:

*   我们为 OpenVPN 创建了一个**唯一的 Route53 托管区域，该区域仅包含它自己的记录，我们赋予 OpenVPN 实例角色更新该区域中记录的能力。尽管这听起来没有必要，但这是保证 OpenVPN 服务器角色不会被恶意使用并删除共享区域中所有其他记录的唯一方法。**
*   我们将一个离线加密的秘密传递给包含数据库密码的实例。我们这样做是为了防止数据库密码以明文形式存储在 terraform 状态文件中。

启动配置和自动缩放组

以下 UserData Terraform 模板将 OpenVPN 本地数据库迁移到 RDS 中，更新其 DNS 记录并设置一些默认的有用运行时设置。

启动配置用户数据脚本。

最后，IAM 策略使自动化成为可能。

OpenVPN 服务器的 IAM 策略

# 包扎

这并不是火箭科学，但是花了一点时间摆弄 OpenVPN 来让这些脚本工作。我希望当你需要在 AWS 中设置一个新的 OpenVPN 服务器时，这可以作为你的参考。

干杯