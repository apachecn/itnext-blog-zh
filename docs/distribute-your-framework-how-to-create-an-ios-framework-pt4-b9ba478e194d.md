# 分发您的框架——如何创建 iOS 框架 Pt4

> 原文：<https://itnext.io/distribute-your-framework-how-to-create-an-ios-framework-pt4-b9ba478e194d?source=collection_archive---------0----------------------->

创建一个好的框架并不一定保证它的成功，为了接触到大量的用户，你，框架作者需要像任何产品一样分发和销售它。为此，我们将从上一篇文章中的[开始，这一次我们将讨论远程打包和包注册。](https://medium.com/elmoezamira/f736f38c5bae)

![](img/f2e318663e51409462bf0efa96d00584.png)

由[伊斯梅尔·帕拉莫](https://unsplash.com/@ismaelparamo?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

## 远程打包:

我们将做必要的修改，使我们的本地包从最后一个帖子开始，一个远程包， **swift.package** 文件的结构不会改变太多，改变的部分将是 targets 数组中的二进制目标。

我们不提供本地 XCFramework 的路径，而是提供一个服务器的链接，我们将在那里托管我们的框架。

> 在远程存储库中包含二进制框架是一种不好的做法，因为它会使存储库变得巨大，为了解决这个问题，我们将在服务器中托管 XCFramework，在远程存储库中托管 Swift 包。

让我们一步一步来:

## 压缩框架:

我们将从创建一个包含 XCFramework 的 zip 文件开始，为此，执行这个终端命令，第一个参数是 zip 文件的名称，第二个参数是框架本身。

## 生成 zip 文件的校验和:

下一步是生成校验和，导航到您的包文件夹，然后键入以下命令:

> 得到的结果应该是这样的:0 a5b 236 f1 b 93 ebfa 0 cf 2242 ce 3 f 354035 a 8 c 03 c 925335 fdb 39 c 860 a 909084284 保存它，我们将在最后一步中用到它。

## 托管您的框架:

将存档的框架上传到个人服务器(或您公司的服务器)。或者，您也可以将它存储在本地服务器上，但仅用于测试目的。

> 如果你跳过了压缩步骤，你必须返回去做，你需要上传你的框架作为一个 zip 文件

## 最后必要的更改:

现在一切就绪，让我们回到清单文件(package.swift)并进行必要的更改，正如我前面所说的，我们将更新二进制目标。

这次我们将提供 3 个参数:

*   名称:您的目标的名称，不需要必要的更改
*   url:我们将使用指向您托管框架的位置的链接，而不是本地路径，确保您的 URL 以 zip 扩展名结尾
*   校验和:我们在第二步中生成的校验和

您的包可能包含多个二进制目标，就像我们在上一篇文章中所做的那样，所以对您拥有的每个二进制目标重复相同的过程。

## 将您的包推送到 git 存储库:

最后一步是将您的包推送到 git 存储库，这样就完成了，开发人员将能够通过在 SPM 中提供存储库的 URL 来使用您的框架。

## 包注册中心:

包注册中心是 Swift 包的集合，通过它们提供的工具，它们将帮助您找到您需要的包。另一方面，作为一个框架作者，注册中心将帮助你推广你的工作。目前，苹果没有提供第一方的软件包注册表，所以我们将使用可用的解决方案，没有一个明确的赢家或默认的选择，以下三个是目前最流行的，我们将简要提及它们:

*   [Swift 包裹登记处](https://swiftpackageregistry.com/) ( + 5647 个包裹)
*   [Swift 包裹指数](https://swiftpackageindex.com)(+4610 个包裹)
*   这个注册表自动抓取 Github 的新包，这解释了为什么它比其他两个有更多的包

Github 还宣布 GitHub 包注册中心也将支持 Swift 包，这一功能仍处于有限的测试阶段。

现在你可能会想你应该使用哪一个？答案很简单，公开对你的框架/包总是有好处的，所以一定要把你的包添加到所有这些注册表中。

> Marco Eidinger 在他的一篇文章[中提供了关于这个主题的更多细节，如果你觉得包注册表很有趣，请务必阅读。](https://medium.com/geekculture/the-best-registries-for-your-swift-package-82c08dd45b05)

包注册是让你的框架对世界可见的一种便捷方式，但它们本身是不够的，你需要做额外的工作，因为你的框架消费者是开发者，请确保在 Reddit、Twitter & slack 等平台上与 iOS 开发者社区分享你的工作。

## 结论:

在今天的帖子中，我们创建了一个远程包，并通过在服务器上托管它来分离二进制框架，然后，我们以包注册表以及它们如何帮助人们找到你的包来结束。

这标志着这篇文章的结束，所以请务必点击鼓掌按钮👏👏如果你喜欢的话。

如何创建一个 iOS 框架是一个每周**一次的**系列，所以**在**媒体**或 [**推特**](https://twitter.com/elmoezamira) 上关注我的**来了解最新的帖子，像往常一样，我们总是很感激你的反馈。

编码快乐！

![El Moez Amira](img/7d99f0795ed331bdc8af37b1613d3144.png)

[El Moez Amira](https://medium.com/@elmoezamira?source=post_page-----b9ba478e194d--------------------------------)

## 如何创建 iOS 框架

[View list](https://medium.com/@elmoezamira/list/how-to-create-an-ios-framework-739461924d14?source=post_page-----b9ba478e194d--------------------------------)6 stories![](img/d58861afc5089f332484756807132da0.png)![](img/40aab67ccf288e3caa47bebef2054971.png)![SwiftUI Documentation viewed in Xcode 13](img/a9d8449abc39ffcb687130fd54fda832.png)