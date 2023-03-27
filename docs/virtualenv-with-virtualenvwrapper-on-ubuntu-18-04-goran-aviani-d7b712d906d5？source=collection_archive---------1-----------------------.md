# 在 Ubuntu 18.04 上使用 Virtualenvwrapper 的 Virtualenv

> 原文：<https://itnext.io/virtualenv-with-virtualenvwrapper-on-ubuntu-18-04-goran-aviani-d7b712d906d5?source=collection_archive---------1----------------------->

![](img/f3fab5cc5b4bc4be76695636c80990ab.png)

在我们开始教程之前，我想告诉你我是如何意识到我需要一个关于如何在 Ubuntu 18.04 中的 virtualenvwrapper 之上设置 virtualenvwrapper 的提示的故事。我已经在不同的电脑上做了几次这个过程，每次看起来都和以前有一点点不同。

最近我给自己买了一台新的笔记本电脑，在回家的路上，我读了几本关于“如何在 Ubuntu 18.04 上设置 virtualenvwrapper”的教程。让我告诉你，这看起来真的很简单，因为所有这些教程都非常简单，基本上解释了如何做下面三件事:

*   安装虚拟
*   安装 virtualenvwrapper
*   编辑。bashrc/。bash_profile 或两者

但是尽管所有这些教程都是为了帮助我们在一个现代的 Ubuntu 18.04 系统上安装 virtualenvwrapper，但没有一个对我有用。

在跟随这些教程的过程中，我在试图找出问题所在时出现了几个错误。首先我有一些“ *mkvirtualenv:命令未找到*”，然后一点“*-bash:/usr/bin/virtualenvwrapper . sh:没有这样的文件或目录*”，然后一点“*错误:virtualenvwrapper 无法在您的路径*中找到 virtualenv”。

经过一些研究，我意识到所有的 virtualenvwrapper Ubuntu 18.04 教程都是在 2016 年 4 月(Ubuntu 16.04 的发布日期)之前写的旧文本的副本。
我知道这是真的，因为从 Ubuntu 16.04 开始，vritualenvwrapper 的 pip 安装位置已经从`/usr/local/bin/virtualenvwrapper.sh` 更改为`~/.local/bin/virtualenvwrapper.sh.`注意，本地目录是隐藏的。

所以让我们开始写一个教程，告诉你如何避免上面提到的所有问题。

## 先决条件

在本文中，我将向你展示如何用 pip 3(Python 3 的 pip)设置 virtualenvwrapper。我选择这个版本的 pip 而不是 Python 2 版本，因为 Python 2 的生命周期是 1 月 1 日。2020.

> Python 2 将于…[https://pythonclock.org/](https://pythonclock.org/)退役

要完成本教程，你需要一台安装了 Ubuntu 18.04 的电脑和一个互联网连接:)此外，一些终端和 Vim 编辑器方面的知识也是有用的。我们假设您已经更新和升级了系统。

## 设置虚拟环境

现在右击并选择“在终端中打开”选项在主目录中打开您的终端，或者您可以同时按下键盘上的`CTRL`、`ALT`和`T`键来自动打开终端应用程序。

您首先需要创建一个特殊的目录来保存您的所有虚拟环境，因此继续创建一个名为 virtualenv 的新的隐藏目录。

```
mkdir .virtualenv
```

现在你应该为 Python3 安装 pip 了。

```
sudo apt install python3-pip
```

确认 pip3 安装。

`pip3 --version`

现在通过 pip3 安装 virtualenv。

```
pip3 install virtualenv
```

要找到 virtualenv 的安装位置，请键入:

```
which virtualenv
```

通过 pip3 安装 virtualenvwrapper:

```
pip3 install virtualenvwrapper
```

我们将修改您的。bashrc 文件，通过添加一行来调整每个新的虚拟环境以使用 Python 3。我们将虚拟环境指向我们在上面创建的目录(。virtualenv ),我们还将指向 virtualenv 和 virtualenvwrapper 的位置。

现在打开。使用 Vim 编辑器的 bashrc 文件。

```
vim .bashrc
```

如果你还没有使用 Vim 编辑器或者你的电脑上没有安装它，你应该现在就安装。它是广泛使用的 Linux 编辑器之一，这是有原因的。

```
sudo apt install vim
```

安装 Vim 后，打开文件。在 *vim 中键入 bashrc 文件。你终端里的 bashrc* 命令。导航到的底部。bashrc 文件，按字母 *i* 进入 Vim 的插入模式并添加这些行:

完成后，按下 *esc* 键，然后键入`:wq`并按 enter 键，该命令将保存并退出 Vim 编辑器。完成后，关闭并重新打开您的终端。

要在 Python3 中创建一个虚拟环境并立即激活它，请在终端中使用以下命令:

```
mkvirtualenv name_of_your_env
```

您应该确认这个环境是为 Python3 设置的:

```
Python -V
```

要停用环境，请使用停用命令。

```
deactivate
```

要列出所有可用的虚拟环境，请在您的终端中使用命令 *workon* 或 *lsvirtualenv* (结果与 workon 相同，但显示方式很奇特):

```
workonlsvirtualenv
```

要激活一个特定的环境，请使用您的环境的 workon +名称:

```
workon name_of_your_env
```

有几个有用命令您可能需要在某一天使用:

Rmvirtualenv 将删除位于您的中的特定虚拟环境。虚拟目录。

```
rmvirtualenv name_of_your_env
```

Cpvirtualenv 会将现有的虚拟环境复制到一个新的虚拟环境并激活它。

```
cpvirtualenv old_virtual_env new_virtual_env
```

> 干得好！
> 
> 现在，您已经创建了第一个独立的 Python 3 环境！

感谢您的阅读！在我的个人主页上查看更多类似的文章:[https://medium.com/@goranaviani](https://medium.com/@goranaviani)和我在 GitHub 页面上创建的其他有趣的东西:[https://github.com/GoranAviani](https://github.com/GoranAviani)