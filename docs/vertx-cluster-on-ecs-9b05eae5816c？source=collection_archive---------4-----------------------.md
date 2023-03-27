# ECS 上的 Vertx 集群

> 原文：<https://itnext.io/vertx-cluster-on-ecs-9b05eae5816c?source=collection_archive---------4----------------------->

![](img/39d70a9249c63c1af62f98a8f93d5563.png)

Vertx 是一个用于高负载应用的异步框架，它使用 Netty、新的 I/O 和事件总线。Vertx 设计用于微服务，许多 vertx 实例可以在一个集群中运行，以实现高可用性。

ECS 是一个面向 docker 的 AWS 服务，它比 Kubernetes 简单得多，允许在集群中运行 docker 容器。

这里我们将创建一个 Vertx 集群，它将在 ECS 集群中运行，使用 ECS 集群和服务名称进行节点发现。

我们的应用程序将由三个`verticles`(类似于`akka`中的`Actor` )— `HttpVerticle`组成，它们监听 HTTP“健康检查”请求，并使用事件总线向另外两个垂直领域发送消息。消息内容将是包含 IP 地址的字符串，因此通过打印我们的“本地”IP 地址和接收到的消息内容，我们可以看到消息是由另一个集群成员发送的。

让我们开始，我们将使用 Kotlin:

现在，棘手的部分——创建一个 Vertx 集群并向 AWS 询问当前 IP。
接收 IP 的代码使用的是仅在 ECS 中工作的[http://169 . 254 . 170 . 2/v2/metadata](http://169.254.170.2/v2/metadata)端点[。
对于 Vertx 集群，我们使用 hazelcast 作为集群管理器](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)

对于 ECS 中的 hazelcast 集群发现，我们将使用这个[库](https://github.com/lightspeed-hospitality/hazelcast-aws-ecs)。这个库似乎没有太多的支持，它仍然在版本`1.0-SNAPSHOT`中，但是它在做它的工作。你不能在公共的 maven 库中找到它，所以你需要自己编译它

ecs-cluster.xml:

**替换示例 XML 文件中的集群和服务名称**

在这个文件中有一个对`“com.ikentoo.hazelcast.AwsEcsDiscoveryStrategy”`的引用，它必须在类路径中。

现在您需要构建您的 docker 映像并将其发布到您的 Docker 注册中心。
Dockerfile:

现在—重要的部分—创建一个 [ECS 任务定义和服务](https://medium.com/boltops/gentle-introduction-to-how-aws-ecs-works-with-example-tutorial-cea3d27ce63d)。至少在两个实例中运行此服务，以查看它们如何相互通信。

这个过程在本文中[有很好的描述，但是要注意以下几点:
1。您应该为 ECS 任务分配一个 AIM 角色，它包含`ecs:ListTasks`和`ecs:DescribeTasks`权限
2。请小心执行运行状况检查任务。我建议给它一个`100` 秒的启动时间，以确保服务器在 ECS 开始检查它之前启动
3。在安全组中打开端口`5701` 和`15701` (事件总线端口，在源代码中硬编码)
4。再次检查您的集群名和服务名是否与 cluster.xml 文件中的相同](https://medium.com/boltops/gentle-introduction-to-how-aws-ecs-works-with-example-tutorial-cea3d27ce63d)

如果一切正常，您将在 cloudwatch 日志中看到类似这样的内容

所有源代码包括一个 gradle 构建文件你可以在这里找到。**注意！**它假设您的 maven 库中有一个 artefact `group: 'com.ikentoo', name: 'hazelcast-aws-ecs', version: '1.0-SNAPSHOT’`！

参考文献:
1。[如何从 ECS 容器](https://medium.com/@pedrojuarez/ec2-ecs-instance-metadata-2939d7a6f2ce)内部获取主机 ip 和端口
2。[实例元数据和用户数据](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
3。[如何在 ECS (EC2 容器服务)环境下设置 hazelcast？](https://stackoverflow.com/questions/32815708/how-to-set-up-hazelcast-in-a-ecs-ec2-container-service-environment)
4。[hazel cast-AWS-ECS](https://github.com/iKentoo/hazelcast-aws-ecs)5。 [hazelcast-aws](https://github.com/hazelcast/hazelcast-aws) —他们说他们支持 ECS，但是我没能运行它
6。 [Fargate support #86](https://github.com/hazelcast/hazelcast-aws/issues/86) —在这里我看到了对[hazel cast-AWS-ECS](https://github.com/iKentoo/hazelcast-aws-ecs)
7 的引用。[用例子温和的介绍 AWS ECS 如何工作教程](https://medium.com/boltops/gentle-introduction-to-how-aws-ecs-works-with-example-tutorial-cea3d27ce63d)
8。[教程:使用 AWS CLI](https://medium.com/boltops/gentle-introduction-to-how-aws-ecs-works-with-example-tutorial-cea3d27ce63d)
9 创建带有 Fargate 任务的集群。[本文来源](https://gitlab.com/bernshtam-articles/vertx-ecs-cluster)