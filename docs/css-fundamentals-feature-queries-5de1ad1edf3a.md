# CSS 基础:功能查询

> 原文：<https://itnext.io/css-fundamentals-feature-queries-5de1ad1edf3a?source=collection_archive---------8----------------------->

![](img/ab6f2e026bec08fb8004235ac3c406bf.png)

CSS 中的特征查询用于精确的特征检测。现在所有现代浏览器都支持它们！

我们使用`@supports`关键字。例如:

```
@supports (height: 100vh) {
  .container {
    height: 100vh;
  }
}
```

这里我们检查是否支持`vh`单元。如果是，则相应地设置`height`值。

如你所见，它们的结构很像[媒体查询](https://www.easeout.co/blog/2020-06-04-css-media-queries)。

🤓*想了解最新的 web 开发吗？*🚀想要将最新的新闻直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

让我们扩展一下我们的例子:

```
.container {
  height: 100%;
}@supports (height: 100vh) {
  .container {
    height: 100vh;
  }
}
```

这里我们已经默认提供了一个回退，给容器一个 100%的`height`值。如果浏览器支持`vh`值，则应用 100vh。

老的浏览器会简单地忽略`@supports`块。

我们可以对任何 CSS 属性使用`@supports`来检查任何值。

我们还可以使用逻辑操作符`and`、`or`和`not`来构建更复杂的特性查询。

让我们用`and`来检查浏览器是否支持 [CSS 网格](https://www.easeout.co/blog/2020-05-29-the-css-grid-guide)和 [Flexbox](https://www.easeout.co/blog/2020-05-22-the-flexbox-guide) :

```
@supports (display: grid) and (display: flex) {
  /* ... */
}
```

另一个例子，询问是否支持 [CSS 网格](https://www.easeout.co/blog/2020-05-29-the-css-grid-guide):

```
.element {
  float: left;
  ...
}@supports (display: grid) {
  .element {
    float: none;
    display: grid;
    /* ... */
   }
}
```

简单有效！

***你准备好让你的 CSS 技能更上一层楼了吗？*** *现在就开始用我的新电子书:*[*《CSS 指南:现代 CSS 完全指南*](https://gum.co/the-css-guide) *。获取从 Flexbox & Grid 等核心概念到动画、架构等更高级主题的最新信息！！*

![](img/d3e2ee6adb6ffa2c189049cea5937e93.png)

*现已上市！👉*[gum.co/the-css-guide](https://gum.co/the-css-guide)

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在我的个人博客上找到。

我目前正在构建我的[自由职业完全指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用时会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉