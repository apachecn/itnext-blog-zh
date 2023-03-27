# 圣诞节的 6 个小贴士

> 原文：<https://itnext.io/6-git-tips-for-christmas-ad320252167?source=collection_archive---------1----------------------->

[Git 周刊](https://medium.com/@aerabi/list/git-weekly-9fe103e35b4b) #23
等级:初学者🎄

![](img/ff6a7c157fcd108da78a2af7c621af92.png)

我从 2015 年开始使用 git。再过几天，2023 年就要开始了，标志着我使用 git 的第八个年头。在这 8 年中，git 发生了很多变化，引入了很多新的实践。

在这篇短文中，我将列举 6 个技巧来处理你的圣诞礼物树上的树枝。🎄

# 🎄1.开关支路

直到最近，切换分支的唯一方法是:

```
git checkout <branch-name>
```

由于切换分支并不是 checkout 命令的唯一用途，因此创建了一个新命令，仅用于在分支之间切换:

```
git switch <branche-name>
```

所以，现在你可以做:

```
git switch master
```

# 🎄2.创建新分支

git 的 branch 命令通常用于列出当前的分支:

```
git branch
```

但它还有一个鲜为人知的功能，创建分支:

```
git branch <new-branch>
```

该命令从当前树创建一个新的分支，但不切换到当前树。

# 🎄3.创建一个新的分支并切换到它

与第一个类似，要创建一个新分支并直接切换到它，需要执行以下操作:

```
git checkout -b <new-branch>
```

git 交换机版本如下:

```
git switch -c <new-branch>
```

旗帜`-c`代表“创造”。该命令相当于:

```
git branch <new-branch>
git switch <new-branch>
```

# 🎄4.切换分支并放弃本地更改

在分支之间切换时，工作树必须不干净。您可以进行一些未提交的本地更改，并且仍然可以在分支之间切换。除非新分支中的改变与未提交的改变冲突，在这种情况下，操作被中止。

要放弃本地更改并切换到目标分支上存在的代码版本，可以使用以下标志:

```
git switch --discard-changes <dest-branch>
```

代替`--discard-changes`，你也可以使用它的别名`--force`或者简称`-f`:

```
git switch --force <dest-branch>
git switch -f <dest-branch>
```

# 🎄5.切换分支和合并

在本地变更和目标分支中的代码版本之间发生冲突的情况下，也可以在切换时启动合并:

```
git switch --merge <dest-branch>
```

通过运行此命令，如果存在冲突，您需要像解决任何其他冲突一样解决它们，然后使用以下内容暂存已解决的路径:

```
git add <path>
```

合并选项的简称是`-m`:

```
git switch -m <dest-branch>
```

# 🎄6.强制创建新分支

如果分支已经存在，通常的分支创建命令(如 git branch、git checkout 或 git switch)会失败。在这种情况下，可以删除旧的分支，并在当前树位置之外强制创建新的分支。这可以通过在 git branch 命令中使用 force 标志来实现:

```
git branch --force <existing-branch>
```

捷径是:

```
git branch -f <existing-branch>
```

如果想强制创建分支，然后立即切换到该分支，可以使用 git switch force 选项:

```
git switch --force-create <existing-branch>
```

快捷键应该是大写的 C:

```
git switch -C <existing-branch>
```

# 请参见

*   [GitHub 降价清单](/github-markdown-cheatsheet-50642835effa)
*   [Git 16 岁生日上的 16 个 Git 技巧和窍门](/16-git-tips-and-tricks-bf08d0602d3b)
*   [Git 17 岁生日上的 17 个 Git 技巧和窍门](/17-git-best-practices-1988c7306e6b)

# 最后的话

我每周都会在 git 上写一篇博文。

*   [订阅](https://medium.com/subscribe/@aerabi)my Medium publishes，以便在新一期 Git 周刊出版时获得通知。
*   在 Twitter 上关注[我，了解更多其他平台上发布的更新和文章。](https://twitter.com/MohammadAliEN)