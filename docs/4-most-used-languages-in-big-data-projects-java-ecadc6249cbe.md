# 大数据项目中使用最多的 4 种语言:Java

> 原文：<https://itnext.io/4-most-used-languages-in-big-data-projects-java-ecadc6249cbe?source=collection_archive---------0----------------------->

![](img/1d84815a38e025bf0398308c6999967a.png)

本文是关于大数据项目编程语言系列的第一部分。本系列中发表的其他文章可以在此处找到:

*   [#2: 4 大数据项目中最常用的语言:Python](https://www.linkit.nl/knowledge-base/196/4_most_used_languages_in_big_data_projects_Python)
*   [#3: 4 大数据项目中使用最多的语言:R](https://www.linkit.nl/knowledge-base/226/4_most_used_languages_in_big_data_projects_R)
*   [#4: 4 大数据项目中使用最多的语言:Scala](https://www.linkit.nl/knowledge-base/255/4_most_used_languages_in_big_data_projects_Scala)

编程是大数据项目的核心；有必要创建能够处理结构化和非结构化数据的算法。虽然大数据是多语言的，但特定的语言在某些任务上比其他语言更好。根据特定任务的功能解决方案的效率来选择正确的语言。

一般来说，大数据项目中常用的有 [Java](https://www.oracle.com/java/index.html) 、 [Python](https://www.python.org/) 、 [R](https://www.r-project.org/) 、 [Scala](http://www.scala-lang.org/) 。Java 是大数据的永久语言。Python 和 R 也已经存在了 20 多年，但是最近几年才流行起来。注意，Java 和 Python 主要用于 Hadoop MapReduce 编程。Scala 创建于 2006 年，在过去的几年里得到了广泛的应用。

未来几周，我将介绍大数据环境中最常用的四种编程语言。在这个版本中，我们从 Java 开始。

## **Java**

*一次编写，随处运行*

Java 编程语言是一种通用的、并发的、基于类的、面向对象的语言。从本质上来说，Java 是 C 和 C++的变体，但组织方式相当不同。Java 被设计为在平台无关的虚拟机上运行，Java 虚拟机(JVM)支持这一点。JVM 使这种语言变得安全和高度[可移植](http://www.fscript.org/prof/javapassport.pdf)，同时提供健壮的内存管理和[异常处理](http://www.javaworld.com/article/2076868/learn-java/how-the-java-virtual-machine-handles-exceptions.html)。

Java 有一套[广泛的标准库](https://docs.oracle.com/javase/8/docs/api/)，包括用于数据挖掘的开源库: [Java 数据挖掘包(JDMP)](http://www.jdmp.org/) 、 [Apache Mahout](http://mahout.apache.org/) 和 [Weka](http://www.cs.waikato.ac.nz/ml/weka/) 等等。

Java 语言塑造大数据的主干；Apache Hadoop 平台——[Hadoop MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html)和[HDFS](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html)——是用 Java 编写的。此外，Hadoop 生态系统中的许多工具都运行在 JVM 上，包括 [Storm](http://storm.apache.org/index.html) 、 [Kafka](http://kafka.apache.org/) 和 [Spark](http://spark.apache.org/) 。也有其他编程语言如 [Clojure](https://clojure.org/) 、 [Scala](http://www.scala-lang.org/) 和 [Groovy](http://www.groovy-lang.org/) 被设计成在 JVM 上运行。甚至像谷歌云数据流和 Apache Beam 这样的新技术也只支持 Java，直到最近。

简而言之，Java 不仅仅是一种编程语言。Java 平台是用于开发和运行 Java 应用程序的计算平台，它有各种各样的工具来帮助开发人员高效地使用 Java 编程语言，包括几个库、API、Java 运行时环境(JRE)、Java 插件和 Java 的虚拟机(JVM)。

总而言之，Java 的优点和缺点可以归纳如下:

优势:

*   可量测性
*   Java 管理它的运行时内存
*   安全—虚拟机在执行之前对 Java 代码进行非常严格的验证
*   可移植—由于运行时环境的原因
*   优秀的开发生态系统
*   一个大型开发者社区

不足之处

*   性能 Java 的主要缺点是速度
*   冗长
*   [缺少 REPL](http://www.javaworld.com/article/2971152/core-java/what-repl-means-for-java.html) (存在于 Python、R 和 Scala 中)
*   不提供与 R 和 Python 相同的可视化质量