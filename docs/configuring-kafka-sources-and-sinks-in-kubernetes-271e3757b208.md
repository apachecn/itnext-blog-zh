# 在 Kubernetes 中配置 Kafka 源和接收器

> 原文：<https://itnext.io/configuring-kafka-sources-and-sinks-in-kubernetes-271e3757b208?source=collection_archive---------1----------------------->

将数据**流式传输到**Kafka 集群，以及将**从**Kafka 集群流式传输到其他地方，通常通过 Kafka connect 集群和相关配置来完成。

这通常是 Kafka 用户真正的痛点。它包括:

*   部署和运行 Kafka connect 集群
*   用 Java 更新和编译连接器
*   将 jar 上传到 Kafka connect 集群中的特定目录

![](img/04b464a28fe068939fd74bc37a27efa7.png)

卡夫卡连接来自[汇流博客](https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/)的数据管道

虽然 Kafka 连接器有一个很好的集合，但融合云上很少完全管理，这使得 Kafka 用户如果想要有效地管理他们的源和汇，就有很多工作要做。

在前两篇文章中，我展示了如何[向 Kafka](https://sebgoa.medium.com/sending-messages-to-kafka-cfb5a246f5eb) 生成消息，以及如何[从 Kafka](https://sebgoa.medium.com/consuming-kafka-messages-in-kubernetes-9e43050d6eb4)消费消息，现在我将向您展示如何以声明方式定义 Kafka 源和接收器，并在 Kubernetes 集群中管理它们。

## 配置 Kafka 源

从卡夫卡群集的观点来看，卡夫卡源是将一个事件产生为一个卡夫卡主题的东西。在 Knative 的帮助下，我们可以使用一个叫做 T0 的对象来配置一个 Kafka 源，我知道这非常令人困惑，对此我非常抱歉:)一切都是相对的，取决于你的立场。

要在 Kubernetes 集群中创建一个可寻址的端点，该端点将成为 Kafka 集群的消息源，您需要创建一个如下所示的对象:

```
apiVersion: eventing.knative.dev/v1alpha1
kind: KafkaSink
metadata:
 name: my-kafka-topic
spec:
 auth:
   secret:
     ref:
       name: kafkahackathon
 bootstrapServers:
 — pkc-456q9.us-east4.gcp.confluent.cloud:9092
 topic: hackathon
```

请注意融合云上的引导服务器和指定的主题。有了这个，你现在可以创建一个 AWS [SQS 源](https://github.com/triggermesh/aws-event-sources)直接发送消息到 Kafka，清单如下:

```
apiVersion: sources.triggermesh.io/v1alpha1
kind: AWSSQSSource
metadata: 
  name: samplequeue
spec: 
  arn: arn:aws:sqs:us-west-2:1234567890:triggermeshqueue
  credentials: 
    accessKeyID: 
      valueFromSecret: 
        name: awscreds 
        key: aws_access_key_id 
    secretAccessKey: 
      valueFromSecret: 
        name: awscreds 
        key: aws_secret_access_key 
  sink: 
    ref: 
      apiVersion: eventing.knative.dev/v1alpha1
      kind: KafkaSink
      name: my-kafka-topic
```

请注意，AWS SQS 由它的 ARN 表示，您可以通过包含 AWS API 密钥的 Kubernetes secret 来访问它。`sink` 部分指向先前创建的`KafkaSink`。有了这个清单，所有的 SQS 消息都将被使用 [Cloudevents](https://cloudevents.io/) 规范消费并发送给 Kafka。

> 两个 Kubernetes 对象，你就有了 Kafka 源代码的声明性定义

## 配置 Kafka 接收器

对于你的 Kafka Sink，我们很遗憾地回到了同样的潜在困惑:从 Kafka 集群的角度来看，什么是 Sink，什么是该集群之外的 source。因此，要定义 Kafka sink，您需要在 Kubernetes 集群中定义一个`KafkaSource`。

例如，下面的清单指定了您的引导服务器指向合流云中的 Kafka 集群，一个从其消费消息的特定主题和一个作为 Kafka 消息的最终目标的`sink`。这里我们要提到一个叫做`display`的 Kubernetes 服务:

```
apiVersion: sources.knative.dev/v1beta1
kind: KafkaSource
metadata:
 name: my-kafka
spec:
 bootstrapServers:
 — pkc-453q9.us-east4.gcp.confluent.cloud:9092
 net:
   sasl:
     enable: true
…
 sink:
   ref:
     apiVersion: v1
     kind: Service
     name: display
 topics:
 — hackathon
```

因此，Kafka Sink 定义只需要一个 Kubernetes 清单和一个作为传统 Kubernetes 服务公开的目标微服务。请注意，它也可以是一个 Knative 服务，在这种情况下，您的目标将受益于自动扩展，包括扩展到零。

# 结论

与编译 jar 并将它们上传到 Kafka connect 集群的复杂工作流程不同，您可以利用您的 Kubernetes 集群并使用 Kubernetes 对象定义您的 Kafka 源和接收器。

这给你的 Kafka 源和汇带来了一个声明性的心态，给你一个真实的来源，并统一了你的 Kafka 流和你的微服务的管理。此外，当 Kafka 消息存在时，这简化了在 Kubernetes 中运行的微服务的定位，并且通过添加 Knative，为您提供了处理不断变化的流容量所必需的自动缩放。

如果你需要一个 Kafka 源的集合，突然所有的 Knative 事件源都可以被使用(例如 GitHub，GitLab，JIRA，Zendesk，Slack，Kinesis…)