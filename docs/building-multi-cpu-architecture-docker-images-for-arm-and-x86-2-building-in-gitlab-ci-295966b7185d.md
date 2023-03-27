# 为 ARM 和 x86 构建多 CPU 架构 Docker 映像(2):在 GitLab CI 中构建

> 原文：<https://itnext.io/building-multi-cpu-architecture-docker-images-for-arm-and-x86-2-building-in-gitlab-ci-295966b7185d?source=collection_archive---------2----------------------->

![](img/4697cad62920dc61a04ac7f02a058a4d.png)

照片由 [Pankaj Patel](https://unsplash.com/@pankajpatel?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

在“[为 ARM 和 x86 构建多 CPU 架构 Docker 映像(1):基础知识](https://medium.com/p/2fa97869a99b)”中，我们介绍了使用 buildx/buildkit 构建多拱 Docker 映像的一般工作流程。然而，要让它在 Gitlab CI 上运行，您将需要一些额外的设置。

# 在 GitLab CI/CD 作业中启用 Docker 命令

在 GitLab CI/CD 作业中启用 Docker 命令有三个选项:

*   [外壳执行程序](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-the-shell-executor)
*   [Docker 插座装订](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-socket-binding)
*   [具有 Docker 图像的 Docker 执行器(Docker-in-Docker)](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-the-kubernetes-executor-with-docker-in-docker) :需要 [Docker 执行器](https://docs.gitlab.com/runner/executors/docker.html)或 [Kubernetes 执行器](https://docs.gitlab.com/runner/executors/kubernetes.html)

在本文中，我们将只讨论 k8s (Kubernetes) executor 的`dind` (Docker-in-Docker)选项，用于设置多拱建筑管道。

# 部署 Kubernetes Executor Gitlab Runner

要部署 Kubernetes executor Gitlab Runner，您可以使用 [Helm](https://helm.sh/) 将 Helm Chart 部署到您的 k8s 集群中。

首先，我们可以为我们的部署创建配置值文件(` config.yaml `):

```
gitlabUrl: "https://gitlab.com/"
runnerRegistrationToken: "xxxxxx"
concurrent: 1
runners:
  tags: "k8s-runners"
  privileged: true
  config: |
    [[runners]]
      [runners.kubernetes]
        image = "ubuntu:20.04"
        privileged = true
      [[runners.kubernetes.volumes.empty_dir]]
        name = "docker-certs"
        mount_path = "/certs/client"
        medium = "Memory"
      [runners.cache]
        Type = "gcs"
        Path = "gitlab_runner"
        Shared = false
        [runners.cache.gcs]
          AccessID = "xxx@xxxx.com"
          PrivateKey = "xxxxxxx"
          BucketName = "xxxxxx"
```

这里，您需要用自己的注册令牌替换“runnerRegistrationToken”。该设置假设使用 GCS (google 云存储)作为缓存存储。如果您使用 AWS s3，您可以将`[runners.cache]`部分替换为:

```
 [runners.cache]
        Type = "s3"
        Path = "runner"
        Shared = false
        [runners.cache.s3]
          ServerAddress = "s3.amazonaws.com"
          BucketName = "xxxxxx"
          BucketLocation = "eu-west-1"
          Insecure = false
          AuthenticationType = "access-key"
          AccessKey = "xxxx"
          SecretKey = "xxxxx"
```

其次，添加 Gitlab Runner Helm Chart repo(如果你还没有):

```
helm repo add gitlab https://charts.gitlab.io
```

使用我们创建的`config.yaml`配置文件部署 Gitlab Runner:

```
helm install --namespace <NAMESPACE> gitlab-runner -f config.yaml gitlab/gitlab-runner
```

这里，`<NAMESPACE>`是要安装 GitLab Runner 的 Kubernetes 名称空间。

# 将环境变量添加到`.gitlab-ci.yml`

部署 Kubernetes executor Gitlab Runner 后，我们还需要将以下 ENV 变量添加到 Gitlab CI/CD 配置文件`.gitlab-ci.yml`:

```
variables:
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_TLS_VERIFY: 1
  DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"
```

# 带有' buildx` Pluigin 的 Docker 图像

要在 CI 中运行`docker buildx build`命令，您需要一个包含 docker 命令& `buildx `插件的 docker 映像。我们已经在“[为 ARM 和 x86 构建多 CPU 架构 docker 映像(1):基础知识](https://medium.com/p/2fa97869a99b)”中介绍了如何创建这样的 Docker 映像。

您也可以使用我发布的预建 docker 图像:`[data61/magda-builder-docker:latest](https://hub.docker.com/r/data61/magda-builder-docker/tags)`。

# 构建多拱 Docker 图像和推送至 Gitlab Docker 注册表

在运行`docker buildx build`之前，我们需要用以下命令创建一个“构建器实例”:

```
# Create a new docker context for builder instance to use
docker context create builder-context# Create a builder instance named "builderx"
docker buildx create --name builderx --driver docker-container --use builder-context
```

以下是构建多拱 Docker 图像并推送到 Gitlab Docker 注册表的完整示例:

```
# All env vars are auto-provided by GitLab CI at runtime
docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY# Create a new docker context for builder instance to use
docker context create builder-context# Create a builder instance named "builderx"
docker buildx create --name builderx --driver docker-container --use builder-contextdocker buildx build -t $CI_REGISTRY/xx/myimage:$CI_COMMIT_REF_SLUG -f [path to Dockerfile] --push --platform=linux/arm64,linux/amd64 [path to build context]
```

# 将多拱图像重新标记和复制到 Docker Hub

在 CI 管道中，我们经常将 docker 映像推送到 CI 平台提供的注册表中进行测试和验证。但是，最终，我们会希望将 Gitlab 注册表中的多拱 Docker 图像重新标记和复制到生产注册表中。例如码头中心。

对于多拱图像，我们不能使用旧的工作流程来使用`docker tag` & `docker push`命令重新标记和推送 docker 图像到不同的注册表，因为它们只处理一个图像，而不是一个清单注册表条目后的多个图像。

要解决这个问题，我们可以使用`[regclient](https://github.com/regclient/regclient)`。它的命令语法非常简单。为了重新标记&复制我们在前面的例子中构建的&推送到 Gitlab 注册表的图像，并推送到 Docker hub，我们可以运行:

```
# Login to gitlab registry
docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY# Login to docker hub
docker login -u my-dockerhub-username -p $DOCKER_HUB_PASSWORDregctl image copy $CI_REGISTRY/xx/myimage:$CI_COMMIT_REF_SLUG docker.io/xx/myimage:v1
```

请注意:我发布的预构建 docker 映像:`[data61/magda-builder-docker:latest](https://hub.docker.com/r/data61/magda-builder-docker/tags)`已经包含了`regclient`实用程序。如果您使用自己预先构建的 docker 映像，您将需要手动安装它。

# 然后

[为 ARM 和 x86 构建多 CPU 架构 Docker 映像(3):在 GitHub Action CI 中构建](https://medium.com/p/a382feab5af9)