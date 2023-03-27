# 如何使用模拟在 Go 中编写更好的单元测试

> 原文：<https://itnext.io/how-to-write-better-unit-tests-in-go-using-mocks-4dd05e867b17?source=collection_archive---------1----------------------->

最近，作为我对[Cloud Foundry](https://www.cloudfoundry.org/)[Bits-Service](https://github.com/petergtz/bitsgo)的 Go 重新实现工作的一部分，我实现了一个特性，它要求一组动作以正确的顺序执行。这样的一组动作也被称为*协议*。

在我的案例中，协议是这样的:

1.  针对`Bits-Service`发出文件上传请求
2.  `Bits-Service`告诉`Cloud Controller`现在是`PROCESSING_UPLOAD`文件时间。
3.  `Bits-Service`上传文件给一个`Blobstore`(比如 S3)。
4.  `Bits-Service`告知`Cloud Controller`(a)`FAILED`或(b)文件上传成功，文件`READY`可以使用。

由于这些调用中的任何一个都可能返回一个错误，在这种情况下，操作集会发生变化，所以我想对 Bits 服务与其两个合作者的交互进行单元测试，以确保这一点完全正确。两个协作者`Cloud Controller`和`Blobstore`已经分别被定义为接口`Updater`和`Blobstore`。尽管如此，为协议创建模拟并不简单，因为模拟的调用顺序很重要。

使用特别的、手工编写的模拟是可能的；他们可以共用一个计数器来跟踪呼叫顺序等。但是让他们的内部行为正确并不简单，并且需要它自己的单元测试集。仅仅编写一个简单的单元测试就要做很多额外的工作。

幸运的是，有 [**Pegomock**](https://github.com/petergtz/pegomock) ，一个极好的模仿围棋的框架。我要做的就是:

```
$ pegomock generate Blobstore 
$ pegomock generate Updater
```

然后单元测试代码看起来微不足道:

真的就这么简单。

当我们在做这件事的时候，当然 Pegomock 不仅仅适用于这个特定的用例。事实上，它对于*所有*种嘲讽用例都是极好的。想存根一些电话簿一样的界面？方法如下:

您可以使用匹配器始终返回相同的值:

```
**When**(phoneBook.**GetPhoneNumber**(**AnyString**())).**ThenReturn**("123-456")
```

或者，如果您需要更复杂的东西，您甚至可以实现自己的存根函数:

需要恐慌？这是:

```
**When**(phoneBook.**GetPhoneNumber**("Unknown")).**ThenPanic**("Invalid name")
```

然而，当您需要将存根和验证结合起来时，Pegomock 的真正魅力就显现出来了，因为您为两者使用了相同类型的匹配器，所以感觉非常自然。当您在它们之间切换时，您甚至不会注意到，除了您的测试看起来非常干净:

下次你需要一个模仿围棋的框架时，为什么不试试 Pegomock 呢？[只是](https://github.com/petergtz/pegomock#getting-pegomock) `[go get](https://github.com/petergtz/pegomock#getting-pegomock)` [它](https://github.com/petergtz/pegomock#getting-pegomock)。

*顺便说一下，Cloud Foundry Bits-Service 的 Go 重新实现正在德国的 IBM-Lab Boeblingen 开发，我们通常在#bits-service* *中的*[*cloudfoundry.slack.com 上闲逛。来和我们聊聊吧！*](https://cloudfoundry.slack.com/messages/C0BNGJY0G)