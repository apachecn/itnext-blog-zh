# 用 lima 替换 Docker 桌面

> 原文：<https://itnext.io/replace-docker-desktop-with-lima-88ec6f9d6a19?source=collection_archive---------0----------------------->

![](img/826a86ba17900b5b6157e560ededcaee.png)

照片由[多梅尼科·洛亚](https://unsplash.com/@domenicoloia?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

终于到了我们不得不抛弃这个名为 Docker Desktop 的渴求更新的工具的那一天。

我最初的目标是从 Docker 桌面转移，但仍然兼容我每天使用的所有 Makefiles 和构建。

# 尝试波德曼

Podman 的文档看起来很有前途，它实现了所有的 docker 命令，并带有一个 Linux VM。

简单的 brew 安装和几个命令就足够了。第一天，我意识到一个令人悲伤的事实，这是我通常不会想到的。Docker desktop 会在 Linux 虚拟机上自动挂载一些路径，比如 Home 文件夹。
有一个问题在讨论如何在波德曼[https://github.com/containers/podman/issues/8016](https://github.com/containers/podman/issues/8016)上挂载卷，似乎维护者会在 4.0 版本上添加，但我今天需要一个 Docker 桌面替换。

# 利马来救援了

当我偶然发现这个项目时，我几乎要放弃购买 Docker 桌面的许可了。
[https://github.com/lima-vm/lima](https://github.com/lima-vm/lima)

它拥有一切，自动卷安装，端口转发，并支持英特尔和 M1 CPU。

我计划让 Lima 管理一个运行 Docker 服务器的 VM，并将套接字暴露给我的 Mac。

**安装 Lima**

`brew install lima docker docker-credential-helper`

**启动 Lima 的虚拟机**

项目存储库中有许多例子可以用来启动 Linux 虚拟机。我们将使用 Docker 一号。
https://github . com/Lima-VM/Lima/blob/master/examples/docker . YAML

与存储库上的目录相比，我所做的唯一更改是将主目录设置为可写的。

运行 Docker 的 Lima 虚拟机配置

你可以将这个片段复制到一个名为`docker.yaml`的文件中。
运行`limactl start docker.yaml`

最后，它会打印最后的设置步骤，然后您就可以享受半无障碍 docker 体验。

跑完最后几步，你就可以开始了。

现在我可以回去继续我的项目，而不用担心我的目录安装在哪里。

如果你对`docker login`有什么问题。确保在您的`~/.docker/config.json`上，您已经将凭证存储设置为使用 Mac OS 钥匙链`"credsStore": "osxkeychain"`