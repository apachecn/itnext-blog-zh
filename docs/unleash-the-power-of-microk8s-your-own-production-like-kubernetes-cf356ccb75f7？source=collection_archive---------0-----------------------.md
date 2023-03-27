# 释放 MicroK8s 的力量:您自己的生产式 Kubernetes

> 原文：<https://itnext.io/unleash-the-power-of-microk8s-your-own-production-like-kubernetes-cf356ccb75f7?source=collection_archive---------0----------------------->

Microk8s 是一个令人兴奋的强大解决方案，可以轻松地在本地运行一个配备齐全的 kubernetes 基础设施(T2、T3、T4、T5)。直接在本地集群上部署和测试，就像在典型的生产环境中一样，没有太多麻烦。您可以在一个命令中启动并运行 ***。随着一些[应用程序已经在野外](https://blog.ubuntu.com/2019/03/28/microk8s-in-the-wild)和更多可能的应用程序，它的受欢迎程度正在理所当然地增长。在这个简短的介绍中，我们将看看 MicroK8s 的一个最典型的用例。***

![](img/a1b258b5c92dd33361f0a36aa4b57df7.png)

让我们以一个 Docker(微)服务环境为例，生产工作负载运行在 Kubernetes 上。一个高效的开发团队(或单个开发人员)最想要的需求是一个一致的、可靠的、类似生产的本地环境，例如，在部署到一个公共的阶段环境之前，在这个环境中测试和集成他们的代码与一个平台的所有必要的组件，全部启动并运行。

实际上，对于日常的集成工作，每个团队成员都带来了他/她自己定制的本地设置:有些人更喜欢*普通 docker* 和容器的手动启动/停止，有些人喜欢 *docker-compose* ，其他人使用 *minikube、vagger、virtualbox* ，以上所有这些的组合或者另一个深奥的解决方案。有些人喜欢模仿某些服务，有些人则喜欢挑战在自己的笔记本电脑上运行整个堆栈。许多人无可挽回地花费数小时来设置和修复他们自己不断变化的本地设置。但是他们只需要测试他们编码到一(1)个服务中的新端点！我见过有人花了几天的时间去追踪一个 bug，结果发现这完全是因为他们的本地环境需要修复，这当然是一种浪费时间的愚蠢方式。

团队成员在本地环境中缺乏一致性将导致低估生产力的缺乏，稍后反映在交付的服务的最终质量中。相反，如果整个团队设法同意使用，比方说，一个公共的 *docker compose* 定义来产生一个本地测试环境，这可以保证一个流畅的体验和共享项目上更可预测的结果。关键是团队有能力找到并接受一个共同共享的解决方案，然后利用它。现在，特别是如果你的平台不是平凡的，并且使用一个真正的、互连的微服务网格(而不仅仅是通过队列通信的独立的 dockerized monolithic 服务)*对于本地环境，有一些比 docker compose 更好的解决方案。*

对于那些在 Linux 上工作的幸运的人或其他在 Mac 或 Windows 上能够负担得起庞大的 Linux 虚拟机的人来说，Canonical 在 2018 年 12 月发布的 **MicroK8s** snap 提供了很大的帮助。 [Snaps](https://snapcraft.io/) ，最初是 Ubuntu 的东西，现在可以在大量的 Linux 发行版中得到支持，MicroK8s 似乎是使用它们的一个很好的激励。这篇文章不会解释什么是快照或者如何在你的发行版中支持它们。

**MicroK8s** 可以非常容易地安装在你的 Linux 系统[上](https://tutorials.ubuntu.com/tutorial/install-a-local-kubernetes-with-microk8s#0)并且成为 snap 将与系统资源紧密集成，因此运行平稳快速，同时与其他应用程序保持分离。集成是这样的，您将看到在您的系统中与 MicroK8s 组件一起运行的 *systemd* 服务。同样令人惊讶的是它带来的*插件*的数量，以及它的普遍兼容性(它是经过*认证的 Kubernetes* )使它成为一个*完整的平台*。

[**Minikube**](https://kubernetes.io/docs/setup/minikube/) 是另一种在本地运行 K8S 的强大方法，可以相对容易地安装，并具有适用于所有平台的设置脚本，但它也运行在 Linux 上的大型虚拟化虚拟机中(需要安装虚拟机管理程序)，具有相对有限的插件数量，并且可能需要大量工作来适应高级需求或由多个团队成员一致部署。当然:每个人都有坚持自己喜好的自由。

那么，在一个典型的 Kubernetes 生产平台中，你会在 MicroK8s 中找到什么样的**或者作为插件*(更多信息[在这里](https://microk8s.io/docs/) ) *？**

*   *K8S 仪表板*
*   *Kube DNS*
*   **注册表**
*   *存储类添加到本地主机上的文件夹中*
*   *Ingress (MicroK8s 没有*负载均衡器*，但是必要时可以使用 Ingress 插件[在外部 IP 上发布)](https://kndrck.co/posts/microk8s_ingress_example/)*
*   *[*RBAC*](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)*
*   *[efk 堆栈](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/) ( *Elasticsearch，Fluentd ans Kibana* )用于汇总日志*

```
*$ microk8s.kubectl cluster-info
Kubernetes master is running at [https://127.0.0.1:16443](https://127.0.0.1:16443)
Elasticsearch is running at [https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy](https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/elasticsearch-logging/proxy)
Heapster is running at [https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/heapster/proxy](https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/heapster/proxy)
Kibana is running at [https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/kibana-logging/proxy](https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/kibana-logging/proxy)
KubeDNS is running at [https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy](https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy)
Grafana is running at [https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy](https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy)
InfluxDB is running at [https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/monitoring-influxdb:http/proxy](https://127.0.0.1:16443/api/v1/namespaces/kube-system/services/monitoring-influxdb:http/proxy)*
```

*以上大部分你也会发现作为 *Minikube* 的插件。但是在 MicroK8s 中，有更多面向微服务和更复杂的有状态设置的功能:*

*   *[指标-服务器](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/metrics-server.md)以及 [*普罗米修斯*](https://github.com/coreos/prometheus-operator) 在 *Grafana* 中可见的指标*
*   **[*istio*](https://istio.io)[服务网](https://glasnostic.com/blog/kubernetes-service-mesh-what-is-istio)(更多信息[此处](https://cloud.google.com/istio/#documentation))**
*   **[*linkerd*](https://linkerd.io/2/overview/) ，另一个服务网格**
*   **[*耶格*](https://www.jaegertracing.io/) [运算符](https://enterprisersproject.com/article/2019/2/kubernetes-operators-plain-english)**

**MicroK8s 在 snap ( `microk8s.kubectl` *)* 中有自己内置的 *kubectl* 工具，并且可以很容易地生成完整的 *kubeconfig* 配置文件，以便在本地使用，如果您已经安装了自己的*kube CTL*(`microk8s.config > kube/config`)。你也可以在 `/etc/hosts`中为最常用的网址添加别名，比如 Dashboard、Grafana 和 Kibana。**

**[*kubesless*](https://kubeless.io/)在 MicroK8s 上轻松运行，对于无服务器设置(谁需要 Lambda？);为什么不在它上面放一个 *Jenkins* (也许以不同于示例中的另一种方式)，如果您想知道的话，也可以轻松地支持*[*Helm*](https://helm.sh/)*[*Charts*](https://github.com/helm/charts/tree/master/stable)。****

**![](img/31011b312c92fa8b04995a0dc1bc84cd.png)**

**要安装*舵柄*作为功能性*舵柄*海图支撑(假设你的 *kubeconfig* 已经就位)，通常的`helm init`将完成这个任务，打开所有可用[海图](https://github.com/helm/charts)的世界。而这仅仅是开始！**

**MicroK8s 的定制或扩展可以从它的[源 repo、](https://github.com/ubuntu/microk8s)进行，这样你就可以定制设置并与你的团队共享，比如选择默认启用的插件，这样*就有了你的团队共享的本地平台。***

**最后回到老式的整体服务，作为一个实验，我花了大约*10 分钟就有了一个成熟的 K8S 集群，并且运行了 *Wordpress* 本地博客(`helm install stable/wordpress`)，就这样我安装了主题并写了文章。你能用 Minikube 做到吗(包括创建一个 VM 需要的时间)？否则，请询问您友好的开发人员，他们花了多长时间在一个普通集群中正确设置稳定的 dns、监控、日志记录和 RBAC..***