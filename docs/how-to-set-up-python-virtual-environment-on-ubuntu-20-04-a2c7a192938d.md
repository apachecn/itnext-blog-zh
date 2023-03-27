# 如何在 Ubuntu 20.04 上设置 Python 虚拟环境

> 原文：<https://itnext.io/how-to-set-up-python-virtual-environment-on-ubuntu-20-04-a2c7a192938d?source=collection_archive---------0----------------------->

![](img/fe38f27e17c87a59c93b581292d007e3.png)

就像上次我又给自己买了一台“新”笔记本电脑，一台 x270 耶！x270 是 x 系列的最后一个型号，具有 1 个内置电池和 1 个外置电池，即所谓的“电源桥”(电池续航时间超过 12 小时)，允许在不关闭笔记本电脑的情况下更换电池。

我又一次需要建立一个 Python 虚拟环境，我又一次搜索解决方案，只是为了找到我自己的文章！

继续我之前的文章，在 Ubuntu 18.04 上处理同样的情况，我决定用新学到的信息更新知识库。

旧文:
freeCodeCamp:[https://www . freeCodeCamp . org/news/virtualenv-with-virtualenvwrapper-on-Ubuntu-18-04/](https://www.freecodecamp.org/news/virtualenv-with-virtualenvwrapper-on-ubuntu-18-04/)

medium it next:[https://it next . io/virtualenv-with-virtualenv wrapper-on-Ubuntu-18-04-Goran-aviani-d7b 712d 906 D5](/virtualenv-with-virtualenvwrapper-on-ubuntu-18-04-goran-aviani-d7b712d906d5)

让我告诉你，这比以前更容易，因为我们将只做下面两件事:

*   安装 virtualenvwrapper
*   编辑。bashrc 文件

## **先决条件**

在本文中，我将向你展示如何用 pip 3(Python 3 的 pip)设置 virtualenvwrapper。我们不再用 Python 2 来做这件事了，因为现在 Python 2 已经死了:[https://www.python.org/doc/sunset-python-2/](https://www.python.org/doc/sunset-python-2/)

要完成本教程，你需要一台安装了 Ubuntu 20.04 的电脑和一个互联网连接:)此外，一些终端和 Vim 编辑器方面的知识也是有用的。

## 设置虚拟环境

现在右击并选择“在终端中打开”选项在主目录中打开您的终端，或者您可以同时按下键盘上的 *CTRL* 、 *ALT* 和 *T* 键来自动打开终端应用程序。

您首先需要创建一个特殊的目录来保存您的所有虚拟环境，因此继续创建一个名为 virtualenv 的新的隐藏目录。

```
mkdir .virtualenv
```

## **pip3**

现在你应该为 Python3 安装 pip 了。

```
sudo apt install python3-pip
```

确认 pip3 安装。

```
pip3 -V
```

## **virtualenvwrapper**

virtualenvwrapper 是 virtualenv 的一组扩展。它提供了 mkvirtualenv、lssitepackages 等命令，尤其是用于在不同 virtualenv 环境之间切换的 workon。

通过 pip3 安装 virtualenvwrapper:

```
pip3 install virtualenvwrapper
```

## **Bashrc 文件**

我们将修改您的。bashrc 文件，通过添加一行来调整每个新的虚拟环境以使用 Python 3。我们将虚拟环境指向我们在上面创建的目录(。virtualenv ),我们还将指向 virtualenv 和 virtualenvwrapper 的位置。

现在打开。使用 Vim 编辑器的 bashrc 文件。

```
vim .bashrc
```

如果你还没有使用 Vim 编辑器或者你的电脑上没有安装它，你应该现在就安装。它是广泛使用的 Linux 编辑器之一，这是有原因的。

```
sudo apt install vim
```

安装 Vim 后，打开文件。bashrc 文件，方法是键入 *vim。你终端里的 bashrc* 命令。导航到的底部。bashrc 文件，按字母 *i* 进入 Vim 的插入模式并添加这些行:

```
#Virtualenvwrapper settings:export WORKON_HOME=$HOME/.virtualenvsVIRTUALENVWRAPPER_PYTHON=/usr/bin/python3. /usr/local/bin/virtualenvwrapper.sh
```

完成后，按下 *esc* 键，然后输入:wq 并按回车键，该命令将保存并退出 Vim 编辑器。现在你需要重新加载 bashrc 脚本，有两种方法:关闭并重新打开你的终端或者执行 source ~/。bashrc 指挥部

要在 Python3 中创建一个虚拟环境并立即激活它，请在终端中使用以下命令:

```
mkvirtualenv name_of_your_env
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

干得好！

现在，您已经创建了第一个独立的 Python 3 环境！

感谢您的阅读！在我的 freeCodeCamp 个人资料上查看更多类似的文章:[https://www.freecodecamp.org/news/author/goran/](https://www.freecodecamp.org/news/author/goran/)，中等个人资料:[https://medium.com/@goranaviani](https://medium.com/@goranaviani)以及我在 GitHub 页面上创建的其他有趣的东西:[https://github.com/GoranAviani](https://github.com/GoranAviani)