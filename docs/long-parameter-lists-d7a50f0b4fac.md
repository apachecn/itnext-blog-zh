# 长参数列表

> 原文：<https://itnext.io/long-parameter-lists-d7a50f0b4fac?source=collection_archive---------4----------------------->

![](img/4191ee60a0455e1770dfa13f421fd09d.png)

马库斯·斯皮斯克在 [Unsplash](https://unsplash.com/photos/xekxE_VR0Ec?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

例如，假设我们有一个`UIButton`子类，我们希望它可以在调用点定制，所以我们向它的`init`方法添加了两个参数:

```
final class Button: UIButton { 
    init(textColor: UIColor, borderColor: UIColor) 
} // ... let button = Button(textColor: .darkText, 
                    borderColor: .darkText)
```

看起来很好。

过了一段时间，需要定制它的背景颜色，这时我们需要另一个参数:

```
final class Button: UIButton { 
    init(textColor: UIColor, borderColor: UIColor, backgroundColor: UIColor) 
} // ... let button = Button(textColor: .white,
                    borderColor: .white, 
                    backgroundColor: .darkText)
```

仍然没有那么糟糕，尽管我开始争论。

又过了一段时间，我们现在需要让它变大或变小，有圆角或没有圆角，并且有时在触摸时有动画效果。再添加三个参数将会阻塞实现站点*和*调用站点。

我们可以添加默认参数，以使调用网站更漂亮，但这仍然不是我们所能做到的最好的。如果我们将所有这些参数包装在一个`Config`对象中并用于`init`会怎么样？

```
final class Button: UIButton { 
    init(config: Config) 
} extension Button { struct Config { 
        var textColor: UIColor = .darkText 
        var borderColor: UIColor = .darkText 
        var backgroundColor: UIColor? = nil 
        var animatesOnTouch = false 
        var cornerRadius: CGFloat? = nil 
    } 
} // ... let button1 = Button(config: Button.Config()) var config = Button.Config() 
config.textColor = .white
config.borderColor = .white 
config.backgroundColor = .darkText 
config.animatesOnTouch = true 
config.cornerRadius = 8 let button2 = Button(config: config)
```

`init`是干净的，调用点也是干净的，我们只在需要的时候创建一个定制的`Config`对象，并且只设置我们需要的属性。

看起来好多了，不是吗？几乎任何语言都可以使用相同的方法。

作为一个额外的提示，我喜欢嵌套相关的对象，就像我们的例子中的`Config`。比起`ButtonConfig`，我更喜欢`Button.Config`；虽然它们基本上是一样的，从字符的数量到发音，但是嵌套的对象在语义上更正确，更有组织性。

*你可以在我的博客上找到更多这样的文章，或者你可以订阅我的* [*月报*](https://rolandleth.us19.list-manage.com/subscribe?u=0d9e49508950cd57917dd7e87&id=7e4ef109bd) *。最初发表于*[*rolandleth.com*](https://rolandleth.com/tech/blog/long-parameter-lists)*。*