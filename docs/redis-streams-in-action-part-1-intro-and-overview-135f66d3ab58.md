# Redis Streams 在运行:第 1 部分(介绍和概述)

> 原文：<https://itnext.io/redis-streams-in-action-part-1-intro-and-overview-135f66d3ab58?source=collection_archive---------1----------------------->

## 这一部分探讨了用例、动机，并提供了解决方案中使用的 Redis 特性的高级概述。

欢迎阅读这一系列博客文章，这些文章通过一个实际例子的帮助涵盖了 [Redis 流](https://redis.io/topics/streams-intro)。我们将使用一个示例应用程序来实时搜索和查询 Twitter 数据。[redi research](https://redisearch.io/)和[redi Streams](https://redis.io/topics/streams-intro)是这个解决方案的支柱，它由几个合作组件组成，我们将在一篇专门的博客文章中介绍每一个组件。

*   第 1 部分—本博客
*   [第二部分](https://abhishek1987.medium.com/redis-streams-in-action-part-2-tweets-consumer-app-674fd3b45f6f)
*   [第三部分](/redis-streams-in-action-part-3-tweets-processor-app-254161838973)
*   [第四部分](/redis-streams-in-action-part-4-serverless-monitoring-service-faef52ee58db)

> *本 GitHub 回购中有代码—*[*https://github.com/abhirockzz/redis-streams-in-action*](https://github.com/abhirockzz/redis-streams-in-action)

这是第一部分，探讨了用例、动机，并提供了解决方案中使用的 Redis 特性的高级概述。

# 解决方案架构

![](img/2dcf1c087cde505b88484082efa002e5.png)

高层建筑

用例相对简单。作为最终目标，我们希望有一个服务，让我们能够根据一些标准，如标签，用户，位置等搜索推文。当然，对此已有解决方案。本博客系列中提供的是一个示例场景，可以应用于类似的问题。

以下是各个组件的摘要:

1.  Twitter 流消费者:一个 Rust 应用程序，用于消费 Twitter 流数据，并将它们传递给 Redis 流。我将在 [Azure 容器实例](https://docs.microsoft.com/azure/container-instances/container-instances-overview?WT.mc_id=data-17927-abhishgu)中演示如何将它作为 Docker 容器运行
2.  Tweets 处理器:来自 Redis 流的 tweets 由 Java 应用程序处理——这也将使用 Azure 容器实例进行部署(和扩展)。
3.  监控服务:最后一部分是一个 [Go](https://golang.org/) 应用程序，用于监控 tweets 处理器服务的进度，并确保任何失败的记录都得到重新处理。这是一个无服务器组件，将被部署到 Azure Functions 中，你可以基于定时器触发器运行它，并且只需为它运行的时间付费。

我已经使用了一些 Azure 服务(包括支持 Redis 模块如`RediSearch`、`RedisTimeSeries`和 Redis Bloom 的 Azure Cache 的[企业层)来运行解决方案的不同部分，但是您可以稍微调整一下指令，并根据您的环境应用它们，例如，您可以使用 Docker 在本地运行一切！尽管各个服务是用不同的编程语言编写的，但同样的概念也适用(在 Redis 流、`RediSearch`、可伸缩性等方面)。)并可以用您选择的语言实现。](https://techcommunity.microsoft.com/t5/apps-on-azure/delivering-an-even-better-redis-experience-on-azure/ba-p/2176217?WT.mc_id=data-17927-abhishgu)

# “规模需求”

我写了一篇博文 [RediSearch in Action](https://redislabs.com/blog/redisearch-in-action/) ，涵盖了相同的用例，即如何实现一组实时消费推文的应用程序，在 [RediSearch](https://redisearch.io/) 中索引它们，并使用 REST API 查询它们。然而，这里提出的解决方案是在 Redis Streams 和其他组件的帮助下实现的，目的是使架构具有可伸缩性和容错性。在这个具体的例子中，它是处理大量推文的能力，但同样的想法可以扩展/应用到处理高速数据的其他用例，如物联网、日志分析等。这类问题得益于一种架构，在这种架构中，您可以横向扩展您的应用程序来处理不断增长的数据量。通常，这涉及到引入一个消息传递系统作为生产者和消费者之间的缓冲。因为这是一个常见的需求，而且问题空间也很好理解，所以在分布式消息传递领域有很多成熟的解决方案，从 JMS (Java 消息传递服务)、 [Apache Kafka](https://kafka.apache.org/) 、 [RabbitMQ](https://www.rabbitmq.com/) 、 [NATS](https://nats.io/) ，当然还有 Redis。

# Redis 中的许多选项！

不过，Redis 有一些独特之处。从消息传递的角度来看，Redis 非常灵活，因为它提供了多种选项来支持不同的范例，因此适用于广泛的用例。它的特性包括[发布订阅](https://redis.io/topics/pubsub)、[列表](https://redis.io/topics/data-types-intro#redis-lists)(工作队列方法)和 Redis 流。因为这个博客系列主要关注 Redis 流，所以在继续之前，我将提供一个其他可能性的快速概览。

*   Pub-Sub:它遵循一个基于广播的范例，其中多个接收者可以消费发送到特定*通道*的消息。生产者和消费者是完全分离的，但是请注意，没有*消息持久性的概念，也就是说，如果消费者应用程序没有启动并运行，当它稍后重新启动时，它不会获得那些消息。*
*   列表:它们允许我们采用基于工作队列的方法，可以在工作应用程序之间分配负载。消息一旦被消费就被删除。它可以使用`RPOPLPUSH`(和`BRPOPLPUSH`)提供一定程度的容错和可靠性

# Redis 流

Redis Streams 是在 Redis 5.0 中引入的，它提供了最好的发布/订阅和列表，以及可靠的消息传递、消息重放的持久性、用于负载平衡的使用者组、用于监控的待定条目列表等等！它的不同之处在于它是一个`append-only log`数据结构。简而言之，生产者可以添加记录(使用`XADD`)，消费者可以订阅到达流的新项目(使用`XREAD`)。支持范围查询(`XRANGE`等。)由于消费者团体，一组应用程序可以分配处理负载(`XREADGROUP`)并可以监控其状态(`XPENDING`等)。

因为 Redis 的魅力在于它强大的命令系统，所以让我们来看一下 Redis Streams 的一些命令，为了更容易理解，这些命令按照功能进行了分组:

**添加条目**

只有一种方法可以将消息添加到 Redis 流中。 [XADD](https://redis.io/commands/XADD) 将指定的流条目追加到流中指定的键处。如果键不存在，作为运行该命令的副作用，将使用流值创建键。

**读取条目**

*   [XRANGE](https://redis.io/commands/xrange) 返回匹配给定 ID 范围的流条目(`-`和`+`特殊 ID 分别表示流内可能的最小 ID 和最大 ID)
*   [XREVRANGE](https://redis.io/commands/XREVRANGE) 与`XRANGE`完全相同，但不同之处在于以相反的顺序返回条目(先使用结束 ID，后使用开始 ID)
*   [xdread](https://redis.io/commands/XREAD)从一个或多个流中读取数据，只返回 ID 大于调用者报告的最后接收到的 ID 的条目。
*   [XREADGROUP](https://redis.io/commands/XREADGROUP) 是`XREAD`命令的特殊版本，支持消费者群体。您可以创建客户端组，使用到达给定流的消息的不同部分

**管理 Redis 流**

*   [XACK](https://redis.io/commands/XACK) 从流使用者组的待定条目列表(`PEL`)中删除一条或多条消息。
*   [XGROUP](https://redis.io/commands/XGROUP) 用于管理与 Redis 流相关的消费者组。
*   [XPENDING](https://redis.io/commands/XPENDING) 用于检查待处理消息列表，以观察和了解流消费者群体的情况。
*   [XCLAIM](https://redis.io/commands/XCLAIM) 用于获取消息的所有权并继续处理。
*   XAUTOCLIAM 转移符合指定标准的挂起流条目的所有权。从概念上讲，`XAUTOCLAIM`相当于先调用`XPENDING`，再调用`XCLAIM`

**删除**

*   [XDEL](https://redis.io/commands/xdel) 从流中删除指定的条目，并返回删除条目的数量，如果某些 id 不存在，该数量可能不同于传递给命令的 id 数量。
*   [XTRIM](https://redis.io/commands/xtrim) 如果需要，通过驱逐旧条目(id 较低的条目)来整理流。

> *详细的话，我强烈推荐阅读*[*《Redis 流简介》*](https://redis.io/topics/streams-intro) *(来自 Redis 官方文档)。*

# 再搜索呢？

Redis 有一套通用的数据结构，从简单的字符串到强大的抽象，如 Redis 流。原生数据类型可以带您走很长一段路，但是有些用例可能需要一个解决方法。一个例子是 Redis 中使用二级索引的要求，以便超越基于关键字的搜索/查找，获得更丰富的查询功能。虽然你可以使用[有序集合、列表等等来完成工作](https://redis.io/topics/indexes)，但是你需要考虑一些权衡。

得益于一流的二级索引引擎，`RediSearch`作为 Redis 模块提供了灵活的搜索功能。它的一些关键特性包括全文搜索、自动完成和地理索引。还有许多其他特性，它们的详细探讨超出了本博客系列的范围。我强烈建议您通过[文档](https://oss.redislabs.com/redisearch/)进行进一步探索。现在，这里是一些`RediSearch`命令的快速概述。您将在后续的博客文章中看到它们的实际应用。

两个最重要的命令包括创建索引和执行搜索查询:

*   [英尺。CREATE](https://oss.redislabs.com/redisearch/Commands/#ftcreate) 用于创建具有给定模式和相关细节的索引。
*   [英尺。SEARCH](https://oss.redislabs.com/redisearch/Commands/#ftsearch) 使用文本查询搜索索引，返回文档或 id。

您可以对索引执行其他操作:

*   英尺。DROPINDEX 删除索引。请注意，默认情况下，它不会删除与索引相关联的文档哈希
*   [英尺。INFO](https://oss.redislabs.com/redisearch/Commands/#ftinfo) 返回索引的信息和统计数据，比如文档数量、不同术语的数量等等。
*   [英尺。ALTER SCHEMA ADD](https://oss.redislabs.com/redisearch/Commands/#ftalter_schema_add) 向索引中添加一个新字段。这将导致将来的文档更新在索引和重新索引现有文档时使用新字段。

要使用自动完成功能，您可以使用“建议”:

*   [英尺 SUGADD](https://oss.redislabs.com/redisearch/Commands/#ftsugadd) 将建议字符串添加到自动完成的建议字典中。
*   [英尺 SUGGET](https://oss.redislabs.com/redisearch/Commands/#ftsugget) 获取前缀的补全建议。

`RediSearch`支持同义词，这是一种由一组组组成的数据结构，每个组都包含同义词。[英尺 SYNUPDATE](https://oss.redislabs.com/redisearch/Commands/#ftsynupdate) 可用于创建或更新带有附加术语的同义词组。

如果你想要查询拼写检查纠正(类似于“你的意思是”功能)，你可以使用 [FT。SPELLCHECK](https://oss.redislabs.com/redisearch/Commands/#ftspellcheck) 对查询执行拼写纠正，返回拼写错误的术语的建议。

字典是一组术语。词典可用于修改 RediSearch 的查询拼写更正行为，方法是在潜在的拼写更正建议中包含或排除其内容。你可以用 [FT。口述](https://oss.redislabs.com/redisearch/Commands/#ftdictadd)和 [FT。DICTDEL](https://oss.redislabs.com/redisearch/Commands/#ftdictdel) 分别添加和删除术语。

这部分到此为止！

# 展望未来…

正如我提到的，这只是一个介绍。在接下来的三篇博文中，您将了解用于构建解决方案的各个组件的细节。您将在 Azure 上部署、运行和验证它们，并浏览代码以更好地理解“幕后”发生的事情。敬请期待！