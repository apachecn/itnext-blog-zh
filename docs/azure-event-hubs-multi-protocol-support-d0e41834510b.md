# Azure 事件中心多协议支持

> 原文：<https://itnext.io/azure-event-hubs-multi-protocol-support-d0e41834510b?source=collection_archive---------8----------------------->

[Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/?WT.mc_id=medium-blog-abhishgu) 是一个完全托管的平台即服务(PaaS ),作为一个数据流平台和事件接收服务，能够每秒接收和处理数百万个事件。客户端应用程序可以利用以下协议与服务交互:AMQP、 [Kafka](https://kafka.apache.org/) 和 HTTPS。

![](img/13ba752521e23cb7306877065ee9064a.png)

这篇博文将简要介绍活动中心的多协议支持，以及它的重要性。第二部分将给出一个简单而实用的例子来演示这一点

# 支持的协议

让我们很快过一遍！

## AMQP

[高级消息队列协议(AMQP) 1.0](https://www.amqp.org/) 是 Azure 事件中心的主要协议。这是一个`OASIS`标准，它定义了在系统之间异步、安全、可靠地传输消息的协议。要使用 Azure Event Hubs，你可以使用任何实现 AMQP 协议的特定语言客户端软件开发套件，包括`.NET`、`Java`、`Python`、`Go`、`Nodejs`和`C`

> *这些都是 SDK*[*在 GitHub 上开源*](https://github.com/Azure/azure-event-hubs)

要深入了解 AMQP 协议如何在活动中心使用[，请参考文档](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-amqp-protocol-guide?toc=https://docs.microsoft.com/en-us/azure/event-hubs/TOC.json&bc=https://docs.microsoft.com/en-us/azure/bread/toc.json&WT.mc_id=medium-blog-abhishgu)

## 阿帕奇卡夫卡

Event Hubs 提供了一个 Kafka 兼容的端点，您现有的基于 Kafka 的应用程序可以使用它作为运行您自己的 Kafka 集群的替代方法。这消除了设置 Kafka 和 Zookeeper 集群以及持续基础设施维护的需要。事件中心和 Kafka 在概念上是相同的，即 Kafka 集群对应于事件中心名称空间，Kafka 主题相当于事件中心等。详情[请参考文档](https://docs.microsoft.com/azure/event-hubs/event-hubs-for-kafka-ecosystem-overview?WT.mc_id=medium-blog-abhishgu#kafka-and-event-hub-conceptual-mapping)

您可以在任何编程语言中利用现有的 Kafka 客户端。我鼓励您查看 GitHub 上的[快速入门和教程，它们展示了在`Java`、`Node`、`dotnet`、`Python`、`Go`等地 Kafka 客户端中使用事件中心的情况。](https://github.com/Azure/azure-event-hubs-for-kafka)

## HTTPS

Event hubs 还提供了一个可以被任何 HTTP 客户端利用的 REST API。一些操作包括:

*   [发送数据](https://docs.microsoft.com/rest/api/eventhub/send-event?WT.mc_id=medium-blog-abhishgu)
*   [向特定分区发送数据](https://docs.microsoft.com/rest/api/eventhub/send-partition-event?WT.mc_id=medium-blog-abhishgu)
*   [批量发送数据](https://docs.microsoft.com/rest/api/eventhub/send-batch-events?WT.mc_id=medium-blog-abhishgu)

# 价值主张

这种多协议功能提供了很大的灵活性，您可以将它们结合起来用于特定的用例。这里有几个例子:

*   编写 Kafka endpoint 并使用基于[触发器的集成](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-hubs?WT.mc_id=medium-blog-abhishgu)构建一个具有 [Azure 功能](https://docs.microsoft.com/azure/azure-functions/?WT.mc_id=medium-blog-abhishgu)的无服务器消费者(这个场景将在后续的博客文章中介绍)
*   使用`HTTPS`写，使用`Kafka`读——例如，您的前端应用程序可以使用 HTTPs 发送点击流事件，您可以使用 Kafka 客户端处理这些事件
*   使用事件中心 SDK(基于 AMQP)发送事件，然后使用 Kafka 协议读取

既然可以混合和匹配协议，那么生产端和消费端的应用程序如何理解发送和接收的消息呢？这里已经介绍了一些很棒的[例子和最佳实践](https://docs.microsoft.com/azure/event-hubs/event-hubs-exchange-events-different-protocols?WT.mc_id=medium-blog-abhishgu)，但这里是 Tl；DR — *这是因为所有的协议客户端(卡夫卡、AMQP、HTTPS)都将消息视为原始字节*。这些字节使用特定的协议通过网络发送，即使使用不同的协议，消费者端也会收到相同的字节。如果生产或消费应用程序使用特定的格式，例如 HTTP 协议的 JSON 和基于 Java 的 Kafka 消费者客户端中的 Java POJO，则由客户端应用程序将字节转换为它想要使用的格式。

正如我所承诺的，下一篇博文将会讨论使用事件中心和 Azure 函数的无服务器处理。敬请期待！