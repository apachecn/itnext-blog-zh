# 大数据项目中最常用的 Java 库、框架和 APIs 第 2 部分

> 原文：<https://itnext.io/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-2-72263d03d147?source=collection_archive---------1----------------------->

![](img/fb99e6c52909579670b19f95c785a001.png)

这是关于大数据项目中最常用的 Java 库、框架和 API 系列的第二篇文章。如果你想阅读这个系列的其他剧集，请查看这些文章:

> [#1:大数据项目中最常用的 Java 库、框架和 API——Java 中 9 个最常用的机器学习库](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-1-c1728f1c26b0)
> 
> [#3:大数据项目中最常用的 Java 库、框架和 APIs 大数据项目中最常用的 5 个 Java 框架](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-3-b229b3c6aec9)
> 
> [#4:大数据项目中最常用的 Java 库、框架和 API——4 个有用的 Java APIs 和 GUI](/most-used-java-libraries-frameworks-and-apis-in-big-data-projects-part-4-8ee50cda8162)

Java 是大数据项目中使用最广泛的编程语言之一，它的流行部分归功于其广泛的生态系统。Java 编程提供了对这个由几个库、框架和 API 组成的生态系统的访问。

在一系列文章中，我将简要介绍大数据项目中最常用的 Java 库、框架和 API。上一篇文章专门介绍了[九个主要的机器学习库](/knowledge-base/268/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_1)，它们对不同项目的开发者都有帮助。

本文致力于 Java 库，这些库解决了自然语言处理(NLP)中的常见问题，NLP 是机器学习的一个子领域。

**Java 中最常用的 5 个自然语言处理库**

## 1.[斯坦福科伦普](http://stanfordnlp.github.io/CoreNLP/)

一套 GPL 许可的语言分析工具，用于处理文本。它包含用于标记化、情感分析、词性标注(POS)、解析、命名实体识别(NER)、指代消解等的工具。

事实上，斯坦福 CoreNLP 为不同的斯坦福 NLP 工具提供了具体的 API。这些工具包括:[分词器](http://nlp.stanford.edu/software/tokenizer.html)、[词性标注器](http://nlp.stanford.edu/software/tagger.html)、[命名实体识别器](http://nlp.stanford.edu/software/CRF-NER.html)、[解析器](http://nlp.stanford.edu/software/lex-parser.html)、[指代消解系统](http://nlp.stanford.edu/software/dcoref.html)、[情感分析](http://nlp.stanford.edu/sentiment/)、[引导模式学习](http://nlp.stanford.edu/software/patternslearning.html)、[开放信息抽取](http://nlp.stanford.edu/software/openie.html)。斯坦福 NLP 小组也有其他的 [NLP 软件](http://nlp.stanford.edu/software/)，它们是开源的，可以在 GNU 通用公共许可证下获得。

有用链接:

[Github 库](https://github.com/stanfordnlp/CoreNLP)

斯坦福 CoreNLP 自然语言处理工具包

[CloudAcademy 博客—斯坦福 CoreNLP 自然语言处理](http://cloudacademy.com/blog/natural-language-processing-stanford-corenlp/)

## 2. [Apache OpenNLP](http://opennlp.apache.org/)

一套用于处理自然语言文本的工具。它提供了[类和接口](http://opennlp.apache.org/documentation/1.7.2/apidocs/opennlp-tools/index.html)来执行各种自然语言处理任务，例如[标记化](https://www.tutorialspoint.com/opennlp/opennlp_tokenization.htm)、[句子分割](https://www.tutorialspoint.com/opennlp/opennlp_sentence_detection.htm)、词性标注、命名实体提取、[组块](https://www.tutorialspoint.com/opennlp/opennlp_chunking_sentences.htm)、[解析](https://www.tutorialspoint.com/opennlp/opennlp_parsing_the_sentences.htm)，以及共指消解。

OpenNLP 旨在填补 ASF 在人类语言处理工具方面的空白。它提供了一个坚实的库，包括最大熵和基于机器学习的感知器。

有用的链接:

[OpenNLP 教程](http://www.programcreek.com/2012/05/opennlp-tutorial/)

[OpenNLP 开发者文档](http://opennlp.apache.org/documentation/1.7.2/manual/opennlp.html)

[Github 库](https://github.com/apache/opennlp)

## 3.[灵管](http://alias-i.com/lingpipe/)

用 Java 编写的计算语言学工具包。它包括最常见的自然语言处理任务的方法，如标记化，句子检测，命名实体检测，共指消解，分类，聚类，词性标注，一般组块，模糊字典匹配。

除了核心 API，LingPipe 还提供了一系列预编译的模型。这些模型基于每种语言、体裁和训练语料库的任务。LingPipe 在运行时加载这些模型，它们向核心 LingPipe API 提供特定的细节。

值得注意的是，Lingpipe 文档丰富，包含许多[教程](http://alias-i.com/lingpipe/demos/tutorial/read-me.html)，对任何 NLP 初学者都很有帮助，不管是使用它还是其他工具。

有用链接:

[用 LingPipe 4 进行文本分析](http://alias-i.com/lingpipe-book/lingpipe-book-0.5.pdf)

## 4.[浇口嵌入](https://gate.ac.uk/family/embedded.html)

用于计算语言处理和文本挖掘的开源 Java 类库(JCL)。Embedded 是一组作为 jar 交付的工具(概述其[包和类](http://jenkins.gate.ac.uk/job/GATE-Nightly/javadoc/index.html?overview-summary.html))。它原本用于 Java、基于 JVM 的语言，如 [Groovy](http://www.groovy-lang.org/) ，并通过 JNI 或其他语言的类似网关使用。它能够在不同的应用程序中实现语言处理功能。

在所有基于 Gate 的系统中使用的 Gate Embedded 实际上是 GATE(文本工程通用架构)的[子项目](https://gate.ac.uk/family/)之一。 [GATE](https://gate.ac.uk/) 是一套解决 NLP 问题的工具，包括一个[开发环境](https://gate.ac.uk/family/developer.html)，一个基于工作流的 [web 应用](https://gate.ac.uk/teamware/)，一个 [Java 库](https://gate.ac.uk/family/embedded.html)，一个架构和一个[流程](https://gate.ac.uk/family/process.html)。

有用的链接:

[闸门嵌入式快速启动](https://gate.ac.uk/sale/tao/splitch7.html#chap:api)

[使用 GATE 版本 8 开发语言处理组件](https://gate.ac.uk/sale/tao/split.html#QQ2-11-176)

[闸门示例代码](https://gate.ac.uk/wiki/code-repository/)

## 5.[语言工具包的机器学习— MALLET](http://mallet.cs.umass.edu/)

一个用于文本统计自然处理的开源 Java 工具包。它包含[包](http://mallet.cs.umass.edu/api/)，用于文档分类、聚类、主题建模、信息提取和文本的多种 ML 应用。它还包括将文本文档转换成数字表示的例程，以便进行有效的处理。

此外，MALLET 还可以添加 [GRMM 包](http://mallet.cs.umass.edu/grmm/index.php)，这是一个在任意结构的图形模型中进行推理和学习的工具包。

有用的链接:

*   [GitHub 库](https://github.com/mimno/Mallet)
*   [用木槌学习机器](http://mallet.cs.umass.edu/mallet-tutorial.pdf)

除了上面提到的库， [Apache UIMA 项目](https://uima.apache.org/)促进了非结构化内容的分析，例如文本、音频和视频。也有一些基于 UIMA 的其他项目可以从不同的地方获得，包括: [Apache cTAKES](http://ctakes.apache.org/) (临床文本分析和知识提取系统) [ccp nlp](https://github.com/UCDenver-ccp/ccp-nlp) (生物医学领域的 nlp-Wrappers)[ClearTK](http://cleartk.github.io/cleartk/)(开发 ML 和 NLP 组件的框架)[dk pro Core](https://dkpro.github.io/dkpro-core)(NLP 的软件组件集合) [DKPro TC](https://dkpro.github.io/dkpro-tc/) (文本分类框架) [JULIE Lab 项目](http://julielab.github.io/)

值得注意的是，UIMA 或非结构化信息管理架构是“一种架构和软件框架，用于创建、发现、组合和部署广泛的多模态分析能力，并将它们与搜索技术相集成。”有关更多信息，请查看 [UIMA 概述& SDK 设置](https://uima.apache.org/d/uimaj-2.4.0/overview_and_setup.html)

总之，可用于机器学习和自然语言处理的 Java 库和项目数不胜数。目前，GitHub 包含用于机器学习的 [2023 Java 知识库](https://github.com/search?l=Java&q=machine+learning&type=Repositories&utf8=%25E2%259C%2593)结果(01–03–2017)。此外，【MLOSS.org】有一个基于 Java 的开源机器学习项目的综合列表。

下一篇文章将致力于大数据项目中最常用的 Java 框架。

其他文章专门针对(LINKIT 网站):

[**第一部分—Java 中最常用的九个机器学习库**](/knowledge-base/268/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_1) [**第三部分—大数据项目中最常用的五个 Java 框架**](/knowledge-base/271/Most_used_Java_libraries_frameworks_and_APIs_in_big_data_projects_part_3)