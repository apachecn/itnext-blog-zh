# OMBD#21:掌握 Git Diff 与这些不太知名的命令

> 原文：<https://itnext.io/master-git-diff-with-these-not-so-known-commands-9fecfa3006d0?source=collection_archive---------3----------------------->

## 在推送前发现错误，以节省时间，同时检查您自己的拉取请求

欢迎来到第 21 期,**O**ne**M**inute**B**etter**D**developer，在这里，通过阅读简短的知识，每次一分钟，你将成为一名更成功的软件开发人员。

## [⏮️](https://jportella93.medium.com/1-minute-to-become-a-better-developer-20-a6f93db2c180) [🔛](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e)⏭️

![](img/e932f71918fdd1675f07f8fc9d618801.png)

我的好友洛尔·尼古拉斯的作品

## 问题是

在请求审查之前，你会花很多时间在 Github 上审查你自己的 pull 请求，你总是会发现需要修改的地方。所以你需要添加/编辑提交，浪费了很多时间。

## 一个解决方案

在提交之前，好好利用`git diff`，这样可以避免以后不得不修改代码。

以下是一些不太为人所知的 git 技巧，从常见到不常见排序如下:

## 暂存或未暂存文件的更改

1.  显示自上次提交以来的所有**未分级的**更改:

```
git diff
```

2.显示自上次提交以来的所有**阶段**变更:

```
git diff --cached
```

3.显示自上次提交以来所有已暂存和未暂存的变更:

```
git diff HEAD
```

## 修订目标

1.  显示特定提交后的变化**:**

```
git log --oneline
# 4833545 cleanup
# c3a1ee6 add navbar
# **ca2f968** initial commitgit diff **ca2f968**
```

**2.显示自最后 n 次提交以来的变化**，例如 2:****

```
git diff HEAD~2
```

## **单行**

1.  **显示自上次提交以来**单个文件的变化:****

```
git diff script.js
```

**2.显示自特定提交以来**单个文件的变化:****

```
git log --oneline
# 4833545 cleanup
# c3a1ee6 add navbar
# **ca2f968** initial commitgit diff **ca2f968** script.js
```

## **变更摘要**

**从您的功能分支运行。**

1.  **显示特定分支上的所有变更( **GitHub PR diff** ):**

```
git diff master
```

**2.显示**在特定分支上有多行**已更改的文件:**

```
git diff master --stat
```

**3.显示特定分支上变更的**单行摘要:****

```
git diff master --shortstat
```

## **奖金**

**最后，这里有一个由 [Shime.sh](https://shime.sh/til/git-diff-tips-and-tricks) 设计的巧妙技巧，通过去掉`+`和`-`符号使你的区分更具可读性，因为它们是有颜色的！**

```
git config --global pager.diff 'sed "s/^\([^-+ ]*\)[-+ ]/\\1/"'
```

## **如果你喜欢这个故事，你可能也会喜欢**

**[](https://jportella93.medium.com/1-minute-to-become-a-better-developer-20-a6f93db2c180) [## 1 分钟成为更好的开发人员(#20)

### 学习如何在一分钟内惰性加载图像。

jportella93.medium.com](https://jportella93.medium.com/1-minute-to-become-a-better-developer-20-a6f93db2c180) [](https://medium.com/codex/a-snazzy-trick-how-to-style-your-console-log-messages-2b23ac281b31) [## 一个时髦的技巧:如何设计 console.log()消息的样式

### 关于在浏览器控制台中应用 CSS 记录消息的快速指南(带有示例)

medium.com](https://medium.com/codex/a-snazzy-trick-how-to-style-your-console-log-messages-2b23ac281b31) 

## (T0)↓↓️ [↓↓(T3) (T4)↓↓年(T5)](https://jportella93.medium.com/one-minute-to-become-a-better-developer-ombd-5b1a1d37468e)**