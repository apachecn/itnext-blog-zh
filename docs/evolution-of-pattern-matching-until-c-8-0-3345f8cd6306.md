# C# 8.0 之前模式匹配的发展

> 原文：<https://itnext.io/evolution-of-pattern-matching-until-c-8-0-3345f8cd6306?source=collection_archive---------3----------------------->

![](img/3e270b0df6b53bbe7d49d17cbdb1ea43.png)

[来源](https://gist.github.com/dimitris-papadimitriou-chr/abbe9fa04008a6d2667e1467435c0a27)

什么是模式匹配？假设你有一个简单的形状层次结构

现在如果我给一个`Shape` 变量赋值。我可以指定矩形或圆形。这里我将选择矩形

`**Shape** shape = new **Rectangle** { Width = 100, Height = 100, };`

> 你如何在运行时知道我放入了什么形状？

**这是模式匹配想要解决的底层问题。我们将看到 C# 8.0 如何最终更接近于一种处理这个问题的函数式方法。**

在这里，我们将看到如何基于形状类型获得不同描述的不同方法。

# 1.使用 Is 关键字

我们可以使用 if 语句来处理一些非常基本的事情**:**

# 2.使用 switch 语句

从 C# 7.0 开始，我们可以使用 switch 语句来分隔子类型:

# 3.使用多态性

是的，通过使用多态，我们可以通过调用多态方法来区分变量被赋给了哪种`Shape` 类型:

**现在所有的箱子都移到了每个类里面。**

```
var description = shape.GetShapeDescription();
```

# 4.我们可以使用自定义模式匹配

现在我们将重构前面的多态性案例，以便抽象出多态性的机制！！

我们有这个 **MatchWith 方法，我们可以用它来分离案例**

```
public abstract T **MatchWith**<T>(
              Func<Rectangle, T> **rectangleCase**, 
              Func<Circle, T> circleCase);
```

我们将这样使用它:

您可以看到这与 switch 语句相同:

```
string GetShapeDescription(Shape shape)
{
 switch (shape)
  {
   case Circle c:
      return $"found a circle with radius {c.Radius}";
   case Rectangle s:
      return $"Found {s.Height }x {s.Width} rectangle";
   default:
      throw new System.Exception(nameof(shape));
   }
 };
```

现在进化的最后一步

# 5.使用 C# 8.0 模式匹配

我们最终可以使用 switch 关键字的另一种语法进行模式匹配，而不需要之前的自定义实现:

习惯这种语法需要一些时间，但是很容易掌握。如您所见，您可以访问子类的公共属性:

```
shape switch
 {
   Rectangle { **Height: var h**, **Width: var w** } 
              => $"Found {h }x {w} rectangle",
   Circle { **Radius: var r** } 
              => $"found a circle with radius {r}",
}
```

再举一个例子。假设我们想要缩放形状，我们可以写这个函数作为利用模式匹配的扩展

像这样使用它:

`var t1 = shape.**Scale**(x => x*x);`

**总之:**

C#模式匹配最后带来了另一个**功能特性**，它将帮助 C#开发者更自然地编写功能代码。