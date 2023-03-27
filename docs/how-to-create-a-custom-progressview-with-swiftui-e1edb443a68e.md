# 如何:使用 SwiftUI 创建自定义 ProgressView

> 原文：<https://itnext.io/how-to-create-a-custom-progressview-with-swiftui-e1edb443a68e?source=collection_archive---------1----------------------->

![](img/160fdba82870344a62e7bca6a5aaf468.png)

照片由[尼克·佩奇](https://unsplash.com/@nickpage?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

对于在 iOS 或 MacOS 应用程序中创建进度条视图，您可能已经熟悉了 UIKit 或 SwiftUI 中的 ProgressView 元素，并且可能还知道它如何缺少简单的定制。默认实现只是一个填充蓝色的细条:

![](img/9574c82287560670a3ff9c8418fb0fad.png)

系统的默认进度视图

定制视图修改器，如*背景*、*高度*、*等*。将会改变视图的所有内容，但不改变工具栏本身。

它将保持原样——蓝色而朴素。

如果你想有自己的自定义进度条，你有两个选择:

1.  创建你自己的进度条视图并实现它
2.  根据 Apple 提供的 ProgressViewStyle 协议，创建您的自定义进度条视图样式

本文将关注后者，尽管您也可以轻松地获取代码，用绑定扩展它，并拥有自己的视图而不是样式——这根本没有什么意义。

> 扩展现有的协议也应该是首选的方式，因为重新发明轮子很少是一个好主意。

# 目标

像往常一样，让我们先看看我们最终想要达到的目标:

![](img/61032d99b0295a8e4fa65f79b8e8c810.png)

我们想要实现的最终进度视图

我们想用一个更宽的颜色渐变条代替蓝色细条，而不是简单的颜色。此外，我们希望能够设置一个显示在视图下方的可选标签。

# 协议

苹果为我们提供了一个在 SwiftUI 中创建自定义流程视图样式的协议: [ProgressViewStyle](https://developer.apple.com/documentation/swiftui/progressviewstyle) 。

协议本身有一个方法要求:

`func makeBody(configuration: Self.Configuration) -> Self.Body`

这个函数基本上构建了我们的定制进度视图。在其中，我们定义了视图的主体以及任何属性、样式等。—其实和 [ViewModifier](https://developer.apple.com/documentation/swiftui/viewmodifier) 协议的**func body(Content:Content)->some View**函数很像。如果您了解 ViewModifier，那么将这一知识应用到样式协议中应该很容易。

# 风格

创建定制的进度视图样式实质上意味着定义整个视图主体，而不仅仅是对其进行样式化。

对于进度视图样式的主体，我们使用 ZStack，我们将它包装在 VStack 中，以提供标签的可能性。此外，我们添加了一些变量来定制选项(如背景、高度等。):

第 4–9 行是用于定制我们的定制视图的属性。我们的大多数属性都有默认值，所以只有*笔画*和*填充*是必需的，其余的都是可选的。

在 ZStack 中，我们放置了一个 GeometryReader 来动态地读取视图的宽度，然后根据进度视图的当前值，使用这个宽度作为封闭矩形的宽度。如果用户旋转屏幕或改变视图分辨率，进度条状态应该是相同的，只是缩放到新的宽度。

可以从 makeBody 函数内部的给定配置对象中读取进度条的当前值。值*configuration . fraction completed*以一个从 0 到 1 的双精度值的形式表示当前完成的总工作量的一部分。

# 实施

让我们看看如何实现我们的自定义进度条。代替进度条的静态值，我们将使用定义进度条状态的滑块视图。有了它，我们可以动态地改变标杆。

为了简单起见，我们只使用 ContentView.swift 文件来实现——在现实生活中，最好将它拆分为单独的文件:

内容视图. swift

第一行(3–36)是我们之前章节中的进度条样式。

第 39–75 行是实现本身。一如既往，它应该是非常直截了当的。首先，我们为值、输入状态(滑块)和进度条的渐变定义了一些状态变量。

在第 59–61 行，您可以看到 ProgressView 实现附加了我们的自定义样式和一些垂直填充。不要太花哨…

如果我们运行代码并转动滑块手柄，我们的进度条将会和上面看到的一模一样。这篇 how-to 应该给出了一些如何创建定制进度视图的基本概念，这样您现在就可以轻松地创建自己的视图，或者只是从这里获取代码并扩展它。