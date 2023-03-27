# 在 5 分钟内创建、构建并发布 Python3 pip 模块

> 原文：<https://itnext.io/create-build-and-ship-a-python3-pip-module-in-5-minutes-31dd6d9d5c8f?source=collection_archive---------3----------------------->

![](img/1d97404a81b0f8b4db2c012a9fc5d9fe.png)

兰迪·法特在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

如果你使用 Python，你可能会在路上遇到 **pip** 。Pip 是 Python 模块最常用的包管理器。

> **Pip** 是“Pip 安装包”[维基百科](https://en.wikipedia.org/wiki/Pip_(package_manager))的[递归首字母缩略词](https://en.wikipedia.org/wiki/Recursive_acronym)

去年是 pip 的 10 岁生日，这使得它成为一款优秀的软件——这在当今是罕见的。我记得皮普被认为是一个累赘的时候。皮普被贴上了船上第二个叛逆船长的标签，众所周知，他与发行的主要包装系统相冲突，带来了各种各样与安全和稳定性相关的问题。

今天，有了容器——比如 Docker——这些担忧就不复存在了。应用程序代码及其依赖关系运行在一个受控的、隔离的环境中。漏洞测试变得更加容易，也不再需要与系统管理员斗争。

Python 的使用和受欢迎程度也在飙升——这要归功于机器学习。这使得世界渴望更多的 Python 模块。

让我们运送一些模块。

# 运送 pip 包裹—自动化方式

## 先决条件

*   PyPi 账户。在此注册[并在安全的地方保存您的凭证。](https://pypi.org/account/register/)
*   码头工人。安装说明: [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce) 或 [Mac](https://docs.docker.com/docker-for-mac/install/) 。

你还需要运行 make 的能力，有一个 POSIX 兼容的 shell 和一个代码编辑器——几乎所有的 Linux、Unix 或 Mac 系统都可以工作。

## PyShipper

PyShipper 是一个自动化的部分，完成建立一个模块结构的繁重工作，添加一些样板代码，并提供一个管道来构建、测试和发布一个模块——当你更新或创建一个模块时节省时间。

相关代码大部分在 Makefile 中，补充了一些 Docker 配置和一个稍加修改的 setup.py。对于好奇的读者来说，之前的这篇文章解释了基础。 [PyShipper](https://github.com/LINKIT-Group/pyshipper) 是一个完成 Python 模块交付工作的版本。

## 步伐

1.  想个名字
2.  创建新的模块目录
3.  配置变量
4.  运行和编辑代码
5.  制作模块
6.  出版

我将介绍每一步，并解释 PyShipper 在哪里以及如何自动完成一些事情。

## 1.想个名字

如果你发现很难想出一个好名字，你并不孤单:

> 计算机科学只有两个硬东西:缓存失效和事物命名。—菲尔·卡尔顿

我总是纠结于给事物命名。在缓慢思考模式下数小时后，我通常会以一个描述解决了什么问题的名称结束——因此命名为 PyShipper。如果取了一个名字，通常可以在名字前或后加上“Py”、“Script”、“Tool”或类似的东西。

命名是我还没有弄清楚如何自动化的事情之一。如果有什么机器学习相关的想法，请分享！

## 2.创建新的模块结构

这个关于 Python 包的[文档](https://python-packaging.readthedocs.io/en/latest/minimal.html)描述了所需的包结构。即使我们自动化了这一步，不言而喻，文档仍然是必不可少的阅读材料——对调试很有用。

让我们来看看俏皮话。

```
# get a copy of PyShipper and change into it
git clone [https://github.com/LINKIT-Group/pyshipper.git](https://github.com/LINKIT-Group/pyshipper.git)
cd pyshipper# fork (copy/ paste) the contents to a new directory
make fork dest=~/${YOUR_NEW_MODULE_NAME}
```

这会将 PyShipper 的剥离版本分支到一个名为~/${YOUR_NEW_MODULE_NAME}的新目录中。LICENSE 和 README.md 等文件不会被复制。毕竟，您拥有新模块，因此需要向它添加许可证和文档——没有附加的许可证条件。

出于回顾的目的，这是 Makefile fork 目标:

通过 Makefile 派生(复制)一个目录

安全说明:只有在没有目录或目录为空的情况下，才会复制文件。

## 3.配置变量

我们的下一步是切换到新目录并编辑/variables 文件:

/变量文件的内容

```
# switch to the new module
cd ~/${YOUR_NEW_MODULE_NAME}# edit the /variables file
{replace_with_your_editor} variables
```

/variables 中的 NAME 变量由 Makefile 拾取，所有变量都导出到 Docker 容器的环境中，在容器执行过程中由 setup.py 使用。

Makefile 行花了我几个小时来工作，正确地去除强引号几乎总是很痛苦。当我偶然发现这份文档时，问题变得简单了——至少对 GNU AWK 来说是第 47 号。

将文件中的变量检索到 Makefile 中

虽然一行 Makefile 代码很难，但是将变量传递给 setup.py 要容易得多。我只需要一种将所有变量传递给容器的方法。一个短~/。容器空间中的 profile 文件完成这项工作——注意，要完成这项工作,/variables 文件也必须通过链接或挂载存在。

环境变量通过。轮廓

最后，这是/setup.py 文件，它在容器环境中加载变量。小 lambda 确保在变量不存在时插入空字符串。

带有环境变量的 Setuptools setup.py

## 4.运行和编辑代码

PyShipper 附带了一个/module 目录，其中包含模块的样板代码。还有一种编码模式，带有一个最小的类似“hello-world”的函数，可以以 CLI 和 import 两种方式调用。

当然，所有的 python 3——正如 Python2 于 2020 年 1 月 1 日停产一样！

```
# enter the container runtime
make shell# test run the module in CLI
python3 -m module --name "PyShipper"# start Python3, import the module and test
python3>>> import module
>>> module.main(name="PyShipper")
```

这会运行/module 下的代码。不使用“模块”这个名称，也可以用自己模块的名称来代替。创建一个符号链接来使两者都工作——在发布的版本中，只有您自己的名字可以用来引用该模块。

如果你是一个 Python 开发者，我确信你需要知道下一步做什么。所有要编辑的模块代码都在/模块中——祝您玩得开心！

## 5.制作模块

当你有一个最小的工作版本时，就是时候构建这个包了。你可能想首先做的一件事是检查/variables 中的版本，并确保它是更新的——我刚刚提醒自己在下一个版本中自动执行这个操作，谁喜欢跟踪版本呢？—右；).

```
# build the module -- this runs setup.py in the container
make module# better practice version of the former
# includes pylint code quality testing
make pylint module
```

这是连接到上面一行程序的 Makefile 配置:

## 6.出版

构建过程的输出是一个 gzipped tar 归档文件，位于/dist 目录中。通过将这个文件上传到[PyPi](https://pypi.org/)——Python 包索引——模块就发布了，每个人都可以通过 Python pip 进行安装。

```
# upload the module
make upload
```

上面的命令提示输入用户名和密码。您需要插入您的 [PyPi](https://pypi.org/) 凭证——参见本章开头的先决条件。

上传目标的 Makefile 片段是:

使用 twine 上传到 PyPi

在您创建了模块，并将其发布在 [PyPi](https://pypi.org/) 上之后，您可以将它安装在任何有能力的系统上，并像使用任何其他 Python3 pip 模块一样使用它。

```
# install the module on system
sudo python3 -m pip install ${YOUR_NEW_MODULE_NAME}
```

# 最后的想法

我自己已经在使用这种自动化，我对它很满意。我定期为特定任务创建小的 Python 模块；之后在容器或无服务器环境中导入它们非常容易。

现在，有了这种自动化，更多的重复性工作被削减，这意味着有更多的时间花在其他创新上。

我希望你会发现这同样有用。如果这有助于(至少对你们中的一些人来说)为 Python 生态系统做出贡献，我会很高兴。

Python 模块写的开心！