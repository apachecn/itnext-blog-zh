# 使用 Kops 升级 Kubernetes 集群，以及需要注意的事项

> 原文：<https://itnext.io/upgrading-kubernetes-cluster-with-kops-and-things-to-watch-out-for-8b5e7dff71c0?source=collection_archive---------6----------------------->

![](img/a2b859667a2957df85b3ddf578c4d157.png)

好吧！我想为一年多来的无所作为道歉。非常尴尬的是，我完全放弃了这个好习惯。不管怎样，今天我想分享一个不太高级也很简短的关于如何用 kops 升级 Kubernetes 的演练。

在 Buffer，自从我们在 [AWS EKS](https://aws.amazon.com/eks/) 之前开始我们的旅程以来，我们在 AWS EC2 实例上托管我们自己的 k8s(简称 Kubernetes)集群。为了有效地做到这一点，我们使用 [kops](https://github.com/kubernetes/kops) 。这是一个非常棒的工具，可以管理集群管理的几乎所有方面，从创建、升级、更新到删除。它从未让我们失望。

# 如何开始？

好吧，升级集群总是让人紧张，尤其是生产集群。相信我，我也经历过！有句话叫，**希望不是策略**。因此，我并不希望事情进展顺利，相反，我总是有偏见，如果你跳过测试，事情就会变得糟糕。另外，祝你好运，向人们解释集群现在停机是因为有人决定尝试一下，看看会发生什么。

那我们该怎么办？

简单！我们在运行生产集群的版本中创建一个新的集群，添加一个基本的测试 web 应用程序，并在升级期间向它发送流量，以确保服务不会中断。让我试着为你更详细地分解这些步骤。

*   升级会中断任何正在运行的应用程序吗
*   新版本能很好地与现有的 EC2 实例类型兼容吗

(有些情况下，使用新的 AWS AMI 的升级版本不能与特定的实例类型一起工作，导致集群范围的暂停，并且无法创建容器)

*   从我们正在升级的版本`from`和`to`来看，基础服务还能顺利工作吗？
*   我们研究关键的网络组件，如`flannel`和`kube-dns`。如果我们发现任何问题，我们应该后退一步，全面调查任何问题。

# 创建新的测试集群

在 AWS 上用 kops 创建一个集群最初有点棘手，因为必须创建一些关键的 AWS 资源。这里是关于如何做的指南。你只需要做一次，永远。之后，创建一个集群就像使用这个命令一样简单

```
kops create cluster \ 
--name steven.k8s.com \ 
--cloud aws \ 
--master-size m4.large \ 
--master-zones=us-east-1b,us-east-1c,us-east-1d \ 
--node-size m4.xlarge \ 
--zones=us-east-1a,us-east-1b,us-east-1c,us-east-1d,us-east-1e,us-east-1f \ 
--node-count=3 \ 
--kubernetes-version=1.11.6 \ 
--vpc=vpc-1234567 \ 
--network-cidr=10.0.0.0/16 \ 
--networking=flannel \ 
--authorization=RBAC \ 
--ssh-public-key="~/.ssh/kube_aws_rsa.pub" \ 
--yes
```

需要注意的是，我们正在尝试将生产集群升级到新版本，因此我们希望测试集群尽可能靠近生产集群。请确保`--kubernetes-version`是当前的生产版本，并在`--networking`中使用相同的覆盖网络。保持实例类型不变以避免任何兼容性问题也是很好的。我以前对主节点的 m5 EC2 实例有一些问题。要点是进行模拟升级，就像它是生产集群一样。

一旦它被创造出来。向它部署一个基本的 web 服务。我们将使用它来建立一些基线测试结果。

# 测试服务

在这一步中，我们将希望获得一些基准结果，以更好地确定它在集群升级期间是否受到影响，而不是调整一些参数(稍后将详细介绍)。

为此我通常使用 [Apache 基准](https://httpd.apache.org/docs/2.4/programs/ab.html)

`ab -n 100000 -c 10 -l http://<URL to the service>`

这将向服务抛出 100000 个并发数为 10 的请求。让它运行并将结果复制下来。我们会将它们与集群升级过程中的结果进行比较，以确保没有服务受到影响，这在生产升级过程中至关重要。

# 试运行升级

# 集群升级模拟

```
kops edit cluster 
# Update the version number 
kops update cluster steven.k8s.com
```

# 集群升级(无间隔)

```
kops update cluster steven.k8s.com --yes 
kops rolling-update cluster 
kops rolling-update cluster --yes
```

发出上述命令后，群集将启动滚动更新过程，每次关闭 1 个节点。这是我们一直在等待的时刻。我们想知道测试服务在这个过程中是否继续运行。现在，当升级正在进行中，让我们扔一些流量

`ab -n 100000 -c 10 -l http://<URL to the service>`

结果应该与基线结果相似，如果不相似，我们可以考虑添加一个时间间隔，以便在节点创建/终止期间有更多的时间。这肯定有助于减少中断，但会延长整个过程。为了确保稳定性，我很乐意做出这样的权衡。

我们可以使用以下命令来添加时间间隔。

# 集群升级(每 10 分钟一次)

```
kops update cluster steven.k8s.com --yes 
kops rolling-update cluster 
kops rolling-update cluster --master-interval=10m --node interval=10m --yes -v 10
```

# 最终检查

一旦我们确信升级不会影响现有的服务，我们就接近真正的升级了。这只是一个小清单，以确保所有重要的东西在升级后仍然工作

*   库贝法兰绒
*   kube-dns
*   kube-DNS-自动缩放器
*   dns 控制器
*   kube-apiserver
*   kube-控制器-管理器
*   kube 代理 ip
*   etcd-服务器-事件 x 3
*   etcd-服务器-ip x 3

如果所有的舱都正常工作。我们应该为真正的行动做好准备。

# 生产集群上的实际升级

升级持续时间的计算方法是节点数量 x 两者之间的时间间隔。更可靠的时间间隔是 10 分钟，因此如果我们有一个由 50 个节点组成的集群，所需时间将是 500 分钟。一定要安排那么多时间。

一旦时间确定，测试步骤通过，我们就准备好了！

# 常见问题

升级期间群集不验证当一个或多个节点拒绝进入就绪状态，或者在`kube-system`命名空间中有任何未就绪的 pod 时，会发生这种情况。问题解决后，我们可以使用此命令进行验证。

`kops validate cluster`

在`kube-system`内部有挥之不去的失败吊舱而不被注意是很常见的。删除失败的吊舱甚至部署将修复它。

# 升级受阻

`kops`的滚动升级工作方式是一次取下一个节点，用一个具有较新版本的新节点替换。这种方法通常在特定部署需要至少一个正在运行(可用)的 pod 来确保基本可用性，并且仅设置一个副本之前有效。用这样一个正在运行的 pod 移除一个节点将导致`availability budget`冲突，导致驱逐失败，从而导致集群升级停止。要解决这个问题，只需删除有问题的 pod，升级将继续逐出节点。

我们通常在各种名称空间的 nginx 入口控制器上看到这种类型的违规。使用详细日志选项升级集群将有助于识别这种类型的违规(请注意`-v 10`选项)

`kops rolling-update cluster --master-interval=10m --node-interval=10m --yes -v 10`

如果升级因其他未知原因停止，我建议使用以下命令清空节点

`kubectl drain --ignore-daemonsets --delete-local-data --force <NODE>`

然后，您可以通过 AWS EC2 控制台手动终止节点。

# 结束语

根据我对 kops 的经验，由于滚动更新的本质，升级通常是非常安全的。如果您希望在进程中恢复到原始版本，您可以简单地终止进程(`kops rolling-update cluster`)并编辑版本，然后重启`rolling-update`命令。我只有感谢`kops`团队，感谢所有的顺利体验。正因为如此，Buffer 作为一家公司能够对 Kubernetes 有如此高的信心，并且已经将我们的大部分服务迁移到了 Kubernetes。

*原载于*[*gist.github.com*](https://gist.github.com/stevenc81/e8d86d68a2aff69b7268938fa1f711fe)*。*