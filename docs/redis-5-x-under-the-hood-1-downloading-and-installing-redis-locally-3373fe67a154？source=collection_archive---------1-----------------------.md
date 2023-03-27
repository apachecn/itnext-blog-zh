# Redis 5。幕后 x:1—Redis-服务器启动并运行

> 原文：<https://itnext.io/redis-5-x-under-the-hood-1-downloading-and-installing-redis-locally-3373fe67a154?source=collection_archive---------1----------------------->

![](img/c73f924f301bd2287b3ba10870133e48.png)

Redis 是远程字典服务器的缩写，自从最初使用强大而快速的键值存储以来，它已经走过了漫长的道路。它以其[高性能](https://redis.io/topics/benchmarks)、多样化的数据类型、高可用和可扩展的架构越来越受到关注。

它在[数据库引擎完整排名](https://db-engines.com/en/ranking)的前 10 个数据库中排名第 7，是前 10 名中唯一的键值。最近，[还将](https://www.prnewswire.com/news-releases/redis-labs-delivers-the-fastest-multi-model-database-on-intel-optane-dc-persistent-memory-300775789.html)认证为多模型数据库，支持针对单个后端的多个数据模型。

作为[meetup 组织者](https://www.meetup.com/en-AU/Redis-Porto/)，我们决定记录这一相互学习的旅程，以更好地了解核心 Redis 5 及其顶级扩展如何工作，讨论这一新版本中的新功能，因此阅读这一系列文章并参加我们一般会议的人可以就它适合和不适合的不同用例做出明智的决定。

我们的旅程将从头开始，从设置本地实例到让多主集群运行在云上不同的可用性区域。到本系列结束时，您应该对在类似生产的环境中运行 Redis 有了深入的了解。

本文档将在每篇文章发布时更新，以跟踪系列文章，其结构如下:

第 1 篇— [本地下载安装 Redis](https://medium.com/@fcosta_oliveira/redis-5-x-under-the-hood-1-downloading-and-installing-redis-locally-3373fe67a154)，包括 Redis 本地服务器的安装和基本操作，如启动和关闭 Redis 服务器，用 redis-cli 连接 Redis，获取服务器信息，了解一些关键的默认配置，如数据持久化。

第 2 篇文章[Redis 命令和数据结构介绍——第 1 部分](https://medium.com/@fcosta_oliveira/redis-5-x-under-the-hood-2-intro-to-redis-commands-and-data-structures-part-1-41f05501cb52)，讲述了为什么在使用 Redis 时需要考虑如何存储、访问和转换数据，同时还讨论了 STRING 数据类型。

# 1 —在本地下载和安装 Redis

如果可能的话，使用最新版本的 Redis 是最佳实践之一。最新的 Redis 主版本(5.0.0)于 2018 年 10 月 17 日在 GitHub 上发布。在这个系列中，我们将采用 2018 年 12 月 12 日推出的最新稳定版本(5.0.3)。

## 1.1 —使用哪种操作系统:

Linux 和 OS X 是 Redis 开发和测试较多的两个操作系统，我们**推荐使用 Linux 部署**。

关于 Windows 操作系统没有官方支持，但是微软开发并维护了一个 [Win-64 的 Redis](https://github.com/MSOpenTech/redis) 端口。

## 1.2 —启动和运行:

我们推荐两种在您的机器上启动和运行 Redis 的方法:

## 1.2.1 —选项 1 —使用 docker 运行(最简单的方式):

如果你的机器上有 docker，这是设置测试环境的最简单的方式:

```
docker run -p 6379:6379 --name redis5-under-the-hood -d redis:[5.0.3](https://github.com/docker-library/redis/blob/7be79f51e29a009fefdc218c8479d340b8c4a5e1/5.0/Dockerfile)
sudo apt install redis-tools
redis-cli 
127.0.0.1:6379>
```

## 1.2.2 —选项 2 —建立 Redis:

如果您想构建 Redis，只需在终端上执行以下命令:

```
wget [http://download.redis.io/releases/redis-5.0.3.tar.gz](http://download.redis.io/releases/redis-5.0.3.tar.gz)
tar xvzf redis-5.0.3.tar.gz
cd redis-5.0.3
make
make test
sudo make install
```

如果一切顺利，在最后一个命令之后，将出现以下消息:

```
Hint: It's a good idea to run 'make test' ;)**INSTALL** **install
INSTALL** **install
INSTALL** **install
INSTALL** **install
INSTALL** **install**
```

要使用内置的默认配置运行 Redis 进行测试(并让它在后台运行),只需输入:

```
redis-server &
```

## 1.3 —通过 Redis 配置文件配置 Redis:

正如在前面的步骤中所看到的，Redis 能够使用内置的默认配置在没有配置文件的情况下启动，但是，这种设置仅推荐用于测试和开发目的。

配置 Redis 的正确方法是提供一个 Redis 配置文件，通常称为`[redis.conf](http://download.redis.io/redis-stable/redis.conf)`。在那里，您应该能够配置:

*   包括设置
*   网络设置
*   常规设置
*   快照设置
*   复制设置
*   安全设置
*   限制设置
*   Lua 脚本设置
*   聚类设置
*   其他高级设置

以下链接应提供更多关于配置文件结构方式的详细信息[https://redis.io/topics/config](https://redis.io/topics/config)。

所有这些设置将在后续文章中讨论。现在，让我们保留文件`[redis.conf](http://download.redis.io/redis-stable/redis.conf)`的默认设置，看看如何使用 docker 或构建的 Redis 运行 Redis。为此，我们必须使用一个额外的参数/ **路径/to/redis.conf** (配置文件的路径)来运行它。

## 1.3.1 —选项 1.1 —使用带有 redis.conf 文件的 docker 运行:

```
docker run -v **/path/to****/redis.conf**:/usr/local/etc/redis/redis.conf --name redis5-under-the-hood-redis-conf -d redis:[5.0.3](https://github.com/docker-library/redis/blob/7be79f51e29a009fefdc218c8479d340b8c4a5e1/5.0/Dockerfile) redis-server /usr/local/etc/redis/redis.conf
```

## 1.3.1 —选项 2.1 —使用 redis.conf 文件运行构建的 Redis:

```
redis-server **/path/to****/redis.conf** &
```

`[redis.conf](http://download.redis.io/redis-stable/redis.conf)`中的所有选项也支持作为使用命令行的选项，名称完全相同。例如，如果我们希望 Redis 接受端口 9999 上的连接，而不是默认的 6379，我们应该这样做:

```
redis-server **/path/to****/redis.conf** --port 9999 &
```

请记住，**如果你传递一个** `[**redis.conf**](http://download.redis.io/redis-stable/redis.conf)` **文件，它应该总是第一个参数。**

## 1.4 —初步了解 Redis 如何保存数据:

既然我们已经建立并运行了一个测试 Redis 服务器，让我们快速地向您介绍 Redis 如何持久化数据。我们将用一整篇文章来讨论 Redis 中数据持久化的优点和缺点，以及实现数据持久化的几种方法，但是现在，让我们向您介绍一下这个主题。

在`[**redis.conf**](http://download.redis.io/redis-stable/redis.conf)` **上启用的数据持久性的默认方法是**通过将数据库快照到本地磁盘，它以指定的时间间隔执行数据集的时间点快照，默认方式如下:

*   **900 秒(15 分钟)后，如果至少有一个键发生变化**
*   **300 秒(5 分钟)后，如果至少 10 个键发生变化**
*   **60 秒后，如果至少 10000 个键发生变化**

如果在发生事故的情况下，几分钟的数据丢失是不可接受的 ，Redis 快照 ***不能提供良好的耐久性保证，因此它的使用仅限于丢失最近数据不是很重要的应用。***

Lyft 下面这个 redisconf 2018 演讲链接:[https://www . slide share . net/redis labs/redis conf 18-2000-instances-and-beyond](https://www.slideshare.net/RedisLabs/redisconf18-2000-instances-and-beyond)。

Redis 提供了一种更高级的持久性模式，称为仅附加文件，通常简称为 AOF，它记录服务器收到的每个写操作。我们将在后面讨论利弊。现在，请记住这两种模式都有利弊。

## 1.5 —停止 Redis:

要停止 Redis，最优雅的方法是执行 [**shutdown**](https://redis.io/commands/SHUTDOWN) 命令。

```
redis-cli
127.0.0.1:6379> shutdown
```

如果启用了持久性，这个命令可以确保 Redis 被关闭而不会丢失任何数据。

## 1.6 —后续步骤:

现在我们已经有了一个正常运行的 Redis 本地服务器，我们可以开始研究 hood 了。我们的下一篇文章将是对 Redis 命令和数据结构 的**，让您对可用的数据类型有一个基本的了解，以及它们如何在实际例子中有用。**