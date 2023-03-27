# 大数据项目中最常用的 Java 库、框架和 APIs 第 3 部分

> 原文：<https://itnext.io/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-3-b229b3c6aec9?source=collection_archive---------2----------------------->

![](img/7b4ace910db6a3b24d606e708145ed5a.png)

这是关于大数据项目中最常用的 Java 库、框架和 API 系列的第三篇文章。如果你想阅读这个系列的其他剧集，请查看这些文章:

> [#1:大数据项目中最常用的 Java 库、框架和 API——Java 中 9 个最常用的机器学习库](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-1-c1728f1c26b0)
> 
> [#2:大数据项目中最常用的 Java 库、框架和 API——Java 中最常用的 5 个 NLP 库](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-2-72263d03d147)
> 
> [*#4:* 大数据项目中最常用的 Java 库、框架和 API——4 个有用的 Java APIs 和 GUI](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-4-8ee50cda8162)

Java 是大数据项目中使用最广泛的编程语言之一，它的流行部分归功于其广泛的生态系统。Java 编程提供了对这个由几个库、框架和 API 组成的生态系统的访问。

在一系列文章中，我将简要描述大数据项目中最常用的 Java 库、框架和 API。

**5 个面向大数据项目的开源 Java 框架**

本质上，“控制反转”将框架与库区分开来。当程序员从一个库中调用一个方法时拥有控制权，而在一个框架中控制权是颠倒的；换句话说，框架调用程序员。

框架经常用于解决特定领域的问题。它们定义了一个过程，使程序员能够根据需求实现某些功能。此外，框架中内置的模块和函数使得编码过程更快。

有很多流行的框架可供 Java 使用，比如 [Spring](https://projects.spring.io/spring-framework/) 、 [Hibernate](https://github.com/hibernate/hibernate-orm) 和 [JSF](https://javaserverfaces.java.net/) 。本文简要介绍了五个对大数据项目有用的开源框架。以下框架提供了不同的功能，同时减少了编程时间:

## 1.[海量在线分析— MOA](http://moa.cms.waikato.ac.nz/)

一个用 Java 编写的用于数据流挖掘的开源框架。MOA 支持与 [WEKA](http://www.cs.waikato.ac.nz/ml/weka/) 的双向交互，包括一系列机器学习算法，包括[分类、回归](http://moa.cms.waikato.ac.nz/details/classification/)、[聚类](http://moa.cms.waikato.ac.nz/details/stream-clustering/)、[离群点检测](http://moa.cms.waikato.ac.nz/details/outlier-detection/)、概念漂移检测和[推荐系统](http://moa.cms.waikato.ac.nz/details/recommender-systems/)以及评估工具。

一些有用的链接:

*   [MOA:海量在线分析——机器学习研究杂志](http://www.jmlr.org/papers/volume11/bifet10a/bifet10a.pdf)
*   [GitHub 库](https://github.com/Waikato/moa)

## 2.[可扩展的高级海量在线分析—萨摩亚](http://samoa.incubator.apache.org/)

Apache SAMOA 是一个用于挖掘大数据流的开源框架。它包含一组分布式流算法，用于最常见的数据挖掘和 ML 任务，如分类、聚类和回归。它还允许开发分布式流 ML 算法，并在多个分布式流处理引擎(DSPEs)上执行它们，包括 [Apache Storm](https://samoa.incubator.apache.org/documentation/Executing-SAMOA-with-Apache-Storm.html) 、 [Apache S4](https://samoa.incubator.apache.org/documentation/Executing-SAMOA-with-Apache-S4.html) 和 [Apache Samza](https://samoa.incubator.apache.org/documentation/Executing-SAMOA-with-Apache-Samza.html) 。

一些有用的链接:

*   [GitHub 资源库](https://github.com/apache/incubator-samoa)
*   [萨摩亚:挖掘大数据流的平台](https://melmeric.files.wordpress.com/2013/04/samoa-a-platform-for-mining-big-data-streams.pdf)

## 3.[大羚羊 2](http://oryx.io/)

Oryx 2 建立在 Apache Spark 和 Apache Kafka 的基础上，是一个专门用于实时大规模机器学习应用程序开发的框架。它还包括用于协作过滤、分类、回归和聚类的预构建包。

其独特的三层架构包括一个通用的[λ架构](http://lambda-architecture.net/)层、一个机器学习层实现(用于超参数选择)，以及一个用于端到端应用实现的顶层(作为应用实现 ML 算法——ALS、随机决策森林、k 均值)。

有用链接:

*   [GitHub 库](https://github.com/michelsalib/oryx)

## 4.[欧米诺](http://neuroph.sourceforge.net/)

Apache 许可的神经网络开发框架。它支持常见的神经网络架构，如 Adaline、感知器、Maxnet 和 Hopfield 网络。

用 Java 编写，Neuroph 由一个神经网络库[和多个类](http://neuroph.sourceforge.net/javadoc/index.html)以及一个用于创建和训练神经网络的 GUI 编辑器组成。

一些有用的链接:

*   [GitHub 库](https://github.com/neuroph/neuroph)
*   [NetBeans 平台上的神经网络](http://www.oracle.com/technetwork/articles/java/nbneural-317387.html)

## 5. [Encog](http://www.heatonresearch.com/encog)

用于高级机器学习的 Apache 许可的多平台框架。它支持多种 ML 算法，如支持向量机、人工神经网络、贝叶斯网络、隐马尔可夫模型、遗传编程和遗传算法。它还包含标准化和处理数据的类。

Encog 的主要优势是其神经网络算法，如反向传播算法。它也有创建各种网络的类。注意，Encog 也适用于微软。Net 和 C++。

一些有用的链接:

*   [GitHub 库](https://github.com/encog)
*   Encog:Java 和 C#的可互换机器学习模型库

其他文章专门针对(LINKIT 网站):

[**第 1–9 部分 Java 中最常用的机器学习库**](/knowledge-base/268/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_1)[**第 2–5 部分 Java 中最常用的 NLP 库**](/knowledge-base/270/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_2)