# 事件采购:为什么 Kafka 不适合作为事件商店

> 原文：<https://itnext.io/event-sourcing-why-kafka-is-not-suitable-as-an-event-store-796e5d9ab63c?source=collection_archive---------0----------------------->

## 不要相信一个公司或者一些顾问想卖给你的一切！

在我们开始之前，如果你对什么是活动采购只有一个粗略的概念，你可以看看 Alexey Zimarev 的这篇[好文章。](https://www.eventstore.com/blog/what-is-event-sourcing)

**更新**:我刚刚发表了自己对 ES 的介绍。

[](https://medium.com/@TonyBologni/event-sourcing-explained-b19ccaa93ae4) [## 事件来源解释

### 互联网上有许多令人困惑的文章，将事件采购与 EDA 和 CQRS 混为一谈。让我们看看它是什么，不是什么。

medium.com](https://medium.com/@TonyBologni/event-sourcing-explained-b19ccaa93ae4) 

又不是没有人写过这个话题。许多事件源、CQRS(命令查询责任分离)和 EDA(事件驱动架构)方面的专家都写过相关的文章。但现在，我刚刚读了另一篇糟糕的文章，它假装 Kafka 是一个微服务架构的伟大事件商店。这似乎需要重复 1000 次，直到每个人都听到它。

你可能读过一个关于大型活动采购/ CQRS“项目”失败的恐怖故事。在大多数文章中，你会发现他们把卡夫卡作为一个事件商店，并大规模地打击粉丝。公平地说，我也读过**的一篇**这样的博文，这篇博文写得非常好，还解释了事件源的优势以及他们将错过的功能，现在他们恢复到完整状态的经典存储，而不是事件。可悲的是，他们得出了错误的结论，并完全抛弃了 ES，而不是仅仅转移到一个合适的活动商店。关于题目的烂文章我看了 20 多遍，走吧。

> *在我继续之前，请注意标题是怎么说“不适合”而不是“完全不可能”的。我稍后将回到这一点。*

# 事件源不是顶级架构

这是格雷格·杨(Greg Young)——活动采购先生和 CQRS 本人——的一句话，这是第一个问题。许多试图将 Kafka 作为事件商店的项目都是围绕 Kafka 构建的。他们试图开发一种“基于事件、事件驱动的微服务架构”。这很有说服力，因为您希望尽可能多地利用您的超级强大而复杂的银弹解决方案，对吗？

Kafka 甚至有一个很好的属性——您只有一个存储/来源来存储您的数据(事件)和消息(发布的事件),这样您就不必解决分布式事务的问题，而其他人可能必须实现*发件箱模式*或找到另一个解决方案。

然而，他们错过的是，现在他们必须事件源的一切，甚至应用程序的一部分，事件源不是一个很好的匹配。如果他们实现了有状态存储的某些部分，他们将一无所获——如果他们不想冒在事情变糟时丢失其中一个或另一个的风险，他们需要解决自动存储数据和消息的问题。

他们还错过的是，这样的解决方案只不过是一个分布式的整体，由死星卡夫卡(Kafka)将他们捆绑在一起。每个服务都可以读取每个其他服务的所有事件，这与在世界中心拥有一个巨大的 Oracle 数据库没有什么不同。最终，一切都将耦合起来，这里的变化*可以打破那里的东西*。**

# **卡夫卡缺乏原子写作**

**想象一个命令/请求/用例导致多个事件。为了保持数据的一致性，您必须原子地存储它们——要么一个都不存储(想想 ACID)。据我所知，Kafka 中没有这种类似事务的机制来保证原子性，所以我们可能会以部分丢失或至少延迟的事件结束，使应用程序处于不一致的状态。**

> ***更新:我错过了卡夫卡现在有某种交易。我快速浏览了一下汇合者的博客*[](https://www.confluent.io/blog/transactions-apache-kafka/)**如何运作。只要看看这篇博文的长度，或者它说选择 transactional.id“有点棘手”的地方。
> 相比之下，使用 EventStoreDB，我只需编写多个事件，无需任何额外的配置或技巧，仅此而已。****

# ***乐观并发控制？***

***人类或其他服务“看到”的每个应用程序状态都已经过时。自从从事件存储(或任何其他存储)请求数据以来，至少已经过去了几纳秒。因此，实现某种并发控制是至关重要的。如果系统的状态在读取状态和保持决策之间发生了变化，那么更新一定无法保证所有的业务规则都得到遵守。***

***通常，最好实现开放式并发控制。这意味着在读取和写入之前不会发生数据锁定，但是写入请求会检测到数据在读取和写入之间发生了更改，并拒绝更新。在大多数系统中，这些并发冲突相对较少，乐观并发控制比悲观锁定数据要便宜得多。***

***Kafka 没有乐观或悲观并发控制的机制。解决这个问题的唯一方法是引入另一种锁，例如 Redis 这样的键值存储。虽然这是可能的，但它容易出错，增加了复杂性，并且否定了用一个卡夫卡来统治所有人的整个(坏)想法。***

> ****更新:我还被告知 Kafka 有一种锁定整个分区的方法，结合上面提到的事务，你可以拥有悲观的并发控制。
> 当我用 PostgreSQL 推出我自己的事件存储时，我们只需要锁定一个事件流(例如，针对一个集合/实体),我敢打赌这比 Kafka 解决方案简单多了。****

# ***事件的全局排序***

***在(部分)基于事件的系统中，您通常希望构建一些读取模型，从多个流或多个事件类型中聚合事件。举个简单的例子，要聚合库存(在仓库或其他地方)的内容，您需要聚合每种类型商品的所有添加和删除商品的事件类型。为了将所有事件正确地投射到任意时间点的正确的当前状态，它们需要处于正确的顺序。假设你想问是否有任何商品曾经低于 X 项。如果事件的顺序是错误的，并且您在最先发生的“ItemsAddedToStock”事件之前投影了“ItemsRemovedFromStock”事件，您可能会得到错误的答案。***

***Kafka 的问题是，它只保证分区内的顺序，而不是跨分区的顺序，这让您不得不以其他方式解决排序问题。同样，现在您需要增加复杂性来解决一个问题，这只是因为您想要一个万能服务。***

# ***所以，活动采购是个坏主意？***

***当然不，你应该喜欢它！！！***

***如果做得正确，用于它所匹配的问题，而不是作为银弹使用，这是一个极好的原则，提供了超越经典的有状态系统的许多好处！这不是本文的主题，但其中一些好处是:***

*   ***领域模型的实现细节具有很大的可变性***
*   ***出色的可调试性——您可以看到哪个事件引入了错误***
*   ***时间旅行——您可以看到系统在过去任何给定时间点的状态***
*   ***时间特性——您的代码不仅可以根据已经发生的事情做出决策，还可以根据事情发生的时间、频率和顺序做出决策***
*   ***自由审计——谁、何时、基于当时的哪个州进行了更改***
*   ***你不需要一个增加意外复杂性的 ORM***

***事件源的核心是存储系统状态的更好的方法！***

***它不是“事件驱动架构”，也不是“微服务”，它与 CQRS 是正交的！***

# ***如果不是卡夫卡，还有什么？***

*   ***https://www.eventstore.com/***
*   ***【https://martendb.io/ ***
*   ***[https://github.com/Eventuous/eventuous](https://github.com/Eventuous/eventuous)***
*   ***[https://eventide-project.org/](https://eventide-project.org/)***
*   ***外面还有很多开源库，只需搜索和评估！***
*   ***滚动您自己的事件存储，可能使用 PostgreSQL 及其强大的 JSONB 数据类型***

> *****警告**:你可能不应该为一个生产系统制作自己的产品，尤其是如果你是活动采购的新手(事实上，我已经做了两次，没有任何问题，但我仍然**不**推荐它)。如果你的团队有时间在一些“学习”会议上这样做，学习 ES 是一个很好的形式。***

# ***你不必相信我的话***

***我已经把上面的内容写出来了。我可能错过了一些要点，我可能稍微偏离了一些。一般来说，不要相信网上某个男人的观点是个好主意。***

***以下是 ES/DDD 社区中一些德高望重的人所说的话:***

*   ***[https://event-driven . io/en/event _ streaming _ is _ not _ event _ sourcing/](https://event-driven.io/en/event_streaming_is_not_event_sourcing/)***
*   ***[https://www.youtube.com/watch?app=desktop&v = Lu-skMQ-vAw](https://www.youtube.com/watch?app=desktop&v=Lu-skMQ-vAw)***
*   ***[https://zimarev . com/blog/event-sourcing/myth-busting/2020-07-09-over selling-event-sourcing/](https://zimarev.com/blog/event-sourcing/myth-busting/2020-07-09-overselling-event-sourcing/)***
*   ***[https://medium . com/serialized-io/Apache-Kafka-is-not-for-event-sourcing-81735 C3 cf 5c](https://medium.com/serialized-io/apache-kafka-is-not-for-event-sourcing-81735c3cf5c)***
*   ***[https://domaincentric.net/blog/eventstoredb-vs-kafka](https://domaincentric.net/blog/eventstoredb-vs-kafka)***
*   ***[https://www.youtube.com/watch?app=desktop&v = y7ca 1-EKsg](https://www.youtube.com/watch?app=desktop&v=Y7ca1--EKsg)***

***还有一个——这是 Oskar 收集的一长串文章和视频——我没有全部看过/看过，但总的来说我们应该相信 Oskar；-)***

***并不是所有的反模式都是关于卡夫卡的，而是一般的反模式***

***[https://github.com/oskardudycz/EventSourcing.NetCore #这不是事件源](https://github.com/oskardudycz/EventSourcing.NetCore#this-is-not-event-sourcing)***

# ***但是，但是…你提到这不是不可能的？***

***我想避免这种情况，但我想公平竞争。这也许是可能的，有大量的[文章像来自 Confluent 的](https://www.confluent.io/blog/event-sourcing-using-apache-kafka/)一样，他们通过让人们把卡夫卡当作万能的杰克来赚钱。它可能在许多项目中起作用，但我敢打赌，遇到严重问题并最终导致你写另一篇“事件采购如何让我们失败”的文章的可能性是非常真实的。一篇像[这篇像](https://medium.com/dna-technology/why-we-dropped-event-sourcing-with-kafka-streams-when-given-a-second-chance-b904a80bc4be)的文章。***

***关键是，为什么你要把时间花在意外的复杂性上，因为你选择了一个不是为工作而构建的工具，而不是选择一个合适的事件存储并解决你的领域的固有问题？或者用阿列克谢的话说:***

# ***TL；博士；医生***

***跟我重复:**卡夫卡不*适合*做活动店！只要使用合适的工具就行了！*****

## ***感谢您的时间和关注！:-)***

> ***非常欢迎提问和评论，如果你为这篇文章鼓掌(如果你喜欢)，或者甚至在 [Medium](https://medium.com/@TonyBologni) 或 [Twitter](https://twitter.com/TonyBologni) 上关注我，我会非常高兴！***

***[](https://medium.com/@TonyBologni/membership) [## 通过我的推荐链接加入 Medium-Anton stckl

### 阅读我和其他伟大的科技作家的每一个故事！你的会费直接支持我和其他…

medium.com](https://medium.com/@TonyBologni/membership)***