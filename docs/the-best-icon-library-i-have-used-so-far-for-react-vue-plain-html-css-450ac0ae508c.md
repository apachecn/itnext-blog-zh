# 迄今为止我为 React，Vue & Plain HTML/CSS 使用的最好的图标库

> 原文：<https://itnext.io/the-best-icon-library-i-have-used-so-far-for-react-vue-plain-html-css-450ac0ae508c?source=collection_archive---------2----------------------->

## 磷光图标——灵活的图标库

![](img/131a30dd2781a04de95aebb8fcf50c1e.png)

# 什么是磷光图标？

磷光图标是一个很棒的图标库，有 **588 个**图标和 **6 个**图标变体，总共为您提供**3529 个**图标！更好的是，它是 100%免费的。

> Phosphor 是一个灵活的图标家族，用于界面、图表、演示等。

# 图标变体

磷光图标有六种变化

*   薄的
*   光
*   规则的
*   大胆的
*   充满
*   两色网版的一种

虽然大多数图标库会让你为不同的变化付费，但磷光图标是免费提供的。

## 这些变化有什么作用？

瘦，轻，常规和粗体都是图标的笔画粗细，这些都是不言自明的，但你可能会有一些问题的两种变化是填充和双色调。

## 充满

填充变化只是将这些轮廓图标变成实心图标

![](img/b101bc98c0698c6a20dc403178e92fd3.png)

填充变化示例

## 两色网版的一种

双色调变化几乎是常规变化和填充变化的组合。笔画颜色将是您定义的图标颜色，而填充颜色只有 20%的不透明度。

![](img/5b5976d794e4dc8f2036ab96606beb9e.png)

双色调变化示例

# React 的磷光图标

## 装置

您可以使用`npm`或`yarn`安装荧光粉进行反应

```
npm install --save phosphor-react
```

或者

```
yarn add phosphor-react
```

## 如何使用荧光粉进行反应

我们需要做的第一件事是导入我们想要使用的图标

现在我们可以开始在我们的应用程序中使用它们了

# 了解 React 荧光粉成分

## 大小

`size`属性将接受任意值的`number`或`string`类型，`string`值可以使用任意 CSS 有效单元，如果使用数字类型，不要忘记`v-bind` ( `:`)。

## 颜色

`color`属性只能接受一种类型的`string`，可以是任何有效的 CSS 颜色，包括`currentColor`。

## 重量

`weight`属性只能采用等于六个变量`"thin"`、`“light”`等之一的`string`类型。如果不包括该属性，则默认为`"regular"`。

## 反映

还有一个接受类型为`boolean`的`mirrored`属性，如果为真，它将翻转图标。

# Vue 的磷光图标

## 装置

您可以使用`npm`或`yarn`为 Vue 安装荧光粉

```
npm install --save phosphor-vue
```

或者

```
yarn add phosphor-vue
```

## Vue 如何使用荧光粉

首先，我们必须将我们想要的图标导入到组件的`<script>`标签中。

现在，我们可以在`<template>`标签中使用磷光图标，如下所示:

# 了解 Vue 荧光粉成分

## 大小

`size`属性将取一个类型为`number`或`string`的任意值，`string`的值可以使用任何 CSS 有效单元别忘了`v-bind` ( `:`)如果使用一个类型的数字。

## 颜色

`color`属性只能接受一种类型的`string`，可以是任何有效的 CSS 颜色，包括`currentColor`。

## 重量

`weight`属性只能采用等于六个变量`"thin"`、`“light”`等之一的`string`类型。如果不包括该属性，则默认为`"regular"`。

## 反映

还有一个接受类型为`boolean`的`mirrored`属性，如果为真，它将翻转图标。

# HTML/CSS 的磷光图标

## 装置

要为一个普通的 HTML & CSS 网站安装 Phosphor，你必须像这样在`<head>`标签中添加 Phosphor 脚本

## 如何为普通 HTML 和 CSS 使用荧光粉

所有荧光图标必须使用`<i>`标签。要让一个图标出现，你必须向`<i>`添加一种磷光体，就像这样

```
<i class="ph-rocket"></i>
```

所有图标类都以这种方式作为前缀

```
ph-{icon name}-{variation/weight}
```

如果没有指定变化/重量，则默认为常规变化

## 式样

你可以自定义`color`、`font-size`、&图标的其他属性，覆盖那里默认的 CSS

```
<i class="ph-rocket custom-icon"></i>
```

磷光体带有一些默认样式，请参见 [*文档*](https://github.com/phosphor-icons/phosphor-icons#styling) 了解更多信息

# Figma 的磷光图标

是的，Phosphor 有三个图标可用于 Figma。您可以通过下载 [*组件库*](https://www.figma.com/file/xMCDSp5g0g7Fw8aMyAdVVr/Phosphor-Icons?node-id=0%3A1) 或安装 [*磷光体 Figma 插件*](https://www.figma.com/community/plugin/898620911119764089/Phosphor-Icons) 来使用 Figma 中的图标，我用它来创建本文的所有图像。

## 磷光图标的创作者之一——海伦娜·张的文章

[](https://medium.com/@minoraxis/phosphor-icons-is-live-fcf017ec2449) [## 磷光图标是活的！

### 6 种重量的灵活图标系列

medium.com](https://medium.com/@minoraxis/phosphor-icons-is-live-fcf017ec2449) 

## 链接

*   [*PhosphorIcons.com*](https://phosphoricons.com/)
*   [*荧光粉起反应*](https://github.com/phosphor-icons/phosphor-react)
*   [*磷 Vue*](https://github.com/phosphor-icons/phosphor-vue)
*   [*磷光 HTML/CSS*](https://github.com/phosphor-icons/phosphor-icons)