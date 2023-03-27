# 我在 2007 年用过 REACT！

> 原文：<https://itnext.io/i-used-react-in-2007-46e4723e7c03?source=collection_archive---------6----------------------->

![](img/f2a6cab4c127b4ab9481fbf2a7e5aa43.png)

照片由[迈克尔·阿丰索](https://unsplash.com/@mafonso?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

## 什么？？？？

与我在 2006 年维护的 WordPress 网站不同，这将是一次有趣的回顾，回顾我的第一个真正的编码项目，它使用了一个非常类似于 React、Redux 和 Redux-Observable 的系统。我在文章的每一部分都加入了一个 Lua to JavaScript Rosetta Stone，让我的大多数追随者更容易理解。

# 历史

你们中的一些人可能知道，我开始写代码是因为我进行了一生中最昂贵、最耗时的探险:开发独立游戏。游戏名字叫 [Pulsen](https://pulsengame.com/game) ，今天可以在 [Steam](https://store.steampowered.com/app/340960/Pulsen/) 上找到。

Pulsen 是基于一个名为 [StepMania](https://www.stepmania.com/) 的现有舞蹈游戏引擎构建的。该引擎创建于 1998 年，并随着时间的推移经历了相当多的重大修改；最初允许通过 INI 文件修改主题，然后一个街机游戏的分支允许通过 XML 文件修改，后来，另一个分支——最终成为主线 StepMania——将许多 UI 逻辑从 C++核心中分离出来，放在 Lua 中。

我在 2007-2008 年的某个时候加入了 XML 版本的修改现场，后来在 2009 年当我想到用 StepMania 引擎构建 Pulsen 时，我转向了 Lua 版本。

这篇文章的标题有点误导。它说明了我在 2007 年是如何使用 React 的，但我真正使用的是一个以类似 React 的方式编写的 XML 或 Lua 格式的自定义图形渲染器。在经历了多年的各种 web 技术之后，看到这个古老的项目与我们今天看到的 React 和 Redux 非常接近，仍然令人着迷。

# 演员是组件

作为 Lua to JS Rosetta Stone 的一部分，我将描述每个部分是如何关联的，以及它们是如何不同的。StepMania 有一个`Actor`的概念，它本质上是一个包含生命周期方法的 React 组件和一个为您预先编写的`render`函数。

通常你会在 React 中自己编写`render`,因为 HTML 元素的组合，但是这个游戏引擎只支持有限数量的元素类型，所以每种类型都有自己的 C++内部`render`函数。

## 行动框架

例如，有一个相当于 HTML `div`的`ActorFrame`，或者更具体地说，是一个反应`Fragment`。它所做的只是把其他演员圈起来，并允许你把他们作为一个团体来定位。

下面是`ActorFrame`的`render`在 React 中的样子:

```
render() {
  return this.props.children
}
```

很简单。

与`div`不同，它不会出现在 UI 中，所以它没有宽度、高度或深度。这就是为什么我把它和 React `Fragment`联系起来，但是它可能有一个 X，Y 和 Z 值，只要你设置了其中的任何一个。

## 其他演员

像众多的 HTML 元素一样，总共有 43 个参与者，例如:

*   不使用图像而显示文本的唯一方法。HTML 中没有对等的东西，因为浏览器处理任何元素中的文本呈现。
*   `Model`:用于加载 3D 模型。类似于`canvas`。
*   `Quad`:一个很像`div`的长方形。
*   `Sound`:像 HTML 中的`audio`一样管理音频文件的播放。
*   `Sprite`:常用类似 HTML 的`img`通过`LoadActor`函数，但也可以用于 sprite 动画或拉出类似 CSS sprites 的静态图片。

## 每个文件一个演员

就像 React 中通常只有一个组件导出(特别是默认导出)一样，StepMania 只允许每个文件有一个 actor。就像你必须在 React 中使用`Fragment`一样，你可以在 StepMania 中使用`ActorFrame`来解决这个问题。

# 反应，但是在 2007 年？

有了这些工具，你可以把一个类似的组件架构放在一起反应；事实上，它看起来更像 React 最初在 v0.13 中所做的，而不是你今天在 JSX 上看到的:

Lua 对象函数有一个类似于 JavaScript 的概念`this`，叫做`self`，但是由于 C++后端的原因，它必须被传递到这些特定的函数中。

我继续将相同的代码翻译成您在 React 中看到的内容:

当传递消息时，我认为这些与传递 Redux 动作是一样的，但是在这个翻译中，我将其屏蔽为渲染道具组件:`PlayerScore`。

另一件有趣的事情是，由于游戏使用位图文本，所以很容易对字体应用渐变，尽管这在 CSS 中很难解决。

这还不是最疯狂的事。旧的 XML 方法甚至更像 JSX:

一定会喜欢那份意大利面的。

当我第一次开始使用这个引擎时，这是我编写组件的风格。时间太久了，我不得不翻遍所有的旧文件，弄清楚我是如何完成这一半工作的。我很难记起任何一件事。

XML 有点不可靠，像 AngularJS 和 Knockout 一样，它在字符串中有 Lua 函数:/。不管怎样，即使是这种格式，你也可以做很多很酷的事情，令人惊讶的是，这与 React 如此相似。

# 生命周期方法

StepMania 提供了与 React 相同的生命周期方法。

有两种类型的生命周期方法可用，命令和消息命令，后者是整个应用程序的全局侦听器，而前者是特定于一个组件，有时是屏幕类型本身。

以下是一些常见的问题:

*   `InitCommand` : `constructor`
*   `BeginCommand` : `componentWillMount`
*   `OnCommand` : `componentDidMount`
*   `OffCommand` : `componentWillUnmount`

令人惊讶的是，没有对应的`render`。就像 HTML 元素如何从浏览器中呈现一样，actors 也是以同样的方式呈现的。

然后，就像 JS 类一样，您可以在 Lua 对象上调用方法。如果我在我的对象上放一个`AnimateCommand`，我可以在我的生命周期方法中使用`self:queuecommand('Animate')`调用它。从那时起，您可以通过一遍又一遍地调用命令本身来循环它，直到调用了`OffCommand`。

消息命令是很酷的命令。您不仅可以像调用其他命令一样调用它们，还可以广播一条消息，以相同的名称触发所有消息命令。

例如:

我能够广播一条调用所有`DifficultyLevelsChangedMessageCommand`函数的消息。如果那些道具看起来很可笑，它们不是；虽然，他们会受益于更好的命名。

我创建了一个主演员，它会听屏幕上的各种变化，然后通知所有其他演员他们是否应该停止补间或消失，等等。想象一下 Context API 在 React 中是如何工作的，在 React 中有一个生产者，然后一群消费者监听来自生产者的状态更新。

这就是我如何能够实现改变困难和歌曲的流畅性，同时在匹配时在玩家之间同步它们；尽管如此，代码的复杂性还是很高。

它类似于普通的 JS 事件和 Redux 动作。在 JS 事件中，它们可以在`window`上全局发生，也可以在单个 DOM 元素上发生:

您甚至可以将此视为 Redux 操作:

## 动画演员 vs 组件

我在 Pulsen 中处理动画的方式是，我会渲染额外的演员，并根据广播消息中可用的内容用补间动画隐藏或显示他们。

在 React 中有几种隐藏组件的方法。您可以不使用 JS 条件呈现它们，或者使用 CSS 隐藏它们。根据我使用 Pulsen 和 React 的经验，CSS 方法几乎总是更优越，因为它是动画 HTML 元素的唯一真正的方法。

除非在内存中保存数百个额外的 DOM 元素存在性能问题，否则我建议使用 CSS 方法，而不是有条件地呈现 React 组件。通过 CSS 隐藏和显示要比创建和删除新的 DOM 元素快得多。

# 2009 年响应式游戏设计

**Pulsen 反应灵敏**。在响应式网页设计出现之前，我记得看到人们在使用 StepMania 时将游戏窗口最大化，引擎会将所有东西都拉长。

我为此创建了一个解决方案，增加了一个名为`ScreenResolutionChangedMessageCommand`的生命周期函数，它会在游戏窗口调整大小时触发。

您通常会看到这样的情况:

我执行了来自`InitCommand`的`ScreenResolutionChangedMessageCommand`,所以它会以那种方式初始化，然后对任何分辨率变化做出反应。通常，这个功能只会修改演员在屏幕上的位置，但有时它也会改变缩放级别，因为早在 2009 年，上网本和非高清显示器是一件事，我希望游戏在所有显示器上都好看。

在 JavaScript 世界里，你会知道这是`window`上的 resize 事件:

想想在每个组件上都做这样的事情！如果你有一个`componentDidResize`生命周期方法，允许你相应地更新你的 CSS-in-JS，那就简单多了，这也是我可以利用的。

StepMania 中没有 CSS，所以你必须自己管理演员的所有信息；非常类似于 CSS-in-JS 和其他 helper 组件。在我创建的一个响应式组件库中，我监听(并抑制)`window` resize 事件，这样当它改变时我可以发出 Redux 动作，这样每个组件可以计算自己的大小并相应地改变其内部内容。

有了响应式 CSS 和`@media`查询，我不得不手动做的许多事情变得自动化了；尽管早在 2009 年，直到 2011 年，大多数浏览器才开始支持这些功能。一些浏览器，如 Firefox 和 Safari，早在 2009 年就开始支持`@media`，但它们直到 2012 年才被广泛使用，因为浏览器自动更新在当时还相对较新。

# 鼠标点击很难

为了判断鼠标是否单击了 HTML 元素，浏览器知道每个元素的宽度，并且父元素通常是所有子元素的宽度之和。它还知道它们的高度，然后通过作为该组件的父组件的所有组件使点击事件冒泡。

React 受益于浏览器对鼠标点击和键盘控制的处理，与 React 不同，我不得不在处理它们的每一个角色中亲自完成这项工作。

任何对鼠标点击有反应的演员都必须倾听`LeftClickMessageCommand`。然后它必须检查鼠标点击的 X 和 Y 位置与它在屏幕上的当前位置。知道一个演员是否被点击的唯一方法是获取它的 X 和 Y 坐标(和 Z，但我们不在这里谈论 3D)，计算出宽度和高度，然后查看鼠标点击是否在范围内。

事实上，给定的 X 和 Y 是相对于演员框架的，这使得事情变得更加复杂。获得真正的 X 和 Y 坐标的唯一方法是沿着演员帧链向上，同时获取父 X 和 Y 值。另一个复杂的问题是宽度和高度可能会受到缩放级别和父缩放级别的影响。

不需要太复杂，这是一个典型的例子，说明我如何判断一个`Quad`是否被点击:

我使用`self:GetZoomX() / 2`作为`ItemXOffset`，因为`GetX`和`GetY`从演员的中心返回坐标，如果我有一个 80px 宽的演员，我需要从那个 X 值看到+40px 和-40px。

为了节省代码，我在这个类上创建了一个函数，既可以在左键点击时运行，也可以在右键点击时运行，就像你为一个`onClick`向基于类的 React 组件添加一个函数一样。

# 让文本流动

在这个特殊的引擎中，文本并不像浏览器那样流动，而是在一个 sprite 中的一组光栅图像，它们彼此相邻。当文本超过特定的行长度时，我必须编写一些代码来处理换行。因为所有字符的大小不同，所以我通过获取当前元素和所有父元素的累积宽度来计算这些值。HTML 为你做的事情太多了。

因此，当我不得不做如何播放屏幕时，这是一件痛苦的事情，从来没有真正正确地工作过，但后来我找到了一种在引擎中做这件事的方法，前提是我有正确的计算:

```
self:wrapwidthpixels(
  (Item.ZoomX / Item.FontSizeBody)
  - (Item.XYPadding * 2)
)
```

是啊，很难看，但很有效。当你在使用 HTML 时，很难体会到它有多好。人们经常抱怨 HTML 和 CSS 的文本流以及它可以有多复杂，但老实说？和 StepMania 比起来超级容易。

想象一切都是`position: absolute`。你现在必须找出一种方法，使文本在到达边界时流动，找出如何放置所有内容，使其适合在一起，并确保所有内容都像 flexbox 或 CSS Grid 一样灵活。祝你好运！

# 使用库(不是一个选项)

如果你在想“可能已经有一个库来做这件事了，你为什么要自己做呢？”，你错了。没有助手库。这个引擎几乎拥有这个社区的所有东西，而且是一个 14-26 岁的小众社区。当时，GitHub 和 BitBucket 是存在的，但并不是真正的东西。那时我运行我自己的私人 Git 服务器。

我写的所有疯狂的代码都是别人在引擎中没有做过的。想想 2015 年之前的反应。没有多余的，所以你必须想出你自己的国家管理计划。和 Node.js，于 2009 年 5 月 27 日发布。我记得在一家初创公司尝试设置它来代替 PHP，但 Express.js 甚至不是一个东西！我不得不自己从头开始组装网络服务器。

我就是这个意思。在这个庞大开发人员社区接管之前，您必须自己解决所有问题。我们决定采用 React 这种流行的、有据可查的方式来处理状态，这真是太棒了，但是多年前对我来说简直就是地狱，尤其是因为这是我的第一个代码项目。即使在今天，我也不像了解 JavaScript 那样真正了解 Lua、C++或 PHP。JavaScript 成为我的首选有点疯狂，但我认为这是因为社区中的大量开发人员对事情应该如何发展有不同的想法。

# 结论

自从我开始研究 Pulsen 以来，大约有 10 年了，我感觉我已经围绕 React、Redux 和 Redux-Observable 完成了一个完整的循环。令人惊讶的是，这些完全不同的技术如此相似，最终都用组件模型解决了相同的问题。

# 更多阅读

如果你对我的 Lua 之旅感兴趣，请查看:

[为什么我在 Lua 里写 Redux](/pointlessly-writing-redux-in-lua-b64894e8ede5)

我还对 React、Redux 和 Redux-Observable 有很多其他独特的看法:

*   [使用 React 路由器的异步 React&悬念](/async-react-using-react-router-suspense-a86ade1176dc?source=your_stories_page---------------------------)
*   [在 React 组件中使用 Redux 还原剂](/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [无冗余可观察](/redux-observable-without-redux-ff4a2b5a4b39)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)