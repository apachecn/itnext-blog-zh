# 使用 CodePipeline/CodeCommit 时如何访问 CodeBuild 中的 git 元数据

> 原文：<https://itnext.io/how-to-access-git-metadata-in-codebuild-when-using-codepipeline-codecommit-ceacf2c5c1dc?source=collection_archive---------0----------------------->

## 如果您希望您的 AWS 代码管道构建代理能够访问 git 历史，请阅读本文

![](img/e42662bd3061c446db5b0868839e7555.png)

希望你的管道不会消失在雾中(图片由在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄)

场景是这样的:被使用一个云服务来做所有事情的便利性所吸引，您已经在 AWS 中建立了一个构建管道。您对源代码使用 CodeCommit，对构建目标使用 CodeBuild 实例，并使用 CodePipeline 将它们连接在一起。一切看起来都很好——部署到 AWS 基础设施的其余部分很容易，从提交开始构建，一切都运行良好。

直到你决定使用 git 元数据，比如说使用`git describe`的输出来计算你正在构建的版本。当您尝试这样做时，您的构建会失败，并显示:

```
fatal: not a git repository (or any parent up to mount point /codebuild)
```

如果你从谷歌来到这里，你只是想要一个解决方案，请跳到文章的末尾寻找指导。如果你对血淋淋的细节感兴趣，请继续阅读。

# 为什么这个不行？

转到谷歌，你会发现[这个论坛帖子](https://forums.aws.amazon.com/thread.jspa?threadID=244197)来自 2016 年。链接在底部的是[另一个论坛帖子](https://forums.aws.amazon.com/thread.jspa?threadID=251732)，AWS 工程师在那里投稿:

> CodePipeline 从源代码提供者下载源代码作为 zip 文件，而不是进行 Git 克隆，这意味着。git 文件夹将不会被保留，像您正在运行的 git 命令将不会工作。
> 
> 你能分享更多关于你的用例的细节吗？看起来您的目标是为您的存储库生成一个 zip 存档，您也可以使用类似 zip 命令的东西来实现。
> 
> 如果您有保留。git 文件夹我有兴趣听到更多关于它们的信息，因为这是我们从几个客户那里听到的一个功能请求。

你没看错。CodePipeline 下载源代码，然后在将它交给 CodeBuild 之前，在没有`.git`目录的情况下重新打包它。我不知道它为什么会这样。

![](img/51b2e834a5fc1851591db7929ccf79c4.png)

可能是另一个 AWS 用户贴了这个标语(照片由[肯·特雷罗亚尔](https://unsplash.com/@kentreloar?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄)

# 我们可以使用环境变量吗？

假设我们想知道`git describe --tags`会打印出什么。CodeBuild [为我们提供了一些环境变量](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html)。也许我们可以使用其中的一个来查看我们想要的 git 元数据？

从文档来看，我们有`CODEBUILD_RESOLVED_SOURCE_VERSION`，它是“提交 ID”。在我的一个构建中，我得到:

```
CODEBUILD_RESOLVED_SOURCE_VERSION=80167249a7009aeff680573a9c40ab122176b361
```

这是正在构建的最近提交的 git SHA。潜在有用。

我们还有`CODEBUILD_SOURCE_VERSION`，它似乎是包含压缩源代码的人工制品的 S3 ARN。不是超级有用。

文档还描述了`CODEBUILD_SOURCE_REPO_URL`。也许我可以用它来克隆存储库——这可能有点麻烦，但是会有用的。这是它的文档(重点是添加的):

> 对于 CodeCommit 和 GitHub，这是存储库的克隆 URL。**如果构建源自 CodePipeline，那么它可能是空的。**

果然，在我的建筑里是空的:

```
CODEBUILD_SOURCE_REPO_URL=
```

据我所知，文档没有说明“可能”在这个上下文中是什么意思。一直都是空的吗？有时候？仅在某些配置中？我不知道。

![](img/3a6aa7b6df736e6395ba890eae0777b2.png)

不要忘记走出去，不要了解大自然中的事物

# 我们可以使用 AWS API 调用吗？

我最初希望我们可以给 [CodeCommit CLI](https://docs.aws.amazon.com/cli/latest/reference/codecommit/index.html) 当前的提交散列来确定标签，但是这相当有限。下面是文档中对 [get-commit](https://docs.aws.amazon.com/cli/latest/reference/codecommit/get-commit.html) 的示例响应:

```
{
  "commit": {
      "additionalData": "",
      "committer": {
          "date": "1484167798 -0800",
          "name": "Mary Major",
          "email": "mary_major@example.com"
      },
      "author": {
          "date": "1484167798 -0800",
          "name": "Mary Major",
          "email": "mary_major@example.com"
      },
      "treeId": "347a3408thisisanexampletreeidexample",
      "parents": [
          "7aa87a031thisisanexamplethisisanexample1"
      ],
      "message": "Fix incorrect variable name"
  }
}
```

我们得到了`treeId`，这是一个[内部 git 概念](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)，但是我们没有得到 Git 标记，即使请求的提交被直接标记。

在主[代码提交 cli 页面](https://docs.aws.amazon.com/cli/latest/reference/codecommit/index.html)上，我们可以看到有一些与标签相关的 API 调用，但它们是针对 AWS 标签，而不是 git 标签。

我想我们必须克隆存储库。

![](img/37df66f1726ef98502eb7b9cb1fe1121.png)

不是这种克隆(照片由[张家瑜](https://unsplash.com/@danielkcheung?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄)

# 克隆存储库

我不是第一个想这么做的人——本文开头链接的一些论坛抱怨通过克隆存储库绕过了这个限制。还有另一篇文章，其中 CodeCommit 存储库是在 CodeBuild [中克隆的，这里是](https://medium.com/maestral-solutions/create-git-tags-in-the-form-of-semantic-build-numbers-using-aws-codebuild-and-aws-codepipeline-e52210d719df)。然而，我认为我使用的方法更简单一些。

*   允许代码构建克隆您的存储库
*   给予 code build git 凭据帮助器

那么我们需要:

*   在右边的分支克隆存储库
*   将存储库返回到正在构建的提交(否则在提交和构建之间会有一个竞争条件，这会导致使用错误的 git 元数据)
*   将`.git`目录复制到源代码树中。

# 以下是步骤:

## 1)允许 CodeBuild 克隆您的库

为此，给你的代码构建执行角色权限`codecommit:GitPull`。在这篇文章的结尾有一个云计算模板，但是这里有一个政策声明:

```
{
  "Effect": "Allow",
  "Action": [
     "codecommit:GitPull"
  ],
  "Resource": [
    "arn:aws:codecommit:*:*:<YOUR_REPOSITORY_NAME>"
  ]
},
```

## 2)为代码构建提供 git 凭据帮助器

像这样启动 buildspec.yml:

```
version: 0.2env:
  git-credential-helper: yes
```

就是这样。现在 git 克隆将在您的代码构建代理中工作。

## 3)克隆存储库

您的 buildspec 中的一个简单的`git clone <REPO_URL>`将会工作，但是您可能不会获得您的构建正在使用的相同的源——所以我们需要检查正确的分支和正在构建的提交的`git reset`。

这足够复杂，可以在一个单独的脚本中处理这个问题，并执行一些[错误检查](https://medium.com/@timothyjones_88921/helpful-errors-in-bash-scripts-c1e3c2c50bf8)。因为我想在许多项目中这样做，所以我写了一个你(或我)可以在任何项目中使用的脚本。您可以在这里找到它[，或者使用以下命令下载它:](https://github.com/TimothyJones/codepipeline-git-metadata-example/blob/master/scripts/codebuild-git-wrapper.sh)

```
curl [https://raw.githubusercontent.com/TimothyJones/codepipeline-git-metadata-example/master/scripts/codebuild-git-wrapper.sh](https://raw.githubusercontent.com/TimothyJones/codepipeline-git-metadata-example/master/scripts/codebuild-git-wrapper.sh) -o codebuild-git-wrapper.sh
```

下面是一个 buildspec，它使用该脚本，然后打印 git 日志的顶部:

```
version: 0.2env:
  git-credential-helper: yesphases:
  build:
    commands:
      - scripts/codebuild-git-wrapper.sh <REPO_URL> <REPO_BRANCH>
      - git log | head -100
```

(注意，您需要为 REPO_URL 和 REPO_BRANCH 提供自己的值)。

![](img/a56fd8fa9f04ad8e601f30a9bbe9c8b8.png)

希望使用你的管道感觉是这样的(照片由 [Imani](https://unsplash.com/@spider_mani?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄)

# 看到它的实际应用

我已经[提供了一个 CloudFormation 模板](https://github.com/TimothyJones/codepipeline-git-metadata-example/blob/master/cfn-pipeline.yaml)，它包含了一个完整的演示原理的管道。如果您想看到整个管道的运行，您可以克隆[这个库](https://github.com/TimothyJones/codepipeline-git-metadata-example)并按照[这些指令](https://github.com/TimothyJones/codepipeline-git-metadata-example#full-instructions)让它启动并运行。

# 摘要

要将 git 元数据放入 CodeBuild/CodePipeline，请遵循以下三个步骤:

1.  在您的代码构建角色中允许`codecommit:GitPull`
2.  将`git-credential-helper: yes`放在 buildspec 文件的`env`部分
3.  克隆 repo，重置为与`CODEBUILD_RESOLVED_SOURCE_VERSION`匹配的提交，然后将`.git`目录复制到构建目录。

如果你不想写你自己的脚本来完成克隆/重置/复制步骤，你可以使用我为你写的这个。

如果你想看到一个完整的管道，我在这里发布了一个端到端的管道示例。