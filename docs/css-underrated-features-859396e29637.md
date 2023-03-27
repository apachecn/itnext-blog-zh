# CSS:被低估的特性

> 原文：<https://itnext.io/css-underrated-features-859396e29637?source=collection_archive---------3----------------------->

## CSS 功能没有广泛使用

正如我们已经看到的[被低估的 HTML 标签](/html-underrated-tags-119ef3e45b94)，现在我将向你展示我认为应该更多使用的 CSS 特性。

![](img/0ccb5e2bf49038725d16bfac58a13e6e.png)

# 变量

如果你认为 CSS 没有任何类型的可编程逻辑，也许你错了，不，我甚至不是在谈论创建可切换按钮的复杂方法，而是在谈论简单的变量声明。

使用它们就像使用任何语言一样。宣布并号召:

```
:root{
    /* Variable declaration */
    --purple: purple;
}element{
    /* Variable usage */
    background-color: var(--purple);
}
```

我建议在引用根文档的 *:root* 元素中声明变量(它们也可以在任何元素中声明)，因为这将给你一个更好的语义。您可以加载一个外部文件，在其中声明您的*变量。css* ，然后将它们导入您的页面。使用变量，您可以轻松管理模板属性、调色板或创建动态主题。

[*https://codepen.io/adrian-legaspi/pen/XWJRNRG*](https://codepen.io/adrian-legaspi/pen/XWJRNRG)

# 文本选择

这个选项比你想象的更强大，覆盖默认的浏览器颜色选择可以很好地给用户体验一个更好的反馈，或者让你的用户界面更具沉浸感。这是一个充满可能性的世界，但那是你的职责，我将向你展示它的用法:

```
::selection {
    color: white;
    background: purple;
}
```

[*https://codepen.io/adrian-legaspi/pen/qBEmqgb*](https://codepen.io/adrian-legaspi/pen/qBEmqgb)

非常容易应用，潜力巨大，可惜没有得到应有的利用。在使用 web 技术的桌面开发中，我认为这会有很大的市场。在你的下一个网络项目中尝试一下。

# 内容

这个属性很好地说明了 CSS 如何努力变得更加动态，我们喜欢这种语言所做的努力。Content 属性主要用于插入一些文本，但是您也可以通过 URL 来设置图像。坏消息是只能用在伪元素 *:before* 和 *:after* 中，这并不意味着它没有用。

```
element:before, element:after{
 content: “A text, url, attribute data or an emoji 😁”
}
```

[https://codepen.io/adrian-legaspi/pen/qBEmRpj](https://codepen.io/adrian-legaspi/pen/qBEmRpj)

内容接受字符串、来自元素属性的数据或图像 URL。我想给出一个关于连接字符串的符号，因为在 JavaScript 中没有必要添加像 *"+"* 或*"这样的粘合符号在 PHP 中，只有将字符串写在你的内容旁边才是我们属性的有效值。*

```
/* Another valid string example */
content: "#"attr(id)" - "url(image.url)" Another content "
```

# 计算

这不是一个属性，而是一个函数，一个逻辑函数！。有了这个，我们就有了动态计算房产价值的超能力。例如，假设我们有一个 *< main >* 内容，其动态高度为 *100vh* ，但是我们有一个 *< navbar >* ，其静态高度为 *40px* ，我们希望这两个元素同时显示… 旧的解决方案是用 JavaScript 处理这个问题，这将导致不止一行来解决它，但是这个问题现在是这样解决的:

```
main{
  height: calc(100vh - 40px);
}
```

就这么简单！

在过去的几年里，有必要使用浏览器选择器，如 *-webkit* 或 *-moz* ，但现在已经没有必要了。

[https://codepen.io/andeersg/pen/gaKadg](https://codepen.io/andeersg/pen/gaKadg)

# 过滤

这是一个多么美好的时代，过去我们没有网格，今天我们甚至可以用滤镜来设计图像。像明亮、饱和、模糊等滤镜都是有效的属性，你现在就可以使用它们。

```
img{
  filter: filterfunction(value);
}
```

属性过滤器接收[有效函数](https://developer.mozilla.org/en-US/docs/Web/CSS/filter)中的一个，玩这个有点意思。

[https://codepen.io/adrian-legaspi/pen/KKwmaOV](https://codepen.io/adrian-legaspi/pen/KKwmaOV)

外面有很好的 CSS 库用来模拟 Instagram 滤镜[，比如这个](https://una.im/CSSgram/)，是很好的选择，因为绝大多数 CSS 库的功能都基于 CSS 滤镜属性。

啊，我不得不说，据我所知，没有必要使用浏览器前缀，因为在 Mozilla 和 Chrome 中，filter 属性本身就很好，但无论如何都要使用 *-webkit* 前缀，以最大限度地兼容相当旧的 Chrome 版本。

# 结论

即使我们和他一起工作了多年，级联样式表语言也比我们想象的更让我们吃惊。希望你有机会继续寻找更多有趣的技巧和特性，你可以用你所有的工作技术来实现，而不仅仅是 CSS。

> *参考:*
> 
> *W3Schools*→*[*https://www.w3schools.com/*](https://www.w3schools.com/)*
> 
> *CSS-TRICKS→[https://css-tricks.com/](https://css-tricks.com/)*
> 
> **MDN web 文档*→[*https://developer.mozilla.org/*](https://developer.mozilla.org/)*
> 
> **我自己*→【https://twitter.com/ImLuyou *