# 要扩展的代码

> 原文：<https://itnext.io/code-to-extend-3b520d2d6745?source=collection_archive---------3----------------------->

![](img/15ad2d587e6f9e8b3bbdf998b50ceede.png)

软件和硬件相比，什么是伟大的？很容易改变、维护和理解。

> 软件是用来操作计算机和执行特定任务的一组指令、数据或程序。

“软件”一词包含软件，根据定义应该是

> 易于成型、切割、压缩或折叠；摸起来不硬或者不结实。

这都是真的，但是

> 任何傻瓜都能写出计算机能理解的代码。优秀的程序员编写人类能够理解的代码。—马丁·福勒

计算机不会给一只飞翔的火烈鸟你的代码是如何构造的，有没有经过评审，有没有包含无意义的注释。编译器将代码归结为指令，计算机执行，工作完成。每个人都很高兴，除了下一个必须做出改变的开发者。如果他的改变破坏了什么，要负全责。

女士们先生们，这是我们的故事，如何使你的代码可读，最重要的是可扩展。我邀请我的朋友参加播客，他的名字是[oļegs·甘辛斯](https://github.com/oganzins)，他有丰富的开发经验。我们讨论了代码中的 if 语句。

If 语句是一件独一无二的艺术品，例如，无法比较两个数字。但是当它们被误用或过度使用时，问题就出现了，例如“如果圣诞树”。

什么是如果圣诞树。当你把嵌套的 if 语句代码旋转 90 度，代码会突然变成树的形状。

第一个问题，上下文切换。当然，在提供的例子中，遵循代码很容易，上下文的切换也不会造成伤害。但在现实生活中，在下一条语句生效之前，已经写了一大堆逻辑。当下一个 if 语句命中时，大脑内部的上下文必须从“转向”切换到“周围”。

第二个问题，代码扩展。让我们增加新的功能，让船慢慢掉头。再来一个 if，然后把参数直接传到兔子洞里。

简单的扩展已经变得如此糟糕。head 里各种红旗弹出，这个怎么测试，每个 if 分支要跳多少个单元测试。当“t”等于“超级慢”时会发生什么。

第三个问题，责任的层次。方法“turn”不仅在设计上很复杂，而且有不止一个职责。

方法知道怎么转快，怎么转慢。这表明当改变一件事时，另一件事会受到意外的影响。这些错误通常被称为特性。

如何使代码可扩展，并记住它应该做一件事。深入研究[坚实的](https://en.wikipedia.org/wiki/SOLID)原则，通过探索每一条原则，如何创建整洁结构的模式和方法就会出现在日常生活中。

我们把注意力集中在 S 上，代表“单一责任原则”做好一件事，O 代表“开放-封闭原则”,开放是为了扩展，封闭是为了修改。

在播客的最后，我给了 Oleg 一个挑战。用 IF 语句编写代码。然后在没有它们的情况下做同样的事情。用例是一个基于网关处理支付的支付系统。其中网关可以是谷歌，脸书等。

让我们看看纯粹的如果结果如何。 [Github](https://github.com/oganzins/alternative-to-ifs/blob/master/src/main/java/com/intelmodus/billing/service/BillingService.java) 。

我们眼前有一个常见的嫌疑犯。但是我不得不说，即使有了 IFs，代码看起来还是干净易读的。我在等更糟糕的东西。这样做的问题是，每当有新的网关时，计费服务就必须改变。这样我们就突破了固体中的 O。至于 SOLID 中的 S，类或者代码应该只有一个改变的理由。但是我们可以列举出计费服务可以改变的多种原因。引入新的支付网关，现有支付处理器的新实现。

让我们看看当他删除相同服务的 IF 语句时发生了什么。 [GitHub](https://github.com/oganzins/alternative-to-ifs/blob/ifs-removed/src/main/java/com/intelmodus/billing/service/BillingService.java) 。

他们走了，代码看起来更干净，在新的网关，这些代码仍然保持不变。因此对扩展开放，对修改关闭，只有一个原因要改变。

奥列格做了哪些改变？他让 BillingService 不再负责创建一个支付处理器，并了解有多少网关。

所有的处理器和网关都在[PaymentProcessorRepository](https://github.com/oganzins/alternative-to-ifs/blob/ifs-removed/src/main/java/com/intelmodus/billing/service/PaymentProcessorRepository.java)中定义。这仍然在某种程度上打破了 O，但至少我们只是扩展了知识库的知识，而不是改变它。

他更进一步，用 [Guice 注入](https://github.com/google/guice/wiki/GettingStarted)实现了相同的 if 墙。让我们看看相同的 BillingService 在 Guice 注入框架中是什么样子的。 [GitHub](https://github.com/oganzins/alternative-to-ifs/blob/ifs-removed-using-guice/src/main/java/com/intelmodus/billing/service/BillingService.java)

抽象程度简直暴涨上天:)。抽象胜于实现是确保解决方案经得起未来考验的一个非常好的方法。这不仅适用于软件世界，也适用于现实世界。比如一辆没有离合机构的车，换挡就有点不可能。

BillingService 的实现在哪里？在这里。 [GitHub](https://github.com/oganzins/alternative-to-ifs/blob/ifs-removed-using-guice/src/main/java/com/intelmodus/billing/service/DefaultBillingService.java) 。

我们仍然看到有一个 PaymentProcessorRepository，但是它有一个显著的变化。但是在我们开始之前，让我们看一下 Guice 模块 PaymentProcessorModule。这是焊接发生的地方，接缝牢固，但易于更换。基本上，在任何新的处理器上，我们都在模块中注册它。紧凑，简洁，有意义，因为这是作为配置的代码。请记住，类发现可以自动进行，但是这样您就失去了什么与什么以及如何绑定的高度概述。

PaymentProcessorRepository 的变化是惊人的。它完全不知道处理器是在哪里、如何创建的，也不知道支持多少个网关。它只是通过注入获得所有的处理器，并在需要时为它们服务。每当创建一个新的网关或交换处理器的实现时，存储库不需要任何改变。

从而实现固体中的 S 和 O。每段代码只做一件事。支付系统对变化是封闭的，但对扩展是开放的。

这种方法开放了很多功能，比如交换实现，甚至在运行时。单元测试变得简单多了，因为它实际上测试的是小单元。

让这种方法真正闪光的是，业务逻辑可以与引擎分开测试。“DefaultBillingService”和“PaymentProcessorRepository”是引擎的一部分，但支付处理器是业务逻辑。为什么这很重要。通常，这样的引擎被创建一次，它就工作了。商业一直在发展。主要目标是不要因为业务而更换引擎。因此，避免它不是一个错误，而是一个特性。

这是我们关于如何处理 IF 语句的故事。完全不用它们是不可能的，但不过度使用它们是可能的。

我们建议阅读坚实的原则，因为它将导致许多有趣的方法和模式。记住，强迫自己去寻找和实现模式需要时间和思考。这增加了您的解决方案的初始成本，但从长远来看，这是值得的。另一方面，最初做快速而蹩脚的代码是便宜的，但从长远来看，将永远得不到回报，并且添加新功能的成本会越来越高。

GitHub
[带 IF 语句](https://github.com/oganzins/alternative-to-ifs) [不带 IF 语句](https://github.com/oganzins/alternative-to-ifs/tree/ifs-removed)
[不带 IF 语句用 Guice](https://github.com/oganzins/alternative-to-ifs/tree/ifs-removed-using-guice) 实现。