# CSS 基础:属性选择器

> 原文：<https://itnext.io/css-fundamentals-attribute-selectors-444d3f4264b4?source=collection_archive---------3----------------------->

![](img/d5123b707e660d816e0242c63167f19f.png)

在本文中，我们将学习所有关于属性选择器的知识。如果你还记得 HTML，元素可以有提供额外细节的属性。

具有`href`属性的`a`标签:

```
<a href="https://easeout.co">This is a link</a>
```

我们使用 CSS 属性选择器来为这些元素指定特定的属性。例如:

```
a[href="https://easeout.co"] {
  color: #FFFF00;
}
```

这里我们有一个**精确匹配**选择器，它只会选择**精确** `href`属性值为“https://easeout.co”的链接。

所以让我们来学习如何使用这些非常有用的选择器！

🤓*想与 web dev 保持同步吗？*
🚀想要将最新消息直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 属性选择器的类型

有许多不同类型的属性选择器，分为存在和值选择器或子串匹配选择器。

每个选择器的语法以属性名开始，以等号结尾，后面跟着属性值，通常用引号括起来。

属性名和等号之间的内容决定了选择器的类型！

# 存在和值选择器

这些选择器将基于单独属性的存在(例如`href`)或者基于针对属性值的各种不同匹配来选择元素。

`[attr]` *属性本身。*

匹配属性名为 *attr 的所有元素。*如`a[title]`。

`[attr=”value”]` *属性有这个确切的值。*

匹配属性名为 *attr* 的所有元素，其值为**精确值** *值* —引号内的字符串。如`a[href="https://easeout.co"]`。

`[attr~=”value”]` *属性在一个空格分隔的列表中有这个值。*

匹配属性名为 *attr* 且值恰好为*值*的元素，或者匹配属性名为 *attr* 且包含一个或多个值的元素，其中至少有一个值匹配*值*。

在多个值的列表中，各个值用空格分隔。例如，`img[alt~="dough"]`将选择带有备选文本“制作*面团*”和“*面团*类型”的图像，而不是“*面团*坚果”的图像。

`[attr|=”value”]` *用破折号分隔的列表中的第一个属性值。*

匹配属性名为 *attr* 的所有元素，属性名的值可以正好是*值*，也可以以*值*开始，后面紧跟一个连字符。例如，`li[attr-years|="1990"]`将选择`attr-years`值为“1990-2000”的列表项，而不选择`attr-years`值为“1900-1990”的列表项。

# 子串匹配选择器

子字符串选择器允许在属性值内部进行更高级的匹配。例如，如果您有`message-warning`和`message-error`类，并且想要匹配以字符串“message-”开头的所有内容，您可以使用`[class^="message-"]`来选择它们！

`[attr*=”value”]` *属性值包含此值。*

匹配属性名为 *attr* 的所有元素，其值在字符串中至少包含一次子字符串 *value* 。例如，`img[alt*="dough"]`将选择包含 alt 文本" dough "的图像。所以“制作*面团*”、“*面团*类型”和“*面团*坚果”都会被选中。

`[attr^=”value”]` *该属性以该值开始。*

匹配任何属性名为 *attr* 的元素，其值以子字符串 *value* 开头。例如`li[class^=”message-”]`。

`[attr$=”value”]` *属性以该值结束。*

匹配任何属性名为 *attr* 的元素，其值的末尾有子字符串 *value* 。例如，`a[href$="pdf"]`选择每个以. pdf 结尾的链接。

# 区分大小写

到目前为止，所有的检查都是区分大小写的。

如果在右括号前添加一个`i`,检查将不区分大小写。它被大多数浏览器支持，但不是所有，检查[https://caniuse.com/#feat=css-case-insensitive](https://caniuse.com/#feat=css-case-insensitive)。

# 组合属性选择器

属性选择器也可以与其他选择器结合使用，比如标签、类或 ID。

```
div[attr="value"] {
  */* style rules */*
}.item[attr="value"] {
  */* style rules */*
}#header[attr="value"] {
  */* style rules */*
}
```

并且可以组合多个属性选择器:

```
img[alt~="city"][src*="europe"] {
  */* style rules here */*
}
```

在这里，我们选择所有带有 alt 文本的图像，包括单词“city”作为唯一值或空格分隔的列表中的值，**和**包括值“europe”的`src`值。

***你准备好让你的 CSS 技能更上一层楼了吗？*** *现在就开始用我的新电子书:*[*《CSS 指南:现代 CSS 完全指南*](https://gum.co/the-css-guide) *。获取从 Flexbox & Grid 等核心概念到动画、架构等更高级主题的最新信息！！*

![](img/d3e2ee6adb6ffa2c189049cea5937e93.png)

*现已上市！👉*[gum.co/the-css-guide](https://gum.co/the-css-guide)

# 结论

就是这样！我们已经讨论了属性选择器的基础知识。包括存在和值选择器、子串匹配选择器、增加区分大小写和组合选择器。

我希望这篇文章对你有用。你可以在 Medium 上[关注我](https://medium.com/@timothyrobards?source=post_page---------------------------)。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正致力于构建我的[自由职业完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果是你感兴趣的东西，你可以[注册，当它可用的时候会通知你👍](https://easeout.eo.page/news)

感谢阅读🎉