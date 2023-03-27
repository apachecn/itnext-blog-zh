# Android 上的(不)有序列表

> 原文：<https://itnext.io/un-ordered-lists-on-android-20defac6b156?source=collection_archive---------1----------------------->

![](img/2073dbdcbd96d5a175a342d75065470a.png)

项目符号列表或编号列表是总结内容的常见方式，但是我们如何在 Android 中实现它们呢？让我们快速看看如何在一个简单的文本视图中实现这些列表！

# 项目符号列表

所以让我们从“简单的”开始:一个项目符号列表。假设你知道我在说什么是安全的，但这里有一个例子只是为了记录:

*   项目 1
*   项目 2
*   项目 3

假设我们有一个简单的内容列表:

```
val items = ["Item 1", "Item 2", "Item 3"]
```

我们希望为每个项目添加一个项目符号，但也要确保每当内容有多行时，项目符号会对齐到`TextView`的顶部。为了实现这一点，我们将看看如何使用跨度！

从一个简单的`SpannableStringBuilder`开始，我们将以此为起点，一行一行地添加我们的代码:

```
val builder =  SpannableStringBuilder()
```

因此，一旦我们有了构建器，我们将简单地遍历我们的列表，添加一些换行符，并使用`BulletSpan`来实现实际的要点:

`BulletSpan`实现`LeadingMarginSpan`，基本上是把你的文字偏移一定的量。在`BulletSpan`的情况下，它会在偏移区域内添加一个项目符号。

> 如果您想手动设置偏移区域或设置项目符号的颜色，只需使用`BulletSpan`的可选`gapWidth`和`color`参数！

现在你需要做的就是将`builder`设置为`TextView`的文本，然后就完成了！`TextView`中的项目符号列表😄

# 编号列表

所以首先:我们对一个编号列表有什么意义？它基本上是一个项目符号列表，但我们将为每个项目符号编号，而不是项目符号。如果您熟悉 HTML，请将其与有序列表进行比较:

1.  项目 1
2.  项目 2
3.  项目 3

实现编号列表的基础和项目符号列表是一样的，但是我们需要一个不同的跨度。Android 没有任何现成的跨度来支持这一点，但幸运的是，创建我们自己的跨度非常容易！我们可以简单地自己实现`LeadingMarginSpan`并在其中添加我们需要的行为。
通过实现，我们将获得两个要覆盖的方法:`drawLeadingMargin`和`getLeadingMargin`:

*   `drawLeadingMargin`负责呈现前导边距
*   `getLeadingMargin`简单返回前导边距的宽度。

我们将向 span 传递 2 个东西:前导边距的宽度和我们希望在前导边距内绘制的文本(在本例中为:数字):

就这些了！现在，我们可以简单地使用这个跨度，就像我们对项目符号列表所做的一样，我们唯一需要传递的是前导边距的宽度和我们想要显示的数字:

作为最后一笔，将它隐藏在一个好的扩展方法后面，你将很快实现有序和无序列表！

就这样，伙计们！Android 上文本视图中的有序或无序列表😄*有问题吗？这里是* [*捏*](https://pinch.nl/en) *，我们很高兴能帮到人，就打我一下吧。祝大家编码愉快！*