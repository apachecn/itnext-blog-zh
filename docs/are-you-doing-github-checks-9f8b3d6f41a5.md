# 你在做 GitHub 检查吗？

> 原文：<https://itnext.io/are-you-doing-github-checks-9f8b3d6f41a5?source=collection_archive---------6----------------------->

![](img/f35ae73558b9a19decf3b5c8d2d8f5be.png)

没有任何借口——在**的 15 分钟**里，我得到了对我们主要分支机构每个公关**的运行检查。而且是**免费**！**

GitHub check 是什么？这是一种自动检查源代码的方式。比如它**造**吗？它是否通过了**测试**？通过林挺的了吗？做所有电脑**擅长**和人**容易**错过**的繁琐工作。**

![](img/85e8e0509a4cf219fa69ce9df3106d15.png)

一个拉取请求，它触发了一个检查并返回 OK

我们使用**检查**的**项目**是一个有角度的单页应用程序。它使用 [Node.js](https://nodejs.org/en/) 和 [TypeScript](https://www.typescriptlang.org/) ， [Jest](https://jestjs.io/) 进行单元测试，使用 [TSlint](https://palantir.github.io/tslint/) 捕捉类似于`[fit](https://jasmine.github.io/2.1/focused_specs.html)`、`[fdescribe](https://jasmine.github.io/2.1/focused_specs.html)`、`console.log`之类的东西以及类似的错误检入的代码。

**检查**的**目标**是让构建、**单元测试**和**林挺**在之前运行****PR**(拉请求)已经被**合并**。所以它会在每个人的签出代码结束之前捕捉到一个坏的构建。或者是一个失败的测试。还是一辆`debugger`入住。
如果代码审查已经完成，那么**审查者**将能够专注于更**高层次的**东西，并确信计算机完成了令人厌烦的部分。**

酷，那我们怎么做？

这个项目是利用**詹金斯**的工作建立起来的，有几天，我努力让詹金斯在新的公关事件上做点什么。**做不到**做不到。我更习惯于 Azure DevOps(以前的 VSTS)，我决定尝试一下。去了[https://github.com/marketplace?category = continuous-integration&query = azure](https://github.com/marketplace?category=continuous-integration&query=azure)并搜索“azure”:

![](img/e4d7b1dca02c88155b6fdd090330ea9a.png)

点击了 Azure 管道。

[](https://github.com/marketplace/azure-pipelines) [## 这是 GitHub Marketplace 中 Azure 管道的链接

### 持续构建、测试和部署到任何平台和云

github.com](https://github.com/marketplace/azure-pipelines) 

注意:如果已经安装了 Azure Pipelines，你会看到文章末尾显示的屏幕。*

![](img/5dc8c4a64c32fc71c5c472a92e54d52e.png)

为所有项目安装或只选择一个项目。

![](img/5fa3a5535918d2b471483617e19ede62.png)

登录后看到这个。选择“创建新项目”。

![](img/207eb6f61f6f26fbfce7565f67a5e132.png)

我将在这里选择我的回购— devops-demo。

![](img/e51f0079c5d2fa5e14fa7c6a8f40da7d.png)

下一步是选择一个模板。在我的例子中， **Angular** 是显而易见的选择。下一步，我们可以**定制**模板

![](img/9cc85da5fc99da0dea6f094d2a5e833b.png)

是它最初的样子。

给定这个脚本—对 master 的任何更改都将触发构建。这可能是我们在某些情况下需要的——例如**持续集成**。在我们的情况下，我们需要别的东西。稍后会有更多的介绍。首先—点击右侧的“**保存并运行**”按钮，这将让您选择是提交给*主*还是用`azure-pipelines.yml`文件开始一个拉取请求。**或者**起作用，你最终将管道**配置为代码**检入**沿着**以**代码**构建！干净利落。这是我的第一个版本。

接下来，让我们将脚本`azure-pipelines.yml`更改为仅在 pull 请求时触发:

我留下了一些详细描述步骤的评论。

请注意，我们在签入和合并时显式地停止了构建。只在 PR-s 上触发它。这将**只**做**检查**。在工作中，我们在 AWS 机器上进行由 Jenkins 编排的实际持续集成构建。

## 开始公关

启动 PR 后，我们可以看到，几秒钟后检查就开始了:

![](img/188c3f6f7f5e7ac70a32a34e1d94c0ab.png)

如果一切顺利，我们可以看到舒缓的绿色滴答声:)

![](img/85e8e0509a4cf219fa69ce9df3106d15.png)

[这是实际公关检查的链接](https://github.com/gparlakov/devops-demo/pull/2/checks)

## 断裂的棉绒

当我们打破一个测试，或者构建，或者我们的一个林挺规则，我们会看到一些即时的反馈。例如，我忘记了一个[公关#3](https://github.com/gparlakov/devops-demo/pull/3/checks?check_run_id=128845520) 中的`fit`电话:

![](img/379e4ff07bb7255f224c239c2e3962ff.png)

在支票中，在到达主分支之前，我们看到:

![](img/20e745e45c27cf0bd67076cd07eda9c3.png)

[https://github.com/gparlakov/devops-demo/pull/3](https://github.com/gparlakov/devops-demo/pull/3)

![](img/57168abdebe397a98a566b0447779ac2.png)

[https://dev . azure . com/gparlakov/devo PS _ Project/_ build/results？buildId=10](https://dev.azure.com/gparlakov/DevOps_Project/_build/results?buildId=10)

我真的很喜欢这个反馈！

> …该检查仅在 **PR** 上**执行，并且**不会影响**您可能拥有的任何其他持续集成或部署**

总而言之，该检查仅在 **PR** 上**执行，并且**不会影响**您可能拥有的任何其他持续集成或部署。它旨在提供快速反馈。**

这是 Azure 项目[链接](https://dev.azure.com/gparlakov/DevOps_Project/_build?definitionId=3)。这是公开的——去看看吧。https://github.com/gparlakov/devops-demo
和 GitHub 回购。

关于文章开头的免费部分。我知道这很难相信，但事实是这样的:

*   对于公共回购-像[这一个](https://github.com/gparlakov/devops-demo)-无限分钟 10 个并行作业
*   对于私人回购—每月 1 份工作和 1800 分钟。喜欢这个项目，我一开始就描述过。我们是一个由 5 人组成的团队，积极参与，每月做大约 1000 分钟。

*如果您的回购上预装了 pipeline 应用程序，您将会看到:

![](img/da56fe106a1d5ea79f7e2c923be00d77.png)![](img/008c0c6a600c62f425e897d9b7e527a9.png)

并继续到一个类似于上面的“安装 Azure Pipelines”的屏幕，在那里你选择应用程序将被安装到哪个仓库。

或者，如果开始一个全新的项目:

*   看一看[哟团队](http://donovanbrown.com/post/yo-Team)
*   如果只是创建 GitHub repo，并且已经安装了 Azure Pipelines GitHub 应用程序，您可以在 repo 创建时立即添加它:

![](img/f998f436068b4c400406790e4fe173b6.png)

通过选中底部附近的复选框，在创建 repo 时添加 Azure 管道。