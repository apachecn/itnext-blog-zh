# 现在，在 SQL 和 NoSQL 之间做出选择比以往任何时候都容易

> 原文：<https://itnext.io/choosing-between-sql-and-nosql-is-now-easier-than-ever-9d467628630c?source=collection_archive---------6----------------------->

## TL；DR:除非 ACID 合规性是一个关键问题(你在金融、电子商务或政府部门)，否则选择 NoSQL。

[ACID](https://en.wikipedia.org/wiki/ACID_(computer_science)) (原子性、一致性、隔离性、持久性)旨在即使在出错、断电等情况下也能保证有效性。

**首先**，让我们吞下青蛙。为什么 NoSQL 在酸性方面不如美国？

NoSQL 数据库牺牲了 ACID 合规性来换取灵活性和处理速度。您可以为了 ACID 而实现定制工作，但这取决于您自己。

**第二个**，NoSQL 是许多数据库(如键值存储、文档存储和图形)的通用分类。我将主要参考本文中的文档存储，它是最常用的，并且与这个比较相关。

如今，当像 MongoDB 这样的 NoSQL 实现具有 [join 查询](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/)、[random select](https://docs.mongodb.com/manual/reference/operator/aggregation/sample/)、 [transactions](https://docs.mongodb.com/manual/reference/method/Session/) 以及其他不久前还是 SQL 的明显优势的特性时，它会让你质疑使用它的必要性。

NoSQL 还保留了它的优点，如更高的性能、更灵活(无模式)、更容易扩展、更环保(因此更便宜)，以及*可能*更简单。

我们的决定应该更令人舒服，有时甚至不需要动脑筋。

## 一个警告

就像在每一个松散的技术中一样，你必须成为优秀的开发人员，并设计和使用好它，否则它会对你不利。

## 但是

随着差异变得模糊，虽然性能、灵活性和规模可能不是主要问题，但其他原因可能会打破这种关系。比如说——像 [Node.js 的 Sequelize](http://docs.sequelizejs.com/) 这样优秀的 [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) 。

一只小狗:

![](img/dbf1e46a0de72bf918d57a87280cc0a5.png)

查尔斯·德鲁维奥·🇵🇭🇨🇦在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

如果您觉得这篇文章有帮助，请点击👏按钮。欢迎在下面评论或给我发电子邮件到 orenykb@gmail.com。