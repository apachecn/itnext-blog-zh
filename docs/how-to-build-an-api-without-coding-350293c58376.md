# 如何在不编码的情况下构建 API

> 原文：<https://itnext.io/how-to-build-an-api-without-coding-350293c58376?source=collection_archive---------3----------------------->

![](img/6b6662fbe8f037a68d66437d95fd92cd.png)

卡斯帕·卡米尔·鲁宾在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

我在过去的 10 年里一直是一名系统工程师，这意味着当我第一次必须构建一个 API 时，我不知道如何去做。我对 python 相当了解，因为作为一名系统管理员，我经常要写一些或大或小的脚本，但是构建整个 API 对我来说是一个挑战。当然——我可以谷歌一下这个短语，找到几十个关于如何构建 API 的教程，但它们都有点复杂，或者是为初级开发人员创建的(这意味着——假设你已经知道很多编程)。幸运的是，在工作中，我被一家公司指派了一个项目，该公司有一个用 Python Eve 框架编写的 API 原型。我稍后会解释 Python Eve，但它本身并不是一个关于 Eve 的故事。这是一个关于我如何让用 Python Eve 构建 API 变得更加容易的故事。我们走吧。

正如我上面提到的——故事从 Python Eve 开始，也正如前面提到的——我们今天不关注 Eve，但我需要给你一些关于它的背景。Eve 是一个以 REST 为中心的 API 框架，构建在由 [Nicola Iarocci](http://nicolaiarocci.com/) 创建的 Flask 之上。它的伟大之处在于构建完整的 API 毫不费力，因为您只需要启动一个正在运行的 MongoDB 服务器和两个简单的 python 文件(app.py，在大多数基本设置中包含 4 行代码和 settings.py，它们定义了端点、数据库连接和额外的配置，如果您愿意的话)。简单是 Eve 的第一个卖点，但是不要搞错——它根本不会限制你。它的功能非常丰富(它提供了您期望从 API 中得到的大部分东西——一整套 CRUD 操作、过滤、排序、分页、条件请求、数据验证等等)。此外，由于它是建立在 Flask 之上的——大多数 Flask 插件也应该可以与 Eve 一起工作。

因此，让我快速地向您展示"*如何在 5 分钟内构建一个 API*"之后，我将向您展示更快的方法；)

从 Eve 开始，您可以简单地用 pip 安装它:

```
**pip install eve**
```

**注意:请使用 Python 3**

如果我们安装了 Eve，我们需要启动一个 MongoDB 数据库。这完全取决于你的平台和你的设置，但是假设你已经安装了 docker，例如，你可以使用官方的 mongo 镜像:

```
**docker run --name mongo -d mongo**
```

现在，我们只需要两个小文件。一个是假设 app.py(或者 run.py，或者您喜欢的其他名称)包含以下内容:

所以我们简单地导入 Eve，创建新的 Eve 对象并调用 run 函数。现在，这足以启动 Eve 实例，但是我们需要定义所有的端点、到 Mongo 的连接等等。为此，我们需要一个名为 settings.py 的文件(默认情况下，Eve expect 文件的名称是这样的，如果您愿意，可以更改它，但之后您需要在 app.py 中指向它——为了简单起见，我们保留默认值)。该文件的内容可以根据您想要使用的功能数量进行定制，但它必须包含以下内容:

1.  MongoDB 定义—在上面的例子中，我使用了 _URI 版本，但是也可以单独定义 MONGO_HOST、MONGO_PORT、MONGO_DBNAME 和更多选项
2.  RESOURCE/ITEMS _ METHODS——顾名思义定义了资源 REST 方法(所以当你向 *http://your_api/example* 发送请求时允许做什么，以及对于像 [*这样的特定项目允许做什么)*](http://your_api/example/[uuid])
3.  模式定义——这是为 Eve 构建 settings.py 文件的核心和最耗时的部分。这里您需要为每个端点定义一个模式。在这个例子中，我们只有一个模式，只有两个条目 firstname 和 lastname，它们是字符串。现在，我故意让它变得不切实际地简单，因为这是我们将在下一步中自动化的部分，但请记住，在现实生活场景中，您可能只有几个端点，而每个端点中都有很多模式。
4.  端点定义—在步骤 3 中，我们创建了一个模式，在域部分，我们必须定义映射端点<>模式。在我们的例子中，我们简单地创建了映射到 people 变量的 people 端点(我们刚刚在第 9 行定义了它)

这就是你构建完整功能所需要的一切(除了它几乎没有用，但要使它有用，我们只需创建更多的端点和模式，这只是定义数据名称和类型(步骤 3)并将它们映射到端点。这个故事的重点来了——让我给你介绍 EveGenie！

[](https://github.com/DavidZisky/evegenie) [## 大卫齐斯基/伊夫吉尼

### 一个使 Eve 模式生成更容易的工具。兼容 Python3 - DavidZisky/evegenie

github.com](https://github.com/DavidZisky/evegenie) 

EveGenie 可以为您生成完整的 settings.py 文件，方法是向 JSON 文件提供您期望从 API 获得的数据。它的意思是，如果你有由某个 UI、某个工具、你的同事或任何其他东西生成的 JSON 文件，你可以执行 evegenie —它会读取它并为你创建模式和端点定义(+ MongoDB 连接线和 RESOURCE/ITEM_METHODS —所以是整个文件)。

那怎么用呢？就像这样:

```
python geneve.py sample.json
```

*注意:evegenie 只兼容 Python 3.x*

所以，忘记在 5 分钟内构建 API 吧…你可以在 5 秒内完成——在 GitHub repo 中，你还会找到简单的 run.py 文件，所以你只需要做两件事(假设你已经安装了 eve，并且 evegenie repo 已克隆)就可以创建功能齐全且可运行的 API:

1.  ***python geneve . py your _ data . JSON***
2.  **python run . py**

请记住，这将创建一个模式并运行 API，但其中没有数据(您为 evegenie 提供的 json 文件仅用于创建模式，而不是加载数据)。但是因为您有 API 在运行，所以您可以简单地用完全相同的 JSON 文件发送一个 POST 请求。

拥有以这种方式创建 API 的能力并不是开发人员的解决方案——虽然 Eve 仍然有许多功能，并且您肯定可以用它来构建非常丰富的生产级 API——我向您展示了 Eve 的最基本用法，因为我将它用作快速微服务/网络测试的系统工程师。我不使用这个解决方案来创建一个 API——我使用它来创建一个 API，它将帮助我调试 Kubernetes 网络、Istio 测试等问题。但关键是，如果你总是想尝试一些需要运行 API 的东西，但你不能这样做，因为你不是一个开发人员，或者你甚至是前端开发人员，并希望开始转向全栈角色，希望这是一个简单的解决方案。

最诚挚的问候！