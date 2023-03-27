# DC/操作系统代理节点维护

> 原文：<https://itnext.io/dc-os-agent-node-maintenance-e95355e6ef51?source=collection_archive---------5----------------------->

![](img/3df6544201eb686002620eca9f74212a.png)

资料来源:toothpastefordinner.com

提供基础设施的高可用性(HA)是我工作的一个主要方面，我对此非常自豪。我关心基础设施的可用性，因为这反映了我的架构设计和我作为工程师的知识，这是我努力获得的。如果我希望开发人员使用我的平台来运行他们的服务，他们需要知道底层基础设施将始终可用，并且不会成为他们正常运行时间 SLA 的瓶颈。当然，这也要求用户将他们的服务设计成 HA，但是如果底层架构不是 HA，他们的服务怎么可能是 HA 呢？

在这篇文章中，我想讨论一下我们目前处理 DC/操作系统代理节点维护的方式。我将解释我们在维护期间为确保用户的高可用性而采取的步骤，解释我们用来处理维护的步骤和工具，并讨论我们在过去几年中在我们的环境中学到的一些东西。

电脑很烂，而且很笨。为了保持基础设施平稳安全地运行，我们必须预计到在某个时间点至少会出现计算机故障和/或维护。这个已知的事实并没有改变能够为服务提供 HA 和高 SLA 的需求。在当今的计算和软件世界中，我们比以往任何时候都有更多的控制和机会来设计架构良好的系统，以实现故障转移、恢复和高可用性。在我看来，如果它不能允许这些事情，那么它就不应该运行。

我想花一点时间来讨论这篇文章背后的基础架构和平台，以及我们用来管理基础架构的一些工具。

# **站台**

如果你没有从标题中猜到的话——是的，我们将要讨论[DC/操作系统](https://docs.mesosphere.com/1.11/overview/)。

> “DC/OS 是基于 Apache Mesos 分布式系统内核的分布式操作系统。它支持管理多台机器，就像它们是一台计算机一样。它自动化了资源管理，安排了进程放置，促进了进程间的通信，并简化了分布式服务的安装和管理。”—docs.mesosphere.com

无需深入解释 DC/操作系统架构，只需注意这是为我们提供管理服务框架的主要平台。我们还选择在 AWS 中运行这个架构，以利用使云计算如此强大和令人敬畏的东西。我们还应该注意到，我们的大部分服务都在[码头](https://www.docker.com)运行，这也不需要解释很多好处。

这里我应该提到的另一个关键点是，我们在 DC 操作系统上大量使用卡夫卡。我们在自己的环境中部署了多个 Kafka 集群，这些集群最能代表不同的应用环境(开发、QA、测试等)。每个 Kafka 集群也可能是不同的版本或风格(如合流 Kafka)，因此我们必须始终确保我们不会一次失去来自同一个集群的太多代理，否则我们会遭受数据丢失。

# **工装**

作为基础设施工程师，我们实践[基础设施即代码](https://www.martinfowler.com/bliki/InfrastructureAsCode.html)方法，并在任何可能的地方应用它们。我们对基础设施管理和 DC/操作系统的各个方面进行版本控制、编码和自动化。这使得我们能够快速有效地为一个非常忙碌的工程师组成的极小团队工作。实现这一目标的一些工具值得一提:

[Packer](http://www.packer.io) —我们使用 Packer 来构建我们的机器映像(AMIs)并将我们所有的环境细节应用于它们(监控、日志、Docker 版本等……)。如果有兴趣了解更多，请看我以前的帖子，关于我们如何使用 [Packer 来构建我们的 DC/操作系统代理 AMIs](https://medium.com/@wbassler23/dc-os-agent-ami-using-packer-and-ansible-8f6184a30e21) 。

[Terraform](http://www.terraform.io) —我们使用 Terraform 创建和管理我们基础设施的所有方面(EC2 实例、SG、角色等)。当然，我们把上面用 Packer 创建的机器图像合并到这里。借助 Terraform，我们能够自动完成 DC/操作系统集群的初始创建，并将其用于扩展和修补。大部分魔力来自定制脚本和用户数据的 [Terraform 模板](https://www.terraform.io/docs/providers/template/index.html)功能。

Ansible——我们非常广泛地使用 ansi ble，不仅自动化我们基础设施的不同方面，还通过特别的命令管理基础设施。我们将行动手册融入到构建我们的机器映像和 ansible-vault 中，以保护敏感信息。

# **流程**

既然我们已经讨论了用于执行维护的工具，我想概述一下我们的流程和方法。

由于我们使用基础设施即代码、AWS 和高度分布式系统，我们用全新的 DC/操作系统代理完全取代了我们的代理。本质上，我们所做的是构建一个全新的机器映像，终止当前的 DC/操作系统代理，然后用新创建的 AMI 重建/替换该代理。当然，在终止实例之前，我们必须做一些事情(下面会详细介绍)，但这是我们处理维护的高级方式。当您看到下面执行的步骤时，这可能会更有意义。

我们利用能够使用我们的 DC/操作系统代理作为短暂的生命，而不是长期永远活着的实例。我们对待它们就好像我们知道我们很可能会因为文件系统损坏、内核崩溃、有人在 AWS 中捣鬼等等而失去代理一样。我们不会浪费时间对故障节点进行故障排除(99%的时间)，我们也不想担心如果我们执行“yum update -y”并重新启动可能会出现什么兼容性问题。我们用经过全面测试的机器映像进行替换，以避免将来发生灾难。记住我之前说的，电脑很烂，而且很蠢。他们会在某个时候让你失望。

如上所述，在终止实例之前，我们必须做几件事来确保我们不会影响集群上运行的服务。尽管我们已经为故障转移设计了我们的服务，但是我们必须安全地从代理节点中排出当前运行的服务。我们不想在维护时冒险造成停机。DC/操作系统实际上有一种执行[节点维护](https://docs.mesosphere.com/1.11/administering-clusters/update-a-node/)的方法，在这种方法中，您可以从主节点取消代理的注册，直到进一步通知或给定的时间。我们实际上更进了一步，我将在下面展示。此外，因为我们也运行几个 Kafka 集群，所以我们还必须替换代理，并让其他可用的代理替代它们，以避免数据丢失。

# **步骤**

现在我们对这个过程有了更好的理解，让我们来看一下在节点维护期间执行的步骤。下面的步骤代表了我们为每个代理所经历的步骤:

1.  从 Amazon 和供应商提供的最新最好的 AMI 构建一个新的机器映像。我们用 CentOS 7。你可以在我之前的文章[中再次看到这一步。](https://medium.com/@wbassler23/dc-os-agent-ami-using-packer-and-ansible-8f6184a30e21)这一步只做一次，所有代理节点共享。
2.  在您的 Terraform 代码中，修改用于管理特定 DC/操作系统代理的 ec2 资源，并用在第一步中创建的新 ami-id 替换旧 ami-id。Terraform 文档示例。

```
resource "aws_instance" "dcos_agent_node" {
  monitoring    = true
  ami           = "NEW_AMI_ID_HERE"
  instance_type = "${var.instance_types["dcos_agent"]}"...
...
...}
```

3.在 AWS 中终止 DC/操作系统代理之前，我们还必须做其他事情。我们必须耗尽节点上所有正在运行的服务，以确保安全的故障转移。

在我们的环境中，我们首先必须检查是否有任何 Kafka 经纪人在运行，如果有[我们必须“替换”他们](https://docs.mesosphere.com/services/confluent-kafka/2.3.0-4.0.0e/troubleshooting/#replacing-a-permanently-failed-node)。您可以使用[DC/操作系统命令行界面](https://docs.mesosphere.com/1.11/cli/)轻松做到这一点，您还可以深入到 DC/操作系统用户界面的节点选项卡，找到代理属于哪个 Kafka 集群。

```
dcos kafka --name=<KAFKA_CLUSTER_NAME> pod replace kafka-0# For older versions
dcos kafka --name=<KAFKA_CLUSTER_NAME> broker replace 0
```

这将停止在该节点上运行的当前代理，然后将新代理重新部署到不同的代理节点，并开始同步指定 Kafka 集群的数据。

其次，我们必须清空在那里运行的所有其他服务，并从集群中删除代理。我之前提到过，我们的大部分服务都在 Docker 中运行。因为我们的 Docker 版本当前需要 Docker 守护进程来运行容器，而 Marathon 需要守护进程运行来部署服务，所以我们向 docker systemd 服务发送一个 kill 信号并停止它。

```
sudo systemctl kill -s SIGUSR1 docker && sudo systemctl stop docker
```

这将立即终止在节点上运行的 docker 容器，Marathon 将开始在集群中的其他地方重新部署它们。此外，因为 docker 守护进程现在在节点上停止了，所以我们不必担心马拉松式地尝试在我们正在工作的当前代理节点上重新部署它们！

我们要做的最后一步是从 DC/操作系统群集中完全删除代理节点。幸运的是， [Mesosphere 为我们提供了一个类似的命令，它将为我们处理上面提到的](https://docs.mesosphere.com/1.11/administering-clusters/delete-node/)。

```
sudo systemctl kill -s SIGUSR1 dcos-mesos-slave && sudo systemctl stop dcos-mesos-slave
```

您将很快看到，如果不是立即看到，您的代理节点在 DC/OS UI 和 Mesos UI 中完全从集群中消失。

我们已经想出了一个很酷的事情，那就是如何杀死和停止 Docker 守护进程和 dcos-mesos-slave 服务，这些服务是为 SSM 自动伸缩组后面的实例提供的。这使我们能够自动化向 DC/操作系统集群添加新实例以及在节点终止时安全清空节点的整个过程。在以后的文章中会有更多的介绍！

4.现在，我们登录 AWS 并启动 DC/操作系统代理实例终止。

5.一旦实例完全终止，现在可以使用我们在步骤 1 中构建的新 AMI，用上面的 terraform 代码替换/重新部署该实例。

```
terraform apply
```

大约 2-3 分钟后，我们将看到我们的 DC/操作系统代理向群集重新注册，并以全新的操作系统重新出现在 DC/操作系统用户界面中，它将开始接受预订请求。第 5 步需要一点编码和一些定制，以使节点自动[正确添加回](https://docs.mesosphere.com/1.11/installing/ent/custom/advanced/)，因为我们运行定制设置，如[属性标签](https://docs.mesosphere.com/1.9/installing/oss/faq/#q-how-to-add-mesos-attributes-to-nodes-to-use-marathon-constraints)和[禁用 CFS](https://docs.mesosphere.com/1.11/monitoring/debugging/slow-docker/) 。也许在某个时候，关于我们如何使用 Terraform 来管理我们的 DC/操作系统集群的不同帖子会更合适。也许有一天，我们也会将整个过程自动化。

让我们能够在进行节点维护的同时为用户提供高可用性需要一些反复试验。我们现在能够在很短的时间内完全升级大约 20 个 DC/操作系统代理节点的集群，而对服务没有任何影响。这很重要，因为它允许我们在白天进行维护，没有人知道发生了什么。用户能够不受影响地完成他们的测试，我们也不必在非工作时间熬夜做测试。如果您的环境是相似的，那么您可以采取我上面介绍的相同步骤，并将其应用到您的节点维护中，以确保最高的可用性。