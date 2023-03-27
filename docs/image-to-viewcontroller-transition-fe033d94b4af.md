# 自定义 UIViewController 转换

> 原文：<https://itnext.io/image-to-viewcontroller-transition-fe033d94b4af?source=collection_archive---------1----------------------->

## 不要拘泥于仅仅是模态和推送。

![](img/d1db51fc0e4029188a3187da216a3530.png)

今天我们将实现一个自定义过渡，使用`UIViewControllerAnimatedTransitioning`将`UIView`从半屏放大到全屏。在这个自定义转换中会发生什么只是一些简单的约束动画，但我们可以做几乎任何事情。

![](img/a6e7fa62a29c7cb8e7fb725e2dcc083a.png)

从我在这里起草的这个 quick-n-dirty 方案中，你可以得到发生在转换上的想法。为了简单起见，我们只对每个屏幕使用一个视图。

因为`NSLayoutConstraints`，这看起来有点夸张，但事实上这里并没有太多。在我们到达`Presenter`之前，请注意屏幕并不知道我们要做什么。逻辑与之完全隔绝。

`Presenter`是从`UIViewControllerTransitioningDelegate`协议中实现这两个功能的协调器。我们在这里要做的就是以正确的方向将转换传递给每个`animationController(for presented/for dismissed)`

> 好了，是时候看看我们在这里的原因了:`*Transition*`。

好的，首先，我们有一个名为 Direction 的枚举，它包含两种情况，一种是 present，另一种是 discover。我们还保持方向，以便我们可以在我们的演示者中设置它。当符合协议`UIViewControllerAnimatedTransitioning`时，我们必须实现`transitionDuration`和`animateTransition`功能。第一个简单地返回一个`CGFloat`，它是过渡的持续时间，第二个是“*大交易*”。

我们将现在分离出来，放入私有函数中，因此在 animateTransition 函数中，我们只需切换方向，并通过 transitionContext 调用适当的函数。另外，请注意，我们存储了两个 NSLayoutConstraints 数组。它们将保留已呈现的状态约束和已取消的状态约束。

*   使用 present 函数上的上下文，我们可以获得将使用 context.view(forKey:。到)。如果我们想在转换之前做些别的事情，我们也可以通过将键改为**来检索将要呈现的视图。来自**
*   现在我们有了将要呈现的视图，我们将它添加为 context.containerView 中的子视图，这是在转换期间显示的视图，然后为每个状态设置约束:呈现和取消。
*   我们需要做的最后一件事是纯粹的 UIView 动画约束，现在(真的)最后一件事是调用 completeTransition(true)以显示正确的屏幕。

解散的时候，我们正好反过来。玩约束和*瞧*。

这里有一个我们可以用它做什么的例子:

![](img/a335c7a95630a24613523db97ac7abba.png)