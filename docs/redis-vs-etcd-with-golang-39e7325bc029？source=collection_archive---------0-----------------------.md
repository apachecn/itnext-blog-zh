# Redis vs etcd 与 Golang

> 原文：<https://itnext.io/redis-vs-etcd-with-golang-39e7325bc029?source=collection_archive---------0----------------------->

![](img/f5cc949001c1e68fa3224836f1fef4ee.png)

克洛维斯伍德摄影在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

今天我们将比较 Redis 和 etcd。
让我们比较一下它们的性能。考虑它们的应用领域-案例。

Redis —内存中的数据结构存储，用作数据库、缓存和消息代理。它通常被称为数据结构服务器，因为键可以包含字符串、散列、列表、集合和排序集合。

Etcd —是一个高度一致的分布式键值存储，它提供了一种可靠的方式来存储需要由分布式系统或机器集群访问的数据。etcd 在网络分区期间优雅地处理主设备选举，并容忍机器故障，包括主设备故障。

Redis 的优点
-性能
-内存缓存
-高级键值缓存
-开源
-简单易用

etcd 的优点
-服务发现
-容错键值存储
-与 coreos 捆绑
-开源
-特权访问管理
-安全

Redis 有优秀的文件[https://redis.io/](https://redis.io/)。几年来，Redis 一直是全世界开发者的首选工具。
与其他键值不同，Redis 数据类型包括字符串、列表、集合、存储集、哈希、位图、超级日志、流、地理空间。

Redis 的用例非常多样:
-缓存
-聊天、消息和队列
-会话存储
-媒体流
-地理空间
-机器学习

etcd 拥有现代文献[https://etcd.io/](https://etcd.io/)。
开箱即用 etcd 有 HTTP API(工具)。您的应用程序可以“监视”特定键或目录的变化，并对值的变化做出反应。

许多组织使用 etcd 来实现生产系统
——所有的 Kubernetes 系统。
- OpenTable —内部服务发现和集群配置管理。【cycoresys.com —为计算系统提供架构和工程设计。
- Radius 智能—多种内部工具、Kubernetes 集群、可引导系统配置。
- Vonage — kubernetes、vault 后端、微服务的系统配置、调度、锁(未来—服务发现)。
- PD — PD(布局驱动程序)是 TiDB 集群中的中央控制器。
-华为——覆盖网络(Canal)系统配置。
-七牛云—微服务、分布式锁的系统配置。
- QingCloud — appcenter 集群，用于作为 metad 后端的服务发现。
- Yandex —服务的系统配置，服务发现。
-腾讯游戏-服务发现、Kubernetes 等的元数据和配置数据。
- Hyper.sh — Kubernetes、分布式锁等。
-美图—测试环境中服务、服务发现、Kubernetes 的系统配置。
- Grab —服务的系统配置，服务发现。
- DaoCloud.io —容器管理。
-branch . io-在 Kubernetes 系统中。
-百度外卖 SkyDNS、Kubernetes、UDC、CMDB 等分布式系统。
-Salesforce.com——在库伯内特系统。
-托管石墨——服务发现、锁定、短暂应用数据。
-Transwarp——Transwarp 数据云，trans warp 操作系统，trans warp 数据中枢，Sophon。

此外，etcd 是一个 CNCF 项目。这真的很酷。

Etcd 用例:
-core OS 的容器 Linux。
——库伯内特系统公司。
-适用于各种应用的快速、分布式、容错和简单配置系统。

更多关于用例及使用领域的信息，请参见第[页 https://etcd.io/docs/v3.3/learning/why/](https://etcd.io/docs/v3.3/learning/why/)页。

# 让我们从一个基本的 Redis + Golang 应用程序开始。

当然，您可以很容易地在本地或您的主机上开始使用它。
例如，用几个命令在本地用 docker 运行 Redis:

在您的机器上运行 Redis

现在从 Golang 应用程序连接到 Redis。创建一个真正基本和简单的 golang 应用程序，并连接到 Redis 主机。

初始化 go 应用程序

和“hello-world”基本应用程序本身。

cat ~/my-golang-redis-app/main . go

检查此应用程序的操作。

这是您在 Golang 开始使用 Redis 的全部内容。然后，您应用团队中采用的代码组织实践和一般架构模式。

# 现在谈谈基本的 etcd + Golang 应用程序。

先说一下如何启动 etcd 和 golang 应用。

如果您的机器上还没有 etcd，让我们安装它。用几个命令检查它的工作。同时，让我们直接了解一下它的 cli。

进入官方安装文档页面[https://etcd.io/docs/v3.5/install/](https://etcd.io/docs/v3.5/install/)。

使 main.go 的内容匹配

这些都是概念性的应用。

现在检查一下它将在循环中执行多少次操作—写/读。参见下面代码中的循环:

```
for i := 0; i < cnt; i++ {
 // write/read
}
```

循环读写

最后，我们看到执行 10，000 次写和读操作需要多长时间。
Redis 显示读写时间相同——7 秒以上。
但 etcd 显示它“写”得慢，但读得快得多。
如果我们只比较读操作，那么 etcd 赢了 5 次。

当然，我们“有义务”检查 golang 应用程序的内部操作花费了多少时间和内存，以进行内置基准测试。

主 _ 测试. go

基准结果

正如所料，在这里，我们看到在写入 etcd 时，在资源消耗和时间方面有很大的优势。
与此同时，etcd 在录音方面成为领头羊。

# 极限呢？

你首先要注意的，当然是 Redis 和 etcd 在性能上的限制。
它们包括:
-将这些技术带入团队的资源；
-为各种环境(开发、阶段和生产)维护和恢复(在故障情况下)Redis 和 etcd 所需的资源；
-任何资源运行的一个不可或缺的部分是它们的技术限制
—考虑数据累积的吞吐量(性能)
—以及内存和数据存储的最终限制(每个节点/主机和群集作为一个整体)。

可以为 Redis 分配的限制:
-100 万个小键- >字符串值对使用~ 85MB 内存。
-100 万个键- >哈希值，代表一个有 5 个字段的对象，使用~ 160 MB 内存。
-单个 Redis 实例的最大键数— 2 个键。
实际上，你把自己限制在 RAM。

Etcd 限制:
-任何请求的最大大小(默认)为 1.5 MiB(该限制可通过 etcd 服务器的 max-request-bytes 标志进行配置)。
-存储容量— 8GB。但实际操作中，即使 16GB 也可以使用 etc。超过 20GB 时，您会发现超时。

Redis 和 etcd 当然是不同类的键值存储，目的也不同，但是它们也有显著的交集。
首先是键值存储。如果您需要从存储中读取更多内容，那么您应该仔细看看 etcd。如果你需要经常写/读，并且你想要更多的特性，那么你应该仔细看看 Redis。

就目前的情况来看，大部分开发者和架构都选择 Redis。首先，这是一个熟悉的工具。它允许你扩展你的功能。
但与此同时，我也遇到过几次使用“非传统”Redis 的提议，即 etcd 作为配置存储的键值存储。

顺便说一句，配置应用程序是一个很大的主题，值得在这方面单独写一篇文章。你怎么想？

本报告中所有可用的源代码—[https://github.com/romanitalian/redis_vs_etc_with_go](https://github.com/romanitalian/redis_vs_etc_with_go)。