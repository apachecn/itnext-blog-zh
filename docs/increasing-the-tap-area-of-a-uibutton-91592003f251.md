# 增加 UIButton 的点击区域

> 原文：<https://itnext.io/increasing-the-tap-area-of-a-uibutton-91592003f251?source=collection_archive---------4----------------------->

前几天， [Soroush](https://twitter.com/khanlou) 写了一篇关于`hitTest(_:with:)`的很棒的[帖子](http://khanlou.com/2018/09/hacking-hit-tests/)(你应该去看看)，这让我想起了我前几周解决的一个问题:我想增加一个`UIButton`的点击区域，而不增加按钮的框架。

下面的代码将放在一个按钮的`superviews`中；例如，您可能有一个`mainContainer`，它有一个包含按钮的`UIStackView`——我们将在`mainContainer`中实现它:

```
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? { 
    let biggerButtonFrame = theButton.frame.insetBy(dx: -30, dy: -30) // 1 
    if biggerButtonFrame.contains(point) { // 2 
        return theButton // 3 
    }     return super.hitTest(point, with: event) // 4 
}
```

我们首先创建一个比我们的按钮大`30px`的 rect，无论水平还是垂直(1)，我们检查它是否包含 point (2):如果包含，我们返回按钮(3)，否则我们退回到`super` (4)。在(1)处，我们可能需要在坐标系之间进行转换，在(2)处，我们可能还希望检查按钮是否启用和可见，它是否可以有不同的状态。

如果你谷歌一下`increase uibutton tap area`，你会得到相当多的结果，提到要么增加`imageEdgeInsets` / `titleEdgeInsets`(不完全是我们想要的)，要么使用`pointInside(point:with:)`。后者的方法与我们上面看到的基本相同，只是实现是在按钮本身上完成的，而不是在它的`superviews`之一中:

```
override func point(inside point: CGPoint, with event: UIEvent?) -> Bool { 
    let biggerFrame = bounds.insetBy(dx: -30, dy: -30)     return biggerFrame.contains(point) 
}
```

有时候，一个按钮可能是视图控制器的`view`的一部分，或者被用作`UIBarButtonItem`的`customView`，在这种情况下，我们无法访问`superview`的实现，所以我们需要使用这种方法。

在某些时候，我真的不想再写这个帖子了，看到这么多关于这个可以追溯到 2010 年的信息，但是，最终，我认为任何知识分享都比保留好。

我以前是怎么做的？通过创建一个包含所需信息的`UILabel` / `UIImageView`并在顶部添加一个更大的按钮🤷🏻‍♂️😳。

*原载于*[*rolandleth.com*](https://rolandleth.com/increasing-the-tap-area-of-a-uibutton)*。*