# 在 Elixir 中加强代码质量

> 原文：<https://itnext.io/enforcing-code-quality-in-elixir-20f87efc7e66?source=collection_archive---------1----------------------->

![](img/b1f35d79d9075269a5a53cd79d609bfb.png)

[杰瑞米·托马斯](https://unsplash.com/@jeremythomasphoto?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

强制你的仙丹代码格式良好，没有警告，希望没有错误。我说的是一个*混合别名*，它将检查代码的质量，可以在本地开发期间使用，也可以在 CI/CD 管道上使用，以强制团队(或您自己)保持一个干净的代码。但是请记住，没有魔法，您仍然有责任创建一个组织良好、高性能、没有安全问题、易于阅读的代码。

## 目录

*   [工具](https://medium.com/p/20f87efc7e66#ef9e)
*   [实施](https://medium.com/p/20f87efc7e66#a4e5)
*   [正在执行](https://medium.com/p/20f87efc7e66#ebb2)
*   [完整的示例文件](https://medium.com/p/20f87efc7e66#48a2)
*   [参考文献](https://medium.com/p/20f87efc7e66#e9c1)

# TL；速度三角形定位法(dead reckoning)

完整的例子在文末的[。](https://medium.com/p/20f87efc7e66#48a2)

# 工具

## 信条和透析器

[Credo](https://github.com/rrrene/credo) 负责检查代码是否符合由社区建立的通用良好代码实践。[透析器](http://erlang.org/doc/man/dialyzer.html)是一个静态分析工具，它可以识别软件差异，如明确的类型错误、由于编程错误而失效或无法访问的代码(解释官方文档)，但我们将使用 [dialyxir](https://github.com/jeremyjh/dialyxir) ，它是用 Elixir 编写的透析器包装器，简化了透析器的使用。

## 索贝洛

Sobelow 是一个面向 Phoenix 框架的安全静态分析工具。如果您没有使用 Phoenix，只需将它从本文描述的 deps、configs 和 alias 中删除即可。

## 混合格式

[混合格式](https://hexdocs.pm/mix/Mix.Tasks.Format.html)任务是[在 Elixir v1.6](https://elixir-lang.org/blog/2018/01/17/elixir-v1-6-0-released/) 中引入的，它用于自动格式化您的代码。使用它的主要好处之一是避免无聊和无休止的讨论，如“我们应该如何格式化代码？”，“线路长度应该是多少？”，等等。 [Elixir 也用它](https://github.com/elixir-lang/elixir/blob/v1.8/Makefile#L215-L216)来执行标准格式。

## 警告为错误

这不是一个工具，实际上，这是一个 [Elixir 编译属性](https://hexdocs.pm/mix/Mix.Tasks.Compile.Elixir.html)，它将当前项目中的警告视为错误，并返回一个非零退出代码，这意味着如果有任何警告，您的项目将不会编译。

## 在我们继续之前，请注意

让所有这些工具和配置都打开*可能会让*很烦。一个没有 doc 的模块、一个错误的规范甚至一个未使用的变量都会阻止你编译和发布代码。实施的级别取决于您，因此您应该根据项目的需要(以及您的压力水平)来调整工具和配置。但是做长期计划是个好主意。

# 履行

好的，让我们更改文件以实施质量检查。让我们一点一点地做，或者更好地说，一个功能一个功能地做。

## mix.exs

首先，让我们安装 dep。在 *deps 列表*中增加以下内容:

```
{:credo, "~> 1.0", only: [:dev, :test], runtime: false},
{:dialyxir, "~> 1.0.0-rc.6", only: [:dev, :test], runtime: false},
{:sobelow, "~> 0.7", only: [:dev, :test], runtime: false}
```

让我们更改项目的配置。将此添加到*项目功能*:

```
elixirc_options: [warnings_as_errors: true],
aliases: aliases(),
dialyzer: [
  plt_file: {:no_warn, "priv/plts/dialyzer.plt"},
  ignore_warnings: ".dialyzer_ignore.exs"
]
```

就像是:

我推荐阅读 [dialyxir 文档](https://hexdocs.pm/dialyxir/0.4.1/readme.html)来了解更多关于它的配置，特别是[持续集成](https://github.com/jeremyjh/dialyxir#continuous-integration)如果你想在你的 CI/CD 管道上执行它。

接下来，让我们在 mix.exs 文件中创建一个别名函数，*如果您还没有创建这个函数*，并添加两个别名:

我喜欢使用一个专用于本地开发的别名(quality)和一个专用于 CI/CD 的别名(quality.ci ),因为我在任务及其参数的顺序上有更多的自由。比如在本地开发上我不想检查代码是否格式化，我只是直接格式化代码；我喜欢在最后看到测试结果。想怎么适应就怎么适应。

## *。透析器 _ignore.exs*

创建文件*。项目根目录下的透析器 _ignore.exs* :

还记得我说过那些工具可能很烦人吗？特别是透析器。不要误会，透析器是惊人的，我相信你应该使用它，但有时你需要忽略一两个警告，这是你可以这样做的文件。

关于这个文件的内容:我们将在测试环境中运行我们的别名(MIX_ENV=test ),透析器声明这些函数是错误的，但是这没什么大不了的，让我们忽略它。

## 克雷多

您不需要为了让 credo 工作而改变任何东西，但是默认的规则可能不适合您的项目。你可以使用一个 [.credo.exs](https://github.com/rrrene/credo#configuration-via-credoexs) 文件或者使用特殊注释的[来改变它。](https://github.com/rrrene/credo#inline-configuration-via-config-comments)

## 。gitignore

添加到。gitignore:

```
# Dialyzer
/priv/plts/*.plt
/priv/plts/*.plt.hash# Sobelow
.sobelow
```

这些文件是环境相关的。

# 执行

终于，你准备好了！😅

首先，让我们创建透析器将存储 plt 文件的目录:

`mkdir -p priv/plts`

然后在您的终端中执行:

`MIX_ENV=test mix quality`

休息一会儿，喝杯咖啡，第一次执行将需要一些时间来建立所有透析器人工制品，但这些文件将被缓存，不要担心。

并将 CI/CD 管道更改为执行:

`MIX_ENV=test mix quality.ci`

每当该命令在您的代码中发现问题时，CI/CD 管道将停止并返回一个失败。

# 完整的示例文件

*mix.exs*

*。透析器 _ignore.exs*

。gitignore

# 参考

*   [采用灵丹妙药——从概念到生产](https://pragprog.com/book/tvmelixir/adopting-elixir)
*   [https://blog.dnsimple.com/2018/10/elixir-dialyzer-in-ci/](https://blog.dnsimple.com/2018/10/elixir-dialyzer-in-ci/)

*你正在寻找一家创意公司来实现你的下一个想法吗？退房* [*LNA 系统*](https://lnasystems.com.br) *和我们聊聊。*