# 使用 CocoaPods 将 EarlGrey 添加到您的 iOS 应用程序

> 原文：<https://itnext.io/adding-earlgrey-to-your-ios-app-with-cocoapods-918d4ce1dfd0?source=collection_archive---------5----------------------->

![](img/cdfd68d72af6c6f3a4102d81b7b3938d.png)

马特·西摩在 [Unsplash](https://unsplash.com/s/photos/earl-grey-tea?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

[EarlGrey](https://github.com/google/EarlGrey/tree/earlgrey2) 是比较流行的 iOS UI 测试框架之一。它由 Google 开发，原生构建在`XCUITest`之上，使用 Google 的 [eDistantObject](https://github.com/google/eDistantObject) 远程调用系统有效运行。

在这篇博客中，我将展示如何使用 CocoaPods 轻松地将其添加到您的 iOS 项目中。

最新版本 EarlGrey2 分为两个部分——应用端和测试端。这两个组件位于各自的区域，应用程序位于被测应用程序中，测试端位于实际测试本身中，它们使用 eDistantObject 进行交互。

因为有两个不同的组件，所以您将在您的 iOS 项目中包含两个不同的 pod。

假设你在做一个名为`TheCatsApp`的 iOS 应用。当您在`TheCatsApp`上执行`pod init`时，应该会生成一个基本的 Podfile，如下所示:

```
target 'TheCatsApp' doendtarget 'TheCatsAppUnitTests' doendtarget 'TheCatsAppUITests' doend
```

其中每个`target`都是 Xcode 项目中定义的目标。`TheCatsApp`目标是你的实际应用，其他分别是你的单元和 UI 测试目标。

由于 EarlGrey 是一个 UI 测试框架，我们将把重点放在应用程序和 UI 测试目标上。

更新您的 Podfile，添加`EarlGreyApp`和`EarlGreyTest`，如下所示:

```
target 'TheCatsApp' do pod 'EarlGreyApp'endtarget 'TheCatsAppUnitTests' doendtarget 'TheCatsAppUITests' do pod 'EarlGreyTest'end
```

然后在该目录上运行`pod install`来安装这些 pod。

如你所见，我们将`EarlGreyApp`添加到我们的应用程序目标中，并将`EarlGreyTest`添加到我们的 UI 测试目标中。您可能会注意到，我们没有包含指向 eDistantObject 的 pod 链接，这是因为如果您查看 EarlGrey2 eDistantObject 的 [Podspec](https://github.com/google/EarlGrey/blob/earlgrey2/EarlGreyTest.podspec) ，就会发现它已经作为一个依赖项包含在内了。

运行`pod install`后，您已成功将 EarlGrey 添加到您的 iOS 应用程序中！

要将它添加到您的 Objective-C UI 测试中，您可以像`#import <EarlGreyTest/EarlGrey.h>`一样导入它。要在 Swift UI 测试中使用 EarlGrey，您只需添加 [EarlGrey Swift 包装器](https://github.com/google/EarlGrey/blob/earlgrey2/TestLib/Swift/EarlGrey.swift)并添加桥接头，如 [EarlGrey 文档](https://github.com/google/EarlGrey/blob/earlgrey2/docs/setup.md)所示。

这里还有一个示例项目[这里](https://github.com/google/EarlGrey/tree/earlgrey2/Demo/EarlGreyExample)有演示代码所有设置，显示我在这里解释的一切。

编码快乐！