# 大数据项目中最常用的 Java 库、框架和 APIs 第 1 部分

> 原文：<https://itnext.io/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-1-c1728f1c26b0?source=collection_archive---------0----------------------->

![](img/38e5fcaf8b67562cf39dda599cc947a9.png)

这是关于大数据项目中最常用的 Java 库、框架和 API 系列的第一篇文章。如果你想阅读这个系列的其他剧集，请查看这些文章:

> [#2:大数据项目中最常用的 Java 库、框架和 API——Java 中最常用的 5 个 NLP 库](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-2-72263d03d147)
> 
> [#3:大数据项目中最常用的 Java 库、框架和 APIs 大数据项目中最常用的 5 个 Java 框架](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-3-b229b3c6aec9)
> 
> [#4:大数据项目中最常用的 Java 库、框架和 API——4 个有用的 Java APIs 和 GUI](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-4-8ee50cda8162)

Java 是大数据项目中使用最广泛的编程语言之一，它的流行部分归功于其广泛的生态系统。Java 编程提供了对这个由几个库、框架和 API 组成的生态系统的访问。

在一系列文章中，我将简要描述大数据项目中最常用的 Java 库、框架和 API。

**Java 中 9 个最常用的机器学习库**

Java 编程语言有许多第三方库。这些库包含扩展 Java 核心功能的包，同时解决了常见的编程挑战。这些库是开源和免费的，可以应用于各种项目。

以下列表包括九个主要的机器学习库，对不同项目的开发人员有所帮助:

## 1.[深度学习 4j — DL4J](https://deeplearning4j.org/)

Apache 2.0 许可的、开源的、为 Java 和 JVM 编写的分布式神经网络库。它由 [ND4J](http://nd4j.org/) 库提供支持，这是一个用于执行线性代数和矩阵的库。

DL4j 集成了 Apache Hadoop 和 Spark，可以在分布式 GPU 和 CPU 上使用。它由 [Skymind](https://skymind.ai/) 维护。

一些有用的链接:

[Github 库](https://github.com/deeplearning4j)

[深度学习和神经网络术语表](https://deeplearning4j.org/glossary)

[对比框架:Deeplearning4j、Torch、Theano、TensorFlow、Caffe、Paddle](http://www.infoworld.com/article/3114175/artificial-intelligence/baidu-open-sources-python-driven-machine-learning-framework.html)

[深度学习软件框架对比研究](https://arxiv.org/pdf/1511.06435.pdf)

## 2.【Java 的 N 维数组— ND4J

ND4J 是在 Apache License 2.0 下发布的，是一个用于 JVM 的开源科学计算库。它用于在生产环境中执行线性代数和矩阵操作。它与 Hadoop 和 Spark 集成，可与分布式 CPU 或 GPU 配合工作。

它的主要特性包括:通用的 n 维数组对象、包括 GPU 在内的多平台功能，以及线性代数和信号处理功能。

一些有用的链接:

[快速入门指南](https://deeplearning4j.org/quickstart#Java)

[Github 库](https://github.com/deeplearning4j/nd4j)

## 3. [Java 统计分析工具— JSAT](https://github.com/EdwardRaff/JSAT/tree/master)

JSAT 包含了 ML 库中最大的[算法集合](https://github.com/EdwardRaff/JSAT/wiki/Algorithms)，它最初被写得比 [Weka](http://www.cs.waikato.ac.nz/ml/weka/) 更快[更直观。它在多线程实现中速度相对较快，主要关注分类、回归和聚类算法。](http://jsatml.blogspot.nl/2015/03/jsat-vs-weka-on-mnist.html)

JSAT 是纯 Java，没有外部依赖性。它可以在 GPL 3 下使用。它支持读写 ARFF 文件、LIBSVM 和二进制格式的数据集。

有用链接:

[爱德华·拉夫的博客——JSAT 的机器学习](http://jsatml.blogspot.nl/)

## 4. [Java 机器学习库— Java-ML](http://java-ml.sourceforge.net/)

Java-ML 是主要用 Java 编写的机器学习和数据挖掘算法的集合。这些算法有一个记录良好的源代码，可以从 [API 文档](http://java-ml.sourceforge.net/api/0.1.7/)中获得。主要算法包括:聚类、分类、特征选择、数据过滤、距离度量和效用。除了这些算法之外，还有几个支持的[类](http://java-ml.sourceforge.net/api/0.1.7/)。

Java-ML 为每种算法提供了一个标准接口。这些接口保持简单，算法严格遵循各自的接口。注意，在撰写本文时，最后一个版本是在 2012 年。

有用链接:

[Java-ML:一个机器学习库](http://www.jmlr.org/papers/volume10/abeel09a/abeel09a.pdf)

## 5. [MLlib(阿帕奇火花)](http://spark.apache.org/mllib/)

[机器学习库](http://spark.apache.org/mllib/)是一个可扩展的机器学习库，简化了大规模 [ML 流水线](http://spark.apache.org/docs/latest/ml-pipeline.html)。它包含主要的学习算法，包括分类、回归、聚类、协同过滤、维度减少和底层优化原语。

MLlib 在 Spark 平台上提供了这些算法的实现(可以使用任何 Hadoop 数据源，如 HDFS 和 HBase)。此外，它还与 Python 和 R 库中的 [NumPy](http://www.numpy.org/) 互操作。

有用链接:

[MLlib 指南](http://spark.apache.org/docs/latest/ml-guide.html)

## 6. [RankLib](http://sourceforge.net/p/lemur/wiki/RankLib/)

学习排序算法的库。它由八个算法组成，包括:多重加性回归树(MART)、RankNet、RankBoost、AdaRank、坐标上升、LambdaMART、ListNet 和随机森林。

RankLib 允许单独指定训练数据，同时从单个输入文件自动分割训练。它还执行 k 重交叉验证。

## 7.[视网膜库](http://www.cortical.io/product_retina_library.html)

Java 虚拟机的语义文本处理库，用于分类、过滤和搜索巨大的文本存储库。它还可以基于含义而不是关键字来传输文本(如社交媒体源)。

Retina Library 与 Apache Spark 或单节点 Java 运行时环境集成，因此支持所有 Hadoop 存储格式。CorticalApi 和 CorticalEngine 是核心算法，能够大幅减少误报数量，同时提高召回率。

有用链接:

[视网膜库入门](http://documentation.cortical.io/retina-spark.html)

## 8. [Java 数据挖掘包— JDMP](https://jdmp.org/)

用于数据分析和机器学习的开源库。它支持机器学习算法，包括聚类、回归、分类、图形模型、优化以及提供可视化模块。

此外，JDMP 还提供了其他机器学习和数据挖掘包的接口，如 [Weka](http://www.cs.waikato.ac.nz/ml/weka/) 、 [LibLinear](https://www.csie.ntu.edu.tw/~cjlin/liblinear/) 、 [Elasticsearch](https://github.com/elastic/elasticsearch) 、 [LibSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvm/) 、 [Mallet](http://mallet.cs.umass.edu/) 、 [Apache Lucene](https://lucene.apache.org/) 和 [Octave](https://www.gnu.org/software/octave/) 。

有用链接:

[GitHub 知识库](https://github.com/jdmp/java-data-mining-package)

## 9.[编码](http://www.heatonresearch.com/encog/)

一个适应性强、可扩展的机器学习库，为回归、分类和聚类提供 ML 算法。它支持支持向量机(SVM)、人工神经网络、贝叶斯网络、隐马尔可夫模型(HMM)、遗传编程和遗传算法等算法。

有用的链接:

[Encog:面向 Java 和 C#的可互换机器学习模型库——机器学习研究杂志](http://www.jmlr.org/papers/volume16/heaton15a/heaton15a.pdf)

其他文章专门针对(LINKIT 网站):

[**第二部分—Java 中最常用的五个 NLP 库**](https://www.linkit.nl/knowledge-base/270/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_2)[**第三部分—大数据项目中最常用的五个 Java 框架**](/knowledge-base/271/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_3)