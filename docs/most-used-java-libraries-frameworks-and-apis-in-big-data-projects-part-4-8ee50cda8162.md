# 大数据项目中最常用的 Java 库、框架和 APIs 第 4 部分

> 原文：<https://itnext.io/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-4-8ee50cda8162?source=collection_archive---------1----------------------->

![](img/7b729009b6b0445e3990c0c21220eea7.png)

这是关于大数据项目中最常用的 Java 库、框架和 API 系列的第四篇文章。如果你想阅读这个系列的其他剧集，请查看这些文章:

> [*#1:大数据项目中最常用的 Java 库、框架和 API——Java 中最常用的 9 个机器学习库*](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-1-c1728f1c26b0)
> 
> [*#2:大数据项目中最常用的 Java 库、框架和 API——Java 中最常用的 5 个 NLP 库*](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-2-72263d03d147)
> 
> [*#3:大数据项目中最常用的 Java 库、框架和 APIs 大数据项目中最常用的 5 个 Java 框架*](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-3-b229b3c6aec9)

Java 是大数据项目中使用最广泛的编程语言之一，它的流行部分归功于其广泛的生态系统。Java 编程提供了对这个由几个库、框架和 API 组成的生态系统的访问。

在一系列文章中，我简要描述了大数据项目中最常用的 Java 库、框架和 API。

**4 个适用于大数据项目的 Java APIs 和 GUI**

API 是简化程序开发的一组预先编写的类、包和接口。事实上，API 提供了可以放在一起的程序的构建块；因此，它最大限度地减少了开发人员编写的行数。

API 通过提供程序员可以请求特性和服务的接口来提高编程效率。有两种类型的 API 可用于 Java:

*   [官方 Java API](http://www.oracle.com/technetwork/java/api-141528.html) ，它是 JDK 中所有类的列表
*   第三方 API，可以从源网站单独下载

虽然许多 Java 库都自带 API，但本文简要介绍了四个开源的、在大数据项目中有用的 API:

# 1. [H2O](https://github.com/h2oai/h2o-3)

面向智能应用的开源机器学习 API。它支持大数据上的规模统计、机器学习和数学。H2O 是可扩展的，允许开发者添加他们选择的数据转换和模型算法。它包括用于监督和非监督学习的高级算法，包括:

*   [深度学习](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/deep-learning.html) | [教程](http://docs.h2o.ai/h2o-tutorials/latest-stable/tutorials/deeplearning/index.html) | [册子](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/booklets/DeepLearningBooklet.pdf)
*   [广义线性建模](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/glm.html) (GLM) | [教程](http://docs.h2o.ai/h2o-tutorials/latest-stable/tutorials/gbm-randomforest/index.html) | [小册子](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/booklets/GBMBooklet.pdf)
*   [分布式随机森林](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/drf.html) (DRF) | [教程](https://github.com/h2oai/h2o-3/blob/master/h2o-docs/src/product/tutorials/rf/rf.md)
*   [堆叠整体](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/stacked-ensembles.html)
*   [朴素贝叶斯](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/naive-bayes.html)
*   [主成分分析](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/pca.html) (PCA) | [教程](https://github.com/h2oai/h2o-3/blob/master/h2o-docs/src/product/tutorials/pca/pca.md)
*   [梯度推进机](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/gbm.html)(用于回归和分类)| [教程](http://docs.h2o.ai/h2o-tutorials/latest-stable/tutorials/gbm-randomforest/index.html) | [小册子](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/booklets/GBMBooklet.pdf)
*   [广义低秩建模](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/glrm.html) (GLRM) | [教程](http://docs.h2o.ai/h2o-tutorials/latest-stable/tutorials/glrm/glrm-tutorial.html)
*   [Word2vec](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/word2vec.html)
*   [K-Means 聚类](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/k-means.html) | [教程](https://github.com/h2oai/h2o-3/blob/master/h2o-docs/src/product/tutorials/kmeans/kmeans.md)

**有用链接:**

[H2O 核心组件 Java API 指南](http://docs.h2o.ai/h2o/latest-stable/h2o-core/javadoc/index.html)

[H2O 所用算法的 Java API 指南](http://docs.h2o.ai/h2o/latest-stable/h2o-algos/javadoc/index.html)

[在 Java 应用程序中创建和实现 POJOs 或 MOJOs 的分步指南](http://docs.h2o.ai/h2o/latest-stable/h2o-genmodel/javadoc/index.html)

[Stackoverflow 社区](https://stackoverflow.com/questions/tagged/h2o+java)

[GitHub 知识库](https://github.com/h2oai/h2o-3)

# 2. [Java-ML](http://java-ml.sourceforge.net/)

一个可扩展的 API，包含一组机器学习和数据挖掘算法。它为每种类型的算法提供了标准接口。这些接口很简单，算法严格遵循各自的接口。注意，在撰写本文时，最后一个版本是在 2012 年。

**有用链接:**

Java-ML:一个机器学习库——机器学习研究杂志

[API 文件](http://java-ml.sourceforge.net/api/)

# 3.[视网膜 API](http://www.cortical.io/product_retina_api.html)

用于开发智能自然语言理解(NLU)应用程序的 API。这个 API 由 [cortical.io](http://www.cortical.io/) 开发，用于将书面材料转换成语义表示。例如，它允许在语义级别上比较这两个内容。

这个 API 简化了任何 Java 应用程序和 Retina 服务器之间的通信。受[语义折叠](http://www.cortical.io/technology.html)的启发，Retina 的主要概念是“将语言的符号表示转化为稀疏分布表示(SDR)形式，使其变得可数值计算”。

**有用链接:**

[GitHub 库](https://github.com/cortical-io/retina-api-java-sdk)

# 4. [RapidMiner 工作室](https://rapidminer.com/products/studio/)

用于设计和执行分析工作流的开源([社区版](https://rapidminer.com/pricing/) ) GUI。RapidMiner 用 Java 编写，为数据科学项目和用例提供了一个环境。它包含几个用于数据访问、探索、混合、清理、建模和验证的预定义函数。

其先进的算法支持数据挖掘和机器学习过程，包括预测分析、统计建模、数据预处理、数据可视化和部署([操作员参考手册](http://docs.rapidminer.com/studio/operators/rapidminer-studio-operator-reference.pdf))。此外，它允许集成 Python 和 R 脚本。

**有用链接:**

[GitHub 库](https://github.com/rapidminer/rapidminer-studio)

[RapidMiner 工作室手册](http://docs.rapidminer.com/downloads/RapidMiner-v6-user-manual.pdf)

*原载于*[*www . linkit . nl*](https://www.linkit.nl/knowledge-base/278/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_4)*。*