# 带有 API 的调度程序:Rocketry + FastAPI

> 原文：<https://itnext.io/scheduler-with-an-api-rocketry-fastapi-a0f742278d5b?source=collection_archive---------0----------------------->

![](img/0405b7b79b688b76bfc7b7202b0148c2.png)

如果你需要一个调度程序和一种与它们交流的方式，FastAPI 和[rockerry](https://rocketry.readthedocs.io/)是很好的一对。FastAPI 是一个现代的 API web 框架，Rocketry 是一个现代的调度后端。它们都易于使用，范围广泛，并且可以无缝地协同工作。

> 提示:我在这里做了一个完整的例子，你可以直接复制。

# 关于火箭技术和 FastAPI

在我们开始我们的项目之前，先简单介绍一下这些技术。这两个框架都很新，但它们都经过了良好的测试和生产质量。它们看起来也非常相似，尽管它们的目的完全不同(Rocketry 的灵感来自 FastAPI)。接下来，我们将对这两者进行一些讨论。

## 关于火箭技术

Rocketry 是一个现代化的调度框架，具有强大的条件系统，可以用简单的语法实现复杂的调度。它还具有:

*   任务并行化(异步、线程和子进程)
*   任务参数化
*   任务流水线(一个接一个地运行任务)
*   完全可修改的运行时间

特别是异步支持和完全可修改的运行时在我们将 Rocketry 和 FastAPI 结合起来的项目中非常有用。我们可以使用我们的 API 读取任务日志、关闭调度程序或者创建、删除、终止或运行任务。

这是火箭技术的样子:

Rocketry 也有 100 多个内置条件，如果你愿意，你可以创建自己的条件。它可以处理非常复杂的调度策略。

火箭的文档:[https://rocketry.readthedocs.io/](https://rocketry.readthedocs.io/)

## 关于 FastAPI

FastAPI 是一个现代的 web 框架，通常用于构建 REST APIs。它是健壮的、快速的和直观的。它内置了数据验证功能，就像 Rocketry 一样，使用起来非常简单:

FastAPI 的文档:[https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)

# 创建我们的应用程序

现在我们已经了解了我们的工具，接下来我们将创建一个调度器，在其上构建一个 API。我们的应用程序将由三部分组成:

*   FastAPI 应用程序，我们称之为 *api.py*
*   Rocketry 应用程序，我们称之为 *scheduler.py*
*   结合了两者的启动器，我们称之为 *main.py*

在我们开始之前，请确保您的 Python 版本为 3.7 或更高。然后我们需要安装一些软件包:

```
pip install rocketry
pip install fastapi
pip install "uvicorn[standard]"
```

Uvicorn 是一个 web 服务器，我们将使用它来运行 FastAPI 应用程序。

## 创建调度程序

接下来，我们将创建我们的调度程序(scheduler.py):

我们首先创建了 Rocketry 实例，其中我们指定默认执行是异步的。您可以将它设置为您想要的任何值，但是如果您没有 CPU 密集型任务，async 通常是一个很好的选择。创建应用程序后，我们创建了两个任务。请根据问题的需要随意修改任务。

## 创建 API

接下来，我们将创建我们的 API (api.py):

我们创建了 FastAPI 应用程序，然后导入了 scheduler 的应用程序，因为我们需要在我们的 routes 中修改它。最后，我们创建了一些路由，其中一些用于检索信息，一些用于执行某种操作，如设置特定的任务运行。

注意，关闭路径不会关闭 FastAPI。为了关闭这两者，需要将 FastAPI 嵌入到 Rocketry 的任务中，这超出了本文的范围。

## 将 Rocketry 与 FastAPI 相结合

现在我们有了 Rocketry 应用程序(调度程序)和 FastAPI 应用程序(API ),我们需要将它们结合起来，这样我们实际上可以同时运行这两个程序。如果你想测试它们，你可以独立运行它们。

Rocketry 和 FastAPI 都使用 [async](https://docs.python.org/3/library/asyncio.html) 运行，所以我们可以用它同时运行两者。我们需要创建一个异步函数来启动两个应用程序，并在应用程序完成之前保持空闲:

您可能想知道为什么我们需要对 uvicorn 服务器进行子类化。你可以不这样做，但是 Rocketry 不会响应 CTRL + C(手动中断)。这是因为 Uvicorn 会覆盖信号，以便对中断做出响应。我们基本上是把火箭的关闭呼叫加到了紫玉米的退出信号上。

在 *main* 函数中，我们创建了两个异步任务:一个用于 Uvicorn 服务器(运行 FastAPI ),一个用于 Rocketry 的应用程序。然后我们*等待*，这样调度程序和 API 就可以像异步一样一次运行一行 Python 代码。

基本上，这个应用程序设置在 Rocketry 和 FastAPI 之间不断切换。例如，rockerry 检查一个任务的开始条件，然后 FastAPI 响应一个到达 API 的请求，然后执行返回 rockerry 并可能运行该任务。换句话说，没有代码是并行运行的，但是代码执行会在应用程序之间不断切换。这听起来可能很慢，但在大多数情况下，它实际上相当快。

# 接下来呢？

为 API 创建所有相关的路由需要大量的手工工作。我做了一个模板，你可以复制来快速上手:[https://github.com/Miksus/rocketry-with-fastapi](https://github.com/Miksus/rocketry-with-fastapi)。

更多资源:

*   火箭的文档:[https://rocketry.readthedocs.io/](https://rocketry.readthedocs.io/)
*   火箭的源代码:[https://github.com/Miksus/rocketry](https://github.com/Miksus/rocketry)
*   火箭学介绍:[https://it next . io/red-engine-insurally-powerful-scheduler-7d 9d 8e 84 b 58 b](/red-engine-insanely-powerful-scheduler-7d9d8e84b58b)

如果你喜欢你所读的，考虑离开 Rocketry 一个明星，并告诉其他人。开发是由创造令人敬畏的事物的热情推动的。