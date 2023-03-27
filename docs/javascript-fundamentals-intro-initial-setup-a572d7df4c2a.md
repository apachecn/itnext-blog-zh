# JavaScript 基础:介绍和初始设置

> 原文：<https://itnext.io/javascript-fundamentals-intro-initial-setup-a572d7df4c2a?source=collection_archive---------3----------------------->

![](img/498e6a29bc950ab721d6dd27fbdd12e5.png)

**JavaScript 是世界上最流行的编程语言**，所以难怪它是 web 开发行业最受欢迎的技能之一。事实上，Devskiller IT 技能和招聘报告(2020 年)报告称，72%的公司希望招聘 JavaScript 专家。

此外，在全球超过 18 亿个网站中，**95%的网站使用 JavaScript**。

选择在 2021 年学习 JS 无疑是一个非常明智的举动！让我们先介绍一下，然后看看初始设置步骤。

🤓*想了解最新的网站开发信息吗？*
🚀想要将最新的新闻直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# **介绍 JavaScript！**

JavaScript 是一种编程语言，它让你能够在网页上实现复杂的功能——而且你可以做的事情几乎没有限制！您可以制作 2D/3D 图形动画，添加滚动视频，交互式地图，可缩放图像，开发游戏，可能性真的是无穷无尽！

谈到标准 web 技术，它与 HTML 和 CSS 配合得非常好。

*   **HTML** 是我们用来构建网页内容的标记语言。我们将内容分为段落、标题、表格、甚至嵌入的图像和视频等元素。
*   CSS 是一种由样式规则组成的语言，我们用它来设计 HTML 内容的样式。我们用它来设置字体、颜色、背景，我们还可以用 CSS 创建我们的整体页面布局(包括定位、flexbox 和网格)。
*   JavaScript 让你可以创造其他任何东西！如前所述，您可以添加复杂的动画、控制多媒体和添加动态内容。

让我们来看一个例子，看看它们是如何相互作用的！

这是我们的基本 HTML，由标题、段落和按钮组成:

```
<h1>I am a headline made with HTML</h1>
<p>And I am a simple text paragraph. The color of this text is styled with CSS. Click the button below to remove me through the power JavaScript.</p>
<button>Hide the text above</button>
```

让我们添加一些 CSS 来使它看起来更好:

```
body {
 font-family: sans-serif;
 text-align: center;
 padding: 3rem;
 font-size: 1.125rem;
 line-height: 1.5;
 transition: all 725ms ease-in-out;
}h1 {
 font-size: 2rem;
 font-weight: bolder;
 margin-bottom: 1rem;
}p {
 margin-bottom: 1rem;
 color: tomato;
}button {
 cursor: pointer;
 appearance: none;
 border-radius: 4px;
 font-size: 1.25rem;
 padding: 0.75rem 1rem;
 border: 1px solid navy;
 background-color: dodgerblue;
 color: white;
}
```

最后，我们可以添加一些 JavaScript 来实现动态行为:

```
document.querySelector("button").addEventListener("click", function () {
 document.querySelector("p").css("opacity", 0);
});
```

在这个例子中，点击按钮后，JavaScript 检测到点击事件，并通过添加`opacity: 0` CSS 规则删除段落。无需刷新页面！

这仅仅是开始..JavaScript 可以做更多的事情——在本系列中，我将深入探讨 JavaScript 的所有内容。

现在，让我们看看如何将 JavaScript 添加到我们的网页中，这样我们就可以开始自己玩了。

# 如何将 JavaScript 添加到网页

JavaScript 以类似于 CSS 的方式添加到 HTML 页面中。如果你熟悉 CSS，你可能记得它使用`<link>`元素连接到外部样式表，使用`<style>`元素定义 HTML 中的内部(内联)样式。

JavaScript 只需要一个元素就可以添加到 HTML 页面:元素`<script>`。

让我们看看如何使用`<script>`来内联**JavaScript 代码和连接到**外部** JavaScript 文件。**

# 如何添加内联 JavaScript

**嵌入在 HTML 文档**中的 JavaScript 代码被称为内联 JavaScript。要将它添加到您的网页中，只需将您的 JavaScript 代码放在一对`<script>`标签之间，就像这样:

```
<script>

  // Your JavaScript code

</script>
```

最好把它放在`</head>`标签之前。这使得你的脚本可以尽快下载而不会阻塞你的浏览器(为什么？参见我的下一篇文章“什么是异步和延迟？”).

所以上面的例子可以添加到网页中，就像这样:

```
<script>document.querySelector("button").addEventListener("click", function () {
 document.querySelector("p").css("opacity", 0);
});</script>
```

# 如何链接到外部 JavaScript

另一方面，HTML 文档可能引用包含 JavaScript 程序的单独文件，在这种情况下，它被称为**外部 JavaScript** 。

您可以创建任意多的独立 JavaScript 文件！只要你给他们**。js** 扩展名，你可以把它们链接到你的网页上。例如，文件`script.js`可以链接到您的页面，如下所示:

```
<script src="script.js" defer></script>
```

在`script.js`中，您有您的 JavaScipt 代码:

```
document.querySelector("button").addEventListener("click", function () {
 document.querySelector("p").css("opacity", 0);
});
```

这段代码与我们的内联示例工作方式相同，只是现在我们将 JavaScript 放在了一个外部文件中。这通常是一件好事，因为您的代码通常会更加有组织和可重用。此外，你的 HTML 将更容易阅读，没有大量的脚本混杂其中。

***你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*JavaScript 掌握完全指南*](https://gum.co/mastering-javascript) *带你从零到英雄！*

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# **包装完毕**

今天就到这里吧！我们已经从 JavaScript 的一点历史开始，包括它如何与 HTML 和 CSS 相关联，从而在本质上形成现代前端开发的基础！我们还研究了如何使用内部和外部方法将 JavaScript 添加到 HTML 网页中。

JavaScript 现在可能看起来有点吓人，但是不要担心——在本系列中，我们将通过简单的步骤学习每个核心概念。毫无疑问，在你的旅途中会有很多“啊哈”的时刻。随着你继续向前推进，随着你建立新的联系和增长知识，这些概念将开始变得更有意义！

我希望这篇文章对你有用！你可以在媒体上关注我。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在[我的个人博客](http://www.easeout.co)上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用的时候会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉