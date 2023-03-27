# CSS 基础:自定义属性

> 原文：<https://itnext.io/css-fundamentals-custom-properties-ededfc2d8d37?source=collection_archive---------5----------------------->

![](img/2e70845a70c22f1ddbe23b67ea417788.png)

**自定义属性**(通常称为 **CSS 变量**)是 CSS 的现代补充。如果你使用过 JavaScript、Python 等编程语言，你就会知道变量非常有用。

一个**变量**是一个引用一个值的名字，通过在 CSS 中使用变量，我们极大地减少了代码重复&创建了更容易维护的样式表。

🤓*想与 web dev 保持同步吗？*
🚀*想把最新消息直接发到你的收件箱里吗？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 定义 CSS 变量

我们使用一个以双连字符(`--variable-name`)开头的名称，后跟一个冒号和一个值(任何有效的 CSS 值)来定义变量:

```
:root {
  --main-color: green;
}
```

`:root`伪类是一个特殊的选择器，它将样式全局应用于 HTML 文档。稍后将详细介绍这一点！

可以使用`var()`访问该变量:

```
p {
  color: var(--main-color)
}
```

# 在元素中定义变量

我们可以在任何元素中定义 CSS 变量:

```
:root {
  --default-padding: 30px 30px 20px 20px;
}body {
  --main-bg-color: brown;
}p {
  --main-color: yellow;
}a:hover {
  --main-color: red;
}
```

# 变量作用域

每个变量只对定义它们的选择器的子元素可用，这就是“作用域”。

让我们在前面使用`:root`的例子的上下文中来看看这一点:

```
:root {
  --main-color: green;
}
```

`:root`是一个特殊的伪类，指的是树的根元素。

在 HTML 文档中，`:root`选择器引用了`html`元素。事实上，它比 html 有更高的特异性，所以它总是优先。

所以当一个变量被应用到`:root`时，它被认为具有“全局范围”&对所有元素都可用。

如果我们向一个`main`元素添加一个变量，它将只对`main`的子元素可用:

```
main {
  --main-color: yellow;
}
```

如果我们试图在这个元素之外使用它，它不会工作。

所以我们的两个变量如下:

```
:root {
  --main-color: green;
}main {
  --main-color: yellow;
}
```

我们的`--main-color`在文档中的任何地方都将是绿色的*，`main`及其任何子元素的**除外，它们将是黄色的*。****

# **额外提示..**

## **区分大小写**

**CSS 变量是区分大小写的！**

```
**:root {
 --color: green;
 --COLOR: yellow;
}**
```

**所以`--color`和`-COLOR`是两个独立的变量。**

## **在 HTML 中使用变量**

**变量可以直接在 HTML 中使用，就像这样:**

```
**<!--HTML-->
<html style="--size: 800px">body {
  max-width: var(--size)
}**
```

## **在变量中使用变量**

**一个变量可以在另一个变量中使用:**

```
**--base-red-color: #f00;
--background-gradient: linear-gradient(to top, var(--base-red-color), #222);**
```

## **在数学中使用变量**

**CSS 变量可以和 [calc()](https://www.easeout.co/blog/2020-05-08-css-calc) 函数一起使用:**

```
**--input-width: 500px;
max-width: calc(var(--input-width) / 2);**
```

## **在媒体查询中使用变量**

**我们可以用媒体查询使 CSS 变量有条件！**

**以下代码根据屏幕大小更改填充值:**

```
**:root {
	--padding: 25px 
}@media screen and (max-width: 750px) {
	--padding: 10px
}**
```

## **为 var()设置回退值**

**当您使用`var()`功能时，您可以定义第二个参数。如果找不到自定义属性，将使用该值:**

```
**width: var(--custom-width, 33%);**
```

# **包扎**

**我确信你可以看出 CSS 变量的使用将会使你的代码更具可读性和可维护性。**

**它们还显著地提高了跨较大代码库的变更的容易程度。如果你在一个单独的 CSS 文件中设置你所有的常量，无论何时你想做一个简单的改变，你都不会发现自己在数千行代码中跳跃！**

*****你准备好让你的 CSS 技能更上一层楼了吗？*** *现在就开始用我的新电子书:*[*CSS 指南:现代 CSS 完全指南*](https://gum.co/the-css-guide) *。获取从 Flexbox & Grid 等核心概念到动画、架构等更高级主题的最新信息！！***

**![](img/d3e2ee6adb6ffa2c189049cea5937e93.png)**

***现已上市！👉*[gum.co/the-css-guide](https://gum.co/the-css-guide)**

# **关于我的一点点..**

**嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在[我的个人博客](http://www.easeout.co)上找到。**

**我目前正在撰写我的[自由职业完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用时会通知你👍](https://easeout.eo.page/news)**

**感谢阅读🎉**