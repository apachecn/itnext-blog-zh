# 一个容器中的多个进程

> 原文：<https://itnext.io/multiple-processes-in-one-container-73d5fa91d5cf?source=collection_archive---------8----------------------->

![](img/aed99f56a596965d8a164338117301ea.png)

保罗·德拉古纳斯在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

我经常需要在一个容器中包含多个进程。例如，一个用于开发目的的 Kafka 实例或一个 Elasticsearch 实例——如果你在微服务架构中开发服务，你就知道我的问题了！—但最新版本的 Kafka 或 Elasticsearch 需要一个 Zookeeper 实例才能启动。非常令人沮丧的是，你需要一个 docker-compose 只是为了做一个测试！

在大多数情况下，你不需要一个集群来测试你的工作，只需要一个代理(例如)，所以我创建了一个包含 Zookeeper 和 Kafka(版本 2.12-2.3.0)的 docker 映像，这个容器启动了一个管理进程生命的*主管*[http://supervisord.org/](http://supervisord.org/)的实例。

我将一步一步地告诉你如何做:

从 https://kafka.apache.org/,[下载卡夫卡的稳定版本](https://kafka.apache.org/](https://kafka.apache.org/),)然后:

*   在第一行中，我想使用 *adoptopenjdk* 作为基础映像，因为它是基于 debian 的(我想使用 **apt** )并且已经安装了有效的 openjdk
*   将 *start.sh* 文件复制到根目录下(详情如下)
*   需要一个 *apt 更新*来同步软件包管理器库
*   然后安装 supervisor 并添加 Kafka 的副本，并复制下面的 *supervisor.conf* 文件

*start.sh* 管理用于配置 KAFKA 广告 url 的环境变量*KAFKA _ ADVERTISED _ listeners*。

在 *start.sh* 结束时，它启动*监管守护进程*，配置如下:

主管启动动物园管理员和一个卡夫卡经纪人:仅此而已。
容器现在运行的是 Zookeeper 和 Kafka 的有效实例，你可以绑定标准端口来与 Kafka 和 Zookeeper 交互。

## 建设

```
*docker build -t paspaola/kafka-one-container .*
```

## 奔跑

```
*docker run -it -p2181:2181 -p9092:9092 -e KAFKA_ADVERTISED_LISTNERS=****{your-host-address}****” paspaola/kafka-one-container*
```

## 贮藏室ˌ仓库

[https://github.com/paspao/kafka-in-one-container](https://github.com/paspao/kafka-in-one-container](https://github.com/paspao/kafka-in-one-container))