# 在 macOS 上升级 Bash

> 原文：<https://itnext.io/upgrading-bash-on-macos-7138bd1066ba?source=collection_archive---------0----------------------->

许多 macOS 用户不知道的一点是，他们使用的是完全过时的 Bash shell 版本。然而，强烈建议在 macOS 上使用 Bash 的新版本，因为它使您能够使用有用的新特性。本文解释了如何做到这一点。

![](img/6f75e1f81e0c019b923664bfef8734a8.png)

# macOS 上的默认 Bash 版本

要查看 macOS 中包含的 Bash 版本有多过时，请执行以下命令:

```
$ **bash --version**
GNU bash, version 3.2.57(1)-release (x86_64-apple-darwin18)
Copyright (C) 2007 Free Software Foundation, Inc.
```

如你所见，这是 [GNU Bash](https://www.gnu.org/software/bash/) 版本 **3.2** ，日期为 **2007** ！Bash 的这个版本包含在 macOS 的所有版本中，甚至是最新的版本。

苹果在其操作系统中包含如此老版本的 Bash 的原因与**许可**有关。从 4.0 版本(3.2 的继承者)开始，Bash 使用 [GNU 通用公共许可证 v3 (GPLv3)](https://www.gnu.org/licenses/gpl.html) ，苹果不(想要)支持。你可以在这里和这里找到一些关于这个[的讨论。GNU Bash 的 3.2 版本是 GPLv2 的最后一个版本，苹果接受了它，所以它坚持使用它。](https://www.reddit.com/r/apple/comments/7v97ls/why_doesnt_apple_use_any_gpl_v3_unix_packages_in/)

这意味着整个世界(例如 Linux)都在使用 Bash 的新版本，而 macOS 用户还停留在十年前的旧版本。在本文撰写之时，GNU Bash 的最新版本是 5.0(见[此处](https://tiswww.case.edu/php/chet/bash/bashtop.html#CurrentStatus))，已经在 2019 年 1 月发布。在本文中，我给出了将系统的默认 shell 升级到 Bash 最新版本的说明。

# 为什么要升级？

但是，如果 Bash 3.2 运行良好，为什么还要费心去获取一个新版本呢？主要原因，对我个人来说，是 [**可编程完成**](https://www.gnu.org/software/bash/manual/html_node/Programmable-Completion.html) 。这是一个允许 Bash 中特定命令自动完成的特性。您可能使用自动补全来完成命令、文件名和变量，方法是开始键入，然后点击*选项卡*来自动补全当前单词(或者点击*选项卡*两次来获得所有可能补全的列表，如果有多个的话)。这是 Bash 的默认完成方式。

然而，可编程补全远远不止于此，因为它允许依赖于上下文的特定命令补全。例如，想象一下，输入`cmd -[tab][tab]`，然后看到适用于该命令的所有选项的列表。或者键入`cmd host rm [tab][tab]`，然后看到某个配置文件中指定的所有“主机”的列表。可编程完成允许这样做。

可编程完成逻辑(由命令的创建者)在完成规范中定义，通常以**完成脚本**的形式。这些完成脚本必须来源于您的 shell，以启用命令的完成功能。

问题是 Bash 的可编程补全特性从 3.2 版本开始已经得到了扩展，大多数补全脚本都使用这些新特性。这意味着这些完成脚本不能在 Bash 3.2 上工作，这意味着如果您继续使用默认的 macOS shell，您将错过许多命令的完成功能。

通过升级到 Bash 的新版本，您可以使用这些完成脚本，这非常有用。我写了一整篇文章，名为 macOS 上 Bash 的 [**可编程完成**](https://medium.com/@weibeld/programmable-completion-for-bash-on-macos-f81a0103080b) ，它解释了在升级到较新的 Bash 版本后充分利用 macOS 上的可编程完成所需要知道的一切。

# 怎么升级？

要将 macOS 系统的默认 shell 升级到 Bash 的最新版本，您必须做三件事:

1.  **安装最新版本的 Bash**
2.  **“白名单”新 Bash 作为登录外壳**
3.  **将新的 Bash 设置为默认 shell**

每一步都非常简单，如下所述。

> **注意:**下面的指令并没有改变 Bash 的旧版本，而是安装了一个新版本，并设置为默认。这两个版本将并存于你的系统中，但是你可以忽略旧版本。

## 安装

我推荐用[自制软件](https://brew.sh/)安装最新版本的 Bash:

```
**brew install bash**
```

已经这样了！

为了验证安装，您可以检查您的系统上现在有两个 Bash 版本:

```
$ **which -a bash**
/usr/local/bin/bash
/bin/bash
```

第一个是新版本，第二个是旧版本:

```
$ **/usr/local/bin/bash --version**
GNU bash, version 5.0.0(1)-release (x86_64-apple-darwin18.2.0)
Copyright (C) 2019 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <[http://gnu.org/licenses/gpl.html](http://gnu.org/licenses/gpl.html)>This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.$ **/bin/bash --version**
GNU bash, version 3.2.57(1)-release (x86_64-apple-darwin18)
Copyright (C) 2007 Free Software Foundation, Inc.
```

由于在`PATH`变量中，新版本的目录(`/usr/local/bin`)默认出现在旧版本的目录(`/bin`)之前，所以当你输入`bash`时使用的版本是新版本:

```
$ **bash --version**
GNU bash, version 5.0.0(1)-release (x86_64-apple-darwin18.2.0)
...
```

到目前为止，一切顺利。现在您已经将此版本设为默认版本。

## 白名单

UNIX 包括一个安全特性，将可用作[登录 shell](https://docstore.mik.ua/orelly/unix3/upt/ch03_04.htm)的 shell(即登录到系统后使用的 shell)限制在“可信”shell 列表中。这些外壳列在`[/etc/shells](https://bash.cyberciti.biz/guide//etc/shells)`文件中。

因为您希望使用新安装的 Bash shell 作为默认 shell，所以它必须能够充当登录 shell。这意味着，您必须将它添加到`/etc/shells`文件中。您可以 root 用户身份编辑该文件:

```
$ **sudo vim /etc/shells**
```

并将`/usr/local/bin/bash` shell 添加到它的内容中，这样文件看起来就像这样:

```
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
/usr/local/bin/bash
```

这一步就到此为止！

## 设置默认外壳

此时，如果您打开一个新的终端窗口，您将仍然使用 Bash 3.2。这是因为`/bin/bash`仍然被设置为默认 shell。要将此更改为您的新 shell，请执行以下命令:

```
$ [**chsh**](https://en.wikipedia.org/wiki/Chsh) **-s /usr/local/bin/bash**
```

就是这样！当前用户的默认 shell 现在设置为 Bash 的新版本。如果您关闭并重新打开终端窗口，您现在应该已经在使用新版本了。您可以按如下方式验证这一点:

```
$ **echo $BASH_VERSION**
5.0.0(1)-release
```

`chsh`命令只为执行该命令的用户更改默认外壳。如果您也想改变其他用户的默认外壳，您可以通过使用另一个用户的身份重复该命令(例如使用`[su](https://en.wikipedia.org/wiki/Su_(Unix))`)。最重要的是，您可能想要更改 root 用户的缺省 shell，您可以这样做:

```
$ **sudo chsh -s /usr/local/bin/bash**
```

这样，如果你以 root 用户身份使用`sudo su`打开一个 shell，它也会使用新的 Bash 版本。

# 重要注意事项

## 脚本中的用法

如前所述，您没有更改 Bash 的默认版本，而是安装了一个新版本并将其设置为默认版本。Bash 的两个版本并存于您的系统中:

*   `/bin/bash`:旧版本
*   `/usr/local/bin/bash`:新版本

在 shell 脚本中，您通常会有一个类似于以下脚本的 [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) 行:

```
#!/bin/bash
echo $BASH_VERSION
```

需要注意的是，这个 shebang 行*明确地*引用了 Bash 的**旧**版本(因为它指定了`/bin/bash`)。这意味着如果你运行这个脚本，它将被旧版本的 Bash 解释(你可以在脚本的输出中看到，它将是类似于`3.2.57(1)-release`的东西)。

对于大多数情况，这可能不是问题。但是如果您想让 Bash 的新版本明确地解释您的脚本，您可以像这样修改 shebang 行:

```
#!/usr/local/bin/bash
echo $BASH_VERSION
```

现在，输出将类似于`5.0.0(1)-release`。但是，请注意，这个解决方案是**不可移植的**，这意味着，它可能无法在其他系统上工作。这是因为，其他系统很可能没有位于`/usr/local/bin/bash`的 shell(而`/bin/bash`几乎是一个标准)。

要结合两者的优点，您可以使用下面的 shebang 代码:

```
#!/usr/bin/env bash
echo $BASH_VERSION
```

这是为 shebang 线推荐的[格式。它通过检查`PATH`并使用第一个遇到的`bash`可执行文件作为脚本的解释器来工作。如果新版本的目录在`PATH`中位于旧版本的目录之前(这是默认的)，那么将使用新版本，脚本的输出将类似于`5.0.0(1)-release`。](https://www.cyberciti.biz/tips/finding-bash-perl-python-portably-using-env.html)

## 为什么不使用符号链接?**?**

与其同时处理 Bash 的两个版本，难道不能删除旧版本，用新版本代替旧版本吗？例如，通过在`/bin/bash`中创建一个指向新版本的符号链接，如下所示？

```
$ **sudo rm /bin/bash**
$ **sudo** **ln -s /usr/local/bin/bash /bin/bash**
```

这样，即使带有`#!/bin/bash`的脚本也可以被新版本的 Bash 解释，为什么不这样做呢？

你可以这么做，但是你要绕开一个 macOS 的安全特性，叫做 [**系统完整性保护(SIP)**](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/Introduction/Introduction.html) ( [维基](https://en.wikipedia.org/wiki/System_Integrity_Protection))。这个特性甚至禁止 root 用户对某些目录进行写访问(这就是为什么它也被称为“无根”)。这些目录在此[列出](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/FileSystemProtections/FileSystemProtections.html)并包括`/bin`。这意味着即使作为根用户，您也不能执行上述命令，因为您不允许在`/bin`中删除任何内容或创建任何文件。

解决方法是禁用 SIP，在`/bin`中进行更改，然后再次启用 SIP。启用和禁用 SIP 可以根据此处的说明[来完成。它要求你启动你的机器进入**恢复模式**，然后使用`csrutil disable`和`csrutil enable`命令。如果您想彻底替换旧的 Bash 版本，或者满足于两个 Bash 版本并存，这取决于您。](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html)

# 参考

*   [**https://apple.stackexchange.com/a/292760**](https://apple.stackexchange.com/a/292760)