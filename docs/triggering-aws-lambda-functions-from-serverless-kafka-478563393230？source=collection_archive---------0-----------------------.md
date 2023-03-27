# 从无服务器 Kafka 触发 AWS Lambda 函数

> 原文：<https://itnext.io/triggering-aws-lambda-functions-from-serverless-kafka-478563393230?source=collection_archive---------0----------------------->

![](img/73b6797a31ed3ab26151588930e6c471.png)

由 [Kelvin Ang](https://unsplash.com/@kelvin1987?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

今天，我们将连接两个世界。一个世界是无服务器的卡夫卡，另一个世界是无服务器的功能。

本教程将演示如何生成触发 Lambda 函数的 Kafka 消息。Kafka 实例将在[上运行，而对于 Lambda 函数，我们将使用 AWS。](http://upstash.com/?utm_source=tobias_tobias4)

Upstash 是一个按需付费的解决方案，无需摆弄硬件、虚拟机或 docker 容器等东西。不用的时候不花什么钱。它实际上缩小到 0，而 0 意味着没有任何成本。

在我们将消息从本地机器发送到 [Upstash](http://upstash.com/?utm_source=tobias_tobias4) 上的 Kafka 之前，需要完成几个步骤，然后消息将被 AWS 接收并触发我们的 Lambda 函数。

如果你更喜欢使用你自己的或任何其他的 Kafka 实例，你只需要相应地修改本教程的不同步骤。

具体来说，这些步骤是:

*   在[上方](http://upstash.com/?utm_source=tobias_tobias4)创建一个 Kafka 集群和主题
*   启动生产者和消费者来测试集群
*   创建秘密
*   创建 Lambda 函数
*   创建 Lambda 角色
*   创建触发器
*   测试设置

有许多事情要做，让我们不要浪费时间，马上开始工作吧。

## 在 Upstash 上创建一个 Kafka 集群和主题

如果还没有这样做，请前往[后台](http://upstash.com/?utm_source=tobias_tobias4)创建一个帐户。完成后，转到[控制台](https://console.upstash.com)并创建一个新的集群。

首先，我们提供集群及其区域的名称。

![](img/2f94a33895de493ed0f19fd360718da8.png)

下一步，我们将创建接收消息的主题。基本上，我只是提供了名称，一切都保持默认。

![](img/1e0bc858f2d06ee95676138c5d3302bf.png)

完成后，将显示集群的概览页面。此页面提供了连接到集群所需的所有必要信息。我们将在下一节中用到它。

![](img/5e090eea4a6aa2d296ecab82da2b4079.png)

现在，我们的 Kafka 集群已经就位，我们将对其进行测试。

## 启动生产者和消费者来测试集群

Kafka 自带命令行工具，对于查询和测试 Kafka 集群非常有用。这些既可以直接安装在本地机器上，也可以通过官方的 Kafka Docker 镜像使用。

特别是，我们将使用命令`kafka-console-producer`和`kafka-console-consumer`。为了简单起见，我使用这些命令，就好像它们在我的本地机器上一样。

这里有一个例子来说明不同之处。如果用户已经直接安装在您的计算机上，启动用户的基本方法是:

```
kafka-console-consumer --topic planes --from-beginning --bootstrap-server kafka:9092
```

做着同样的事情，但这一次借助了官方的卡夫卡码头工人形象，看起来是这样的:

```
docker-compose exec kafka bash -c 'kafka-console-consumer.sh --topic planes --from-beginning --bootstrap-server kafka:9092'
```

对于 Upstash，我们需要添加一些配置，因为我们需要进行身份验证。不应该让每个人都连接到您的集群，对吗？

在本地机器的工作目录下创建一个名为`upstash.config`的文件，并将[控制台上的](http://upstash.com/?utm_source=tobias_tobias4)属性放入其中。

![](img/ab8c0a3ef50759c7611f8fbbc0581e31.png)

不需要属性`bootstrap.servers`。这是它应该看起来，除了凭据不同于我的。

```
sasl.mechanism=SCRAM-SHA-256
security.protocol=SASL_SSL
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username=\"YXBwYXJlbnQtYXNwLTExNjU2JMPd9pk9KbaNkvMiqW7h0l3FOHgPks2Ia3lgup4\" password=\"WcsQuBndTvo1zRYIHffSj5ZwN5A8G6sryjeG2k_1JyThzOzMDTURls2Fo-pboUX4hoza9g==\";"
```

然后启动制作人。

```
kafka-console-producer --bootstrap-server apparent-asp-11656-eu1-kafka.upstash.io:9092 --producer.config upstash.config --topic tutorial-topic
```

在另一个控制台中，启动消费者。

```
kafka-console-consumer --bootstrap-server apparent-asp-11656-eu1-kafka.upstash.io:9092 --consumer.config upstash.config --from-beginning --topic tutorial-topic
```

最后，我们可以从生产者那里发送一些消息，并在消费者那里接收它们。

![](img/c18248e70aee76feb378aaf449be26d2.png)

现在，是时候去 Amazon AWS 创建和设置不同的部分了，我们需要用 Kafka 消息触发我们的 Lambda 函数。

## 创建 AWS 机密

转到 AWS 秘密管理器，点击“存储新的秘密”。

![](img/548a2f1f6d9576db71d350861cb82e00.png)

选择“其他类型的秘密”并为您的 [Upstash](http://upstash.com/?utm_source=tobias_tobias4) 用户名和密码添加以下密钥/值对。

![](img/ca1108da1ed78d9c0676ff1e1567740f.png)

单击“下一步”,并提供密码的名称。

![](img/9f17e81aa4adf528640a09d4a89b7ef1.png)

然后点击“下一步”，然后在“存储新的秘密”-页面也点击“下一步”，然后“存储”，不做任何更改。

完成后，让我们继续下一部分，创建 Lamdba 函数。

## 创建 Lambda 函数

在 AWS Lambda 中，单击“创建函数”。

![](img/0bc406e0a9f2bc374e4af3dda29e8f64.png)

在那里，我们将创建一个名为“kafka-consumer”的 Node.js 函数。完成后，再次单击“创建函数”。

![](img/a0c9a99b9ca95dca7b8cdd2c4e740809.png)

将`index.js`的内容替换为

```
exports.handler = async (event) => {
    for (let key in event.records) {
      console.log('Key: ', key)
      event.records[key].map((record) => {
        console.log('Record: ', record)
        const msg = Buffer.from(record.value, 'base64').toString()
        console.log('Message:', msg)
      }) 
    }
}
```

这个函数基本上只打印触发它的 Kafka 事件的内容。

![](img/14430a1e371f320c9fec586f2fdabbcb.png)

然后单击“部署”。

## 创建 Lambda 角色

点击“配置”，然后点击“权限”，选择角色名称，这里是“Kafka-consumer-role-eov2 kfe”。

![](img/99081a67d29d2f5d3716bfc7fa08b105.png)

现在，“身份和访问管理(IAM)”控制台已经打开。

在“添加权限”下，选择“创建内嵌策略”

![](img/bd9fe5685be67886ad4223bb3a0e4a1d.png)

在“JSON”选项卡中添加以下内容。

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue"
            ],
            "Resource": "arn:aws:secretsmanager:eu-west-1:450055871228:secret:upstash-xcYDmy"
        }
    ]
}
```

请确保替换您可以在 lambda 函数的概述页面上找到的函数 ARN。

![](img/ce5fb13c79cae0728f9eb21413669997.png)

完成后，策略应该是这样的。

![](img/f7cc6a2350384f0158f0229750ebb6c9.png)

点击“审查政策”。

在“审查策略”下添加名称，例如“SelfHostedKafkaPolicy”，然后单击“创建策略”。

如果一切正常，该策略现在会列在 IAM 角色页面下

![](img/bff97ba51c42c47faa7275c9e40c28d0.png)

这是教程中最棘手的部分。如果你能走到这一步，那就太好了。在我们测试所有东西之前，只剩下一小步了。

## 创建触发器

点击“添加触发器”并选择“阿帕奇卡夫卡”-触发器。

在“Bootstrap servers”下添加 Kafka 集群的端点名称和主题名称。

在“身份验证”下，单击“添加”并提供上面的密码。

![](img/291e94550a0ffa4f3404a015abc0bc0d.png)

然后点击“添加”。如果一切顺利，我们将返回到函数页面。

![](img/fdd56bbe5d4457643db33e22dfd910e7.png)

我们现在完成了所有的设置工作。我希望这对你有用。

是检验一切的时候了。

## 测试设置

在 AWS 中，点击“监控”

如果不再运行，在本地机器上再次启动`kafka-console-producer`。

```
kafka-console-producer --bootstrap-server apparent-asp-11656-eu1-kafka.upstash.io:9092 --producer.config upstash.config --topic tutorial-topic
```

然后发送一条或多条消息。

![](img/10361647c8f5921a99cc44b137e8e3a0.png)

在您的 AWS CloudWatch 指标下，您应该看到 Lambda 函数的调用。

![](img/a5e0655e032eb21b739610f2400d99c4.png)

点击“查看 CloudWatch 中的日志”,进入此页面。

![](img/9179c70b75a9c03bb7c1d74bbd2c5f6b.png)

在这里，选择日志流，这将打开一个显示日志流内容的页面。

![](img/6a4f6aaa3c963eae0c72e6963e9c93b0.png)

在那里！倒数第三行是我发送的消息“你好”。根据您发送的内容，您也可以在这里找到您的消息。

## 结论

多棒的旅行啊！我们从在本地机器上生成 Kafka 消息开始，然后这些消息被发送到位于[upshort](http://upstash.com/?utm_source=tobias_tobias4)上的 Kafka 实例，最后由 Amazon AWS 上的 Lambda 函数接收以执行操作。

我希望你喜欢这个教程，它会对你有用。如果你有任何问题，关于如何改进本教程的建议，或者如果你有一个想在我的博客上看到的新主题或教程的想法，请随时在评论区给我打电话。

感谢您的阅读！

*   如果你喜欢这个，请[跟随我在媒体](https://twissmueller.medium.com/)
*   给我买杯咖啡让我继续前进
*   在这里注册[支持我和其他媒体作者](https://twissmueller.medium.com/membership)

[](https://twissmueller.medium.com/membership) [## 通过我的推荐链接加入媒体

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

twissmueller.medium.com](https://twissmueller.medium.com/membership) 

这篇文章包含附属链接，由 [Upstash](http://upstash.com/?utm_source=tobias_tobias4) 赞助。

## 资源

*   [使用无服务器 Kafka 作为 AWS Lambda 的事件源](https://docs.upstash.com/kafka/howto/kafka-with-lambda?utm_source=tobias4)
*   [使用 Lambda 和自我管理的 Apache Kafka](https://docs.aws.amazon.com/lambda/latest/dg/with-kafka.html)
*   [使用自托管的 Apache Kafka 作为 AWS Lambda 的事件源](https://aws.amazon.com/blogs/compute/using-self-hosted-apache-kafka-as-an-event-source-for-aws-lambda/)