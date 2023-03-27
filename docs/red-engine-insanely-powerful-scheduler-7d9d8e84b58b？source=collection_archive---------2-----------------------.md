# 火箭技术:功能强大得惊人的调度程序

> 原文：<https://itnext.io/red-engine-insanely-powerful-scheduler-7d9d8e84b58b?source=collection_archive---------2----------------------->

![](img/a3bec18a454e29129b953a3a2cc7250e.png)

> **注:**该项目最近更名为 Rocketry(之前为 Red Engine)。

有很多用于调度(Python)代码的库:

*   Crontab
*   Sched
*   日程安排
*   APS scheduler
*   气流

如果您需要简单的东西，Crontab 或 Sched 可能适合您的需要。如果您需要企业解决方案，您可以选择 Airflow。你可以有非常简单或复杂的东西，但不能介于两者之间。开始使用气流需要很多时间，而其他替代品很快就会在更高级的用例中出现功能不足的情况。没有类似 FastAPI 或 Flask 的调度解决方案:一些简单易用但适合复杂问题的解决方案。

然而，时过境迁，现在我们有了这样的解决方案:[火箭](https://rocketry.readthedocs.io/)。Rocketry 是一个现代的调度框架，它是干净的、高效的、广泛的和可定制的。看起来是这样的:

它不仅仅是一个漂亮干净的语法。它还具有:

*   **并发**:你也可以在不同的线程或进程中运行任务
*   持久化:任务日志可以放在任何数据存储中，包括 SQL 数据库
*   **流水线**:一个接一个地运行任务，或者将一个任务的输出作为另一个任务的输入
*   **定制**:扩展调度语法或在运行时修改会话

Rocketry 是自动化 Python 应用程序的优秀后端。它不会对你应该如何做事做很多假设。

## 入门指南

那么，如何使用它呢？首先，我们需要 pip 安装它:

```
pip install rocketry
```

在[rockerry 的教程部分](https://rocketry.readthedocs.io/en/stable/tutorial/index.html)有几个不同难度的教程。本文的其余部分不是重复那些教程中的内容，而是展示该框架必须提供的一些核心特性。

但是在开始使用这些功能之前，如果您想暂停一下四处看看，这里有相关的链接:

*   源代码:[https://github.com/Miksus/rocketry](https://github.com/Miksus/rocketry)
*   文件:[https://rocketry.readthedocs.io/](https://rocketry.readthedocs.io/)
*   发布(PyPI):[https://pypi.org/project/rocketry/](https://pypi.org/project/rocketry/)

## 行程安排

Rocketry 有一个独特的条件语法，也支持逻辑运算。条件是根据时间或某种状态为真或为假的语句，它们用于确定是否允许启动任务或是否应该终止任务。它们是框架中调度机制的构建块。有超过 100 个内置的条件语句，可以使用逻辑运算符任意扩展:AND、OR 和 NOT。您也可以从简单的函数中创建自己的条件。

以下是调度语法的一些示例:

关于调度语法和选项的更多信息可以在文档中找到[。如果由于静态分析或健壮性的原因，您不希望使用字符串语法，您可以自由地传递底层 Python 类。](https://red-engine.readthedocs.io/en/stable/condition_syntax/index.html)

## 管道铺设

Rocketry 具有内置的条件，可以在一个任务成功、失败或简单启动后执行另一个任务。还可以将一个任务的返回值设置为另一个任务的输入参数。下面是利用这两种功能的简短演示:

## 并行化

任务也可以在主循环上运行，或者可以并行化，在单独的线程或单独的进程上运行。您可能希望在单独的线程上运行 IO 密集型任务，在单独的进程上运行 CPU 密集型任务，以提高系统的性能。在某些情况下，您可能希望没有并行化，而在主线程和进程中运行您的任务。执行类型是任务设置中的一个简单参数，很容易根据需要进行更改。以下是这些选项的示例:

## 元任务

火箭技术在运行时也暴露了它的许多机制。您可以创建任务，修改任务，删除任务，关闭调度程序或只使用常规任务重新启动它。下面是一个在运行时操作会话的示例:

如果您希望与运行时会话交互，执行类型必须是*主*或*线程*。元任务允许您创建真正自主的应用程序，这些应用程序能够自动更新自己或者动态地改变任务。您可以创建一个通过元任务与运行时环境通信的 API。他们提供了大量的定制。

## 还有什么？

这里有很多我们没有涉及的特性:自定义条件、任务终止、自定义参数和大量的调度条件。Rocketry 也建立在几个抽象层上，所以它支持几种类型用户的需求:新手到高级用户。

如果你喜欢这个库所提供的东西，可以考虑在 Github 上留下它的名字，并向其他人提及它。该库是开源的(有麻省理工学院的许可)，它的开发是因为编程很有趣，项目很棒。

**相关链接:**

*   文档:[https://rocket ry . readthedocs . io](https://rocketry.readthedocs.io)
*   源代码:【https://github.com/Miksus/rocketry 
*   上映日期:【https://pypi.org/project/rocketry 