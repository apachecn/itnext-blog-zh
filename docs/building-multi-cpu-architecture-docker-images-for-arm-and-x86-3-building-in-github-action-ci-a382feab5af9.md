# 为 ARM 和 x86 构建多 CPU 架构 Docker 映像(3):在 GitHub Action CI 中构建

> 原文：<https://itnext.io/building-multi-cpu-architecture-docker-images-for-arm-and-x86-3-building-in-github-action-ci-a382feab5af9?source=collection_archive---------4----------------------->

![](img/d10d2f95e4fad64f967c0c365cee5cf3.png)

照片由[rich Great](https://unsplash.com/@richygreat?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

在“[为 ARM 和 x86 构建多 CPU 架构 Docker 映像(1):基础知识](https://medium.com/p/2fa97869a99b)”中，我们介绍了使用 buildx/buildkit 构建多拱 Docker 映像的一般工作流程。在本文中，我们将介绍如何让它在 GitHub Action CI 上运行。

# 跑步者环境

使用`ubuntu-latest` runner 环境将是让您的多拱管道在 Github action CI 上运行的最简单的选择。例如，您可以通过以下方式指定作业运行器环境:

```
jobs:  
  build-docker-images:    
    name: Build Docker Images
    runs-on: ubuntu-latest
```

# 建立 QEMU

[Buildkit](https://github.com/moby/buildkit) 在一台机器上为不同的目标平台构建 Docker 映像时，使用 QEMU 来模拟不同的目标平台。要设置/安装 QEMU，我们可以使用`[docker/setup-QEMU-action](https://github.com/docker/setup-qemu-action)`:

```
- name: Set up QEMU        
  uses: docker/setup-qemu-action@v1
```

# 安装 Buildx

要设置/安装 buildx，我们可以使用`[docker/setup-buildx-action](https://github.com/docker/setup-buildx-action)`:

```
- name: Set up Docker Buildx        
  uses: docker/setup-buildx-action@v1
```

# 完整的例子，建立多拱图像和推动到' GitHub 软件包'注册表

```
jobs:  
  build-docker-images:    
    name: Build Docker Images
    runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v2
    - name: Set up QEMU        
      uses: docker/setup-qemu-action@v1
    - name: Set up Docker Buildx        
      uses: docker/setup-buildx-action@v1
    - name: Login to GitHub Package Registry        
      run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
    - name: Build & Push Docker image
      run: docker buildx build -t ghcr.io/${{ github.repository_owner }}/myimage:${GITHUB_SHA} -f [path to Dockerfile] --push --platform=linux/arm64,linux/amd64 [path to build context]
```

# 将多拱图像重新标记和复制到 Docker Hub

在 CI 管道中，我们经常将 docker 映像推送到 CI 平台提供的注册表中进行测试和验证。但是，最终，我们会希望将 GitHub Packages Registry 中的多拱 Docker 映像重新标记和复制到生产注册表中。例如码头中心。

对于多拱图像，我们不能使用旧的工作流程来使用`docker tag` & `docker push`命令重新标记和推送 docker 图像到不同的注册表，因为它们只处理一个图像，而不是一个清单注册表条目后的多个图像。

要解决这个问题，我们可以使用`[regclient](https://github.com/regclient/regclient)`。它的命令语法非常简单。为了重新标记&复制我们在前面的例子中构建的&推送到 GitHub Packages 注册表的图像，并推送到 Docker hub，我们可以运行:

```
- name: Login to Docker Hub        
  env:          
    DH_TOKEN: ${{ secrets.DOCKER_HUB_PASSWORD }}               
  run: docker login -u my-docker-hub-username -p ${DH_TOKEN}- name: Re-tag & Push Docker Image to Docker Hub        
  run: |          
    # make config.json avaiable to regclient in docker container
    chmod +r $HOME/.docker/config.json
    # Run regclient in docker image
    docker container run --rm --net host \
      -v regctl-conf:/home/appuser/.regctl/ \            
      -v $HOME/.docker/config.json:/home/appuser/.docker/config.json \            
      regclient/regctl:v0.3.9 image copy ghcr.io/${{ github.repository_owner }}/myimage:${GITHUB_SHA} docker.io/xxxx/myimage:v1
```