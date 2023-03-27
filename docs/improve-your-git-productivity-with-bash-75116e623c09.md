# 使用 Bash 提高您的 Git 生产率

> 原文：<https://itnext.io/improve-your-git-productivity-with-bash-75116e623c09?source=collection_archive---------0----------------------->

![](img/ada5b297c4bede7ffefaef62a98f1fb3.png)

有一些方法可以使用 Git。您可以使用一些用户友好的软件，如 [SourceTree](https://www.sourcetreeapp.com/) 来管理您的工作流和存储库。它有一个漂亮的 Git GUI 供你“点击”。然而，如果你和我一样，只是想使用命令行，那么你来对地方了。

想象一下，当你完成你的工作，然后你想添加所有的变更，写一个提交消息，并推到你正在工作的分支。

```
git status
git add .
git commit -m "Some random messages" 
git push origin dev
```

这对我来说是一个漫长而重复的过程。完成所有这些可能需要几秒到一分钟的时间。我在一周或一年内损失了多少时间——我在这里很夸张，:D

所以我决定找到一种方法来改进这个过程，多亏了 bash，有了一种将这些命令组合在一起的方法。

从现在开始，每次我完成一项任务，这是我唯一使用的命令:

```
gf "Some random messages"
```

看起来这一年我节省了多少时间，:D

我将通过利用 bash 向您展示我是如何做到这一点的。我用的是 Mac，但你也可以在 Linux 机器上用。

在您的主目录中，编辑或创建**。bash_profile** 文件。你可以用任何程序来编辑它。我用的是 Vi。

```
vi /Users/dnguyen/.bash_profile
```

然后我们开始添加快捷方式:

之后，我们需要将任何函数文件加载到当前的 shell 脚本中。

```
source .bash_profile
```

现在，享受黑客。希望对此有所帮助；)