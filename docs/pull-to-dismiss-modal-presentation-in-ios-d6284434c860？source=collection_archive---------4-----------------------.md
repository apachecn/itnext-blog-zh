# iOS 中的拖放模式演示

> 原文：<https://itnext.io/pull-to-dismiss-modal-presentation-in-ios-d6284434c860?source=collection_archive---------4----------------------->

![](img/6b464f6793cb1d5ef14bec4e148b211d.png)

朱利安·迈尔斯在 [Unsplash](https://unsplash.com/s/photos/california?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

**TL；**这是[*博士最后的代号*](https://gist.github.com/vinczebalazs/ee1f2b466f969fa70d424d4480695325) *。*

我用它建立了一个示例模型:

嘿！

今天，我将快速向您介绍如何创建一个可以通过下拉手势消除的模态演示样式，就像 iOS 13 的默认行为一样。

你可能会问，为什么不使用默认的呢？

嗯，主要是两个原因:

1.  向后兼容性。
2.  定制的需要。例如，将所呈现的模型的高度改为仅半屏。

让我们开始吧！

首先，我写这篇文章的原因是因为关于这个主题的文档很少，我花了相当多的时间来弄清楚所有的部分是如何组合在一起的。

构建这样的东西有几个挑战，第一个挑战实际上是理解以下协议如何协同工作:

**UIViewControllerTransitioningDelegate**

**UIViewControllerAnimatedTransitioning**

**UIPresentationController**

**uipercentdriveninteractive transition**

苹果在[解释](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/CustomizingtheTransitionAnimations.html)它方面做得相当不错，然而他们的代码样本确实给读者留下了许多漏洞需要填补。我不会在这里解释每个协议做什么，因为本文的目的是向您展示它们是如何协同工作的。

关键点是:

*   UIViewControllerTransitioningDelegate 作为一个中介，负责提供动画对象、交互控制器和演示控制器。
*   UIViewControllerAnimatedTransitioning 是我们的 animator 对象，这里是实际动画代码的位置。
*   **UIPresentationController**设置演示环境，这包括在模型后面添加一个“调光视图”,并附加一个平移手势识别器用于交互式消除。
*   **UIPercentDrivenInteractiveTransition**将根据完成的百分比来擦除动画，我们根据平移手势的翻译来计算完成的百分比。

我发现自己陷入的第二个**陷阱**是使用 **UIView.animate** 而不是**uiviewpropertyimator。UIView.animate** 应该在纸上完成工作，但是当与**uipercentdriveninteractive transition**配合使用时，动画与平移手势完全不同步。

第三件让我感到困难的事情是让用户感觉解雇过渡很自然。这意味着如果用户的滑动手势具有更高的速度，则以更快的动画关闭模式。在找到解决问题的方法后，解决方案确实非常简单:

在***UIPercentDrivenInteractiveTransition 的*****completion speed**中设置的值作为动画持续时间的乘数。我们使用 **max(1，x)** 是因为如果用户缓慢向下拖动，然后放开，我们不希望以这种缓慢的速度关闭模态，而是使用默认速度。否则会感觉怪怪的。

第四件事并不是真正的挑战，而是一个**干**的问题。当使用定制的表示风格表示视图控制器时，每次都必须在表示视图控制器中编写样板代码:

相反，我把它包装成了一个扩展，这样我就不用每次都写了。

我希望这有助于消除一些困惑，并为您在自己的应用程序中实现如此琐碎(但却令人惊讶地具有挑战性)的东西指明正确的方向！

感谢您的阅读，敬请期待更多内容！🎉

***附言*** *有问题吗？请在评论中告诉我。*

**资源:**

[要点与最终代码和示例用法。](https://gist.github.com/vinczebalazs/ee1f2b466f969fa70d424d4480695325)