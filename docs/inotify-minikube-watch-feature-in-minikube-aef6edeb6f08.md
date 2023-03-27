# inotify mini kube(mini kube 中的跑步观察功能)

> 原文：<https://itnext.io/inotify-minikube-watch-feature-in-minikube-aef6edeb6f08?source=collection_archive---------8----------------------->

如果你正在为你的开发环境运行 Minikube，并且发现你因为你的自动重载构建系统(Webpack [watch](https://webpack.js.org/configuration/watch/) ， [watchdog](https://pypi.org/project/watchdog/) )不工作而感到烦恼，这篇文章是为你准备的。

[Minikube](https://github.com/kubernetes/minikube) 允许你在本地运行 Kubernetes。我没有说服您使用它，但是如果您在您的生产环境中使用 Kubernetes 集群，那么使用它是有意义的，因为它为您带来了许多优势。

> 如果您在生产环境中使用 Kubernetes 集群，我推荐使用 Minikube。

![](img/fee585ecf42bab501770351ecadd7dc6.png)

照片由[莫里茨·金德勒](https://unsplash.com/@moritz_photography?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/mini-shipping-container?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

回到自动重载构建系统的问题，这是因为共享文件夹之间不支持 [inotify](http://man7.org/linux/man-pages/man7/inotify.7.html) 事件，因为虚拟机具有不同的文件系统(这是 VirtualBox 中的一个已知问题，在这里它甚至被标记为 wont fix)。

像`CHOKIDAR_USEPOLLING=true`这样的另一种解决方案不是一个选项，因为根据我们的经验，随着时间的推移，它会消耗大量的 RAM 和 CPU，导致我们不得不重新启动我们的虚拟机，从而影响生产力。

我们相信这些手表功能和 Minikube 应该让我们更有效率，但它们并没有无缝地工作。但是我们有解决办法！

**Notify-forwarder** 是一个可以将文件系统事件转发到另一台有共享文件夹的机器上的工具。

[](https://github.com/mhallin/notify-forwarder) [## mhall in/通知-转发器

### 一个简单的工具，用于将文件系统通知从一个主机转发到另一个主机，可能会在…

github.com](https://github.com/mhallin/notify-forwarder) 

它有两个运行选项:*观看*和*接收。*

```
# Run in host
$ notify-forwarder watch -c <remote-ip> <host-path> <to-path># Run in VM
$ notify-forwarder receive
```

*监视*选项监视<主机路径>中的文件系统事件，并通过端口 29324 中的 UDP 协议将它们发送到<远程 ip >。< to-path >是虚拟机中的远程路径或共享路径。

*接收*选项然后在运行自动重载构建系统的 VM 中重放事件。

## 在 Minikube 中运行通知-转发器

想法是在 Minikube 内部运行一个容器，运行**通知-转发器接收**。其他容器应该神奇地获得这些文件更改，因为它们现在实际上是从 VM 中的同一个文件系统挂载的。

要运行 **notify-forwarder receive** ，请应用此清单文件

但首先，这是我的设置:

*   马科斯·莫哈维
*   minikube 1 . 5 . 2 版
*   VirtualBox v5.2.32(旧的我知道:P)
*   项目目录路径:/Users/esh/workspace

> 注意:如果你已经使用 Minikube 和 VirtualBox 有一段时间了，你会知道它会把你的整个主目录挂载到 VM 中的同一个路径，所以如果你使用的是 MacOS，你会把整个/Users 挂载到里面。更少的路径需要担心。

notify-forwarder.yaml

运行**ku bectl apply-f notify-forwarder . YAML**

这将创建一个 Kubernetes 服务和一个部署资源，后者创建一个监听端口 29324 的通知-转发器应用程序。然后，该服务会将其公开给主机。另一个重要的方面是容器中的挂载目录，所以要确保它设置正确。

要检查它是否成功运行，

```
$ kubectl get pod | grep notify-forwarder
notify-forwarder-fd66445c8-2gkmf    1/1     Running   1          1m$ kubectl get svc | grep notify-forwarder
notify-forwarder   LoadBalancer   10.107.35.88     10.107.35.88   29324:30732/UDP                1m
```

记下 minikube ip 和服务外部端口

```
$ export NOTIFY_LISTENER=$(minikube ip):$(kubectl get svc notify-forwarder --no-headers | awk '{print $5}' | cut -d / -f 1 | cut -d : -f 2)echo $NOTIFY_LISTENER
192.168.99.100:30732
```

现在从主机运行**手表**选项。

下载适用于您的操作系统的预构建二进制版本

*   Mac OS X 10.10 或更新版本，64 位:[通知-转发器 _osx_x64](https://github.com/mhallin/notify-forwarder/releases/download/release%2Fv0.1.0/notify-forwarder_osx_x64)
*   Linux 64 位:[通知-转发器 _linux_x64](https://github.com/mhallin/notify-forwarder/releases/download/release%2Fv0.1.0/notify-forwarder_linux_x64)
*   FreeBSD 64 位:[通知-转发器 _freebsd_x64](https://github.com/mhallin/notify-forwarder/releases/download/release%2Fv0.1.0/notify-forwarder_freebsd_x64)

```
$ ./notify-forwarder_osx_x64 watch -c $NOTIFY_LISTENER /Users/esh/workspace /srv
```

这样，我们告诉 notify-forwarder 从`Host:/Users/esh/workspace`到`$NOTIFY_LISTENER:/srv`发送文件系统更改事件。

现在为什么是`/srv`？因为那是容器的挂载路径，在那里**接收**选项正在监听。运行构建系统的其他容器现在也应该能够接收文件更改了。

通过这种方法，可以使用 Ansible 或 Chef =)轻松实现自动化

这是**通知-运输**的文档，供您参考

```
FROM bitnami/minideb:jessie as buildRUN apt update && apt install -y curl
RUN curl -OL [https://github.com/mhallin/notify-forwarder/releases/download/release%2Fv0.1.0/notify-forwarder_linux_x64](https://github.com/mhallin/notify-forwarder/releases/download/release%2Fv0.1.0/notify-forwarder_linux_x64)
RUN mv ./notify-forwarder_linux_x64 ./notify-forwarder
RUN chmod u+x ./notify-forwarderFROM bitnami/minideb:jessie as final
COPY --from=build /notify-forwarder /usr/bin/notify-forwarder
ENTRYPOINT ["notify-forwarder"]
```

感谢阅读！我希望这有所帮助，如果你对这个问题有不同的看法，请告诉我。

一如既往，我欢迎反馈和建议=)

参考资料:

[](https://github.com/mhallin/notify-forwarder) [## mhall in/通知-转发器

### 一个简单的工具，用于将文件系统通知从一个主机转发到另一个主机，可能会在…

github.com](https://github.com/mhallin/notify-forwarder) [](https://www.datadoghq.com/container-report/) [## 关于真实世界容器使用的 8 个事实

### 编排正在成为使用容器的组织的标准实践。这一点在……最为明显

www.datadoghq.com](https://www.datadoghq.com/container-report/) [](https://minikube.sigs.k8s.io/) [## 迷你库贝

### 只需一个命令，就能轻松再现您的生产环境。始终支持…

minikube.sigs.k8s.io](https://minikube.sigs.k8s.io/) [](https://www.virtualbox.org/ticket/10660) [## #10660(共享文件夹中的文件更新不会触发 Ubuntu 上的 inotify)-Oracle VM VirtualBox

### 大家好！我使用的是 virtualbox 4.1.16 和最新的来宾操作系统。我的主机是 Windows 7，客户是 Ubuntu 11.10，我有…

www.virtualbox.org](https://www.virtualbox.org/ticket/10660)