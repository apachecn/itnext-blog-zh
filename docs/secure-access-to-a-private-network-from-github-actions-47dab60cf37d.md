# 从 GitHub actions 安全访问专用网络

> 原文：<https://itnext.io/secure-access-to-a-private-network-from-github-actions-47dab60cf37d?source=collection_archive---------1----------------------->

![](img/8ded5d17b3081087cf66492b55a927cf.png)

克劳迪娅·苏拉娅在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

GitHub 的动作很牛逼。我喜欢它。但是通常构建步骤必须连接到云中的私有资源。比如运行在私有网络的 MySQL 服务器，或者私有 k8s 集群。这可能会成为一个真正的障碍，你不应该做的是向世界开放你的资源。互联网充满了威胁，如果不将基础设施锁定在专用网络后面，坏事可能会发生。在这篇文章中，我将演示一种叫做 **ssh 端口转发**的简单技术，它将包括使用 Terraform 旋转一个堡垒虚拟机，然后通过 ssh 经由这个实例连接到一个私有的 MySQL 服务器。

在我们进入代码之前，让我们先讨论一下相关的网络。GitHub actions build step 正在某个远程数据中心(可能是 azure，因为微软几年前收购了 GitHub)的某个容器上运行，并且想要连接到在另一个数据中心运行的专用 MySQL 服务器，我说它的专用是指它连接到虚拟专用网络，让我们以 GCP 为例。因此，MySQL 实例只有一个私有 IP 地址，看起来类似于 10.4.20.15，如果您尝试 *ping* 此 IP，您将会丢失 100%的数据包，因为它在私有网络之外是不可到达的，为了能够从外部世界到达此 IP，我们可以做一些技巧，在私有网络中安装 VPN 并连接到它[(例如使用开放式 VPN )](https://github.com/helm/charts/tree/master/stable/openvpn) 。尽管这对于本地开发来说非常方便，但在我看来，在 GitHub 的每个版本上安装和配置 VPN 客户端有点太复杂了。

现在，让我们进入代码:)

端口转发命令

让我们将它上面的命令分解成它的组件:

*   **-o stricthostkey checking = no**告诉 ssh 不要检查我们将要连接的主机的 **known_hosts** 文件
*   **-tt** 将强制 tty 分配(关于什么是 tty 的好帖子可以在[这里](https://www.howtogeek.com/428174/what-is-a-tty-on-linux-and-how-to-use-the-tty-command/)找到)
*   -I**github_id_rsa " $ { SHH _ USER } @ $ { BASTION_IP } "**使用位于工作目录中的 github _ id _ RSA 私钥作为 SSH_USER 连接到 BASTION _ IP
*   **-L " 3306:$ { MYSQL _ IP }:3306 "**L 标志作为 ssh 手册页准确地告诉:指定到本地(客户端)主机上的给定 TCP 端口或 Unix 套接字的连接将被转发到远程端的给定主机和端口或 Unix 套接字。
*   **&** 在后台运行

为了在构建步骤的工作指导中拥有私钥，我们需要以下命令。

从 github actions secret 读取 base 64 编码的私钥，并写入本地文件

一旦执行了上面的命令，连接到 *localhost:3306* 上的 MySQL 服务器将通过我们创建的 ssh 隧道连接到远程 MySQL 服务器。以安全的方式。我们唯一缺少的是一个 bastion 服务器，它也安装了 ssh，位于我们想要到达的私有网络中，并注入了 github_id_rsa 的 pair 公钥。在 GCP 的 web 控制台上点击几下就可以创建这个服务器，但是使用 Terraform 更有趣，所以让我们看一下代码。

使用 terraform 创建堡垒 ssh 服务器

这里有几件事情:

*   为我们的堡垒虚拟机创建一个新的服务帐户(GCP 服务帐户就像一个应用程序标识符)
*   创建公共静态 ip 地址
*   在私有网络中创建一个虚拟机，注入我们之前生成的 ssh 密钥对的公钥(ssh-keygen -c -C user@local -f key)，并为其分配网络标记
*   创建防火墙规则，使用我们创建的网络标记将我们的虚拟机作为目标，并向全世界开放端口 22

我在这里没有提到的是私有网络本身的创建和 Terraform GCP 提供商的设置，这超出了本文的范围。(但真的没那么复杂)

为了测试在 GCP 项目上应用上述 Terraform 更改后一切正常，尝试对实例使用 ssh。

要从 Terraform run 获取其 bastion IP:

> *地形状态显示 Google _ compute _ address . bastion _ IP _ address*
> 
> ssh ${BASTION_IP} -i 用户 id_rsa

如果 ssh 正常工作，并且您发现自己在 vm 中，那么恭喜您，一切都应该没问题了，防火墙已经配置好了，您可以安全地进入私有网络了。

关于 k8s 私有集群的一个注意事项:如果您想在私有集群中获得服务，有几种方法可以实现。在我看来，在您的集群中安装一个[外部 DNS k8s 操作器](https://github.com/kubernetes-sigs/external-dns)，它将为每个服务分配域名，然后通过与上述相同的技术访问这些服务，这是最简单的方法。(您需要一个外部 DNS，因为您试图从 k8s 外部访问 k8s 中运行的服务，因此无法使用 k8s 内部 DNS 名称。但这是另一篇文章的内容。

最后，不要忘记从容器中删除私钥，把私钥到处乱放是不好的做法。

清理我们用于 ssh 隧道的私钥的构建步骤

今天到此为止。

再见，谢谢所有的鱼；)