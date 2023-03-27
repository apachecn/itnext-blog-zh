# 大数据项目中使用最多的 4 种语言:Scala

> 原文：<https://itnext.io/4-most-used-languages-in-big-data-projects-scala-59ff02e707c9?source=collection_archive---------0----------------------->

![](img/e4aea89cc775fd5ac6b1a23169d2414a.png)

本文是关于大数据项目编程语言系列的最后一部分。本系列中发表的其他文章可以在此处找到:

> [#1:大数据项目中使用最多的 4 种语言:Java](https://www.linkit.nl/knowledge-base/177/4_most_used_languages_in_big_data_projects_Java)
> 
> [#2: 4 大数据项目中使用最多的语言:Python](https://www.linkit.nl/knowledge-base/196/4_most_used_languages_in_big_data_projects_Python)
> 
> [#3: 4 大数据项目中使用最多的语言:R](https://www.linkit.nl/knowledge-base/226/4_most_used_languages_in_big_data_projects_R)

大数据项目常用的有 Java、Python、R、Scala。在一系列文章中，我将简要描述这些语言以及它们在数据科学家中流行的原因。 [Java](https://www.linkit.nl/knowledge-base/177/4_most_used_languages_in_big_data_projects_Java) 、 [Python](https://www.linkit.nl/knowledge-base/196/4_most_used_languages_in_big_data_projects_Python) 和 [R](https://www.linkit.nl/knowledge-base/226/4_most_used_languages_in_big_data_projects_R) 在之前的文章中有描述。本文主要关注 Scala，并概述了这种语言，以及为什么它在大数据项目中很常见。

[**Scala**](http://www.scala-lang.org/)

Scala 代表“可扩展语言”,是一种开源、多范例、高级编程语言，具有健壮的静态类型系统。它的类型系统支持参数化和抽象。

Scala 因集成了函数式和面向对象的特性而受到欢迎。因此，每个值都是一个对象，每个操作符都是一个方法，而您可以将函数作为变量传递。Scala 有一个灵活的模块化 mixin 组合，它结合了 mixin 和 traits 的优点(它允许程序员重用没有继承的新类定义)。它还有一个支持匿名函数和高阶函数的语法。

Scala 还支持其他范例，包括命令式和声明式。然而，Scala 在并行化方面比传统的命令式编程语言有优势。Scala 能够在更高的抽象层次上描述算法。正如 D. Wilkinson 教授描述的那样，这种抽象允许相同的算法在一台机器上的可用内核之间串行、并行运行，或者在一个机器集群上并行运行，而无需更改任何代码。

Scala 运行在 Java 虚拟机上，与 Java 无缝互操作。在 Scala 中可以直接使用 Java 库，调用 Java 代码，实现 Java 接口，子类化 Java 类，反之亦然。然而，有一些 Scala 的特性，包括带有定义方法的[特征](http://alvinalexander.com/scala/scala-trait-examples)，以及 [Scala 的高级类型](https://www.safaribooksonline.com/library/view/learning-scala/9781449368814/ch10.html)，是不能从 Java 访问的。

此外，Scala 编程语言非常简洁。几个循环可以用一个单词来代替，这使得它比标准 Java 要简单得多。此外，它的静态类型和函数性质使它是类型安全的。

值得注意的是，在[对比](https://days2011.scala-lang.org/sites/days2011/files/ws3-1-Hundt.pdf)中，一个详细说明的紧凑算法是用四种语言实现的:Scala、Java、Go 和 C++。Scala 简洁的符号和强大的语言特性“允许代码复杂度的最佳优化”。

不可否认，Scala 已经成为大规模数据科学和机器学习的决定性工具。包括 Twitter、LinkedIn 和 The Guardian 在内的一些大公司都用 Scala 构建了他们的网站。这个级数主要是因为:

**1。Scala 是一种简洁的编程语言**

Scala 在可读性和简洁性之间创造了良好的平衡。因此，代码更容易理解。Scala 的简洁主要是因为:

*   它的[类型推理](http://docs.scala-lang.org/tutorials/tour/local-type-inference.html)——与其他函数式语言不同，Scala 的类型推理是局部的。因为类型可以被编译器推断出来，所以它们不会碍事。
*   模式匹配机制 Scala 第二个最常用的特性，它允许使用第一匹配策略匹配任何类型的数据。Brian Clapper 对 Scala 中的模式匹配有很好的介绍
*   将函数用作变量和重用效用函数的能力

**2。前沿班级作文**

作为一种面向对象的语言，Scala 允许用子类化和灵活的混合组合来扩展类；这是一种代码重用的好方法，也是多重继承的替代方法，可以避免继承的不确定性。此外，模块化混合成分结合了混合成分和特征的优点。

**3。实时流处理**

虽然 Hadoop MapReduce 可以并行处理和生成大型数据集，但它因无法处理实时流处理而受到批评。Spark 让 Scala 在实时处理流方面比其他编程语言更有优势。这使得 Scala 成为快速数据处理的计算引擎。

**4。Scala 庞大的生态系统源于与 Java 的无缝互操作性**

Scala 与基于 Java 的大数据生态系统完美集成。Java 库、ide(比如 Eclipse 和 IntelliJ)、框架(比如 Spring 和 Hibernate)和工具都可以完美地与 Scala 一起工作。甚至流行的框架也包含 Java 和 Scala 的双重 API。

Scala 程序员通常在大数据项目中使用以下框架和 API:

*   [Apache Spark](http://spark.apache.org/) —一个快速通用的大规模数据处理框架

Spark 是用 Scala 写的，运行在 JVM 上。它提供 Java、Scala 和 Python 中的 API，还支持 R 和 Clojure。弹性分布式数据集(RDD)是 Spark 的基本数据结构。不可变、分布式、延迟求值、可捕捉是它的共同属性。

Scala 还为大数据分析和机器学习提供了额外的库，包括 Spark Streaming、Spark SQL、Spark MLlib(机器学习)和 Spark GraphX(图形分析)。

*   Apache Flink —一个分布式流和批处理数据处理的框架

Flink 的核心是用 Java 和 Scala 编写的混合(实时流+批处理)分布式数据处理引擎。Flink 包含几个用于批处理(数据集 API)、实时流(数据流 API)和关系查询(表 API)的 API，以及用于机器学习(flink ml-pure Scala)、复杂事件处理(CEP)和图形处理(Gelly)的特定领域库。

*   [Apache Kafka](http://kafka.apache.org/) —用于处理实时数据馈送的分布式流媒体平台

Kafka 用 Java 和 Scala 编写，是一个快速、可伸缩、持久和容错的发布-订阅消息系统。它与 Apache Storm、Apache HBase 和 Apache Spark 结合使用，可以实时分析和呈现流数据。 [Kafka 文档](https://kafka.apache.org/documentation.html)和 [cloudera](http://blog.cloudera.com/blog/2014/09/apache-kafka-for-beginners/) 描述了 Kafka 的设计和实现方面。值得注意的是，Kafka Manager(由 Yahoo 开发)是一个基于 web 的开源工具，用于管理 Apache Kafka。

*   Apache Samza —一个分布式流处理框架

Apache Samza 使用 Apache Kafka 进行消息传递，使用 Apache Hadoop YARN 提供容错、处理器隔离、安全性和资源管理。Samza 类似于 Apache Storm，但更容易操作。有关用 Scala 编写的 Samza 流处理作业的示例，请查看 GitHub 页面。

*   Akka —一个构建分布式应用的并发框架

Akka 是一个基于 actor 的消息驱动的运行时，用于在支持 Java 和 Scala 的 JVM 上管理并发性、弹性和弹性。Akka 使用 Actor 模型，这是高度可伸缩和并发系统的理想模型。

*   一个集成批处理和在线 MapReduce 计算的框架

Summingbird 是一个数据处理框架，用于以类型安全的方式流式传输 MapReduce。它提供了一种在 Scala 中实现的特定领域语言(DSL ),用于表达分析查询，生成 Hadoop 作业(批处理计算——使用烫洗/级联)或 Storm 拓扑(在线计算),而无需对程序逻辑进行任何更改。它也可以在混合批处理/在线处理模式下运行。

*   [烫手](http://www.cascading.org/projects/scalding/) —级联的 Scala API，MapReduce 的抽象

burning 建立在 Cascading 之上，是一个抽象 Hadoop MapReduce 的 Java 库，简化了在 Scala 中编写 MapReduce 作业。烫伤可与猪媲美，同时提供与 Scala 的紧密集成

*   Scrunch —一个用于编写、测试和运行 MapReduce 管道的框架

Scrunch 是 Apache Crunch 的 Scala 包装器，为编写、测试和运行 MapReduce 管道提供了一个框架

此外，有几个基于 java 的数据存储解决方案可以很好地与 Scala 配合使用，包括:Apache Cassandra (phantom 和 Cassie 是 Scala Cassandra 的客户端)、Apache HBase、Datomic 和 Voldemort。

**5。用于数据科学和数据分析的几个库**

尽管 Scala 的库不如 Python 或 R 库全面，但它们为大数据项目提供了坚实的基础。 [Awesome Machine Learning](https://github.com/josephmisiti/awesome-machine-learning) 是一个机器学习框架、库和软件的精选列表(涵盖多种语言)，列出了用于机器学习、数据分析、数据可视化和 NLP 的有用 Scala 库和工具。此外， [Typelevel](http://typelevel.org/projects/) 为 Scala 提供了几个有用的库和扩展。

以下是一些最常用的机器学习和数据分析库:

*   [Saddle](http://saddle.github.io/) —一个高性能的数据操作库(深受 Python 的 [pandas](http://pandas.pydata.org/) 库的影响)
*   [ScalaNLP](http://www.scalanlp.org/) —一套不同的库，包括 [Breeze](https://www.github.com/scalanlp/breeze) (一套用于机器学习和数值计算的库)和 [Epic](https://www.github.com/dlwh/epic) (高性能统计解析器和结构化预测库)。
*   Apache Spark ml lib——Scala、Java、Python 和 R 的机器学习库
*   [Apache PredictionIO](https://github.com/apache/incubator-predictionio) —基于 Apache Spark、HBase 和 Spray 的机器学习服务器，可以作为完整的机器学习堆栈安装
*   [DEEPLEARNING4J](https://deeplearning4j.org/) —面向 Java 和 Scala 的分布式深度学习库
*   [Scala-datatable](https://github.com/martincooper/scala-datatable) 和[fra mian](https://github.com/tixxit/framian)——针对数据框架和数据表( [Darren Wilkinson 的研究博客](https://darrenjw.wordpress.com/)有一篇关于这个主题的[很棒的文章](https://darrenjw.wordpress.com/tag/framian/)

值得注意的是， [Awesome Scala](https://github.com/lauris/awesome-scala) 和 [Scala Wiki](https://wiki.scala-lang.org/display/SW/Tools+and+Libraries) 分类并列出一些对 Scala 有用的库、框架和工具。Scaladex 表示所有已发布的 Scala 库的地图。任何与 Scala 库相关的更新都会在最新版本发布时由 implicit.ly 宣布。

**6。充满活力的成长社区**

Scala 有一个活跃的社区，正在迅速扩大。根据[KD nuggets Analytics/Data Science 2016 年软件调查](http://www.kdnuggets.com/2016/06/r-python-top-analytics-data-mining-data-science-software.html)，Scala 是增长最快的工具之一。

Scala 在 [Stack Overflow](http://stackoverflow.com/questions/tagged/scala) 上有一个活跃的社区，此外还有在 [GitHub](https://github.com/scala/scala) 和 [Reddit](https://www.reddit.com/r/scala/) 上的大型社区。此外，scala 为 GitHub 库的用户提供了三个 Gitter 通道: [scala/scala](https://gitter.im/scala/scala) (用于一般讨论和问题)；[scala/contributors](https://gitter.im/scala/contributors)(用于贡献者讨论 Scala 的变更工作)；[spark-Scala/Lobby](https://gitter.im/spark-scala/Lobby)(用于讨论和询问关于使用 Scala 进行 Spark 编程的问题)。有关 GitHub 上 Scala 的最新趋势，请查看[趋势 Scala 库](https://github.com/trending?l=scala&since=monthly)。

[Scala Times](http://scalatimes.com/)(Scala 周报)和 Scala 本周(由 [cakesolutions 博客](http://www.cakesolutions.net/teamblogs)每周出版)是 Scala 世界最新信息的良好来源。此外， [Scala Space](http://scala.space/) 和 [Scala meetup](https://www.meetup.com/topics/scala/) 提供了全世界 Scala meetup 群组的信息。

Scala 也有很好的社区支持的学习资源，包括:

*   [Scala 练习](http://scala-exercises.47deg.com/)
*   [斯卡拉学校](http://twitter.github.com/scala_school/)
*   [有效标量](http://twitter.github.com/effectivescala/)
*   [Scala 拼图游戏](http://scalapuzzlers.com/)
*   [互动游](http://scalatutorials.com/tour)
*   [一点点学习 Scala](http://matt.might.net/articles/learning-scala-in-small-bites/)
*   [学习 Scala](http://www.scala-lang.org/node/1305)
*   [易浩的编程博客](http://www.lihaoyi.com/)

总之，Scala 的优点和缺点描述如下:

**优点**

*   Scala 统一了面向对象和函数式编程

作为一种面向对象的编程语言，所有的值都是对象，对象的类型和行为由类和特征来描述。作为函数式编程语言，每个函数都是一个值。Scala 包含了许多函数式编程，包括:currying、类型推断、不变性、惰性求值和模式匹配(Java 缺少这些特性)

*   简洁和稳健
*   Scala 是为[并发](https://twitter.github.io/scala_school/concurrency.html)和[并行](http://docs.scala-lang.org/overviews/parallel-collections/overview.html)(并行集合中的隐式并行)而设计的

Hadoop 的几个高性能数据框架是用 Scala 或 Java 编写的。在这些环境中使用 Scala 的主要原因是它惊人的并发支持，这是并行处理大型数据集的关键。

*   与 Java 的无缝互操作性

Scala 在 JVM 上运行，因此 Java 类和库可以直接在 Scala 代码中使用，反之亦然。

*   除了访问 Java 庞大的生态系统，Scala 还有各种各样的用于科学计算和大数据项目的本地库
*   默认情况下，Scala 具有不变性，内置于标准库中
*   Scala 允许命令式编程

虽然命令式和[函数式风格](http://docs.scala-lang.org/glossary/)形成对比，但 Scala 被设计成函数式构造，命令式构造。

*   Scala 包含了一个有用的交互式 REPL
*   Scala 有原生元组
*   Scala 是一种类型安全的编程语言
*   Scala 有一个强大的静态类型系统，将代数数据类型和类层次结构统一起来
*   Scala 有一个内置的类型推断，允许省略某些类型注释
*   Scala 的类型系统支持抽象类型和路径依赖类型将 *v* Obj 演算应用到具体的语言设计中。
*   [在 scala 和其他编程语言之间架起桥梁的包，比如 rscala 包](https://cran.r-project.org/web/packages/rscala/index.html)，一个 R 和 Scala 之间带有回调的双向接口

**缺点**

最初，Scala 是为了更好地开发 java 而编写的，然而，特别是对于初学者来说，这不是一个容易的转变，很难学习或适应。此外，与 Java 相比，Scala 的工具/ IDE 支持较弱。此外，除非你有一个超快的处理器，否则 Scala 编译器会花费大量与 Java 相关的时间。为了克服 Scala 的批评，dotty 项目应运而生。