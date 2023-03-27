# 如何使用 Kafka 绑定集成 Dapr 和 Azure 事件中心

> 原文：<https://itnext.io/how-to-integrate-dapr-and-azure-event-hubs-using-kafka-bindings-44fb8de647cb?source=collection_archive---------1----------------------->

[之前的一篇博文](/tutorial-using-azure-event-hubs-with-the-dapr-framework-81c749b66dcf)展示了如何使用 [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/?WT.mc_id=medium-blog-abhishgu) 作为 [Dapr](http://dapr.io/) 中的资源绑定。[资源绑定](https://github.com/dapr/components-contrib/blob/master/bindings/Readme.md)提供了一种方法，通过来自外部系统的事件(*输入绑定*)来触发应用程序，或者通过可选的数据有效负载来调用外部系统(*输出绑定*)。这些“外部系统”可以是任何东西:队列、消息传递管道、云服务、文件系统等等。在这篇博客中，我们将通过一个示例来浏览[如何将](https://github.com/abhirockzz/dapr-kafka-eventhubs-bindings) [Kafka](https://kafka.apache.org/) 作为`Dapr`绑定来集成到您的应用程序中。

> ***Dapr*** *(又名分布式应用运行时)是一个开源的、可移植的运行时，帮助开发者构建弹性的、微服务的无状态和有状态应用。如果你对* `*Dapr*` *还不了解，我推荐你查阅一下*[*Github repo*](https://github.com/dapr/dapr)*并浏览一下* [*【入门】*](https://github.com/dapr/docs/tree/master/getting-started) *指南。你也可以阅读我以前的一些博客文章。*

azure Event Hubs[还通过公开 Kafka 兼容端点来提供 Apache Kafka 支持](https://docs.microsoft.com/azure/event-hubs/event-hubs-for-kafka-ecosystem-overview?WT.mc_id=medium-blog-abhishgu)，Kafka 兼容端点可由您现有的基于 Kafka 的应用程序使用，作为运行您自己的 Kafka 集群的替代方案。`SASL`在 [Dapr 版本 0.2.0](https://github.com/dapr/dapr/releases/tag/v0.2.0) 中增加了 Kafka 绑定的 auth，从而使得通过`Dapr`中的 Kafka 绑定支持使用 Azure 事件中心成为可能——这就是这篇博客将要展示的。

在我们继续下一步之前，让我们先设置我们需要的东西。

# 先决条件

*   CLI 和运行时组件
*   Azure 活动中心

## Dapr

请阅读`Dapr` [入门指南](https://github.com/dapr/docs/blob/master/getting-started/)了解如何安装 Dapr CLI 的说明

例如用于 mac(安装`Dapr CLI`到`/usr/local/bin`)

```
curl -fsSL https://raw.githubusercontent.com/dapr/cli/master/install/install.sh | /bin/bash
```

一旦有了 CLI，就可以使用`dapr init`在本地运行，或者使用`dapr init --kubernetes`在 Kubernetes 集群上运行。

## 设置 Azure 事件中心

如果你还没有一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)，那就去注册一个免费账户吧！。完成后，您可以使用以下快速入门工具之一快速设置 Azure 事件中心:

*   Azure 门户网站— [这里是一个分步指南](https://docs.microsoft.com/azure/event-hubs/event-hubs-create?WT.mc_id=medium-blog-abhishgu)
*   Azure CLI 或 Azure Cloud shell(在您的浏览器中！)— [这里有一个分步指南](https://docs.microsoft.com/azure/event-hubs/event-hubs-quickstart-cli?WT.mc_id=medium-blog-abhishgu)

现在，您应该有一个带有名称空间和关联事件中心(主题)的事件中心实例。作为最后一步，您需要获取连接字符串，以便[向事件中心](https://docs.microsoft.com/azure/event-hubs/authenticate-shared-access-signature?WT.mc_id=medium-blog-abhishgu) — [进行身份验证。使用本指南](https://docs.microsoft.com/azure/event-hubs/event-hubs-get-connection-string?WT.mc_id=medium-blog-abhishgu)完成此步骤。

# 概观

示例应用程序包括:

*   向 Azure 事件中心发送事件的生产者应用程序。这是一个独立的 Go 应用程序，它使用 [Sarama 客户端](https://github.com/Shopify/sarama)与 Azure Event Hubs Kafka 端点对话。
*   一个消费应用程序，从 Kafka 主题消费并打印出数据。此应用程序使用 Dapr 运行

## 使用 Dapr 运行消费者应用程序

从克隆回购开始

```
git clone [https://github.com/abhirockzz/dapr-kafka-eventhubs-bindings](https://github.com/abhirockzz/dapr-kafka-eventhubs-bindings)
```

这里是绑定组件 YAML

```
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: timebound
spec:
  type: bindings.kafka
  metadata:
    - name: brokers
      value: [replace]
    - name: topics
      value: [replace]
    - name: consumerGroup
      value: $Default
    - name: authRequired
      value: "true"
    - name: saslUsername
      value: $ConnectionString
    - name: saslPassword
      value: [replace]
```

更新`components/eventhubs_binding.yaml`以包含 Azure 事件中心的详细信息

*   `brokers` -替换为 Azure 事件中心端点，例如`foobar.servicebus.windows.net:9093`，其中`foobar`是[事件中心名称空间](https://docs.microsoft.com/azure/event-hubs/event-hubs-features?WT.mc_id=medium-blog-abhishgu#namespace)
*   `saslPassword` -这需要更换为[事件集线器连接字符串](https://docs.microsoft.com/azure/event-hubs/authenticate-shared-access-signature?WT.mc_id=medium-blog-abhishgu) - [使用本指南](https://docs.microsoft.com/azure/event-hubs/event-hubs-get-connection-string?WT.mc_id=medium-blog-abhishgu)(如前所述)
*   `consumerGroup` -你可以继续使用`$Default`作为价值或者在 Azure 活动中心创建一个新的消费者群体(使用 [Azure CLI](https://docs.microsoft.com/cli/azure/eventhubs/eventhub/consumer-group?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-eventhubs-eventhub-consumer-group-create) 或 portal)并使用它

启动使用 Azure 事件中心输入绑定的 Go 应用程序

```
cd app
export APP_PORT=9090
dapr run --app-port $APP_PORT go run consumer.go
```

您应该会看到类似如下的日志:

```
ℹ️  Starting Dapr with id Bugarrow-Walker. HTTP Port: 52089\. gRPC Port: 52090
✅  You're up and running! Both Dapr and your app logs will appear here.== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="starting Dapr Runtime -- version 0.3.0 -- commit v0.3.0-rc.0-1-gfe6c306-dirty"
== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="log level set to: info"
== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="standalone mode configured"
== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="dapr id: Bugarrow-Walker"
== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="loaded component messagebus (pubsub.redis)"
== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="loaded component statestore (state.redis)"
== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="loaded component eventhubs-input (bindings.kafka)"
== DAPR == time="2020-01-14T19:35:09+05:30" level=info msg="application protocol: http. waiting on port 9090"
== DAPR == time="2020-01-14T19:35:10+05:30" level=info msg="application discovered on port 9090"
```

## 运行 Azure 事件中心生成器应用程序

设置所需的环境变量:

```
export EVENTHUBS_CONNECTION_STRING="[replace with connection string]"
export EVENTHUBS_BROKER=[replace with broker endpoint]
export EVENTHUBS_TOPIC=[replace with topic name]
export EVENTHUBS_USERNAME="\$ConnectionString"
```

> *不需要修改* `*EVENTHUBS_USERNAME*`

运行 producer 应用程序——它会一直向指定的事件中心主题发送消息，直到停止(按`ctrl+c`停止应用程序)

```
cd producer
go run producer.go
```

如果一切正常，您应该在 producer 应用程序中看到以下日志:

```
Event Hubs broker [foobar.servicebus.windows.net:9093]
Event Hubs topic test
Waiting for ctrl+c
sent message {"time":"Tue Jan 14 19:41:53 2020"} to partition 3 offset 523
sent message {"time":"Tue Jan 14 19:41:56 2020"} to partition 0 offset 527
sent message {"time":"Tue Jan 14 19:41:59 2020"} to partition 4 offset 456
sent message {"time":"Tue Jan 14 19:42:02 2020"} to partition 2 offset 486
sent message {"time":"Tue Jan 14 19:42:06 2020"} to partition 0 offset 528
```

## 确认

检查 Dapr 应用程序日志，您应该看到从事件中心收到的消息。

```
== APP == data from Event Hubs '{Tue Jan 14 19:35:21 2020}'
== APP == data from Event Hubs '{Tue Jan 14 19:41:53 2020}'
== APP == data from Event Hubs '{Tue Jan 14 19:41:56 2020}'
== APP == data from Event Hubs '{Tue Jan 14 19:41:59 2020}'
== APP == data from Event Hubs '{Tue Jan 14 19:42:02 2020}'
== APP == data from Event Hubs '{Tue Jan 14 19:42:06 2020}'
```

# 幕后…

下面是其工作原理的总结:

输入绑定定义了 Kafka 集群要连接的连接参数。除了这些参数之外，`metadata.name`属性也很重要。

消费者应用程序在`/timebound`公开了一个 REST 端点——这与输入绑定组件的名称相同(不是巧合！)

```
func main() {
	http.HandleFunc("/timebound", func(rw http.ResponseWriter, req *http.Request) {
		var _time TheTime err := json.NewDecoder(req.Body).Decode(&_time)
		if err != nil {
			fmt.Println("error reading message from event hub binding", err)
			rw.WriteHeader(500)
			return
		}
		fmt.Printf("data from Event Hubs '%s'\n", _time)
		rw.WriteHeader(200)
	})
	http.ListenAndServe(":"+port, nil)
}
```

`Dapr`运行时执行从事件中心消费的繁重工作，并确保它在带有事件有效负载的`/timebound`端点使用`POST`请求调用 Go 应用程序。然后执行 app 逻辑，在这种情况下只是记录到标准输出。

# 摘要

在这篇博客文章中，您看到了如何使用 Azure Event Hubs 使用`Dapr`绑定来连接和集成基于 Kafka 的应用程序。

在撰写本文时，`Dapr`正处于 alpha 状态(`v0.3.0`)，并乐于接受社区贡献😃前往 https://github.com/dapr/dapr 潜水吧！

如果你觉得这篇文章有帮助，请喜欢并关注🙌很高兴通过 [Twitter](https://twitter.com/abhi_tweeter) 获得反馈或发表评论。