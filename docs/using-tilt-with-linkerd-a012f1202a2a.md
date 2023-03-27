# 将倾斜与 Linkerd 一起使用

> 原文：<https://itnext.io/using-tilt-with-linkerd-a012f1202a2a?source=collection_archive---------8----------------------->

如果将应用程序部署到 Kubernetes 是您日常开发工作流程的一部分，您会发现 [Tilt](https://tilt.dev) 非常有用。Tilt 可以持续构建您的本地代码变更并将其部署到您的 Kubernetes 集群。

在这篇文章中，我将向您展示我如何使用 Tilt 在 Minikube 上自动连续构建和部署 Linkerd 控制平面组件。

# Linkerd 控制平面

[Linkerd 控制平面](https://linkerd.io/2/reference/architecture/)由几个部件组成；即控制器、身份、Grafana、Prometheus、代理注入器、服务配置文件验证器和 Web。这些组件中的每一个都至少有一个服务容器、一个代理 sidecar 容器和一个代理初始化 init 容器。

控制平面库可以在 [GitHub](https://github.com/linkerd/linkerd2) 上找到。这个存储库包含所有组件的源代码、CLI、“未注入的”舵图、protobuf 定义和集成测试。

# 它是如何工作的

在具有 2 个 CPU 和 5096 MB RAM 的 Minikube 实例上(使用 kvm2 驱动程序)，以下视频展示了 Tilt 如何:

1.  对变更的组件执行选择性的构建和部署
2.  用新仪表板实时更新 Grafana 容器

https://youtu.be/nqrR6Fbj2nU(1080 p)

# 倾斜文件

Tiltfile 可以在我的 [GitHub 库](https://github.com/ihcsim/linkerd2-tilt)中找到。一些亮点包括:

1.  `[k8s_yaml](https://docs.tilt.dev/api.html#api.k8s_yaml)`函数使用由`linkerd install`命令生成的 YAML 斑点来识别控制平面组件
2.  `[custom_build](https://docs.tilt.dev/api.html#api.custom_build)`函数用于根据发生变化的文件和文件夹执行选择性构建
3.  `custom_build`函数的`live_update`参数用于将我的本地`grafana/dashboards`文件夹与容器的`/var/lib/grafana/dashboards`文件夹同步。
4.  `[local](https://docs.tilt.dev/api.html#api.local)`函数用于调用 shell 脚本，该脚本生成控制平面所需的 mTLS 资产。自签名的 mTLS 资产被传递给 Linkerd2 `identity`组件，以确保在重新部署`identity`组件时不会被覆盖
5.  `[k8s_resource](https://docs.tilt.dev/api.html#api.k8s_resource)`功能用于配置端口转发
6.  对于 GKE 上的开发，`[default_registry](https://docs.tilt.dev/api.html#api.default_registry)`函数用于重写图像库。

# 结论

当在 Linkerd 控制平面上工作时，我需要一个工具来帮助加速我的开发反馈循环。从引入代码更改到看到它在我的 Kubernetes 集群上工作的时间必须相当短。该工具还必须提供一组灵活的 API 来处理现有的存储库布局。

Tilt 是一个很棒的工具，它帮助我简化了开发工作流程。我很高兴能与我的团队和 Linkerd 社区分享它。

我还想在我的倾斜文件中添加几个项目(其中一些目前可能还不支持):

1.  更新`custom_build`函数以使用 Linkerd2 [构建脚本](https://github.com/linkerd/linkerd2/tree/master/bin)来保持构建步骤与脚本同步
2.  在`web`组件上使用`live_update`
3.  将命令行参数传递给`init.sh`脚本中的`linkerd install`命令，以控制启动行为；例如，在 HA 模式下启动控制平面
4.  观察“未注入”的舵图和 protobuf 文件的变化
5.  忽略对`*_test.go`文件的更改

特别感谢[丹·本特利](https://medium.com/u/d3a2fbd2b0ab?source=post_page-----a012f1202a2a--------------------------------)审阅我的草稿和倾斜文件。