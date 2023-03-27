# NoSQL 数据库的 5 个陷阱

> 原文：<https://itnext.io/5-pitfalls-of-nosql-databases-c35012431a80?source=collection_archive---------1----------------------->

![](img/45f3c16be0187e799aa40b89ec1825e0.png)

我录制了一段视频，在视频中我讲述了 NoSQL 数据库的优势。这个回答很有趣，但我的印象是，并不是每个人都看到硬币的两面。事实是它们会给我们带来很多问题😉。

# 模式管理

每个 NoSQL 数据库都以自己的方式处理模式。有些没有模式( [MongoDB](https://www.mongodb.com/) )，有些是动态的( [Elasticsearch](https://www.elastic.co/elasticsearch/) )，有些类似于来自关系数据库的模式( [Cassandra](https://cassandra.apache.org/) )。在概念模型中，数据**总是**有一个模式。实体、字段、名称、类型、关系。无论基础的类型如何，物理模型都是概念模型的表示。

NoSQL 数据库在模式方面给了我们更多的自由。在 MongoDB 中，我们可以添加两个具有相同字段名但不同类型的不同文档。这有道理吗？不太可能。这可能发生吗？当然可以。一个简单的人为错误可能会破坏我们的应用程序。

另一个问题与实体之间的关系有关。即使我们的数据库中没有关系，我们也必须记录数据之间的关系。从关系数据库中我们可以生成一个 [ERD 图](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model)。在 NoSQL 数据库的情况下，这可能不起作用。

使用 NoSQL 数据库时，我们必须记住模式管理和数据验证问题。没有它，数据可能会“爆炸”。有趣的事实:一些[公司用 PostgreSQL](http://blog.shippable.com/why-we-moved-from-nosql-mongodb-to-postgressql) 替换 MongoDB。

# 误差下限

NoSQL 数据库的性能是正确的数据建模、索引和分区的结果。在关系数据库中，我们可以添加列、转换表、将数据从一个表翻转到另一个表、添加索引(如果我们以前忘记了)。对于 NoSQL 数据库，这并不是在所有情况下都可行的。我们可能需要使用一些外部工具，比如 Apache Spark，或者甚至删除并重新创建我们的数据模型。

在 Elasticsearch 中，如果我们没有得到索引的模式/映射，我们必须使用例如 [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) ，这意味着我们必须将数据重新索引到另一个索引。

在 Cassandra 中，我们只能通过分区和聚类键进行过滤。如果我们忘记向键中添加一列，就有可能添加一个索引，但是如果集合的基数很大，这可能会降低性能。

# 没有酸

ACID 属性简化了代码编写。我们不需要处理与 X 表中的数据已经存在而 Y 表中还没有(如果有的话)这一事实相关的错误。从 [CAP](https://en.wikipedia.org/wiki/CAP_theorem) 定理我们知道有一致的和[最终一致的](https://en.wikipedia.org/wiki/Eventual_consistency)数据库。这种类型最流行的数据库是 Apache Cassandra。最终的一致性需要不同的数据建模和应用程序逻辑方法。应该以一种更具防御性的方式编写代码，因为不能确定您刚刚更改的记录是否已经可以从应用程序的另一部分获得。HBase 是一致性数据库的一个例子，但即使是 [Cloudera 也认为它不会取代关系数据库。](https://blog.cloudera.com/apache-hbase-dos-and-donts/)有些数据库标榜自己是一致的，只是在一定程度上保证了一致性。比如 MongoDB 提供了事务，但是[多文档事务只有从 4.0 版本开始才有。](https://www.mongodb.com/blog/post/mongodb-multi-document-acid-transactions-general-availability)

# 没有 SQL

NoSQL 的缺点是缺乏… SQL😁。我们可能喜欢也可能不喜欢，但是 SQL 是数据的基础。许多分析师每天都在使用 SQL，学习另一种语言可能会阻碍他们使用数据库。创建[火花 SQL](https://databricks.com/glossary/what-is-spark-sql) 或[光束 SQL](https://beam.apache.org/documentation/dsls/sql/overview/) 是有原因的。

# 有限的分析和/或无连接

这是对 [OLTP](https://en.wikipedia.org/wiki/Online_transaction_processing) 和 [OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) 系统之间差异的一点讨论。我们习惯于使用 GROUP BY 和 JOIN 子句，但并不是每个数据库都提供这样的功能。由于数据库的性质，聚合和合并可能非常有限(如果可能)。在 Apache Cassandra 的例子中，分析功能通常是通过组合 Apache Spark 集群来实现的。你将从[我的这个故事](/how-to-start-with-apache-spark-and-apache-cassandra-886a648bd2fb)中学到如何把一个和另一个联系起来。

# 摘要

那么，使用 NoSQL 值得吗？当然是了。我们必须记住，每个数据库都是为了解决一类问题而创建的。弯曲“命运”是可能的，但会有后果。螺丝更容易用螺丝刀拧进去，用锤子敲钉子，用滚筒敲打墙壁。

一个有趣的选择是 NewSQL 数据库。这种底座的一个例子是[cocroach db](https://www.cockroachlabs.com/product/)和[扳手](https://cloud.google.com/spanner)。它们解决了传统关系数据库的问题(主要是伸缩问题)，同时保持了 [ACID](https://pl.wikipedia.org/wiki/ACID) 属性。