# 库伯内特斯上的卡夫卡，斯特里姆兹之路！(第三部分)

> 原文：<https://itnext.io/kafka-on-kubernetes-the-strimzi-way-part-3-19cfdfe86660?source=collection_archive---------1----------------------->

欢迎来到这个关于在 Kubernetes 上运行卡夫卡的博客系列:

*   [第一部分](/kafka-on-kubernetes-the-strimzi-way-part-1-bdff3e451788?source=friends_link&sk=dd9cb0a8b07ec65e62c9b9f5ae7b3986)
*   [第二部分](/kafka-on-kubernetes-the-strimzi-way-part-2-43192f1dd831?source=friends_link&sk=e5089aa7fc2d879e11989ba141b9d3a4)
*   第 3 部分:这个博客
*   [第四部分](https://medium.com/@abhishek1987/kafka-on-kubernetes-the-strimzi-way-part-4-bf1e651e25b8)

![](img/b153c5eaeee335658e895a19a688f7b6.png)

在本博客系列的前两部分中，我们在 Kubernetes 上设置了一个单节点 Kafka 集群，使用 TLS 加密保护它，并使用内部和外部客户机访问代理。我们继续迭代吧！在本帖中，我们将通过 Strimzi 和 cover 继续 Kafka 在 Kubernetes 的旅程:

*   如何应用不同的认证类型:`TLS`和`SASL SCRAM-SHA-512`
*   使用 Strimzi 实体运算符管理 Kafka 用户和主题
*   如何配置 Kafka CLI 和 Go 客户端应用程序以安全地连接到 Kafka 集群

> *代码在 GitHub 上可以找到—*[*https://github.com/abhirockzz/kafka-kubernetes-strimzi/*](https://github.com/abhirockzz/kafka-kubernetes-strimzi/)

# 我需要什么来浏览这个教程？

https://kubernetes.io/docs/tasks/tools/install-kubectl/`kubectl`-

我将使用[Azure Kubernetes Service(AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes?WT.mc_id=medium-blog-abhishgu)来演示这些概念，但总的来说，它是独立于 Kubernetes 提供者的(例如，可以随意使用本地设置，如`minikube`)。如果你想使用`AKS`，你只需要一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)，如果你还没有的话，你可以[免费获得](https://azure.microsoft.com/free/?WT.mc_id=medium-blog-abhishgu)。

> *在本系列的这一部分或后续部分，我不会重复一些常见的部分(例如安装/设置(Helm、Strimzi、Azure Kubernetes 服务)、Strimzi 概述),请参考第一部分*

# 创建带有 TLS 身份验证的 Kafka 集群

为了实施双向相互`TLS`授权，我们需要做的就是调整 Strimzi `Kafka`资源。我在下面突出重点部分。其他部分保持不变([这是来自第 2 部分](https://github.com/abhirockzz/kafka-kubernetes-strimzi/raw/master/part-2/kafka.yaml)的清单)，即单节点 Kafka 和 Zookeeper、短暂存储以及`TLS`加密

```
external:
  type: loadbalancer
  tls: true
  authentication:
    type: tls
```

我们所做的就是将所有的`tls`认证类型作为`external`监听器的一个属性。除此之外，我们还包括`entityOperator`配置:

```
entityOperator:
  userOperator: {}
  topicOperator: {}
```

这将激活 Strimzi `Entity Operator`，该 Strimzi 由`Topic Operator`和`User Operator`组成。正如`Kafka` CRD 允许您控制 Kubernetes 上的 Kafka 集群一样，主题操作符允许您通过名为`KafkaTopic`的自定义资源管理 Kafka 集群中的主题，即您可以在 Kafka 集群中创建、删除和更新主题。

> *有趣的是，这是一个双向同步，即您仍然可以通过直接访问 Kafka 集群来创建主题，这将反映在正在创建/更新/删除的* `*KafkaTopic*` *资源中*

用户操作者的目标是在`KafkaUser` CRD 的帮助下，使 Kafka 用户管理更加容易。您所做的就是创建`KafkaUser` CRDs 的实例，Strimzi 负责 Kafka 特定的用户管理部分

> *与主题操作符不同，这不是双向同步*

在这里阅读更多关于实体操作符的内容[https://strim zi . io/docs/operators/master/using . html # assembly-Kafka-Entity-Operator-deployment-configuration-Kafka](https://strimzi.io/docs/operators/master/using.html#assembly-kafka-entity-operator-deployment-configuration-kafka)

在接下来的章节中，我们将深入探讨这两个操作符的实际应用。

要创建 Kafka 集群，请执行以下操作:

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/kafka-tls-auth.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/kafka-tls-auth.yaml)
```

在这种情况下，Strimzi 运算符为我们做了什么？

我们在[第一部分](https://dev.to/azure/kafka-on-kubernetes-the-strimzi-way-part-1-57g7) — `StatefulSet`(和`Pods`)、`LoadBalancer`服务、`ConfigMap`、`Secret`等中涵盖了其中的大部分。如何实施`TLS`授权配置？为了弄清楚这一点，让我们反思一下 Kafka 服务器的配置

> *如第 1 部分所述，它存储在*T6 中

```
export CLUSTER_NAME=my-kafka-cluster
kubectl get configmap/${CLUSTER_NAME}-kafka-config -o yaml
```

查看`server.config`中的`External listener`部分:

```
listener.name.external-9094.ssl.client.auth=requiredlistener.name.external-9094.ssl.truststore.location=/tmp/kafka/clients.truststore.p12listener.name.external-9094.ssl.truststore.password=${CERTS_STORE_PASSWORD}listener.name.external-9094.ssl.truststore.type=PKCS12
```

上面突出显示的片段是添加的部分——注意`listener.name.external-9094.ssl.client.auth=required`是和信任库细节一起添加的。

**别忘了实体操作符**

实体操作员运行一个单独的`Deployment`

```
export CLUSTER_NAME=my-kafka-clusterkubectl get deployment $CLUSTER_NAME-entity-operator
kubectl get pod -l=app.kubernetes.io/name=entity-operatorNAME                                                READY   STATUS     
my-kafka-cluster-entity-operator-666f8758f6-gj54h   3/3     Running
```

实体操作符`Pod`运行三个容器- `topic-operator`、`user-operator`、`tls-sidecar`

我们已经配置了集群来验证客户端连接，但是客户端应用程序将使用的用户凭证呢？

# 是时候使用用户操作符了！

用户操作符允许我们创建`KafkaUser`来表示客户端认证凭证。正如博文开头提到的，支持的认证类型包括`TLS`和`SCRAM-SHA-512`。在幕后，Strimzi 创建了一个 Kubernetes `Secret`来存储凭证

> *OAuth 2.0 也受支持，但它不由用户操作者处理*

让我们创建一个`KafkaUser`来存储 TLS auth 的客户端凭证。下面是用户信息的样子:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
  name: kafka-tls-client-credentials
  labels:
    strimzi.io/cluster: my-kafka-cluster
spec:
  authentication:
    type: tls
```

我们将用户命名为`kafka-tls-client-credentials`，与我们之前创建的 Kafka 集群相关联(使用标签`strimzi.io/cluster: my-kafka-cluster`，并指定`tls`认证类型

> *您还可以在一个* `*KafkaUser*` *定义中定义授权规则(本博客未涉及)——参见*[*https://strim zi . io/docs/operators/master/using . html # type-KafkaUser-reference*](https://strimzi.io/docs/operators/master/using.html#type-KafkaUser-reference)

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/user-tls-auth.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/user-tls-auth.yaml)
```

自省`Secret`(与`KafkaUser`同名)；

```
kubectl get secret/kafka-tls-client-credentials -o yaml
```

# TLS 客户端身份验证

就是这样！现在由客户机来使用凭证。我们将使用 Kafka CLI 和 Go 客户端应用程序对此进行测试。首先要做的是:

**提取并配置用户凭证**

```
export KAFKA_USER_NAME=kafka-tls-client-credentialskubectl get secret $KAFKA_USER_NAME -o jsonpath='{.data.user\.crt}' | base64 --decode > user.crtkubectl get secret $KAFKA_USER_NAME -o jsonpath='{.data.user\.key}' | base64 --decode > user.keykubectl get secret $KAFKA_USER_NAME -o jsonpath='{.data.user\.p12}' | base64 --decode > user.p12kubectl get secret $KAFKA_USER_NAME -o jsonpath='{.data.user\.password}' | base64 --decode > user.password
```

将`user.p12`中的条目导入到另一个密钥库中

```
export USER_P12_FILE_PATH=user.p12
export USER_KEY_PASSWORD_FILE_PATH=user.password
export KEYSTORE_NAME=kafka-auth-keystore.jks
export KEYSTORE_PASSWORD=foobar
export PASSWORD=`cat $USER_KEY_PASSWORD_FILE_PATH`sudo keytool -importkeystore -deststorepass $KEYSTORE_PASSWORD -destkeystore $KEYSTORE_NAME -srckeystore $USER_P12_FILE_PATH -srcstorepass $PASSWORD -srcstoretype PKCS12sudo keytool -list -alias $KAFKA_USER_NAME -keystore $KEYSTORE_NAME
```

就像我们在第 2 部分中所做的一样，TLS 加密配置需要在客户端信任库中导入集群 CA 证书

**提取并配置服务器 CA 证书**

提取群集 CA 证书和密码

```
export CLUSTER_NAME=my-kafka-clusterkubectl get secret $CLUSTER_NAME-cluster-ca-cert -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca.crtkubectl get secret $CLUSTER_NAME-cluster-ca-cert -o jsonpath='{.data.ca\.password}' | base64 --decode > ca.password
```

将它导入`truststore` -我使用的是 JDK (Java)安装自带的内置信任库-但这只是为了方便，你可以自由使用其他信任库

```
export CERT_FILE_PATH=ca.crt
export CERT_PASSWORD_FILE_PATH=ca.password# replace this with the path to your truststoreexport KEYSTORE_LOCATION=/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home/jre/lib/security/cacerts
export PASSWORD=`cat $CERT_PASSWORD_FILE_PATH`# you will prompted for the truststore password. for JDK truststore, the default password is "changeit"
# Type yes in response to the 'Trust this certificate? [no]:' promptsudo keytool -importcert -alias strimzi-kafka-cert -file $CERT_FILE_PATH -keystore $KEYSTORE_LOCATION -keypass $PASSWORDsudo keytool -list -alias strimzi-kafka-cert -keystore $KEYSTORE_LOCATION
```

现在，您应该能够使用 Kafka CLI 客户端对 Kafka 集群进行身份验证

> *请注意，下面详述的 Kafka CLI 配置步骤也适用于 Java 客户端——也可以随意尝试一下*

**为 Kafka CLI 客户端创建属性文件**

提取 Kafka 集群的`LoadBalancer`公共 IP

```
export KAFKA_CLUSTER_NAME=my-kafka-clusterkubectl get service/${KAFKA_CLUSTER_NAME}-kafka-external-bootstrap --output=jsonpath={.status.loadBalancer.ingress[0].ip}
```

用以下内容创建一个名为`client-ssl-auth.properties`的文件:

```
bootstrap.servers=[LOADBALANCER_PUBLIC_IP]:9094
security.protocol=SSL
ssl.truststore.location=[TRUSTSTORE_LOCATION]
ssl.truststore.password=changeit
ssl.keystore.location=kafka-auth-keystore.jks
ssl.keystore.password=foobar
ssl.key.password=[contents of user.password file]
```

> `*changeit*` *是默认的信任库密码。如果需要，请使用另一个*

如果你还没有卡夫卡，请下载—[https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)

继续之前还有最后一件事

**创建一个**

正如我前面提到的，主题操作符使得以`KafkaTopic`清单的形式嵌入主题信息成为可能，如下所示:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: strimzi-test-topic
  labels:
    strimzi.io/cluster: my-kafka-cluster
spec:
  partitions: 3
  replicas: 1
```

要创建主题:

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/topic.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/topic.yaml)
```

> *这里参考一个*`*KafkaTopic*`*CRD*[*https://strim zi . io/docs/operators/master/using . html # type-KafkaTopic-reference*](https://strimzi.io/docs/operators/master/using.html#type-KafkaTopic-reference)

您所需要做的就是通过将`kafka-console-producer`和`kafka-console-consumer`指向您刚刚创建的`client-ssl-auth.properties`文件来使用它们

```
export KAFKA_HOME=[replace with kafka installation] e.g. /Users/foobar/kafka_2.12-2.3.0
export LOADBALANCER_PUBLIC_IP=[replace with public-ip]
export TOPIC_NAME=strimzi-test-topic# on a terminal, start producer and send a few messages
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list $LOADBALANCER_PUBLIC_IP:9094 --topic $TOPIC_NAME --producer.config client-ssl-auth.properties# on another terminal, start consumer
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server $LOADBALANCER_PUBLIC_IP:9094 --topic $TOPIC_NAME --consumer.config client-ssl-auth.properties --from-beginning
```

你应该看到生产者和消费者协同工作。太好了！

> *如果您遇到 SSL 握手错误，请检查密钥和证书是否已正确导入，以及您是否使用了正确的密码。如果 Kafka 集群不可达，请确保您对公共 IP 使用了正确的值*

现在，让我们尝试一个编程式客户端。因为 Java 客户端行为(必需的配置属性)与 CLI 相同，所以我使用 Go 客户端来尝试一些不同的东西。不要担心，如果你不是一个围棋程序员，这应该很容易理解。

我不会介绍整个程序，只介绍创建连接相关配置的部分。以下是片段:

```
bootstrapServers = os.Getenv("KAFKA_BOOTSTRAP_SERVERS")
caLocation = os.Getenv("CA_CERT_LOCATION")
topic = os.Getenv("KAFKA_TOPIC")userCertLocation = os.Getenv("USER_CERT_LOCATION")
userKeyLocation = os.Getenv("USER_KEY_LOCATION")
userKeyPassword = os.Getenv("USER_KEY_PASSWORD")producerConfig := &kafka.ConfigMap{"bootstrap.servers": bootstrapServers, "security.protocol": "SSL", "ssl.ca.location": caLocation, "ssl.certificate.location": userCertLocation, "ssl.key.location": userKeyLocation, "ssl.key.password": userKeyPassword}
```

请注意，`bootstrap.servers`和`security.protocol`与您在 Kafka CLI 客户端中使用的相同(对于 Java 也是如此)。

*   对于 TLS 加密:`ssl.ca.location`用于直接指向 CA 证书，而不是信任库
*   客户端认证:`ssl.certificate.location`、`ssl.key.location`和`ssl.key.password`分别指用户证书、用户密钥和密码

如果你已经安装了`Go`，可以尝试一下。克隆 Git repo

```
git clone https://github.com/abhirockzz/kafka-kubernetes-strimzi
cd part-3/go-client-app
```

..并运行程序:

```
export KAFKA_BOOTSTRAP_SERVERS=[replace with public-ip:9094] e.g. 20.43.176.7:9094
export CA_CERT_LOCATION=[replace with location of ca.crt file] e.g. /Users/code/kafka-kubernetes-strimzi/part-3/ca.crt
export KAFKA_TOPIC=test-strimzi-topicexport USER_CERT_LOCATION=[path to user.crt file] e.g. /Users/code/kafka-kubernetes-strimzi/part-3/user.crt
export USER_KEY_LOCATION=[path to user.key file] e.g. /Users/code/kafka-kubernetes-strimzi/part-3/user.key
export USER_KEY_PASSWORD=[contents of user.password file]go run kafka-tls-auth-client.go
```

日志应该确认消息是否被生成和使用

# 强制紧急停堆-SHA-512 授权

`SCRAM`代表“Salted Challenge Response 认证机制”。我不会假装是安全专家或`SCRAM`专家，但是我想强调的是，它是 Kafka 中受支持的和常用的认证机制之一(除此之外还有其他如`PLAIN`

> *请注意，在编写*时，Strimzi 不支持 `*SASL PLAIN*` *auth*

**更新卡夫卡集群**

要应用`SCRAM`认证方案——您需要做的就是将`authentication.type`设置为`scram-sha-512`

```
external:
  type: loadbalancer
  tls: true
  authentication:
    **type: scram-sha-512**
```

更新 Kafka 集群以使用`SCRAM-SHA`认证

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/kafka-tls-auth.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/kafka-tls-auth.yaml)
```

让我们看看 Kafka 服务器配置在这种情况下是什么样子的:

```
export CLUSTER_NAME=my-kafka-cluster
kubectl get configmap/${CLUSTER_NAME}-kafka-config -o yaml
```

检查`server.config`中的`External listener`部分，注意配置是如何更新的

```
listener.name.external-9094.scram-sha-512.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;

listener.name.external-9094.sasl.enabled.mechanisms=SCRAM-SHA-512
```

**创建急停凭证(** `**KafkaUser**` **)**

就像我们对`TLS` auth 所做的一样，我们也需要为`SCRAM`创建客户端凭证。它只是在名称和类型上不同于它的 TLS 等价物(当然！)

```
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
  name: kafka-scram-client-credentials
  labels:
    strimzi.io/cluster: my-kafka-cluster
spec:
  authentication:
    **type: scram-sha-512**
```

> *注意，* `*authentication.type*` *是* `*scram-sha-512*`

创建`KafkaUser`

```
kubectl apply -f [https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/user-scram-auth.yaml](https://raw.githubusercontent.com/abhirockzz/kafka-kubernetes-strimzi/master/part-3/user-scram-auth.yaml)
```

自省`Secret`(与`KafkaUser`同名):

```
kubectl get secret/kafka-scram-client-credentials -o yaml
```

`Secret`包含`base64`编码形式的`password`

```
apiVersion: v1
kind: Secret
name: kafka-scram-client-credentials
data:
  password: SnpteEQwek1DNkdi
...
```

> *用户名与* `*KafkaUser*` */* `*Secret*` *名称相同，在本例中为*`*kafka-scram-client-credentials*`

***运行客户端应用***

*为了运行客户端示例，请下载密码:*

```
*export USER_NAME=kafka-scram-client-credentialskubectl get secret $USER_NAME -o jsonpath='{.data.password}' | base64 --decode > user-scram.password*
```

*为了测试 Kafka CLI 客户端，创建一个包含以下内容的文件`client-scram-auth.properties`:*

```
*bootstrap.servers=[replace with public-ip:9094]
security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512
ssl.truststore.location=[replace with path to truststore with kafka CA cert]# "changeit" is the default password for JDK truststore, please use the one applicable to yoursssl.truststore.password=changeitsasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="kafka-scram-client-credentials" password="[replace with contents of user-scram.password file]";*
```

*请参考上面的说明来运行控制台生产者和消费者*

> **请确保你使用的是* `*client-scram-auth.properties*` *而不是* `*client-tls-auth.properties*` *文件**

*在结束之前，让我们看一下 Go 客户端，看看它是如何处理`SCRAM`认证的。和往常一样，我将只强调展示配置的部分:*

```
*bootstrapServers = os.Getenv("KAFKA_BOOTSTRAP_SERVERS")
caLocation = os.Getenv("CA_CERT_LOCATION")
topic = os.Getenv("KAFKA_TOPIC")kafkaScramUsername = os.Getenv("SCRAM_USERNAME")
kafkaScramPassword = os.Getenv("SCRAM_PASSWORD")producerConfig := &kafka.ConfigMap{"bootstrap.servers": bootstrapServers, "security.protocol": "SASL_SSL", "ssl.ca.location": caLocation, "sasl.mechanism": "SCRAM-SHA-512", "sasl.username": kafkaScramUsername, "sasl.password": kafkaScramPassword}*
```

*`security.protocol`和`sasl.mechanism`分别更新为`SASL_SSL`和`SCRAM-SHA-512`。同时，我们使用`sasl.username`和`sasl.password`来指定客户端凭证*

*运行`Go`客户端应用程序:*

```
*export KAFKA_BOOTSTRAP_SERVERS=[replace with public-ip:9094]
export CA_CERT_LOCATION=[path to ca.crt file] f.g. /Users/code/kafka-kubernetes-strimzi/part-3/ca.crt
export KAFKA_TOPIC=strimzi-test-topicexport SCRAM_USERNAME=kafka-scram-client-credentials
export SCRAM_PASSWORD=[contents of user-scram.password file]go run kafka-scram-auth-client.go*
```

# *包裹..目前*

*这篇文章覆盖了相当多的领域！我们学习了如何应用不同的身份验证类型，使用实体操作符来管理 Kafka 用户和主题，更重要的是，了解了如何配置客户端应用程序，以使用 TLS 加密和所选身份验证方案的组合来安全连接。*

*我们还远没有结束！与此同时，我们一直在创建没有持久性的`ephemeral`集群——我们将在接下来的文章中修复这个问题。*