# 通过发誓来改善你的工作流程

> 原文：<https://itnext.io/improve-your-workflow-by-swearing-3d6855052cf4?source=collection_archive---------0----------------------->

## 如何

## 不，真的让 F 字流

![](img/d34d4f832dfbdfae6def4233f3223fab.png)

[学分](https://st2.depositphotos.com/6367796/9368/v/600/depositphotos_93684108-stock-illustration-pop-art-comics-icon-wtf.jpg)

## 介绍

*免责声明，本文包含脏话。*主要是 F**k

几个月前我在我的博客里写了一些化名。**bashrc文件。有人提示我安装一个名为[他妈的](https://github.com/nvbn/thefuck)的库，这立刻改变了我的工作流程。这是一个基于 shell 的应用程序，可以纠正控制台命令中以前输入的错误。默认情况下，每当你犯了一个错误，你必须在你的控制台上输入 ***fuck*** 。但是他们也引入了一个奇妙的即时模式。**

当我第一次听说这个工具时，我笑了，我认为这是一个很好的噱头，但老实说，在使用了几个月之后，我不知道没有它我是如何使用终端的。这可能是我拥有的最好的命令行助手之一。

## 安装库

根据您的机器使用的不同，有许多不同的命令。

OSX / Mac 使用自制软件(或通过 linux 上的 linuxbrew)

```
brew install thefuck
```

在 Ubuntu / Mint 上，用以下命令安装它:

```
sudo apt update
sudo apt install python3-dev python3-pip python3-setuptools
sudo pip3 install thefuck
```

在 FreeBSD 上，使用以下命令安装它:

```
pkg install thefuck
```

在 ChromeOS 上，使用 chromebrew 通过以下命令安装它:

```
crew install thefuck
```

在其他系统上，使用 pip 安装它:

```
pip install thefuck
```

## 设置别名

一旦库安装到您的机器上，您将需要设置一个别名。这可以在以下文件 ***中完成。bash_profile*** ， ***。巴沙尔*，。zshrc** 或机器上的任何其他启动脚本。

```
eval $(thefuck --alias)
# You can use whatever you want as an alias, for those who don't like to swear:
eval $(thefuck --alias duck)
```

要验证库现在是否已安装，请重新加载您正在使用的终端，并键入以下内容。

```
fuck --yeah
```

## 启用实验即时模式

只有 Python 3、bash 或 zsh 支持即时模式。Zsh 的自动更正功能也需要被禁用，这样库才能正常工作。要启用即时模式，请添加标志

```
--enable-experimental-instant-modeEXAMPLE:
eval $(thefuck --alias --enable-experimental-instant-mode)
```

## 使用图书馆

假设你打错了 git branch

```
git brnch
```

他妈的会在你的终端显示以下信息

```
did you mean
	git branch
```

现在我们不用输入 git branch -r，而是直接说

```
fuck
```

这将自动纠正你的最后一个命令，节省你的打字时间。

```
thewebuiguy@mbp Desktop % git brnch
git: 'brnch' is not a git command. See 'git --help'.The most similar command is
	branch
thewebuiguy@mbp Desktop % fuck
git branch [enter/↑/↓/ctrl+c]
```

## 结论

虽然它有一个有趣的名字，但它是目前最有用的命令行工具之一。它允许您快速纠正在执行简单的重复性任务(如推送 Git 或运行 bash 脚本)时可能犯的错误。

要了解更多信息，你可以在这里阅读他们的[完整文档。](https://github.com/nvbn/thefuck)

如果您想注册访问更多媒体内容，请点击以下[链接](https://thewebuiguy.com/membership)。这将使我能够写更多的故事，通过一个小的委员会从媒体。