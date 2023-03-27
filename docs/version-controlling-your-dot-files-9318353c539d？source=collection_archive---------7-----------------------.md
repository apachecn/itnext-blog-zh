# 版本控制您的点文件

> 原文：<https://itnext.io/version-controlling-your-dot-files-9318353c539d?source=collection_archive---------7----------------------->

## 与版本控制配置文件类似。巴沙尔？以下是方法。

![](img/732fd858dbd79dd71fcd9d0a770b4bde.png)

照片由 [Goran Ivos](https://unsplash.com/@goran_ivos?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/source-code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

在将您的点文件置于源代码控制之下时，有一些独特的挑战。天真的解决方案——仅仅在您的主目录中创建一个 git 存储库——有缺陷。它们是什么，我们如何做得更好？

## 为什么不在主目录中创建一个 git repo 呢？

1.  很难控制什么进什么出。
    你的主目录中很可能有一大堆你*不想*添加到你的回购中的东西。一种方法是制作一个时髦的`.gitignore`来忽略除了点文件之外的所有东西，这是可以做到的，但是还有另一个问题…
2.  根据我的经验，如果我把一个`.gitignore`放在我的主目录中，然后在`~/myrepo`中有一个 git 存储库，那么在那个子目录中运行的 git 命令似乎会“看到”上一级的`.gitignore`……这可能会导致奇怪的行为。

## 取而代之做什么

我遇到的最简单的方法是:

1.  创建一个名为`dotfiles`的 git 存储库，通常在这里存储您的 git repos。(对我来说，这是`~/git/dotfiles`)。
2.  将要跟踪的文件移动到新的存储库中，并从它们的原始位置对它们进行符号链接。如果愿意，您可以将它们移到不带前导点的名称中。

## 例子

在这个例子中，我们将设置存储库并进行包含我们的`.bashrc`的初始提交。

```
> mkdir ~/git/dotfiles
> mv ~/.bashrc ~/git/dotfiles/bashrc
> ln -s ~/git/dotfiles/bashrc ~/.bashrc
> cd ~/git/dotfiles
> git init
> git add bashrc
> git commit -m '.bashrc'
```

然后，确保每次对版本控制进行更改时都提交:

```
> cd ~/git/dotfiles
> git add bashrc
> git commit -m 'New alias.' # Example commit
```

您可能希望将回购推送到一个源(比如 github ),这样您就可以在其他地方获得它的副本。

现在，您可以尝试更改配置，而不会丢失之前的设置。不再有被注释掉的配置文件，当你不记得之前是如何配置的时候，你的理智会感谢你的！