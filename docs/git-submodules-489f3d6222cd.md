# Git 子模块

> 原文：<https://itnext.io/git-submodules-489f3d6222cd?source=collection_archive---------2----------------------->

在开发一个项目时，您通常希望在其中包含另一个项目。这种需求的一些常见情况如下:

*   包含格式样式指南或构建工具的共享存储库
*   第三方库
*   您要在当前项目中登记的回购

在这些情况下，您可以使用 git 子模块。

![](img/629a1406c4fb406fdbe904fd2617ab83.png)

# 创建子模块

要在 git repo 中创建子模块，您需要运行以下命令:

```
git submodule add <url> <directory>
```

这里的`<url>`是另一个 repo 的 URL 的占位符，而`<directory>`是您希望子模块所在的目录名。例如，以下命令从 GitHub repo `django-twitter-clone`创建一个子模块，并将其存储在目录`django-twitter-clone`中:

```
git submodule add git@github.com:aerabi/django-twitter-clone.git django-twitter-clone
```

执行该命令后，在回购的根目录下会创建一个名为`.gitattributes`的文件，内容如下:

```
[submodule "django-twitter-clone"]
   path = django-twitter-clone
   url = git@github.com:aerabi/django-twitter-clone.git
```

对于每个额外的子模块，该文件中会记录一个新条目。

# 克隆带有子模块的项目

让我们假设您想要克隆一个带有子模块的 repo。在通常的 git 克隆之后，包含子模块的路径仍然是空的。要填充它，您需要另外运行以下命令:

```
git submodule init
git submodule update
```

或者一气呵成:

```
git submodule --init update
```

# 有助于子模块

子模块是它自己的 git repo。主 repo 及其子模块上有不同的分支。

让我们假设您的子模块存在于路径`shared`下，您可以并且想要直接提交给它的主模块。要创建子模块，首先，转到该路径并更改分支:

```
cd shared
git checkout master
git pull
```

进行更改并提交给子模块。然后推送到子模块的主分支:

```
git push origin master
```

返回主机存储库，验证您对子模块所做的更改:

```
cd ..
git diff --submodule
```

Git 应该向您显示自从您的分支上的上一个版本以来添加到子模块的提交列表。如果您满意，添加子模块并提交更改:

```
git add shared
git commit -m "Update the submodule"
```

总之，这就是我们在这里所做的:

*   对“其他”回购进行一些更改
*   将更改推送到主服务器
*   移动“子模块指针”,指向主 repo 上最新版本的主模块
*   在主 repo 中提交新的指针值

这些动作中的每一个都可以独立完成。

*   人们可以独立地克隆“其他”回购协议，并对其进行一些修改。
*   人们可以通过接受和合并拉取请求来改变“其他”回购协议的主协议。
*   一个人只能通过改变子模块上的分支并提取最新的改变来更新已经被其他人贡献的子模块。

# 摘要

处理 git 子模块并不简单。人们可能需要几年才能适应子模块。这是许多人不愿意使用它们并寻找替代品的主要原因。我会说，它们非常有用和强大，只是对不接触它们的人来说，一些指导和教育可能是必要的。

还有更多的子模块。稍后我会做一篇相同主题的后续文章。

*   [订阅](https://medium.com/subscribe/@aerabi)my Medium publishes，以便在新的 Git 周刊发布时获得通知。
*   关注 Twitter 上的[我，获取 git 上的每周文章和每日推文。](https://twitter.com/MohammadAliEN)