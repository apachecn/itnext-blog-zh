# 库伯内特斯上的卡夫卡，斯特里姆兹之路！(第二部分)

> 原文：<https://itnext.io/kafka-on-kubernetes-the-strimzi-way-part-2-43192f1dd831?source=collection_archive---------1----------------------->

欢迎来到这个关于在 Kubernetes 上运行卡夫卡的博客系列:

*   [第一部](/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788)
*   第二部分——这个博客
*   [第三部分](/kafka-on-kubernetes-the-strimzi-way-part-3-19cfdfe86660?source=friends_link&sk=f4adf9abd626390e8425422e7cd725a6)
*   [第四部分](https://medium.com/@abhishek1987/kafka-on-kubernetes-the-strimzi-way-part-4-bf1e651e25b8)

我们通过建立一个单节点 Kafka 集群开始了本系列第的第[部分，该集群只有同一个 Kubernetes 集群中的内部客户端可以访问，没有加密、认证或授权，使用临时持久性。在这个博客系列的过程中，我们将继续迭代/改进这个问题。](/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788)

![](img/9db541c0cf9f2110a3bc6bc61f37de50.png)

本部分将涵盖以下主题:

*   向外部应用程序公开 Kafka 集群
*   应用`TLS`加密
*   探索 Kubernetes 幕后的资源
*   使用 Kafka CLI 和 Go 客户端应用程序来测试我们的集群设置

> *代码在 GitHub 上可以找到—*[*https://github.com/abhirockzz/kafka-kubernetes-strimzi/*](https://github.com/abhirockzz/kafka-kubernetes-strimzi/)

# 我需要什么来尝试这个？

`kubectl`-[https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

我将使用[Azure Kubernetes Service(AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes?WT.mc_id=medium-blog-abhishgu)来演示概念，但总的来说，它是独立于 Kubernetes 提供商的(例如，可以随意使用本地设置，如`minikube`)。如果你想使用`AKS`，你只需要一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)，如果你还没有的话，你可以[免费获得](https://azure.microsoft.com/free/?WT.mc_id=medium-blog-abhishgu)。

> *在本系列的本部分或后续部分中，我不会重复一些常见部分(如 Helm、Strimzi、Azure Kubernetes 服务以及 Strimzi 概述的安装/设置),并会要求您* [*参考第一部分*](/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788) *了解那些详细信息*

# 让我们创建一个外部可访问的 Kafka 集群

为了实现这一点，我们只需要稍微调整 Strimzi `Kafka`资源。我强调了下面的关键部分- [这是第 1 部分的原始清单](https://github.com/abhirockzz/kafka-kubernetes-strimzi/raw/master/part-1/kafka.yaml)

```
spec:
  kafka:
    version: 2.4.0
    replicas: 1
    listeners:
      plain: {}
 **external:
        type: loadbalancer
        tls: true**
```

**什么变了？**

为了让外部客户端应用程序可以访问 Kafka，我们添加了一个`loadbalancer`类型的`external`监听器。由于我们将把我们的应用程序暴露给公共互联网，我们需要额外的保护层，如传输层(`TLS/SSL`加密)和应用层安全性(认证和授权)。在这一部分中，我们将只配置加密，并在另一篇博客中探讨其他方面。为了配置端到端的 TLS 加密，我们添加了`tls: true`

> `*tls: true*` *配置实际上被用作默认设置，但是为了清楚起见，我已经明确地添加了它*

要创建集群，请执行以下操作:

```
kubectl apply -f [https://github.com/abhirockzz/kafka-kubernetes-strimzi/raw/master/part-2/kafka.yaml](https://github.com/abhirockzz/kafka-kubernetes-strimzi/raw/master/part-2/kafka.yaml)
```

## 库伯内特魔法！

Strimzi 操作员开始行动，为我们完成所有繁重的工作:

*   它创建了一个 Kubernetes `[LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)`服务..
*   ..并在`ConfigMap`中播种适当的 Kafka 服务器配置

我将重点介绍与外部侦听器和 TLS 加密相对应的资源。要浏览作为 Kafka 集群的一部分创建的所有资源，[请参考第 1 部分](/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788)

如果您查找`Service` s，您将会看到与此类似的内容:

```
kubectl get svcmy-kafka-cluster-kafka-0                    LoadBalancer   10.0.162.98    40.119.233.2    9094:31860/TCP               60s
my-kafka-cluster-kafka-bootstrap            ClusterIP      10.0.200.20    <none>          9091/TCP,9092/TCP            60s
my-kafka-cluster-kafka-brokers              ClusterIP      None           <none>          9091/TCP,9092/TCP            60s
my-kafka-cluster-kafka-external-bootstrap   LoadBalancer   10.0.122.211   20.44.239.202   9094:32267/TCP               60s
my-kafka-cluster-zookeeper-client           ClusterIP      10.0.137.33    <none>          2181/TCP                     82s
my-kafka-cluster-zookeeper-nodes            ClusterIP      None           <none>          2181/TCP,2888/TCP,3888/TCP   82s
```

注意类型`LoadBalancer`的`my-kafka-cluster-kafka-external-bootstrapService`？因为我使用的是 Azure Kubernetes 服务，所以它由一个 [Azure 负载均衡器](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview?WT.mc_id=medium-blog-abhishgu)提供支持，这个负载均衡器有一个公共 IP(在这个例子中是`20.44.239.202`)，并通过端口`9094`向外部客户端公开 Kafka。通过使用`[az network lb list](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-network-lb-list)`命令，您应该能够使用 Azure CLI(或者 Azure portal，如果您喜欢的话)找到它

```
export AKS_RESOURCE_GROUP=[replace with resource group name]
export AKS_CLUSTER_NAME=[replace with AKS cluster name]
export AKS_LOCATION=[replace with region e.g. southeastasia]az network lb list -g MC_${AKS_RESOURCE_GROUP}_${AKS_CLUSTER_NAME}_${AKS_LOCATION}
```

**加密部分呢？**

为了弄清楚这一点，让我们反思一下 Kafka 服务器的配置:

> *如* [*上一篇博客*](https://dev.to/azure/kafka-on-kubernetes-the-strimzi-way-part-1-57g7) *中所解释的，这是存储在* `*ConfigMap*`中的

```
export CLUSTER_NAME=my-kafka-cluster
kubectl get configmap/${CLUSTER_NAME}-kafka-config -o yaml
```

这就是`server.config`中的`Common listener configuration`所揭示的:

```
listeners=REPLICATION-9091://0.0.0.0:9091,PLAIN-9092://0.0.0.0:9092,**EXTERNAL-9094://0.0.0.0:9094**advertised.listeners=REPLICATION-9091://my-kafka-cluster-kafka-${STRIMZI_BROKER_ID}.my-kafka-cluster-kafka-brokers.default.svc:9091,PLAIN-9092://my-kafka-cluster-kafka-${STRIMZI_BROKER_ID}.my-kafka-cluster-kafka-brokers.default.svc:9092,**EXTERNAL-9094://${STRIMZI_EXTERNAL_9094_ADVERTISED_HOSTNAME}:${STRIMZI_EXTERNAL_9094_ADVERTISED_PORT}**listener.security.protocol.map=REPLICATION-9091:SSL,PLAIN-9092:PLAINTEXT,**EXTERNAL-9094:SSL**
```

请注意，除了代理间复制(通过端口`9091`)和非`TLS`端口`9092`上的未加密内部(Kubernetes 集群内)客户端访问之外，还为通过端口`9094`的 TLS 加密访问添加了适当的监听器配置

# 关键时刻到了…

为了证实这一点，让我们尝试几个客户端应用程序，它们将与我们在 Kubernetes 上新创建的 Kafka 集群进行通信！我们将使用以下内容生成和使用消息:

*   Kafka CLI(控制台)生产者和消费者
*   Go 应用程序(使用[合流 Kafka Go 客户端](https://github.com/confluentinc/confluent-kafka-go)

与 Kafka 集群的通信必须加密(非 TLS 客户端连接将被拒绝)。`TLS/SSL`隐含着单向认证，客户端验证 Kafka 经纪人的身份。为此，客户端应用程序需要信任群集 CA 证书。记住集群 CA 证书存储在 Kubernetes `Secret`中(参见第 1 部分中的[细节)。默认情况下，这些是由 Strimzi 自动生成的，但是您也可以提供自己的证书(请参考](/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788)[https://strim zi . io/docs/operators/master/using . html # Kafka-listener-certificates-str](https://strimzi.io/docs/operators/master/using.html#kafka-listener-certificates-str))

首先提取群集 CA 证书和密码:

```
export CLUSTER_NAME=my-kafka-clusterkubectl get secret $CLUSTER_NAME-cluster-ca-cert -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca.crtkubectl get secret $CLUSTER_NAME-cluster-ca-cert -o jsonpath='{.data.ca\.password}' | base64 --decode > ca.password
```

> *你应该有两个文件:* `*ca.crt*` *和* `*ca.password*` *。请随意查看他们的内容*

一些 Kafka 客户端(例如 Confluent Go 客户端)直接使用 CA 证书，而其他客户端(例如 Java 客户端、Kafka CLI 等)则直接使用 CA 证书。)要求通过`truststore`访问 CA 证书。我使用的是 JDK (Java)安装自带的内置`truststore`——但这只是为了方便，你可以自由使用其他选项(比如创建自己的选项)

```
export CERT_FILE_PATH=ca.crt
export CERT_PASSWORD_FILE_PATH=ca.password# replace this with the path to your truststoreexport KEYSTORE_LOCATION=/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home/jre/lib/security/cacertsexport PASSWORD=`cat $CERT_PASSWORD_FILE_PATH`export CA_CERT_ALIAS=strimzi-kafka-cert# you will prompted for the truststore password. for JDK truststore, the default password is "changeit"
# Type yes in response to the 'Trust this certificate? [no]:' promptsudo keytool -importcert -alias $CA_CERT_ALIAS -file $CERT_FILE_PATH -keystore $KEYSTORE_LOCATION -keypass $PASSWORDsudo keytool -list -alias $CA_CERT_ALIAS -keystore $KEYSTORE_LOCATION
```

这就是基本设置—您已经准备好试用 Kafka CLI 客户端了！

> *请注意，下面详述的 Kafka CLI 配置步骤也适用于 Java 客户端——试一试吧！*

提取 Kafka 集群的`LoadBalancer`公共 IP

```
export KAFKA_CLUSTER_NAME=my-kafka-clusterkubectl get service/${KAFKA_CLUSTER_NAME}-kafka-external-bootstrap --output=jsonpath={.status.loadBalancer.ingress[0].ip}
```

用以下内容创建一个名为`client-ssl.properties`的文件:

```
bootstrap.servers=[LOADBALANCER_PUBLIC_IP]:9094
security.protocol=SSL
ssl.truststore.location=[TRUSTSTORE_LOCATION]
//for JDK truststore, the default password is "changeit"
ssl.truststore.password=changeit
```

要使用 Kafka CLI，如果你还没有的话，下载`Kafka`-[https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)

您所需要做的就是通过将`kafka-console-producer`和`kafka-console-consumer`指向您刚刚创建的`client-ssl.properties`文件来使用它们

```
export KAFKA_HOME=[replace with Kafka installation path] e.g. /Users/foobar/kafka_2.12-2.3.0export LOADBALANCER_PUBLIC_IP=[replace with public IP of Load Balancer]export TOPIC_NAME=test-strimzi-topic# on a terminal, start producer and send a few messages
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list $LOADBALANCER_PUBLIC_IP:9094 --topic $TOPIC_NAME --producer.config client-ssl.properties# on another terminal, start consumer
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server $LOADBALANCER_PUBLIC_IP:9094 --topic $TOPIC_NAME --consumer.config client-ssl.properties --from-beginning
```

你应该看到生产者和消费者协同工作。太好了！

> *如果您遇到 SSL 握手错误，请检查 CA 证书及其正确密码是否已正确导入。如果 Kafka 集群不可达，请确保您对公共 IP 使用了正确的值*

现在，让我们尝试一个编程式客户端。因为 Java 客户端行为(必需的配置属性)与 CLI 相同，所以我使用 Go 客户端来尝试一些不同的东西。不要担心，如果你不是一个 Go 程序员，这应该很容易理解——我不会遍历整个程序，只是我们创建连接相关配置的部分。

以下是片段:

```
bootstrapServers = os.Getenv("KAFKA_BOOTSTRAP_SERVERS")
caLocation = os.Getenv("CA_CERT_LOCATION")
topic = os.Getenv("KAFKA_TOPIC")config := &kafka.ConfigMap{"bootstrap.servers": bootstrapServers, "security.protocol": "SSL", "ssl.ca.location": caLocation}
```

请注意，`bootstrap.servers`和`security.protocol`与您在 Kafka CLI 客户端中使用的相同(对于 Java 也是如此)。唯一的区别是`ssl.ca.location`用于直接指向 CA 证书，而不是`truststore`

如果你安装了`Go`，你可以试试。克隆 Git repo...

```
git clone https://github.com/abhirockzz/kafka-kubernetes-strimzi
cd part-2/go-client-app
```

..并运行程序:

```
export KAFKA_BOOTSTRAP_SERVERS=[replace with loadbalancer_ip:9094] e.g. 42.42.424.424:9094export CA_CERT_LOCATION=[replace with path to ca.crt file which you downloaded]export KAFKA_TOPIC=test-strimzi-topicgo run kafka-client.go
```

您应该会看到类似这样的日志，并确认正在生成和使用消息

> *按* `*ctrl+c*` *退出 app*

```
started consumer
started producer delivery goroutine
started producer goroutinedelivered messaged test-strimzi-topic[0]@122
delivered messaged test-strimzi-topic[0]@123
delivered messaged test-strimzi-topic[0]@124
received message from test-strimzi-topic[0]@122: value-2020-06-08 16:23:05.913303 +0530 IST m=+0.020529419
received message from test-strimzi-topic[0]@123: value-2020-06-08 16:23:07.915252 +0530 IST m=+2.022455867
received message from test-strimzi-topic[0]@124: value-2020-06-08 16:23:09.915875 +0530 IST m=+4.023055601
received message from test-strimzi-topic[0]@125: value-2020-06-08 16:23:11.915977 +0530 IST m=+6.023134961
....
```

# 这就是现在的全部，但还有更多的来了！

所以我们取得了一些进步！我们现在在 Kubernetes 上有一个 Kafka 集群，可以公开访问，但由于 TLS 加密，它是(部分)安全的。我们还使用不是一个，而是两个(不同的)客户端应用程序做了一些健全性测试。在下一部分，我们将进一步改进这一点…敬请期待！