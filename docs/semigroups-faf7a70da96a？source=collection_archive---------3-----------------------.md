# 半群和幺半群

> 原文：<https://itnext.io/semigroups-faf7a70da96a?source=collection_archive---------3----------------------->

半群是一个简单的函数概念，它本质上允许我们获取许多对象并返回单个对象。在那方面和`reduce`很像。

半群最基本的实现是:`(A, A) -> A`。你可能会看着这个并想，这对于像`+`这样的中缀操作符来说是完美的。你可能是正确的，在`Haskell`中，操作符是`<>`，以便不与现有的`+`功能冲突，因为实现可能不同。

在 Swift 中，我们可以实现这样一个半群:

```
**infix** **operator** <>: AdditionPrecedence**public** **protocol** Semigroup { 
    **func** combine(with other: Self) -> Self
    **static** **func** <> (lhs: Self, rhs: Self) -> Self
}**extension** Semigroup {
    **public** **static** **func** <> (lhs: Self, rhs: Self) -> Self {
        **return** lhs.combine(with: rhs)
    }
}
```

许多 Swift 类型已经在使用半群逻辑，包括*数字*、*字符串*和*数组*。它们都与`+`操作符相关联。因此，我们可以扩展这些类型，也使用`<>`操作符。

```
**extension** Numeric **where** Self: Semigroup {
    **public** **func** combine(with other: Self) -> Self {
        **return** **self** + other
    }
}**extension** Int: Semigroup { }**extension** Array: Semigroup {
    **public** **func** combine(with other: Array) -> Array {
        **return** **self** + other
    }
}**extension** String: Semigroup {
    **public** **func** combine(with other: String) -> String {
        **return** **self** + other
    }
}**extension** Bool: Semigroup {
    **public** **func** combine(with other: Bool) -> Bool {
        **return** **self** && other
    }
}
```

现在我们可以写一个简单的函数来连接任何类型的半群:

```
**public** **func** concat<S: Semigroup>(**_** values: [S], initial: S) -> S {
    **return** values.reduce(initial, <>)
}
```

这个函数只是减少了我们代码中的`<>`符号的数量，也会对编译器有所帮助。

现在，我们可以像这样组合一组值:

```
**let** a = **true
let** b = **true
let** c = **false**concat([a, b, c], initial: **true**) *// false*
```

我们还可以使用此功能组合阵列:

```
**let** spicy = ["Pepper", "Chilli"]
**let** sweet = ["Mango", "Pineapple"]
**let** sour = ["Lemon", "Sauerkraut"]**let** dish = concat([spicy, sweet, sour], initial: [])*// ["Pepper", "Chilli", "Mango", "Pineapple", "Lemon", "Sauerkraut"]*
```

现在我们有了这个操作符，我们也可以将它用于我们自己的类型。我们所要定义的只是一种结合它们的方式。

# 一个实际的例子:造型

我讨厌到处都是布局代码。

我将创建一个很好的构建`Styles`的可组合方式，我们可以将它应用于我们的`Views`。

首先，让我们定义我们的`Style`结构，这样我们就有了一个要组合的类型。

```
**public** **struct** Style<T> {
    **private** **let** callback: (T) -> Void
    **public** **init**(**_** callback: **@escaping** (T) -> Void) {
        **self**.callback = callback
    } **public** **func** apply(to candidate: T) {
        callback(candidate)
    }
}
```

非常简单的概念，我们传入一个回调函数，稍后我们可以应用它。它本质上是一个`defer`但是有回调。

到`combine`，我们只需要向两边`apply`。`combine`函数如下所示:

```
**extension** Style: Semigroup {
    **public** **func** combine(with other: Style) -> Style {
        **return** Style {
            **self**.callback($0)
            other.callback($0)
        }
    }
}
```

太好了，现在我们有必要的基础来开始组合它们，就像乐高积木一样。再为`CALayer`补充一些吧。

```
**func** border(width: CGFloat) -> Style<CALayer> {
    **return** .init { $0.borderWidth = width }
}**func** border(color: UIColor) -> Style<CALayer> {
    **return** .init { $0.borderColor = color.cgColor }
}**func** corner(radius: CGFloat) -> Style<CALayer> {
    **return** .init { $0.cornerRadius = radius }
}
```

我建议将这个*可重用的*代码存储在你的 *App* 项目之外的地方，比如一个*框架/目标*。

然后，在您的应用程序项目中，您可以像这样开始定义样式:

```
**let** rounded = border(width: 0.5) <> border(color: .black) <> corner(radius: 4)*// It can be applied like so:* rounded.apply(to: view.layer)
```

你可能想知道为什么我们没有使用`concat`方法，这是因为我们还没有为我们的`Style`定义一个`empty`(或初始)状态。为此，我们需要创建一个*幺半群*。

# 幺半群

```
**public** **protocol** Monoid: Semigroup {
    **static** **var** empty: Self { **get** }
}
```

酷，现在让我们为我们的`Style`定义一个`empty`值…

```
**extension** Style: Monoid {
    **public** **static** **var** empty: Style<T> {
        **return** .init { **_** **in** }
    }
}
```

现在我们可以使用我们之前定义的`concat`方法，或者……我们也可以创建一个新方法，它接受一个*幺半群*并为我们设置初始值。

```
**public** **func** concat<M: Monoid>(**_** values: [M]) -> M {
    **return** values.reduce(M.empty, <>)
}
```

这允许我们像这样定义样式:

```
**let** rounded = concat([border(width: 0.5), border(color: .black), corner(radius: 4)])
```

我们也可以翻转操作符，并在扩展中幺半群序列上定义 concat 来删除一些括号。

```
**extension** Sequence **where** Element: Monoid {
    **func** joined() -> Element {
        **return** concat(Array(**self**))
    }
}**let** rounded = [border(width: 0.5), border(color: .black), corner(radius: 4)].joined()
```

现在，上面的代码小而简洁，没有向初级开发人员暴露任何可怕的操作符。

对于这个简单的概念来说，用例是无穷无尽的，并且可以减少代码中的循环次数，因为您总是将代码展平到一个组合中，而不是传递数组。

这应该是开始将它包含在您自己的代码中的足够好的入门。尽情享受吧！

半群:[https://gist . github . com/CJ nevin/9e 9 ef 3 F4 F9 a2 f 223 c 723 ace 9 B0 a 959 BF](https://gist.github.com/cjnevin/9e9ef3f4f9a2f223c723ace9b0a959bf)

mono id:[https://gist . github . com/CJ nevin/f 683619 e 2841 e 71543 C5 E8 fa 0405 c 969](https://gist.github.com/cjnevin/f683619e2841e71543c5e8fa0405c969)