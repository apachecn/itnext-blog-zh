# 走向 Elixir/Phoenix 任务自动化

> 原文：<https://itnext.io/towards-elixir-phoenix-tasks-automation-684c0388b100?source=collection_archive---------2----------------------->

![](img/3d3dc4e9767745a62e4c013286a1401e.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Ftowards-elixir-phoenix-tasks-automation-684c0388b100)

作为一个 Elixir 迷，我一直在寻找新的工具来帮助我提高 Elixir/Phoenix 的开发技能。我最近遇到了 [ExGuard](https://github.com/slashmili/ex_guard) — *ExGuard 是一个混合命令，用于处理文件系统修改事件。*

可以定制 ExGuard 来自动执行各种药剂/凤凰任务，以满足个人需求。这样做需要一些步骤。

**第一步**:

将 ex_guard 作为依赖项添加到 mix.exs 中—

```
defp deps **do** [
     {:ex_guard, "~> 1.3", only: :dev}
  ]
end
```

**第二步:**

在 Elixir/Phoenix 应用程序的根目录下创建. exguard.exs 文件，并将其定制为自动运行 commads。一个示例配置文件—

```
use ExGuard.Config

guard("formatter", run_on_start: **true**)
|> command("mix format")
|> watch(~r{\.(erl|ex|exs|eex|xrl|yrl)\z}i)
|> ignore(~r{deps})
|> notification(:auto)

guard("credo", run_on_start: **true**)
|> command("mix credo")
|> watch(~r{\.(erl|ex|exs|eex|xrl|yrl)\z}i)
|> ignore(~r{deps})
|> notification(:auto)

guard("test", run_on_start: **true**)
|> command("mix test")
|> watch(~r{\.(erl|ex|exs|eex|xrl|yrl)\z}i)
|> ignore(~r{deps})
|> notification(:auto)
```

第一个将运行 Elixir 1.6 附带的新的 awesome **mix format** 命令。您可能需要提供一个. formatter.exs 文件，它才能正常工作。查看此处的链接[了解更多详情。](https://hashrocket.com/blog/posts/format-your-elixir-code-now)

第二个将运行 credo — *一个针对 Elixir 的静态分析工具。*您需要添加它自己的依赖项，如这里的[所述。](https://github.com/rrrene/credo)

第三个将运行**混合测试**命令并执行你的药剂/凤凰测试。

显然，可以适当地添加/删除/修改上面的命令。

这里 **run_on_start: true** 表示**混合保护**将在启动时运行该命令。

**第三步:**

运行下面的命令，只要任何具有上述模式的文件发生变化，相应的命令就会被执行

```
mix guard 
```

# **通知**

为了让通知正常工作，您需要正确设置您的终端通知程序。在 mac 机上，你可以通过自制软件安装 [**终端通知程序**](https://github.com/julienXX/terminal-notifier)—

```
brew install terminal-notifier
```

对于其他平台，请遵循 [github](https://github.com/slashmili/ex_guard) 上的说明。

一旦设置完成，**混合防护**应该会显示一个漂亮的通知—

![](img/de291328e3b4dff4e689a683e82956ee.png)

对于混合格式命令

# 其他类似工具

此外，检查此项目以运行 Elixir 项目的测试—[https://github.com/lpil/mix-test.watch](https://github.com/lpil/mix-test.watch)

虽然，主要是为了运行测试，它可以被配置为运行其他命令，如 **mix guard** 。

*这是我在 medium 上的第一篇帖子。我希望它对一些人有用。更多详细和深入的技术文章，请点击这里或点击* [*twitter*](https://twitter.com/meraj_enigma) *关注我。*