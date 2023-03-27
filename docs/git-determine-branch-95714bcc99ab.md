# Git:确定分支

> 原文：<https://itnext.io/git-determine-branch-95714bcc99ab?source=collection_archive---------1----------------------->

## 你在哪个部门？

分支是 git 最常用的特性之一。人们通常总是改变分支，并且需要不断地确定他们在哪个分支上。

![](img/bed7f90fb89fcced3bdacafdb6a4267d.png)

# TL；速度三角形定位法(dead reckoning)

```
git branch
```

# 使用分支命令

第一个处理分支的 git 子命令是 branch 命令。只要写下这个命令，就会显示所有本地分支和您所在的分支的列表。输入:

```
git branch
```

输出会是这样的:

```
 aerabi/add-readme
  aerabi/add-github-actions
*** master**
  the-hotfix-branch
```

您当前的分支用星号和不同的颜色突出显示。

# 使用状态命令

status 命令是一个广泛使用的强大命令，它可以显示当前的分支。输入:

```
git status
```

输出会是这样的:

```
On branch master 
Your branch is up to date with 'origin/master'. 

Changes not staged for commit: 
  (use "git add <file>..." to update what will be committed) 
  (use "git restore <file>..." to discard changes in working directory) 
        modified:   docker-compose.dev.yml 

Untracked files: 
  (use "git add <file>..." to include in what will be committed) 
        .idea/ 
        Dockerfile 
        docker-compose.override.yml.bak 

no changes added to commit (use "git add" and/or "git commit -a")
```

第一行显示您在哪个分支上。第二个显示您的上游分支(如果有)。

# 使用您的 Bash 配置

*这部分适用于基于 Debian 的 Linux 发行版如 Ubuntu 上的 Bash。*

我在 Ubuntu 上用的是 bash。当我在一个目录上时，我的 bash 会显示如下内容:

```
mohammad@pc:~/Dev/my-git-project$ 
```

因此，我一直都知道我在使用哪个用户帐户，在哪个主机上，在那个主机的什么地方。如果您的 git 分支能够额外显示出来，那将会非常有用。为此，使用您选择的编辑器打开您的`.bashrc`文件:

```
vim ~/.bashrc
```

并将下面一行添加到文件的末尾:

```
**PS1**="${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\[\e[91m\]\$(__git_ps1 ' [%s]')\[\e[00m\]\$ "
```

保存并退出。现在，打开一个新的终端(或键入`source ~/.bashrc`)，每当您在 git 存储库中时，git 分支就会出现:

```
mohammad@pc:~/Dev/my-git-project **[master]**$
```

# 结论

试着配置你的 shell 来显示 git 分支，这样可以节省你很多时间。

*   阅读我的[“16 个 Git 技巧和窍门”](https://medium.com/@aerabi/16-git-tips-and-tricks-bf08d0602d3b)
*   [订阅](https://medium.com/subscribe/@aerabi)git 上的每周内容