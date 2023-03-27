# terraform 中不确定的 null_provider 行为

> 原文：<https://itnext.io/non-deterministic-null-resource-provider-behaviour-in-terraform-f5aaba64179f?source=collection_archive---------1----------------------->

![](img/f502e50dee626de20ea47a491b677994.png)

我们非常依赖 terraform 在 AWS 云上设置我们的基础设施。我们需要提供`EC2`实例，并在创建时用一些引导代码和代理设置对它们进行配置，以便它们可供我们的内部用户使用。这个引导脚本是使用一个`null_resource`应用的，这个`null_resource`可以通过 terraform 的`null_provider`获得。我们有一个平台，允许我们的内部用户为多种用例(数据科学、面向 Web 的应用程序等)构建不同类型的`EC2`实例。)

null_resource 我们用来配置新创建的 EC2 实例的代码

这些实例有一个与它们相关的`cloud-init`模板，该模板在实例初始化时安装了各种工具，这是我们的用户经常要求的。bootstrap 脚本也使用其中的一些工具来下载包并对它们进行配置。`unzip`是我们用来提取打包的 tarballs 并从源代码安装的工具之一。但是我们很快注意到这个`null_resource`经常失败，并抱怨`unzip`不可用。简单地重新运行作业就可以解决问题，但是最大的问题是为什么它会出现在第一个位置！

我们会在游戏机上不断出现的错误。这是我们没有接触到的地球形态丑陋的一面！s

我坐下来深入研究这个问题，花了相当长的时间才意识到它的发生并不是不确定的；该错误有时是可重现的，而其他时候代码完全正常。很快就清楚了，这是 terraform 的一面，我联系了谷歌搜索，看看我是不是一个人。几个点击[这里和那里](https://github.com/hashicorp/terraform/issues/16656)证明我不是一个人，这实际上是一个非常令人讨厌的，但从 terraform 的`null_provider`已知的行为。

[](https://github.com/hashicorp/terraform/issues/16656) [## remote-exec provisioner: apt-get 安装偶尔会失败问题#16656 hashicorp/terraform

### 此时您不能执行该操作。您已使用另一个标签页或窗口登录。您已在另一个选项卡中注销，或者…

github.com](https://github.com/hashicorp/terraform/issues/16656) 

[显然](https://unix.stackexchange.com/a/471192/398090)`null_resource`不会等待`cloud_init`结束，并且会假设因为`EC2`实例已经被创建，它可以继续运行`null_resource`。这个动作甚至会绕过我们为解决这个问题而引入到资源中的`depends_on`属性，因为根据 terraform，实例已经被创建了。实际的修正是在引导脚本中引入下面的代码头，允许脚本等到`cloud_init`完成安装。

游戏规则改变者[评论](https://github.com/hashicorp/terraform/issues/16656#issuecomment-444379459)关于地形问题！

[](https://unix.stackexchange.com/questions/315502/how-to-disable-apt-daily-service-on-ubuntu-cloud-vm-image/471192#471192) [## 如何在 Ubuntu 云 VM 镜像上禁用' apt-daily.service '？

### Ubuntu 16.04 服务器 VM 镜像显然每 12 小时左右启动一次“apt-daily . service”；该服务执行…

unix.stackexchange.com](https://unix.stackexchange.com/questions/315502/how-to-disable-apt-daily-service-on-ubuntu-cloud-vm-image/471192#471192)