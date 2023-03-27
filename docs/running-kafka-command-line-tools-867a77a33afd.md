# 运行 Kafka 命令行工具

> 原文：<https://itnext.io/running-kafka-command-line-tools-867a77a33afd?source=collection_archive---------2----------------------->

## 事件流身份验证和授权

![](img/2afa439b02d44f51e6bf37361daa83c6.png)

Kafka 提供了一套丰富的命令行工具来管理主题和集群。最新 IBM 事件流的默认设置(基于 Strimzi 操作符)给运行这些工具带来了一些挑战。

## 代理监听程序

当 Kafka 资源作为操作员部署时，通常会应用强安全配置。请看下面的集群监听器设置摘录

```
listeners:
  external:
    authentication:
      type: scram-sha-512
    type: route
  tls:
    authentication:
      type: tls
```

经纪人将会监听

1.  端口 9094 用于 SRAM-SHA-512 认证的外部连接。
2.  用于内部通信的端口 9093，其中使用了 MTL。这里的 m(mutual)表示客户端将对代理进行身份验证，代理也将对客户端进行身份验证。

由于只定义了这些侦听器，Kafka 命令行常用的普通端口 9092 将不起作用。

## KafkaUser 资源—客户端验证和授权

由于 IBM event streams 打开了授权，一旦用户通过了身份验证，就需要授权它对主题、消费者组和 Kafka 集群执行任何操作。在操作员模型中，这是通过 KafkaUser 资源实现的。

让我们创建以下 KafkaUser 资源

```
apiVersion: eventstreams.ibm.com/v1beta1
kind: KafkaUser
metadata:
  name: super-user-tls
  labels:
    eventstreams.ibm.com/cluster: es
spec:
  authentication:
    type: tls
  authorization:
    type: simple
    acls:
      - resource:
          type: topic
          name: '*'
          patternType: literal
        operation: All
      - resource:
          type: cluster
          name: '*'
          patternType: literal
        operation: All
      - resource:
          type: group
          name: '*'
          patternType: literal
        operation: All
```

有两个折叠，一个是认证，一个是授权。

**客户端认证**

名为 super-user-tls 的用户将由 mTLS 进行身份验证。一旦这个用户被创建，同时，一个同名的秘密将被创建。秘密是这个用户的证书和密钥。看看这个，

```
$ kubectl -n eventstreams describe secret super-user-tls
Name:         super-user-tls
Namespace:    eventstreams
Labels:       app.kubernetes.io/instance=super-user-tls
              app.kubernetes.io/managed-by=strimzi-user-operator
              app.kubernetes.io/name=strimzi-user-operator
              app.kubernetes.io/part-of=eventstreams-super-user-tls
              eventstreams.ibm.com/cluster=es
              eventstreams.ibm.com/kind=KafkaUser
Annotations:  <none>Type:  OpaqueData
====
user.p12:       2402 bytes
user.password:  12 bytes
ca.crt:         1164 bytes
user.crt:       1017 bytes
user.key:       1704 bytes
```

让我们打印出用户证书

```
$ kubectl -n eventstreams get secret super-user-tls -o jsonpath='{.data.user\.crt}' | base64 -di | openssl x509 -text | head -15Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            8a:3f:a0:88:9a:cd:bf:78
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: O = io.strimzi, CN = clients-ca v0
        Validity
            Not Before: Dec  8 15:39:15 2020 GMT
            Not After : Dec  8 15:39:15 2021 GMT
        Subject: **CN = super-user-tls**
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
```

当客户端试图通过代理使用此证书进行身份验证时，由于作为客户端 CA 的签名者受到代理的信任，因此由客户端 CA 签名的证书将会成功通过身份验证。该用户将被标识为 CN 名称 super-user-tls。

另一方面，经纪人的证书也必须经过验证，因为它是双向的。为此，必须将另一个 CA(集群 CA)呈现给客户端工具，以便客户端可以基于该 CA 验证代理的证书。

**客户授权**

KafkaUser 资源的另一半是分配给该用户的授权。Kafka 主题、消费群和集群都有非常精细的 ACL 控制。例如，主题有读、写、创建、描述等操作。

出于测试目的，让我们通过将资源名称设置为*并将操作设置为 all 来创建一个拥有所有权限的超级用户。

## Kafka 命令行工具的 K8s 部署

让我们创建一个 K8s 部署，在一个独立的 pod 而不是代理中运行 Kafka 命令行工具。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kcmd
  namespace: eventstreams
  labels:
    app: kcmd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kcmd
  template:
    metadata:
      labels:
        app: kcmd
    spec:
      containers:
      - name: kcmd
        image: cp.icr.io/cp/ibm-eventstreams-kafka@sha256:d116a3c77b2290bc9fb7dfe54ceda9ce4df46853d3446a0bfb1f114eab555763
        command:
        - sh
        - -c
        - while true; do sleep 100; done
        volumeMounts:
        - name: certs
          mountPath: /certs
        - name: cluster-ca
          mountPath: /clusterCA volumes:
      - name: certs
        secret:
          secretName: super-user-tls
      - name: cluster-ca
        secret:
          secretName: es-cluster-ca-cert
```

使用与所有 Kafka 命令行工具都可用的代理相同的图像。容器命令只是一个等待的虚拟睡眠循环，我们可以执行 pods 来运行实际的 Kafka 工具。

注意安装在豆荚里的秘密。第一个是用户的证书密码，pkcs12 格式可用。证书将由信任客户端 CA 的代理进行身份验证。另一方面，为了验证代理的证书，集群 CA 必须安装到 pod 中。

Exec 进入 pod，使用以下命令创建属性文件，

```
echo security.protocol=SSL > /tmp/ssl.propertiesecho ssl.truststore.type=PKCS12 >> /tmp/ssl.properties
echo ssl.truststore.location=/clusterCA/ca.p12 >> /tmp/ssl.properties
echo ssl.truststore.password=$(cat /clusterCA/ca.password) >> /tmp/ssl.propertiesecho ssl.keystore.type=PKCS12 >> /tmp/ssl.properties
echo ssl.keystore.location=/certs/user.p12 >> /tmp/ssl.properties
echo ssl.keystore.password=$(cat /certs/user.password)  >> /tmp/ssl.properties
```

这个属性文件将作为参数提供给不同的 Kafka 命令行工具。下面显示了一个示例，

```
security.protocol=SSL
ssl.truststore.type=PKCS12
ssl.truststore.location=/clusterCA/ca.p12
ssl.truststore.password=LqEdssNJKmP2
ssl.keystore.type=PKCS12
ssl.keystore.location=/certs/user.p12
ssl.keystore.password=kEH9oqXfRRpa
```

keystore 是存储用户证书的地方，证书将被发送到代理，以对用户进行相应的认证和授权。信任库用于客户端验证代理的证书。所以使用集群 CA。所有证书和密钥都是 PKCS12 格式。

## 运行 Kafka 工具

准备好属性文件后，我们就可以运行 Kafka 命令行工具了

我们来描述一个话题，

```
export POD=$(oc -n eventstreams get pods | grep Running | grep kcmd | awk '{print $1}')oc exec -i $POD -- sh -c "bin/kafka-topics.sh --describe --bootstrap-server es-kafka-bootstrap.eventstreams.svc:9093 --topic demo --command-config /tmp/ssl.properties"
```

请注意，我们传入了 bootstrap-server 参数，其中包含 bootstrap 的 K8s 服务名和内部 TLS 端口 9093。命令配置被定义到属性文件所在的位置。

显示了结果，

```
Topic: demo PartitionCount: 3 ReplicationFactor: 3 Configs: min.insync.replicas=2,retention.ms=604800000,message.format.version=2.6-IV0
 Topic: demo Partition: 0 Leader: 2 Replicas: 2,1,0 Isr: 2,1,0
 Topic: demo Partition: 1 Leader: 1 Replicas: 1,0,2 Isr: 2,0,1
 Topic: demo Partition: 2 Leader: 0 Replicas: 0,2,1 Isr: 2,1,0
```

在需要集群操作授权的情况下，我们还可以触发分区领导者的选举，

```
bin/kafka-leader-election.sh --bootstrap-server es-kafka-bootstrap.eventstreams.svc:9093 --topic demo --partition 0 --election-type preferred --admin.config  /tmp/ssl.properties
```

命令输出如下所示。这并不奇怪，因为领导者应该是这样的。

```
Valid replica already elected for partitions
```

如果我们重新启动一个代理，而不等待默认的 5 分钟来进行领导者选举，我们可以在代理启动后强制进行选举，再次运行命令，

```
Successfully completed leader election (PREFERRED) for partitions demo-0
```