# git——记录您的 CI 流程

> 原文：<https://itnext.io/git-notes-your-ci-process-46b2fd5ac52?source=collection_archive---------1----------------------->

![](img/27bc516279a99a0cac648a5007f8bc3b.png)

图片由来自 [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1080592) 的 [Michal Jarmoluk](https://pixabay.com/users/jarmoluk-143740/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=1080592) 拍摄

很长一段时间以来，我们一直使用 Jenkins 作为我们的 CI 系统。从构建过程的角度来看，这很好，但是它也有其[问题](/jenkins-is-getting-old-2c98b3422f79)。我们遇到的最恼人的问题之一是丢失构建历史。我们通常告诉 Jenkins 保留大约 10 个以前的版本。但是一旦他们离开，这些信息就永远消失了。现在，我们可以增加或完全取消限制，但是，硬盘空间不是无限的，随着更多的历史被创建，Jenkins 需要越来越长的时间来呈现构建页面。此外，您保留的历史越多，Jenkins 需要的内存(RAM/JVM 堆)就越多。理想情况下，我们总是试图将詹金斯视为一种短暂的资源。

我们如何继续存储我们的构建历史，而不依赖于 Jenkins 或其他数据库？

Git 有一个很少被利用的特性，叫做 [Git Notes](https://git-scm.com/docs/git-notes) ，它允许你在不改变原始提交散列的情况下将元数据附加到任何提交上。这对我们来说是完美的，因为我们试图在每次提交时开始构建，并且希望知道那些构建与原始提交一致的结果。现在，git-notes 并不是没有批评者，但是只要做一点工作，我们就可以利用这个特性在 git 存储库中存储所有构建的构建信息。

## CI 设置

首先，我们需要设置 CI 流程，以便在构建结束时发布构建信息。鉴于我们使用 Jenkins，本指南将重点关注这一点，但大多数构建系统都应该能够实现这一点。

Jenkinsfile:

*post-always* 将确保 git-notes 进程始终执行。我们需要一个节点，并会在一个超时执行以防万一。最后，我们需要签出代码，因为我们将与 git 存储库进行交互。最后，我们调用我们的 *addNotes* 方法。

*addNotes* 方法捕获构建元数据，并使用 Jenkins 管道方法 [writeYaml](https://www.jenkins.io/doc/pipeline/steps/pipeline-utility-steps/#writeyaml-write-a-yaml-from-an-object-or-objects) 将其存储到 yaml 文件中。最后，在一个 credentials 块中，我们调用 git-notes shell 脚本来推送 notes。

gitNotes.sh:

有关详细信息，请参见脚本中的注释，但是基本的*要点*(双关缩进)是 gitNotes.sh 脚本将:

1.  获取当前笔记
2.  将注释 yaml 合并到名称为 Jenkins 的注释 REF 中，我们使用名称为 *jenkins* 的 REF，但是您可以使用任何您希望的名称
3.  将重试循环中的结果回推到 git 存储库

重试循环非常重要，因为其他分支可能会同时推送到同一个回购，REF 必须始终在最前面才能成功。

为了阅读笔记，每个用户必须设置他们的本地 git 克隆来获取笔记。

## 用户设置

要获取笔记:

```
git fetch origin "refs/notes/*:refs/notes/*"
```

或者，您可以编辑您的 git 配置来默认获取注释:

```
[remote "origin"]
  fetch = +refs/heads/*:refs/remotes/origin/*
  fetch = +refs/notes/*:refs/notes/*
```

要查看提交日志中的注释，请执行以下操作:

```
git logs --notes=jenkins
```

样本输出:

```
commit ABCDEFG...
Author: Jenkins <[no-reply@github.com](mailto:no-reply@optum.com)>
Date:   Mon Jun 7 19:58:57 2021 +0000Commit Message, helloNotes (jenkins):
    build-job-env%2Fdev-93:
      duration: 49 sec
      full_display_name: 'github » build-job » env/dev #93'
      id: '93'
      number: 93
      branch: env/dev
      result: FAILURE
      time: '2021-06-07T14:59:07.720Z'
      causes:
      - _class: jenkins.branch.BranchEventCause
        shortDescription: Push event to branch env/dev
      url: [https://sample-url/job/github/job/github/job/env%252Fdev/93/](https://jenkins-datacore-jenkins.ocp-elr-core-nonprod.optum.com/job/datacore/job/datacore-ops/job/env%252Fdev/93/)
```

如果同一个提交发生了多个作业，那么将会显示两个注释，因为我们合并了注释。

## 摘要

通过这种设置，我们能够在 git 存储库中保存 Jenkins 构建的历史，这是我们最终的事实来源。我们可能会丢失实际的构建控制台日志和任何 Jenkins 构建工件，但是随着时间的推移，它们的价值会逐渐减少。

有人可能会说，这种设置在像 [Github Actions](https://github.com/features/actions) 或 [Gitlab CI](https://docs.gitlab.com/ee/ci/) 这样的构建系统中价值有限，因为它们已经在本地存储了构建信息。但是，Github 不再支持在 web 界面中查看 git-notes。在 Gitlab 中可以看到注释。

然而，我们发现这对我们非常有用，因为:

1.  我们利用内部(公司防火墙后)git 存储库
2.  提供可追溯性
3.  可以从命令行查看日志(即使脱机)
4.  与 git 日志很好地集成，提供搜索和解析
5.  确保我们不依赖任何供应商的 git 产品或 web 界面，只依赖原生 git