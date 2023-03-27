# 容器:优雅地终止

> 原文：<https://itnext.io/containers-terminating-with-grace-d19e0ce34290?source=collection_archive---------4----------------------->

![](img/0f1bb5b170f72c0ac8fd06afe85d31be.png)

塞缪尔·泽勒在 [Unsplash](https://unsplash.com/search/photos/containers?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

因此，我了解到信号被用来从主机与您的进程进行通信。无论你想重新加载配置，做一些清理，还是**优雅地退出你的应用**。

让我们关注一下常用的信号，尤其是在执行滚动更新部署时。**信号术语**和**信号技能**。

在 Kubernetes 中，当 pod 启动终止程序时，它执行 ff:

1.  Pod 被认为是“死的”, API 服务器将它从服务端点列表和复制控制器中删除。
2.  执行**预停止**钩子。
3.  **SIGTERM** 信号被发送到 pod，然后等待默认的 30 秒宽限期。
4.  发出 **SIGKILL** 。

阅读[这篇](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods) Kubernetes 关于吊舱终结的概念，了解更多信息。

这看起来好像我们不需要做任何事情，因为我们假设我们的应用程序在容器中优雅地退出。因此，根本没有停机时间。我们错了。我错了，:D

注意:对于示例，我们将使用 **docker stop** 命令，因为它发送 **SIGTERM** 。所以当一个豆荚在 Kubernetes 被终结时会有类似的信号。

> 我们如何确保我们的应用程序接收到信号？

让我们从最简单的方法开始…

## **应用程序作为主进程(PID 1)**

当您的应用程序是主进程时，它可以直接接受信号。

这是一个简单的 python 应用程序，我无耻地从 Stack Overflow(这里是[链接](https://stackoverflow.com/a/31464349/2591014) btw)复制了它，它实现了 SIGINT 和 SIGTERM 的信号处理程序。

在我们的文档中，

```
FROM python:3.7.3-alpine3.9
COPY ./signals.py ./signals.py
ENTRYPOINT ["python", "signals.py"]
```

现在构建图像

```
$ docker build . -t python-signals
```

然后跑，

```
$ docker run -it --rm --name="python-signals" python-signals
```

> 如果最后两步成功，您应该在 stdout 中看到“Running…”消息。

打开一个新的终端，检查我们在 Dockerfile 入口点运行的应用程序是否在 PID 1 中。

```
$ docker exec -it python-signals psPID   USER     TIME  COMMAND
    1 root      0:00 python signals.py
    6 root      0:00 ps
```

如果您运行一个 **docker stop** 命令，应用程序将接收到发出的 **SIGTERM** ，并能够根据您希望它执行的操作对其进行相应的处理。

```
$ docker stop python-signalsReceived SIGTERM signal
End of the program
```

太棒了！但是，如何使用 bash 脚本运行您的应用程序呢？

## 应用程序由 bash 脚本启动

通常情况下，脚本是从 bash 脚本启动的。

问题是，应用程序不会收到任何信号，也不会做出相应的反应。在这种情况下，当容器被杀死时，app 将被残忍地杀死:(

假设您有这样一个 run.sh 脚本:

```
#!/bin/sh

ENVIRONMENT="staging"
OTHERAPPVAR=true
python signals.py

## or for example run it using gunicorn
## with lots of different parameters# gunicorn [OPTION] [OPTION] ... app:app
```

在 Dockerfile 中，我们将用 run.sh 替换入口点。

```
FROM python:3.7.3-alpine3.9
COPY ./signals.py ./signals.py
COPY ./run.sh ./run.sh
RUN chmod +x ./run.sh
ENTRYPOINT ["./run.sh"]
```

构建然后运行容器…

打开一个新的终端并运行`ps`来检查 PID 1 中的进程:

```
$ docker exec -it python-signals ps
PID   USER     TIME  COMMAND
    1 root      0:00 {run.sh} /bin/sh ./run.sh
    6 root      0:00 python signals.py
    7 root      0:00 ps
```

我们的应用程序现在处于不同的进程 ID 中。因此，让我们看看通过运行 **docker stop，**我们的应用程序是否会接收到`SIGTERM`信号。

```
$ docker stop python-signals
...
```

这一次，没有来自应用程序的说再见的打印消息。它刚刚被杀死了。

现在，让我们回到运行脚本，添加 **exec**

```
#!/bin/sh

ENVIRONMENT="staging"
OTHERAPPVAR=true
**exec** python signals.py
```

`exec`命令允许我们执行一个完全取代当前过程的命令。

在我们的第一个 run.sh 版本中，当前进程是实际的 bash 脚本。因此，它是 PID 1。然后，它为 python 应用程序创建了一个子进程，在我们的示例中，它变成了 PID 6。

这个`exec`命令取代了当前进程，使应用程序成为 PID 1。

现在建立，运行，然后运行`ps`，

```
PID   USER     TIME  COMMAND
    1 root      0:00 python signals.py
   12 root      0:00 ps
```

瞧啊。应用程序现在将能够处理信号，并可以优雅地退出。

感谢您的阅读。:)

> 我乐于接受反馈和建议，这样我就可以改进。:)