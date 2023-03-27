# 庆祝 UX 的微互动 Angular 的帆布纸屑

> 原文：<https://itnext.io/a-ux-micro-interaction-to-celebrate-canvas-confetti-for-angular-18bdadffeec7?source=collection_archive---------8----------------------->

## 如果你喜欢 UX 和 UI 设计中的小东西(和纸屑)，那你来对地方了。

![](img/ca2ab113eeff533721ec229feaac6885.png)

一旦你发射了你的五彩纸屑炮，你的网站的现场图像。

# 这都是些小事

**用户体验中的小事**(UX)&用户界面(UI)设计很重要。它们是**让你与众不同的地方**。他们创造**性格**和**性格**。他们称赞设计良好的用户界面，并巩固了主题。

如果你处于这样的开发阶段，你可以通过**微交互**关注**更详细的 UX** ，为你的用户提供额外的**多巴胺**，同时展示**令人难以置信的 web 开发技能**(或者只是导入一个为你做令人难以置信的事情的库)，那么像这样实现**微交互**可以给你的用户留下**持久的印象**。这可能是他们在竞争的海洋中记住你的原因。

> 我记得当我第一次在 Medium 上拍一个故事时，看到了五彩纸屑。受到这样的影响真是可笑。但是我想“我该怎么做”。

> 我会给你看的。

**React** 恰好有一个针对这些微交互的库，称为用户‘奖励’。它叫做`react-rewards`，支持开箱即用的**五彩纸屑**、**表情符号**或**孟菲斯**。我以为就这样了。我在 Angular 中没有任何类似的东西可以轻松使用(我最初是通过搜索 clap 五彩纸屑的原因找到这个库的)。

[](https://github.com/thedevelobear/react-rewards) [## 减速熊/反应-奖励

### 这个包是用 React-Pose，react-dom-confetti 和 Lottie-web 构建的。我为什么要用那个？阅读我的博客…

github.com](https://github.com/thedevelobear/react-rewards) 

**Angular** 没有这些小‘奖励’的库。但是 npm 包`canvas-confetti`是魔法的源头。所以我们将安装这个包，并在一个**角形组件**中使用它来实现我们的五彩纸屑功能。

[](https://www.npmjs.com/package/canvas-confetti) [## 帆布五彩纸屑

### catdad.github.io/canvas-confetti 你可以从 NPM 安装这个模块作为一个组件:然后你可以…

www.npmjs.com](https://www.npmjs.com/package/canvas-confetti) ![](img/1be3a0e322be16557141aaf838da5b35.png)

一张我们将会在 NgOven 中烹制和烘焙的五彩纸屑的 GIF 图

# 让我们一起庆祝棱角分明

## 安装帆布-五彩纸屑

首先从节点包管理器安装`canvas-confetti`包。

```
**$** npm install canvas-confetti
```

## 创建角度组件—庆祝组件

我将使用 **Angular CLI、**创建一个组件，它将与**Angular****Service**和 **spray 全屏五彩纸屑**进行交互，以响应**庆祝事件**。

```
**$** ng g c modules/shared/celebrate
```

这也将**将**celebrate component 类导入 SharedModule(模块结构不重要，只是一个例子)。

> 您还会注意到，我默认使用`ChangeDetectionStrategy.OnPush`。
> 
> 我这样做是出于性能原因；以避免触发太多不必要的变更检测运行。
> 
> 因为你可以手动地让 Angular 知道什么时候期望用`this.changeDetectorRef.detectChanges();`改变(在将`private changeDetectorRef: ChangeDetectorRef`注入构造函数之后)。

您的 **CelebrateComponent** 类将像这样开始。

开始庆祝. component.ts

## 声明实例字段

我们希望一些实例字段**存储**重用**的**canvas 元素，加上**动态导入的 canvas-confetti 库**，此外还创建一些观察对象来监听事件。

**实质上**，我们想:

*   侦听来自 **CelebrateService** (我们将创建的)的 **RXJS Observable** ( `celebrate$`),这样我们就可以通过编程触发 confetti 来响应用户事件。
*   **被通知**当组件**被破坏，**通过**主题**提供该事件通知。我们将让该主题负责确保该组件(`destroy`)的可观察订阅中没有**内存泄漏**。

> 为此，将创建`destroy`主题。然后我们可以在`NgOnDestroy`生命周期中调用`destroy.next()`。主体的可观察对象(`destroy`)将被创造(`destroy$`)，为我们提供了一种倾听被破坏事件的方式。

*   **监听**已创建的`destroy$`可观测值，通过提供它作为 RXJS 运算符`takeUntil`的值，以便我们最终取消订阅`celebrate$`可观测值。
*   **在实例变量`confettiLib`中存储**canvas-confetti 库，这样我们**就不会导入它两次。**我们将使用 ES6 动态导入`import('canvas-confetti')`，随后存储该值。

> 这也确保了除非使用它，否则我们不必导入它，因为它将很少被使用，除非你使用它与一个按钮或类似的频繁事件。

*   **存储**confetti canvas 函数(confetti canvas ),这样我们可以重复使用它，并且只创建一次。

> 使用 ES6 动态导入允许 Angular 将库识别为自己的块，从而减小 main.js 包的大小。

![](img/f9a3c1a74de60e7bbfe63dea4c8f7def.png)

如果您的应用程序不支持 ES6 代码，您可以用动态导入删除代码，并将`import * as canvasConfetti from 'canvas-confetti'`放在组件文件的顶部。

设置 pressure . component . ts

## 创建庆祝服务

在我们更深入地研究**庆祝组件**之前，让我们创建**庆祝服务**，对于本例，它将保持**简单**。

我们只需要提供一种方式来响应用户行为，从应用程序的任何地方切换 T21 庆祝。这种用户行为的一些例子可能是:

*   在用户**注册**时。
*   **点击**按钮(喜欢、分享、创建)。
*   **已完成的**个人资料(如果您提供一份他们完成的清单)。
*   教程/演示/演练**完成**。

**所以我们需要使用 Angular CLI 来创建服务**:

```
**$** ng g s services/celebrate
```

这将在`services/celebrate.service.ts`中**创建**服务。

在 CelebrateService 中，我们创建了一个由服务专用的`celebrate` **RXJS 主题**，以及一个能够让我们监听由那个**主题控制的**排放**的`celebrate$`可观察对象。**

`celebrate$`可观察对象将在**庆祝组件**中订阅，因此它可以对庆祝事件做出反应。

我们还定义了一个函数`startCelebrate()`，当你想要**触发**一个庆典时，这个函数就会被调用。

创建庆功网站

**就是这样**为**庆祝服务**。现在回到组件…

## 通过依赖注入提供依赖

我们需要通过构造函数来解析**依赖关系**。我将注入 **PLATFORM_ID** 令牌，我们用它来检查这个组件**没有在 **SSR 环境**中初始化**(因为我们没有访问 HTMLCanvasElement 的权限)。

我们还需要使用主机元素( **ElementRef** )用 **Renderer2** 动态渲染**canvas 元素，另外我们需要 **CelebrateService** 依赖项(我们刚刚创建的)，以绑定到前面讨论的`celebrate$`可观察对象。**

属国

我们检查我们是客户端，如果使用 **SSR** 和`isPlatformBrowser`，订阅`celebrate$`观察值，同时用`takeUntil`处理从`subscribe()`调用返回的订阅对象。

## 主要的“庆祝”功能

我把这个函数留在最后，因为这是魔法发生的地方。

完整的庆祝. component.ts 文件

使用`celebrate`功能，我们可以:

*   **检查**画布纸屑库不存在。
*   使用 Renderer2 将画布附加到主机元素(ElementRef)上。
*   **创建**五彩纸屑画布，如果尚未创建，提供我们创建的画布。
*   创建一个**间隔，**以便每隔 200 毫秒调用一次五彩纸屑，使用`setInterval`。
*   设置一个**计时器**，这样我们就可以用`clearInterval()`最终摧毁纸屑。
*   **点燃纸屑**🎉 🎊

![](img/1be3a0e322be16557141aaf838da5b35.png)

繁荣

## 全屏幕

为了用你自己的画布元素让画布**全屏**，你需要**为你的组件**设置样式。画布定位需要**固定**，并且是**全宽全高**。

你可以在`celebrate.component.scss`里面这样做。一旦五彩纸屑函数**完成**，画布将**销毁**并且**从 DOM 中移除**，因此使用 fixed is 上下文不会导致任何问题。

## 五彩纸屑结构

你可以配置**粒子计数**，开始**速度**，**扩散**，**原点**(它将从哪里开火)，**形状**(可以给你一个**雪花效果**带‘圈’)**加更多！**查看下面的**演示**页面，看看有什么可能！

 [## 🎊

### 但是如果你对五彩纸屑的疯狂速射感兴趣，还有什么比向每个人展示你自己更好的用途呢…

www.kirilv.com](https://www.kirilv.com/canvas-confetti/) 

完成了🚀 — — — — — — — — — — — — —

## 我错过了什么，你尽管指出来。任何问题，我都会尽力帮忙！