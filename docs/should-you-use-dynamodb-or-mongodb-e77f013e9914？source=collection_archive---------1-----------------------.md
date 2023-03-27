# 应该用 DynamoDB 还是 MongoDB？

> 原文：<https://itnext.io/should-you-use-dynamodb-or-mongodb-e77f013e9914?source=collection_archive---------1----------------------->

## 从开发者的角度比较 NoSQL。

在我的职业生涯中，我大部分时间都在使用 MongoDB，所以我决定在我当前的项目中使用 DynamoDB。这是我的发现。

![](img/46df003bbb435dd9b850c807d7801d7e.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [Max Duzij](https://unsplash.com/@max_duz?utm_source=medium&utm_medium=referral) 拍照

# 设置

DynamoDB 或 MongoDB 的设置相对容易。

## DynamoDB 设置

设置 DynamoDB 与其他 Amazon 服务(如 EC2)协同工作非常简单。访问数据库不需要 AWS 键。相反，为 EC2 实例分配正确的 DynamoDB IAM 角色，它将自动拥有对数据库的正确访问权。

到 DynamoDB 的连接代码再简单不过了，如下面的代码示例所示。

## MongoDB 设置

使用 MongoDB 设置，我们最终会得到稍微多一点的代码语句，但是设置仍然非常简单，正如我们在下面的例子中看到的。

主要区别是 MongoDB 连接 URI 需要设置为环境变量，以便应用程序访问数据库。

# 编写查询

MongoDB 和 DynamoDB 库的结构非常不同，这使得一些查询不太方便。

在接下来的例子中，我们将使用 JavaScript 作为编写查询的基础。

# 用 MongoDB 编写查询

用 MongoDB 编写查询非常简单，不需要特定的预先设置。唯一要确保的是正确设置索引，以确保高性能的查询。最坏的情况是，索引可以随时添加。

一个例子获取查询，获取一个用户的电子邮件地址和电话号码如下:

# 使用 DynamoDB 编写查询

对于 DynamoDB，我们需要更加小心。初始设置需要深思熟虑，因为过滤或搜索不同的字段可能最终需要更改初始设置！

更改表的设置需要您删除并重新创建它。可以想象，这并不理想。

## DynamoDB:使用表键获取用户

通过电子邮件地址和电话号码获取用户的查询示例如下所示。

到目前为止一切顺利，对吧？这似乎很容易获得用户！关键来了。该数据定义了作为查询的一部分发送给 DynamoDB 的**主键**。这意味着这只适用于主键！

主键在获取数据时非常有效，但是 AWS 只允许每个表最多定义 2 个键，即分区键和排序键。组合起来，这些被称为组合键。

## DynamoDB:使用过滤器表达式获取用户

到目前为止，我们已经确认了使用 DynamoDB 通过主键获取数据是很容易的。让我们试着复制我们用 MongoDB 做的例子，其中我们选择的数据不一定是键，而可能是索引列。

这类似于前面的 MongoDB 查询。有一些关键的区别。使用 DynamoDB，我们需要显式地发送类似 SQL 条件的过滤器表达式。当我们想按主键过滤时，需要在对象的 key 字段中显式指定键。

# 将查询与 MongoDB 进行比较

然而，使用 MongoDB，就像根据您的需求传递一个对象一样简单。

就是这样，没有多余的大惊小怪。至少，如果 DynamoDB 有一些 API 允许我们从一个对象自动生成过滤器表达式就好了。这将是一个非常受欢迎的改进，而不是让每个客户端应用程序都编写过滤器生成代码。

# 弥合差距

为了避免反复编写表达式过滤器生成代码，我创建了一个小型开源库，名为 **Dynamongo** 。它的目标是为开发人员简化 DynamoDB 查询语法，以便自动生成表达式过滤器。

[](https://github.com/KevinVR/dynamongo) [## GitHub - KevinVR/dynamongo:一个简单的库，可以使用 MongoDB 风格轻松查询 DynamoDB

### 从 MongoDB 到 DynamoDB，有一些抽象可以让我们的生活变得更容易。对于…

github.com](https://github.com/KevinVR/dynamongo) 

基本的功能已经可用，并使查询 DynamoDB 数据变得更容易，但是，仍然有许多项目可以添加。如果您发现任何缺失的特性，请随时向存储库发送请求！

# 定价模型

## MongoDB Atlas 定价

MongoDB 的定价是在云中运行一个 MongoDB 服务器的价格。无论查询次数多少，服务器的成本都是一样的，除非升级。

在撰写本文时，一个专用的 MongoDB 服务器每月的费用是 57 美元。

MongoDB Atlas 正在推出一个名为 serverless 的新功能，目前正在预览中。MongoDB 不会为服务器付费，而是在任何可用的服务上运行您的查询。其定价可降至每百万次读取请求 0.30 美元，此外还有数据存储和写入请求的额外成本。

## AWS DynamoDB 定价

AWS 的 DynamoDB 类似于 MongoDB Atlas 的无服务器模式。您不需要支付运行具有特定容量的实例的成本。相反，您只需为进入数据库的请求数量付费。

在写这篇文章的时候，在美国东俄亥俄州，价格下降到 0.25 美元/百万次阅读请求。请记住，除了读取请求，还有更多成本。写请求和存储成本也应该考虑在内。

如果您想了解更多关于典型 AWS 定价的信息，我们已经专门为此撰写了一篇文章！

[](https://aws.plainenglish.io/how-much-does-aws-really-cost-1cf5f30b6a81) [## AWS 到底值多少钱？

### MERN 堆栈的典型 AWS 定价示例

aws .平原英语. io](https://aws.plainenglish.io/how-much-does-aws-really-cost-1cf5f30b6a81) 

# 最终比较

虽然查询 API 是 AWS 仍然可以在 DynamoDB 上改进的地方，但对于 AWS 上托管的应用程序来说，它仍然是一个令人惊叹的选择。与 AWS 服务的集成程度非常高。此外，就成本而言，您实际上只需为您使用的东西付费。

MongoDB 很棒，提供了非常直观的查询 API，然而，MongoDB Atlas 的主要产品是一种非常不同的定价策略。您实际上为缩放付费，而不管您实际使用的是什么。除非您使用无服务器 MongoDB 选项，否则迁移到更昂贵的 MongoDB 实例在开始时会很昂贵，该选项仍在预览中。我建议快速浏览他们的定价，计算哪一个更适合你的项目，不仅仅是在开始，还要考虑成本的长期影响。

# 结论

这是否意味着，您应该只使用 DynamoDB，或者只对您的所有项目使用 MongoDB？一点也不。这是我们可以使用的两个伟大的数据库引擎，它们都有各自的优缺点。选择数据库引擎和托管是任何项目的重要技术决策。根据您项目的标准，其中一个可能比另一个更适合。本文中的比较会让您对不同之处有更多的了解。

为了易用性，我们还是更喜欢 MongoDB。对于更复杂的功能，将会有更多的差异，我们可以在下一篇文章中进行比较。至于定价，MongoDB 现在提供了一个无服务器选项，尽管它仍处于预览阶段。它不提供 AWS 与其他 AWS 服务的紧密集成。

我将继续在我的项目中使用 DynamoDB，纯粹是为了与 AWS 紧密集成，以及非常可伸缩的定价模型。这是获得更多 DynamoDB 经验的绝佳借口！