# 大数据项目中使用最多的 4 种语言:Python

> 原文：<https://itnext.io/4-most-used-languages-in-big-data-projects-python-8a400aa6884b?source=collection_archive---------1----------------------->

![](img/f28d08ed05d311599a8613c371cbfb6a.png)

本文是关于大数据项目编程语言系列的第二部分。本系列中发表的其他文章可以在此处找到:

*   [#1:大数据项目中使用最多的 4 种语言:Java](https://www.linkit.nl/knowledge-base/177/4_most_used_languages_in_big_data_projects_Java)
*   [#3: 4 大数据项目中使用最多的语言:R](https://www.linkit.nl/knowledge-base/226/4_most_used_languages_in_big_data_projects_R)
*   [#4: 4 大数据项目中使用最多的语言:Scala](https://www.linkit.nl/knowledge-base/255/4_most_used_languages_in_big_data_projects_Scala)

[Java](https://www.oracle.com/java/index.html) 、 [Python](https://www.python.org/) 、 [R](https://www.r-project.org/) 、 [Scala](http://www.scala-lang.org/) 是大数据项目中常用的。在一系列文章中，我将简要描述这些语言以及它们在数据科学家中流行的原因。Java 在[之前的文章](https://www.linkit.nl/knowledge-base/177/4_most_used_languages_in_big_data_projects_Java)中有描述。本文主要关注 Python，并概述了这种语言，以及为什么它在大数据项目中很常见。

**Python**

[Python](https://www.python.org/) 是一种开源的、解释型的、具有动态语义的高级编程语言。尽管作为面向对象的语言而闻名，它也支持[函数式](https://docs.python.org/3/howto/functional.html)、命令式和过程式编程范例。

内置的高级数据结构、动态类型和动态绑定的结合，使得 Python 程序[比等价的 Java 程序](https://www.python.org/doc/essays/comparisons/)短 3 到 5 倍。Python 的流行主要是由于其清晰的语法和可读性，源语句的缩进使得代码更容易阅读。

Python 是独立于平台的。这种平台独立性体现在解释器的实现中。Python 的[实现](https://wiki.python.org/moin/PythonImplementations)是指为 Python 语言编写的程序的执行提供支持的程序。注意，每个实现都可以利用不同的相关库和模块，而语言语法在每个实例中都保持一致。Python 的主要实现如下:

*   [CPython](https://www.python.org/)—Python 的参考实现，用 c 语言编写。它通常被简称为“Python”
*   [Jython](http://www.jython.org/archive/21/docs/whatis.html) —用 Java 编写的利用 JVM 的 Python 实现
*   IronPython —另一个 Python 实现，用 C#编写。NET 框架。它运行在。NET 虚拟机，微软的[公共语言运行时(CLR)](https://msdn.microsoft.com/en-us/library/8bs2ecf4(v=vs.110).aspx) ，并且可以同时使用 Python 和。NET 框架库
*   [PyPy](http://pypy.org/) —用 [RPython](https://rpython.readthedocs.io/en/latest/) 编写的 Python 实现，这是 Python 语言的静态类型子集。它具有一个实时编译器，并支持多个后端(C、CLI、JVM)。
    Python 自带丰富的[标准库](https://docs.python.org/3/library/)，在自然语言处理(NLP)中非常流行。它有很好的处理语言数据的功能。Python 的优势在于它为 NLP 包含了几个库。这些用于文本处理的库包括:
*   [NLTK](http://www.nltk.org/) —自然语言工具包
    NLTK 是 NLP 的一套程序模块和数据集，它定义了用 Python 构建 NLP 程序的框架。它包括超过 50 个语料库和词汇资源的接口以及一套文本处理库。
*   Gensim
    一个开源库，用于大型语料库的主题建模、文档索引和相似性检索。设计用于分析非结构化数字文本的 Gensim 基于[语料库](https://radimrehurek.com/gensim/intro.html%23term-corpus)、[向量](https://radimrehurek.com/gensim/intro.html%23term-vector)和[模型](https://radimrehurek.com/gensim/intro.html%23term-model)的概念。性能取决于两个 Python 包 [NumPy](http://www.numpy.org/) 和 [SciPy](http://www.scipy.org/install.html) 。
*   [TextBlob](http://textblob.readthedocs.io/en/dev/) —简化的文本处理
    一个用于处理文本数据的库，它提供了一个用于名词短语提取、情感分析、分类、翻译、标记等的 API。
*   [Orange](https://pypi.python.org/pypi/Orange) 一个用 Python 编写的基于组件的数据挖掘软件。其可视化编程前端包括一系列数据可视化、探索、预处理和建模技术，可以用作 Python 库。
*   [模式](http://www.clips.ua.ac.be/pattern) Python 的 web 挖掘模块，包含了抓取、自然语言处理、机器学习、网络分析和可视化的工具。
*   PyNLPl
    一个包含各种 NLP 任务模块的库，包括提取 n 元语法和频率列表，以及构建简单的语言模型。
*   spaCy
    Python 和 Cython 的高级自然语言处理库。

    Python 还包含大量用于数据分析的有用库，包括: [Pandas](http://pandas.pydata.org/) (用于数据分析和数据操作) [Statsmodels](http://statsmodels.sourceforge.net/) (用于数据探索、统计模型估计和执行统计测试) [matplotlib](http://matplotlib.org/) (用于使用通用 GUI 工具包如 Qt 和 GTK+将绘图嵌入应用程序) [scikit-learn](http://scikit-learn.org/stable/) (用于数据挖掘和数据分析) [Mlpy](http://mlpy.sourceforge.net/) (提供广泛的范围) [NumPy](http://www.numpy.org/) (为数值例程提供快速预编译函数) [SciPy](http://www.scipy.org/) (用于科学计算) [Theano](http://deeplearning.net/software/theano/) (用于定义、优化和评估涉及多维数组的数学表达式)。

简而言之，Python 的优点和缺点如下:

**优势**

*   易于学习、阅读和维护——明确定义的语法，易于理解，源代码易于维护。
*   广泛的标准库和广泛的[模块](https://wiki.python.org/moin/UsefulModules)和库
*   多范例——面向对象、命令式、过程式和一些功能特性，例如[λ](https://pythonconquerstheuniverse.wordpress.com/2011/08/29/lambda_tutorial/)和[列表理解](http://treyhunner.com/2015/12/python-list-comprehensions-now-in-color/)
*   高效的高级数据结构和对动态类型检查的支持
*   可移植—运行在广泛的硬件平台上
*   高级语言
*   解释
*   [可扩展的](https://docs.python.org/3/extending/extending.html)——向 Python 添加一段用 C 或 C++等语言编写的代码的可能性
*   [可嵌入](https://docs.python.org/3/extending/embedding.html) — Python 代码可以嵌入到用 C、C++和 Java 等其他语言编写的程序中
*   对基本数据结构的本地支持
*   对 GUI 应用程序的支持
*   [Juypter](http://jupyter.org/) —一个 web 应用程序，用于创建和共享包含代码(如 python)和富文本元素(可视化、等式、说明性文本、链接等)的文档，文档采用可共享的笔记本格式

**缺点**

Python 中的缩进既有好处也有坏处；虽然许多人讨厌它，但更多的人喜欢它。不考虑缩进，Python 的主要劣势是速度；Python 是一种解释语言，这使得它比编译语言慢。此外，Python 用于开发很少的智能手机应用程序，并且几乎在 web 浏览器中不可用。