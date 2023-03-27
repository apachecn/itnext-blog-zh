# 创建 Kafka 主题运行 Kafka Brokers

> 原文：<https://itnext.io/creating-a-kafka-topic-running-kafka-brokers-d9aab5530fb2?source=collection_archive---------0----------------------->

![](img/54be031dbaf7be5b6f3edc0bfbd5b7bc.png)

从 1996 年开始编程，从 2001 年开始开发 Java 的 Raúl Estrada 在这篇文章中学习如何使用 Kafka brokers 运行 Kafka 主题。他是 BEA Systems 和 Oracle Inc .的企业架构师，但他也喜欢 web、移动和游戏编程。劳尔是自由软件的支持者，喜欢尝试新的技术、框架、语言和方法。

服务器背后真正的艺术在于它的配置。本文将研究如何在独立模式下处理 Kafka 代理的基本配置。有两种类型的配置:独立和集群。当以集群模式运行复制并且所有主题都被正确分区时，Kafka 的真正力量被释放出来。

集群模式有两个主要优势:并行性和冗余性。并行性是指在集群成员之间同时运行任务的能力。冗余保证了当 Kafka 节点关闭时，集群是安全的，并且可以从其他运行的节点访问。

本文展示了如何在我们的本地机器上配置一个包含多个节点的集群，尽管在实践中，让多台包含多个节点的机器共享集群总是更好。

为了开始这个练习，让我们使用合流平台开源版本。汇合平台在 Docker 镜像中也是可用的，但是在这里我们将把它安装在本地。在[https://www.confluent.io/download/](https://www.confluent.io/download/)打开合流平台下载页面。

Confluent Platform 的当前版本是 5.0.0 稳定版。请记住，由于 Kafka 核心运行在 Scala 上，因此有两个版本:Scala 2.11 和 Scala 2.12。

我们可以从我们的桌面目录运行融合平台，但是让我们对 Linux 用户使用`/opt/`，对 macOS 用户使用`/usr/local`。

要安装合流平台，在目录中提取下载的文件`confluent-5.0.0-2.11.tar.gz`，如下所示:

```
**> tar xzf confluent-5.0.0-2.11.tar.gz**
```

现在，转到合流平台安装目录，从现在开始称为`<confluent-path>`。

代理是一个服务器实例。服务器(或代理)实际上是运行在操作系统中的一个进程，并根据其配置文件启动。

Confluent 的人员为我们提供了一个标准代理配置的模板。这个名为`server.properties`的文件位于 Kafka 安装目录的`config`子目录中:

1.在`<confluent-path>`里面，做一个带有名字标记的目录。

2.对于我们想要运行的每个 Kafka 代理(服务器),我们需要制作一个配置文件模板的副本，并相应地对其进行重命名。在这个例子中，我们的集群将被称为`mark`:

```
**> cp config/server.properties <confluent-path>/mark/mark-1.properties****> cp config/server.properties <confluent-path>/mark/mark-2.properties**
```

3.相应地修改每个属性文件。如果文件名为`mark-1`，则`broker.id`应为`1`。然后，指定服务器将运行的端口；推荐的是`mark-1`的`9093`和`mark-2`的`9094`。请注意，模板中没有设置 port 属性，因此添加以下行。最后，指定 Kafka 日志的位置(Kafka 日志是存储所有 Kafka 代理操作的特定档案)；在这种情况下，使用`/tmp`目录。在这里，写权限的问题是很常见的。不要忘记向用户授予写和执行权限，以便通过日志目录执行这些过程，如下例所示:

在`mark-1.properties`中，设置以下内容:

```
**broker.id=1****port=9093****log.dirs=/tmp/mark-1-logs**
```

在`mark-2.properties`中，设置以下内容:

```
**broker.id=2****port=9094****log.dirs=/tmp/mark-2-logs**
```

4.使用`kafka-server-start`命令启动 Kafka 代理，并将相应的配置文件作为参数传递。不要忘记合流平台必须已经在运行，并且端口不应该被另一个进程使用。按如下方式启动 Kafka 代理:

```
**> <confluent-path>/bin/kafka-server-start <confluent-****path>/mark/mark-1.properties &**
```

在另一个命令行窗口中，运行以下命令:

```
**> <confluent-path>/bin/kafka-server-start <confluent-****path>/mark/mark-2.properties &**
```

不要忘记结尾的`&`是指定你想要回你的命令行。如果您想查看代理的输出，建议在各自的命令行窗口中分别运行每个命令。

请记住，属性文件包含服务器配置，位于`config`目录中的`server.properties`文件只是一个模板。

现在有两个代理，`mark-1`和`mark-2`，运行在同一个集群中的同一台机器上。

请记住，没有愚蠢的问题，如下例所示:

**问**:每个经纪人如何知道自己属于哪个集群？

**答**:代理知道它们属于同一个集群，因为在配置中，它们都指向同一个 Zookeeper 集群。

问:同一个集群中的每个代理与其他代理有什么不同？

**答**:每个代理在集群中由在`broker.id`属性中指定的名称来标识。

**问**:如果没有指定端口号会怎么样？

**答**:如果未指定 port 属性，Zookeeper 将分配相同的端口号，并将覆盖数据。

**问**:如果没有指定日志目录会怎么样？

**A** :如果未指定`log.dir`，所有代理将写入相同的默认值`log.dir`。如果代理计划在不同的机器上运行，那么可以不指定 port 和`log.dir`属性(因为它们在相同的端口和日志文件中运行，但是在不同的机器上)。

问:我如何检查在我想要启动我的代理的端口上没有正在运行的进程？

**答**:有一个有用的命令，看看具体端口上运行的是什么进程；本例中的`9093`端口:

```
**> lsof -i :9093**
```

前面命令的输出如下所示:

```
**COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME****java 12529 admin 406u IPv6 0xc41a24baa4fedb11 0t0 TCP *:9093 (LISTEN)**
```

轮到您了:尝试在启动 Kafka 代理之前运行该命令，并在启动它们之后运行该命令以查看变化。此外，尝试在正在使用的端口上启动一个代理，看看它是如何失败的。

好吧，如果我想让我的集群在几台机器上运行呢？

要在不同的机器上但在同一个集群中运行 Kafka 节点，在配置文件中调整 Zookeeper 连接字符串；其默认值如下:

```
zookeeper.connect=localhost:2181
```

请记住，DNS 必须能够找到彼此的机器，并且它们之间没有网络安全限制。

只有当您在与 Zookeeper 相同的机器上运行 Kafka broker 时，Zookeeper connect 的默认值才是正确的。根据不同的架构，需要决定是否在同一台 Zookeeper 机器上运行一个代理。

要指定 Zookeeper 可以在其他机器上运行，请执行以下操作:

```
zookeeper.connect=localhost:2181, 192.168.0.2:2183, 192.168.0.3:2182
```

前一行指定 Zookeeper 运行在本地主机的端口`2181`，IP 地址为`192.168.0.2`的机器的端口`2183`，以及 IP 地址为`192.168.0.3`的机器的端口`2182`。Zookeeper 的默认端口是`2181`，所以通常它在那里运行。

轮到您了:作为一个练习，尝试使用关于 Zookeeper 集群的错误信息启动一个代理。另外，使用`lsof`命令，尝试在一个正在使用的端口上启动 Zookeeper。

如果您对配置有疑问，或者不清楚要更改什么值，那么`server.properties`模板(作为 Kafka 项目的一部分)是开源的，如下所示:

[https://github . com/Apache/Kafka/blob/trunk/config/server . properties](https://github.com/apache/kafka/blob/trunk/config/server.properties)

# **运行卡夫卡主题**

代理内部的权力是主题，即其中的队列。现在我们有两个经纪人在运行，让我们为他们创建一个 Kafka 主题。

像几乎所有现代基础设施项目一样，Kafka 有三种构建方式:通过命令行、通过编程和通过 web 控制台(在这种情况下是汇合控制中心)。Kafka 代理的管理(创建、修改和销毁)可以通过用大多数现代编程语言编写的程序来完成。如果不支持该语言，可以通过 Kafka REST API 进行管理。上一节展示了如何使用命令行构建代理。

是否可以只通过编程来管理(创建、修改或销毁)代理？不，我们也可以管理主题。也可以通过命令行创建主题。正如我们已经看到的，Kafka 预先构建了管理经纪人和管理主题的工具，我们将在接下来看到。

要在我们正在运行的集群中创建一个名为`amazingTopic`的主题，请使用以下命令:

```
**> <confluent-path>/bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic amazingTopic**
```

输出应该如下所示:

```
**Created topic amazingTopic**
```

这里使用了`kafka-topics`命令。用`--create`参数指定我们想要创建一个新的主题。`--topic`参数设置主题的名称，在本例中是`amazingTopic`。

你还记得并行和冗余这两个术语吗？嗯，`–-partitions`参数控制并行度，`--replication-factor`参数控制冗余度。

`--replication-factor`参数是基本的，因为它指定了主题将在集群的多少个服务器上复制(例如，运行)。另一方面，一个代理只能运行一个副本。

显然，如果指定的数量大于集群上正在运行的服务器的数量，就会导致错误(你不信？在您的环境中尝试一下)。错误将是这样的:

```
Error while executing topic command: replication factor: 3 larger than available brokers: 2[2018-09-01 07:13:31,350] ERROR org.apache.kafka.common.errors.InvalidReplicationFactorException: replication factor: 3 larger than available brokers: 2(kafka.admin.TopicCommand$)
```

要考虑的是，代理应该正在运行(不要害羞，在您的环境中测试所有这些理论)。

`--partitions`参数，顾名思义，表示主题将有多少个分区。数量决定了在消费者端可以实现的并行度。在进行集群微调时，这个参数非常重要。

最后，正如所料，`--zookeeper`参数指出了 Zookeeper 集群正在哪里运行。

创建主题时，代理日志中的输出如下所示:

```
[2018-09-01 07:05:53,910] INFO [ReplicaFetcherManager on broker 1] Removed fetcher for partitions amazingTopic-0 (kafka.server.ReplicaFetcherManager)[2018-09-01 07:05:53,950] INFO Completed load of log amazingTopic-0 with 1 log segments and log end offset 0 in 21 ms (kafka.log.Log)
```

简而言之，这条消息读起来像一个新的话题已经在我们的集群中诞生了。

我如何检查我的新的闪亮的话题？使用相同的命令:`kafka-topics`。

参数比`--create`多。要检查主题的状态，请运行带有`--list`参数的`kafka-topics`命令，如下所示:

```
**> <confluent-path>/bin/kafka-topics.sh --list --zookeeper localhost:2181**
```

我们知道，输出是主题列表，如下所示:

```
**amazingTopic**
```

该命令返回集群中所有正在运行的主题的名称列表。

如何获取某个主题的详细信息？使用相同的命令:`kafka-topics`。

对于特定主题，运行带有`--describe`参数的`kafka-topics`命令，如下所示:

```
**> <confluent-path>/bin/kafka-topics --describe --zookeeper localhost:2181 --topic amazingTopic**
```

命令输出如下所示:

```
**Topic:amazingTopic PartitionCount:1 ReplicationFactor:1 Configs: Topic: amazingTopic Partition: 0 Leader: 1 Replicas: 1 Isr: 1**
```

以下是对输出的简要说明:

`PartitionCount`:主题上的分区数量(并行度)

`ReplicationFactor`:主题的副本数量(冗余)

`Leader`:负责给定分区读写操作的节点

`Replicas`:复制该话题数据的经纪人列表；其中一些甚至可能已经死亡

`Isr`:当前为同步副本的节点列表

让我们创建一个具有多个副本的主题(例如，我们将在集群中运行更多的代理)；我们键入以下内容:

```
**> <confluent-path>/bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 2 --partitions 1 --topic redundantTopic**
```

输出如下所示:

```
**Created topic redundantTopic**
```

现在，调用带有`--describe`参数的`kafka-topics`命令来检查主题细节，如下所示:

```
**> <confluent-path>/bin/kafka-topics --describe --zookeeper localhost:2181 --topic redundantTopic****Topic:redundantTopic PartitionCount:1 ReplicationFactor:2 Configs:****Topic: redundantTopic Partition: 0 Leader: 1 Replicas: 1,2 Isr: 1,2**
```

可以看到，`Replicas`和`Isr`是同一个列表；我们推断所有节点都是同步的。

轮到您了:使用`kafka-topics`命令，尝试在死去的代理上创建复制的主题，并查看输出。此外，创建关于运行服务器的主题，然后杀死它们以查看结果。输出是您期望的吗？

所有这些通过命令行执行的命令都可以通过编程方式执行，或者通过合流控制中心 web 控制台执行。

*如果您觉得这篇文章很有趣，可以探索一下* [*Apache Kafka 快速入门指南*](https://www.amazon.com/Apache-Kafka-Quick-Start-Guide/dp/1788997824?utm_source=itnext&utm_medium=referral&utm_campaign=ThirdPartyPromotions) *使用最新的 Apache Kafka 2.0 构建高性能、健壮的数据流处理管道的同时，实时处理大量数据。* [*Apache Kafka 快速入门指南*](https://www.packtpub.com/big-data-and-business-intelligence/apache-kafka-quick-start-guide?utm_source=itnext&utm_medium=referral&utm_campaign=ThirdPartyPromotions) *将帮助您学习如何使用 Apache Kafka 高效处理分布式应用程序，并熟悉解决快速数据和处理管道中的日常问题。*