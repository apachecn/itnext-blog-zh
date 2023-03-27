# 在 React 中创建一个简单的灯箱

> 原文：<https://itnext.io/create-a-simple-lightbox-in-react-2ef5facd1148?source=collection_archive---------0----------------------->

## 循序渐进的反应。JS 操作方法

我的一个客户要求为他们的网站提供一个简单的灯箱。这个网站是用 React+Gatsby 制作的，使用起来非常好，但是，可用的 LightBox 解决方案通常不太兼容。

![](img/a3921ffb181fd2c25ab07944d726ea7a.png)

[埃里克·帕克](https://unsplash.com/@deuxdoom?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

# 为什么不使用现有的灯箱？

React 在 NPM 上有很多 LightBox 实现。这种实现的最大问题是，它们经常在组件周围添加额外的包装器和 CSS。这反过来会导致您的网站上不必要的组件和布局变化，这可能很难缓解。

从我试用大多数可用的 React LightBox 包的经验来看，它们中的大多数要么没有足够的可定制性，要么它们额外的 CSS 破坏了你的网站。最后，从零开始创建一个新的 LightBox 更容易，它不会干扰你的网站的内部风格。

如果已经存在，我通常不赞成从头创建。不过，在 React LightBoxes 的情况下，这是必要的，因为很难调整现有的 light box 来满足您特定的网站需求。

# 创建灯箱组件

在 React 中创建一个简单的 LightBox 非常容易。我们只需要以下项目来创建一个灯箱。

*   一个组件，我们将使其可点击
*   打开灯箱的事件
*   一个固定的 div，一旦 LightBox 打开，它将包含我们的全尺寸图像

让我们开始吧。

## 创建 React 组件

我们需要一个简单的 React 组件，我们可以用大多数 ide 的代码片段生成一个无状态组件。根据你的编辑器，`rsc`或者`rc`应该可以。

现在，不要把 div 强加给这个组件的用户，让我们通过一个 prop 来配置它。我们还将让用户传入孩子、图像源和 alt 标签。孩子就是可以点击来触发灯箱的东西。

(!)确保大写`Wrapper`道具，以便 React 将其视为动态组件。

## 保持打开状态

我们需要保持 LightBox 的打开状态，以控制何时打开或关闭它。我们可以通过引入一个新的状态变量`isOpen`来做到这一点。

一旦我们有了可用的状态，我们就可以设置一些处理程序来处理发生在我们的`Wrapper`上的`onClick`事件。这意味着点击任何`children`元素都会导致灯箱打开。我们还将在`Wrapper`中显示`children`。

## 创建灯箱元素

我们现在只缺少拼图的最后一块，实际的具有全尺寸图像的 div。只有当灯箱打开时，才应该显示这个 div。让我们就这样创造吧！

如果您正在使用`LightBox`组件，您应该已经看到此时发生了一些事情！您可能会注意到 div 出现了，但是我们还没有办法关闭它。让我们创建另一个处理程序，它将在按下 Lightbox 的 div 元素时关闭 Lightbox。

# 灯箱样式

我们有一个工作的，基本的灯箱！这就足够了，这取决于你的网站的需求。剩下的，就是给它添加一些风格。为了教程的缘故，我们将使用一些内联样式，但请使用样式组件，或 CSS 模块代替！

这种情况下，我们的`div`应该是`position:fixed`。绝对定位在这里不起作用，因为我们希望 div 相对于浏览器窗口绝对定位，而不是相对于它的父窗口。您可以在以下 W3 文章中阅读更多关于`fixed`定位的内容。

[](https://www.w3.org/wiki/CSS_absolute_and_fixed_positioning?source=post_page---------------------------) [## CSS 绝对和固定定位

### 现在是时候将注意力转向第二对位置属性值了——绝对的和固定的。第一对…

www.w3.org](https://www.w3.org/wiki/CSS_absolute_and_fixed_positioning?source=post_page---------------------------) 

随着样式的增加，我们应该有类似下面的东西。

# 最终演示

完整的代码可以在下面的代码栏中找到。试一试，看看它是如何工作的，以及它是多么容易设置！

从现在开始，你可以很容易地用更多需要的功能来调整它，并拥有一个可重用的 Lightbox。一个可以真正用于不同项目的灯箱！

# 进一步阅读

如果你想尝试使用现有的 npm 软件包，Bri Turner 有一篇很好的文章解释了如何去做。这篇文章解释了如何使用`react-image-lightbox`包。

[](https://medium.com/swlh/how-to-use-react-image-lightbox-b29b55f3da62) [## 如何使用 React 图像灯箱

### 如何用 React Image Lightbox 创建快速响应的 lightbox

medium.com](https://medium.com/swlh/how-to-use-react-image-lightbox-b29b55f3da62) 

[订阅我的媒体](https://kevinvr.medium.com/membership)到**解锁** **所有** **文章**。通过使用我的链接订阅，你是支持我的工作，没有额外的费用。你会得到我永远的感激。