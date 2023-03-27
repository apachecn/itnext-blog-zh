# 基于 GitLab 的条件构建

> 原文：<https://itnext.io/conditional-build-gitlab-ee039b1d4fba?source=collection_archive---------4----------------------->

![](img/40364c77aa389a1933121cd080a6bea5.png)

这个博客的老读者知道我正在使用 [Jekyll](https://jekyllrb.com/) 来生成静态站点。我正在使用 GitLab:当我按下`master`分支时，它触发生成作业。

然而，Jekyll 是基于 Ruby 的，需要几个 Gem 依赖项。我还添加了一些插件。出于这个原因，我创建了一个 [Docker 映像，其中包含所有必需的依赖关系](https://blog.frankel.ch/musings-dockerfile-jekyll/)。我定期通过 Bundler 更新`Gemfile.lock`中的版本。之后，我需要重建 Docker 映像。

因此，两份工作是必要的:

1.  在包含影响 Docker 映像的更改的推送之后，构建它
2.  任何推送之后，生成站点。

首先，触发条件是以下文件中的任何一个是否被更改:`Gemfile.lock`、`Dockerfile`、`.gitlab-ci.yml`。

问题是如何在满足条件的情况下只运行构建**版本**，因为这是一个昂贵且耗时的操作。GitLab 的构建文件配置为此提供了一个解决方案。在作业中，您可以配置一个`only`子句，使其仅在满足条件时运行。条件可以是:

*   一个引用，*，例如*，一个分支，或者一个标签
*   触发*，例如*，推送，web UI 或 API 调用
*   变量的值
*   对特定文件的更改
*   其他几个

倒数第二个选项是我们问题的答案。我们可以配置一组文件，如果其中任何一个文件被更改，构建应该会运行。否则，什么都不做。

它转化为以下结构:

```
stages:
  - image                                        # 1
  - deploy                                       # 1build:                                           # 2
  stage: image                                   # 2
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script: # Build the Docker image
  only:
    refs:
      - master                                   # 4
    changes:
      - Gemfile.lock                              # 5
      - Dockerfile                                # 5
      - .gitlab-ci.yml                           # 5pages:                                           # 3
  stage: deploy                                  # 3
  image:
    name: registry.gitlab.com/nfrankel/nfrankel.gitlab.io:latest
  script: # Generate the site
  only:
    refs:
      - master
```

1.  定义两个阶段
2.  在`image`阶段定义`build`工作。该作业创建 Docker 图像(通过 Kaniko)
3.  在`deploy`阶段定义`pages`工作。该作业通过 Jekyll 生成网站
4.  仅构建`master`分支
5.  仅当这些文件中的任何一个发生更改时才构建映像

此时，每次更改都会触发构建:映像构建作业在站点生成之前运行，但是如果映像没有更改，GitLab 会跳过前者。

**更进一步:**

*   [仅:变更/除外:变更](https://docs.gitlab.com/ee/ci/yaml/index.html#onlychanges--exceptchanges)
*   [仅:变更/除外:变更示例](https://docs.gitlab.com/ee/ci/jobs/job_control.html#onlychanges--exceptchanges-examples)
*   [GitHub:仅在推送影响特定文件时运行您的工作流](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#running-your-workflow-only-when-a-push-affects-specific-files)

*原载于* [*一个 Java 怪胎*](https://blog.frankel.ch/conditional-build-gitlab/)*2022 年 5 月 8 日*