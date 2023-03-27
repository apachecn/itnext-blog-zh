# 容器是新类别

> 原文：<https://itnext.io/container-is-new-class-1c0eae6c78a6?source=collection_archive---------8----------------------->

![](img/1f3b71817d5ff8de8d071da0568476bd.png)

Guillaume Bolduc 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

在面向对象编程中，类是解决问题的基本单位。几十年来，设计、分配职责和创建类的层次结构来构建可维护和可扩展的应用程序的方式一直是软件开发人员面临的基本挑战。关于这个问题，有大量的资料和书籍。像 GoF 的[设计模式](https://goo.gl/LhtQmD)、Martin Fowler 的[PoEAA](https://goo.gl/cah5Df)这样的书对软件设计和架构有着深远的影响。

代理、适配器、中介、装饰等模式，以及面向方面编程等技术有助于扩展类的行为，并在不污染类的情况下管理配置、日志等横切关注点。

随着时间的推移，这个基本单元从类变成了库和框架，提供独立自主的可重用的东西来组成你的应用程序。

随着容器和基于云的环境的出现，容器正在成为思考、解决问题和组成面向云的分布式系统的基本单元。像 [sidecar](https://goo.gl/HXwMnj) 这样的模式成为扩展容器行为和处理横切关注点的一种方式。用容器托管的应用程序不需要依赖日志框架，作为 sidecar 附加的容器可以处理它。将这样的关注转移到容器中，使得独立于应用程序容器的开发、部署、配置和更新变得容易。Kubernetes 和像 Istio 这样的服务网格框架将 sidecar 带到了下一个层次，并提供了一种更好的方式来部署和管理应用程序。

> *“容器不仅是部署单位和包装单位；它也是重用的单位、缩放的单位和资源分配的单位。”约翰·阿伦德尔和贾斯汀·多明格斯*

有两本很棒的书(目前免费)可以进一步探讨这些话题。

1.  设计分布式系统——*作者布伦丹·伯恩斯。这本书就像容器的 GoF，查看作者的[这篇演讲](https://goo.gl/mQju7j)。*
2.  [Cloud Native devo PS with Kubernetes](https://goo.gl/z9dFrV)—*作者 John Arundel 和 Justin Domingus。*为开发人员提供管理容器、工具和实践的优秀实用建议

感谢您的阅读，请随意点赞、评论和分享。