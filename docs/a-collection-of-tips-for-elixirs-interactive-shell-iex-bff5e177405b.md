# Elixir 的交互式 Shell (IEx)技巧集

> 原文：<https://itnext.io/a-collection-of-tips-for-elixirs-interactive-shell-iex-bff5e177405b?source=collection_archive---------3----------------------->

**更新:** *我为这篇文章制作了一个视频版本，介绍了所有提到的技巧。你可以在 youtube 上看到这里——https://youtu.be/ovjsGZwv4Jw*

距离我上次写**仙丹**或者**凤凰**已经有一段时间了。因此，当我考虑继续写关于**灵药**的文章时，我认为我应该从一些非常基础的东西开始，并且是任何**灵药**开发人员日常工作流程的一部分——那就是**交互外壳(IEx)** 。在本文中，我将重点介绍一些使用 IEx 时非常有用的技巧。

![](img/116698c9c9eed95216f3261cbf0286d5.png)

***我会用 IEx 1 . 10 . 2 版本。如果您使用不同版本的 IEx，有些东西可能会有所不同，甚至可能无法工作。***

您可以从命令行启动 **IEx** shell，只需键入 **iex** ，或者如果您想要启动 phoenix 项目，可以在基于 mix 的 **Elixir** 项目中键入 **iex -S mix phx.server** 或键入 **iex -S mix** 。

在整篇文章中，我将展示来自真实的 iex 会话的输出，但是在大多数情况下，输出将被截断和注释以适应本文。

# **提示 1:寻求帮助**

进入 **IEx** shell 后，首先应该知道的是如何获得内置的帮助，如下所示

```
Erlang/OTP 22 [erts-10.7] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [hipe]Interactive Elixir (1.10.2) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> **h <-- Prints this help**IEx.HelpersWelcome to Interactive Elixir. You are currently seeing the documentation for
the module IEx.Helpers which provides many helpers to make Elixir's shell more
joyful to work with.This message was triggered by invoking the helper h(), usually referred to as
h/0 (since it expects 0 arguments).
```

' **h** '命令打印获得帮助的可用方法的摘要。如果您需要关于特定模块或功能的帮助，您可以按如下方式操作—

```
iex(1)> **h(Enum) <-- Prints help about Enum module**EnumProvides a set of algorithms to work with enumerables.In Elixir, an enumerable is any data type that implements the Enumerable
protocol. Lists ([1, 2, 3]), Maps (%{foo: 1, bar: 2}) and Ranges (1..3) are
common data types used as enumerables:
```

或者需要模块内特定功能的帮助，您可以按如下方式操作—

```
iex(4)> **h(Enum.reverse/1) <-- Prints help about reverse/1 function from Enum module**def reverse(enumerable)[@spec](http://twitter.com/spec) reverse(t()) :: list()Returns a list of elements in enumerable in reverse order.## Examplesiex> Enum.reverse([1, 2, 3])
    [3, 2, 1]
```

# 技巧 2:获取过去表达式的值

获取过去执行的命令的值通常是有用的。特别是，我经常忘记给一个长的**向量查询或复杂的表达式赋值，向上滚动并使用箭头键转到表达式的开头会很麻烦。所以，这个建议有助于解决这个问题。**

```
iex(5)> Enum.reverse([1, 2, 3])
[3, 2, 1]
iex(6)> reversed = **v(5) <-- Assigns value from expression at line (5) to reversed** 
[3, 2, 1]
iex(7)> reversed
[3, 2, 1]
iex(8)>
```

如果是最后一个需要访问其值的表达式，快捷方式就是' **v** ' —

```
iex(8)> **new_reversed = v <-- Assigns value from last expression**
[3, 2, 1]
iex(9)> new_reversed
[3, 2, 1]
iex(10)>
```

# 技巧 3:重新编译项目或模块

我经常在 **IEx** 会议中修改源代码。在这种情况下，要编译完整的项目，你可以做—

```
iex(4)> **recompile**
Compiling 1 file (.ex)
**:ok <-- *Prints :ok when something has changed and compilation successful***iex(2)> recompile
**:noop <-- Prints :noop when nothing has changed**
```

有时我只想编译某个模块，而不是整个项目。在这种情况下，下面的工作—

```
iex(7)> **r MinitwitterWeb.PostController <-- Compiles and reloads module** 
warning: redefining module MinitwitterWeb.PostController (current version defined in memory)
  lib/minitwitter_web/controllers/post_controller.ex:1**{:reloaded, MinitwitterWeb.PostController, [MinitwitterWeb.PostController]}**
```

# **提示#4:搜索并完成命令**

有时对于之前执行的长命令，我需要搜索而不是滚动或重新输入。 **IEx** 提供了进入搜索模式的快捷方式***Ctrl+r*****—**

```
**(search)`': r MinitwitterWeb.PostController **<-- Ctrl+r puts in search mode** iex(9)> r MinitwitterWeb.PostController **<-- TAB/Arrow puts it on a new row****
```

# ****提示 5:启用外壳历史****

**通常，如果退出会话， **IEx** 不会保留 shell 历史。但是如果你想这样做，你可以调用如下的 IEx**

```
**iex --erl "-kernel shell_history enabled"** 
```

**这将保存 shell 历史，并在下次启动时可用。一个有用的方法是在您的 OS shell 中创建一个别名，以便在键入 **iex** 时自动启动该命令。对于 bash shell，可以把下面的放在 **~/。bashrc** —**

```
**alias iex='iex --erl "-kernel shell_history enabled"'**
```

# ****提示#6:突破表达式****

****IEx** 提供了一个特殊的 break 触发器( **#iex:break** )当单独在一行中遇到时，将强制 shell 从任何挂起的表达式中脱离出来，并返回到正常状态:**

```
**iex(17)> defmodule Break do
...(17)>  def break_out do
...(17)> **#iex:break <-Breaks out of current expression**
** (TokenMissingError) iex:17: incomplete expression**
```

# **技巧 7:iex . exs 文件**

**IEx 寻找一个本地 **.iex.exs** 文件或者一个全局 **~/.iex.exs** 文件加载它找到的第一个文件(如果有的话)。你可以在这里读到这一切——https://hexdocs.pm/iex/IEx.html#module-the-iex-exs-file**

**简而言之，您可以导入某些模块，绑定某些变量，或者在这里创建某些别名，这些将在 shell 上下文中自动执行，并且在 shell 启动后可用，这对于大型项目来说非常方便。**

**我创建了一个示例. iex.exs，有两个别名，如下所示**

```
**alias Minitwitter.Accounts.User
alias Minitwitter.Accounts.Relationship**
```

**因此，在调用 shell 之后，我可以将它们称为**用户**或**关系**，因为这些别名将变得可用——**

```
**iex(2)> **Relationship**
Minitwitter.Accounts.Relationship
iex(3)> **User**        
Minitwitter.Accounts.User**
```

# **技巧 8:在 IEx 中加载模块或脚本**

****c/1** 命令编译一个文件。在 **IEx** 会话中，您可以执行以下操作，从文件中编译并加载模块——**

```
**iex(1)> **c "lib/issues.ex" <-- Loads Issues module from this file**
[Issues]**
```

**在**药剂脚本(。exs)** ，它会立即编译并执行文件。所以你会看到如下输出—**

```
**iex(1)> **c "./elixir_book.exs"** **<-- Compiles and executes this script**
FizBuzz
Fizz
Buzz**
```

# **技巧 9:显示信息**

****i/1** 命令可用于显示关于**药剂**期限的信息。如果我在 **IEx** shell 中为之前加载的 **Issues** 模块使用此命令，它会显示如下信息—**

```
**iex(4)> **i Issues <-- Prints information about this term**
Term
 ** Issues**
Data type
  **Atom**
Module bytecode
  []
Source
  **lib/issues.ex**
Version
  [74242156420198707021276849165021118249]
Compile options
  [:dialyzer, :no_spawn_compiler_process, :from_core, :no_auto_import]
Description
  Call Issues.module_info() to access metadata.
Raw representation
  :"Elixir.Issues"
Reference modules
  Module, Atom
Implemented protocols
  **IEx.Info, Inspect, List.Chars, String.Chars****
```

**如果我在之前加载的**用户**别名上运行它，它会显示如下信息—**

```
**iex(1)> **i User**
Term
  **User**
Data type
  **Atom**
Raw representation
  :"Elixir.User"
Reference modules
  Atom
Implemented protocols
  **IEx.Info, Inspect, List.Chars, String.Chars****
```

# ****提示#10:打印所有可用的模块****

**有时，为了诊断问题，您可能必须在 shell 中打印所有可用的模块。这可以通过快捷键 **:+TAB 轻松完成。****

# ****提示#11:列出模块中的所有宏和函数****

**要列出模块中的所有宏和函数，可以使用下面的命令—**

```
**iex(3)> **exports Minitwitter.Accounts <-- Shows all functions and macros from Accounts module**
authenticate_by_email_and_pass/2   authenticated?/3                   change_user/1                      
create_user/0                      create_user/1                      delete_user/1                      
follow/2                           followers/1                        following/1                        
following?/2                       following_ids/1                    forget_user/1                      
get_user/1                         get_user_by/1                      list_users/1                       
remember_user/1                    reset_hash/1                       reset_user_pass/2                  
unfollow/2                         update_user/2                      valid_user?/2**
```

**另一种更好的方法如下—**

```
**iex(8)>  Minitwitter.Accounts.module_info
[
  module: Minitwitter.Accounts,
  **exports: [**
    __info__: 1,
    authenticate_by_email_and_pass: 2,
    authenticated?: 3,
    change_user: 1,
    create_user: 0,
    create_user: 1,
    delete_user: 1,
    follow: 2,
    followers: 1,
    following: 1,
    following?: 2,
    following_ids: 1,
    forget_user: 1,
    get_user: 1,
    get_user_by: 1,
    list_users: 1,
    remember_user: 1,
    reset_hash: 1,
    reset_user_pass: 2,
    unfollow: 2,
    update_user: 2,
    valid_user?: 2,
    module_info: 0,
    module_info: 1
  ],
  attributes: [vsn: [165218174362312500220516424562981707158]],
  compile: [
    version: '7.5.3',
    options: [:dialyzer, :no_spawn_compiler_process, :from_core,
     :no_auto_import],
    source: '/Users/meraj/Development/test/elixir/Phoenix_Playground/1.4/minitwitter/lib/minitwitter/accounts/accounts.ex'
  ],
  native: false,
  md5: <<124, 75, 220, 233, 97, 246, 150, 35, 170, 236, 247, 9, 216, 63, 157,
    150>>
]**
```

**没有参数， **module_info** 返回关键字列表，关键字:`[:module, :exports, :attributes, :compile, :native, :md5]`。其中任何一个都可以作为参数传递，以限定返回值的范围。比如可以做 **Minitwitter。Accounts.module_info(:md5)** 只打印 **md5** 密钥。**

# ****提示#12:列出模块中的所有类型****

****t/1** 命令列出了模块中可用的所有类型，如下所示**

```
**iex(4)> **t Enum <-- Lists all types in Enum module**
[@type](http://twitter.com/type) acc() :: any()[@type](http://twitter.com/type) default() :: any()[@type](http://twitter.com/type) element() :: any()[@type](http://twitter.com/type) index() :: integer()[@type](http://twitter.com/type) t() :: Enumerable.t()**
```

# **技巧#13:多行表达式**

**如果你粘贴一个从某处复制的多行表达式， **IEx** 可能会因为换行而报错。为了做到正确，你可以用括号把它括起来—**

```
**iex(15)> [1, [2], 3] 
[1, [2], 3]
iex(16)> |> List.flatten()
**** (SyntaxError) iex:16: syntax error before: '|>' <-- Error while pasting****iex(16)> ([1, [2], 3]   <-- Works fine when surrounded by parenthesis
...(16)> |> List.flatten()
...(16)> )**
[1, 2, 3]**
```

**在这篇文章中，我提出了一些我认为对任何 **Elixir/Phoenix** 开发者都非常有帮助的技巧，让他/她的生活稍微轻松一些。我希望阅读将是愉快和有益的。仙丹编码快乐！**

# **参考资料:**

1.  **IEx 官方文件—[https://hexdocs.pm/iex/IEx.html](https://hexdocs.pm/iex/IEx.html)**
2.  **本文提到的 mini Twitter app—[https://github . com/imeraj/Phoenix _ Playground/tree/master/1.4/mini Twitter](https://github.com/imeraj/Phoenix_Playground/tree/master/1.4/minitwitter)**

***更多详细和深入的未来技术帖子请关注我这里或上*[*Twitter*](https://twitter.com/meraj_enigma)*。***