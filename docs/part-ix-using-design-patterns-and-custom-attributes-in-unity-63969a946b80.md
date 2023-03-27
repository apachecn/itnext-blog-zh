# (第九部分。)在 Unity 中使用设计模式和定制属性

> 原文：<https://itnext.io/part-ix-using-design-patterns-and-custom-attributes-in-unity-63969a946b80?source=collection_archive---------2----------------------->

![](img/700bfac729d32753ed43dc023aae2eae.png)

当你在做一个长期项目时，保持事物整洁、有序和可扩展是至关重要的。让我们来看看有助于此的 3 种设计模式——观察者、状态和命令模式。就属性而言，它将是标题和范围修饰符。

# 设计模式

## 观察者模式

> 观察者模式是一种软件设计模式，在这种模式中，一个名为 subject 的对象维护一个名为 observer 的依赖者列表，并自动通知它们任何状态变化，通常是通过调用它们的方法之一。—维基百科

首先，我创建一个模型场景。观察者模式由主体和观察者组成。

举个例子，我有 3 个观察者——刚体*汽车*，一旦玩家走过主题，*立方体*触发，它们将向任一方向移动。

![](img/4e3f406f0aa71eaebe3900f5533727e3.png)

现在我将创建一个观察者和主题类。Observer 是抽象接口，因为许多对象必须自己实现它的功能。

![](img/0b0b11daabeca614f77df6baa7331626.png)

Subject 拥有一个关于它最终将通知的所有观察者的列表。

![](img/ee60ee2839b4469a45306c5d5f04fe4c.png)

一切都由游戏控制器控制。

![](img/844ba1bf05af71d5bced32bab2cf8c39.png)

我需要汽车类:

![](img/5dc6dd28d49e3985e544c33850d4cfd2.png)

我需要关心的事:

![](img/d37c4d3065adbaac5088bf8660a662bb.png)

我需要做的最后一件事是将汽车对象分配给游戏控制器(触发器立方体持有它):

![](img/9fa91cda37b98bc172cc7940209e350e.png)

现在，当我输入触发器时，汽车将按照我分配给它们的移动函数指定的方式移动(向左、向上、向后移动):

![](img/f0e67396d8a4b959e8a098547c9d08b7.png)

如果你刚刚开始学习如何开发游戏，你可以关注我的 udemy 初学者 Unity 开发课程:

[](https://www.udemy.com/course/make-a-3d-game-in-unity-2020-from-scratch-with-free-assets/?referralCode=8B96F6C67527AEEA39D9) [## 完整指南:Unity 2020 中的动作恐怖 3D 游戏

### 大家好，我叫 Jan Jileč ek，是一名拥有计算机科学硕士学位的专业游戏开发人员，我…

www.udemy.com](https://www.udemy.com/course/make-a-3d-game-in-unity-2020-from-scratch-with-free-assets/?referralCode=8B96F6C67527AEEA39D9) 

## 命令模式

> 命令模式是一种行为设计模式，其中使用一个对象来封装执行一个动作或在以后触发一个事件所需的所有信息。这些信息包括方法名称、拥有该方法的对象以及方法参数的值。—维基百科

对于像控制键绑定这样的事情非常有用。如果不使用*命令模式*，控制键的实现将如下所示:

![](img/8d80f8989b9c0e58b78db84abfacb1d1.png)

如果你有 20 个键分配给不同的游戏功能，而你需要重新绑定其中的一个，该怎么办？你需要重写整本书。现在是利用**命令模式的时候了。**

首先定义一个抽象接口:

![](img/a4170626a94fc26cbe0bbc592d275ef1.png)

准备函数类:

![](img/352e494a6e191a74c6916e4d0757c8d2.png)

现在，只要您需要为一个键使用不同的函数，您就可以调用 Execute 函数并根据需要交换该函数。

![](img/7c055c6323c154e823b380524f13cd27.png)

## 状态模式

> 状态模式是一种行为软件设计模式，它允许对象在其内部状态改变时改变其行为。这种模式接近于有限状态机的概念。状态模式可以解释为策略模式，它能够通过调用模式接口中定义的方法来切换策略。—维基百科

如果你有一个想要控制多个状态的对象，你将需要一个*有限状态机*来区分所有可能发生的情况。

让我们先创建一个敌人接口类。

![](img/14a534367a3d7ef37e1dae9ba3211b22.png)

然后是阿尔法和贝塔的敌人:

![](img/ef901f6702ce78ed0c712638cb4e582d.png)![](img/f950b4898364a6d2c29120e4e4384ef9.png)

和一个控制一切的游戏控制器。

![](img/a0f46eb9a48243f8f88522b7dd8fb450.png)

现在状态将在敌人接口的继承成员中保持一致，并且将来很容易对状态机进行更改。

# Unity C#属性和代码组织

![](img/8b4e73340a1a2ef971ab9d8c8f2ba903.png)

在 Unity 中，您肯定已经尝试过这样的事情:

![](img/aad2640fa708a32f5609b71f8b7e00da.png)

*范围属性*在相关对象/脚本的 Unity 检查器中创建一个滑动条，标题会整齐地划分变量。

![](img/18089a78084cc573ca14c6ab7066cfca.png)

但是属性比这个有用得多。

另一个很好的用途是，您可以检索应用该属性的字段和类的列表。这意味着你不需要直接引用对象！

这在准备游戏的本地化和翻译时非常有用，因为你可能有成百上千的字符串需要翻译。

![](img/421efdd62d8540ef1141ceeec60fda06.png)

然后你可以这样使用它:

![](img/532fc0e8e295874463550aa339d3f02b.png)

稍后，当您需要添加翻译时，您可以简单地搜索所有标有 *ModularText* 属性的对象，并设置翻译后的文本:

![](img/9878276aac6fa9ea4d2cb9cb76dc1228.png)

正如你所想象的，这个方法是编写任何 UI 元素的天赐之物，需要很多额外的功能(工具提示等)。).

我在我的 [github](https://github.com/janjilecek/unity_tutorial/commit/7a0f090d8d22c0ab4a45582753684a3446a18e60) 上提供了这篇文章的代码。

也可以看看我的 gamedev 系列的前几部分！:)

[](/setting-up-high-definition-render-pipeline-in-unity-68ed41062371) [## 在 Unity 中设置高清渲染管道

### 我们将看看全球和本地 HDRP 卷，并修补后处理和雾。

itnext.io](/setting-up-high-definition-render-pipeline-in-unity-68ed41062371) [](/limbo-style-menu-in-unity-4fe3fdee3420) [## Unity 中的 Limbo 风格菜单

### 在本教程中，我将复制 2010 年著名游戏《地狱边缘》的菜单风格:

itnext.io](/limbo-style-menu-in-unity-4fe3fdee3420) 

明天的部分将是系列的最后一部分，我将完成我们一直在创造的项目的游戏设计和视觉风格。