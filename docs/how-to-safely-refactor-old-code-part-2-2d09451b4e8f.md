# 不要只是重构，要让它可维护

> 原文：<https://itnext.io/how-to-safely-refactor-old-code-part-2-2d09451b4e8f?source=collection_archive---------4----------------------->

![](img/2b838eaa995688f95f6baf91de97914f.png)

拉·罗宾逊公司在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

## 安全重构旧代码:第 2 部分

*在这个系列中，我们一步一步地回顾安全重构旧代码的概念设计模式。我编写本指南是为了适应任何类型的重构场景，示例项目是 Node.js with RxJS。*

*这是关于安全重构代码的 3 部分系列文章中的第二篇文章**:*

*   [*第 1 部分:防止现有系统的突破性变化*](https://medium.com/@Sawtaytoes/how-to-safely-refactor-old-code-part-1-a1a853263fec)
*   [*第二部分:不要只是重构，要让它可维护*](https://medium.com/@Sawtaytoes/how-to-safely-refactor-old-code-part-2-2d09451b4e8f)
*   [*第 3 部分:重构 AJAX 调用和观察对象*](https://medium.com/@Sawtaytoes/how-to-safely-refactor-old-code-part-3-f285a6200988)

# 我们的故事到此为止

在编写测试时，我们完全无法在不修改主要代码的情况下完成单元、集成甚至端到端(E2E)测试。相反，我们最终创建了 E2E 测试，在这个过程中需要人来验证功能。现在我们已经有了一些东西来确保我们的应用程序不被破坏，我们可以开始重构代码，这样它对于 npm 包来说实际上是可测试的和可展示的。

# 从哪里开始重构？

现在我们终于找到了检验破损的方法，我们从哪里开始呢？我们是重构还是重写？这篇文章是关于安全重构的，所以我们要避免重写。这意味着我们必须确保每一个微小的变化都让应用程序处于工作状态，但这也意味着我们可以逐个模块地重构。

*免责声明:虽然我提到我们编写测试是为了确保我们不会破坏功能，但在某个时候，我们将不得不升级 RxJS 版本，其中包括重大更改。除非我们更新导入，否则我们的单元测试将无法正常运行，所以到时候我们将不得不再次测试测试。*

在上一篇文章中，我们做了一点返工，只是为了确保我们可以完成一些可重复的测试。我们今天将继续这样做，但我想聪明地开始。

# 从图书馆开始

## 升级 RxJS

因为对我们代码库最大的改变将是 RxJS 升级，所以让我们完成它。你不会想到先处理大鱼，但这次升级将改变我们设计的一些基本原则。重构后不升级 RxJS，意味着我们有更多的地方需要更新，包括单元测试。

我们将利用 rxjs-compat 库来保留现有的 RxJS v5 代码以及新的 RxJS v6 代码。这样，我们不必一次改变所有的东西，我们可以分块来做。完成重构后，删除 rxjs-compat 非常重要。我可以预见更大的项目会安装这样的拐杖，然后在产品的整个生命周期中继续有两种不同的使用 RxJS 的方法。

最安全的方法是让我们一次做一个小的改变，而不是在最后一刻做大规模的架构改变。从经验来看，最后升级 RxJS 意味着它永远不会更新，因为你会有太多根深蒂固的旧方式要移动。当 RxJS 升级时，我们已经不得不更新我们的一个单元测试，那么为什么还要在那些都需要潜在突破性改变的基础上写一堆额外的单元测试呢？

另一方面，如果你的代码是以一种重写比重构更容易的方式编写的，你就不应该首先升级 RxJS。在我们的例子中，我们将获取一点代码，并将其移动到单独的单元可测试模块中。这极大地改变了它的结构，因此越早升级越重要。

简单地更新 rxjs 并添加 rxjs-compat 就足以让代码像以前一样工作:

```
yarn add rxjs rxjs-compat
```

这个小小的升级是迄今为止超级安全的改变。我们不知道性能、内存使用等；但是我要假设，如果你还没有这些东西的测试，那么它对你的项目来说是不值得的。

## 其他图书馆

我知道我的其他库有更新的版本，但我只打算更新那些没有重大变化的库。想继续用 RxJS 所以升级了。另一方面，我不想继续使用 moment.js，所以我会保持原样，直到我准备好。

在这个项目中，一些安全的升级是 nodemon、node-fetch 和我们已经看到的 rxjs。升级 app-module-path 和 moment.js 是不安全的，因为现有 API 可能会出现破损。如果项目维护得很好，我至少可以更新补丁版本(语义版本的最后一个小数:2.19。尤其是如果我目前的版本有已知的安全缺陷。你应该也能够升级次要版本(2。**19【T3 . 1】与大多数 npm 包，但如果你还没有单元测试，这是不安全的。**

## 避免更换库

在最初的重构中，您会希望避免更改库。这样，在用新代码替换旧代码之前，您可以确保旧代码仍然有效。

例如，我真的很想使用 date-fns 而不是 moment.js，但不是去所有使用 moment.js 的地方并交换它们，我应该首先重构那些点以允许针对 moment.js 的单元测试，然后在我得到工作单元测试之后交换到 date-fns。

# 是万圣节吗？

让我们看看之前的`isHalloween`例子:

我们需要给它一个日期，而不是硬编码`moment()`,这样我们就可以在任何一天对它进行单元测试。我们还需要把`currentYear`作为一个参数传入，或者把它改成一个函数，并给它传递一个日期。这两项更改都可以很容易地进行单元测试，以确保没有任何问题。

## 超级安全重构

一个简单的重构如下所示:

注意我们所做的一切是如何让`moment()`接受一个传入的日期的。这正是我们想要的。最小的变化，但我们可以进行单元测试。

我们的`isHalloween`函数以前没有参数，所以我们保留了这个功能，并且只在没有传递日期时默认为`new Date()`(没有参数调用`moment()`的老方法)。

## 重命名函数

作为最后一项更改，我想将这个函数从`isHalloween`重命名为`isDuringHalloweenNight`，因为这在语义上是正确的:

本着安全重构的思想，我用重命名的函数创建了新文件，并弃用了旧函数。这样，旧的功能仍然有效，但是现在我们可以通知用户新的文件，这样他们就可以知道不赞成使用的更改。

我建议使用弃用库，而不是在整个代码库中编写自定义的弃用代码。对于这个例子，我想保持简单，但实际上我已经编写了自己的弃用库来处理这些情况。

## 编写单元测试

我继续前进，引入了 [AVA](https://github.com/avajs/ava) 作为我们的单元测试库，并更新了我们导入文件的方式。

下面是我们的单元测试的样子:

我也可以对旧的`isHalloween`进行单元测试，但是最初的`isHalloween`是不可测试的，因为它没有参数。因为它仍然不接受参数，所以仍然无法测试。

## 更新库

现在我们的单元测试已经就绪，我升级到了 date-fns 并重写了`isDuringHalloweenNight`。由于功能没有改变，我们的单元测试也没有改变。这是结果:

基本的日期库完全是个人偏好，但是 date-fns 有一个函数编程文件夹，它允许你从它们的基本函数组合新的函数。在新的`isDuringHalloweenNight`中，我们正在编写一个新的日期函数。

如果我们不需要将`date`传递给`getYear`，它会看起来更清晰，但是这种重构并不意味着完全重写。

# 颜色索引

在`flashRandomLight`中，我们有很多不可测试的功能。为了使它们可测试，我们将做和`isDuringHalloweenNight`一样的事情；我们希望编写可以组合的基本函数，并使用我们在`flashRandomLight`中的函数在语义上组合这些通用基本函数。

我们来看看`getRandomColorSetIndex`:

这次重构与上一次不同。我们正在创建基本函数(就像我们对 date-fns 所做的那样),并在我们的旧函数中组合它。当进行单元测试时，对一般的基本函数进行单元测试是有意义的，因为组合函数不一定需要进行单元测试。

由于`getRandomColorSetIndex`仍然不接受任何参数，我们无法对其进行单元测试。这就是为什么我们最好创建一个新的通用函数来进行单元测试:`pluckRandomNumberFromRange`。

除了将`dependencies`作为第二个参数传递——将它暴露给任何人传入——我们还可以将它分离到一个单独的函数中，并将其组合成我们的随机数提取函数:

这种重构更加简洁，并且真正简化了我们正在做的事情。在这一点上，将来有人可以重构`pluckRandomNumberFromRange`来使用 lodash-fp 或 Ramda 中的通用范围函数；尽管这是 StackOverflow 上的一个[常见问题，但这些库中并不存在这种特定的功能。](https://stackoverflow.com/questions/4959975/generate-random-number-between-two-numbers-in-javascript)

我们仍然可以将它进一步重构为最流行的 StackOverflow 答案:

```
Math.floor(Math.random() * upperBound) + lowerBound
```

我不会为这个项目做那样的改变，但这绝对是可能的。只需通过`0`中的一个`lowerBound`，您现有的功能不会改变。

我编写了单元测试，并在更改 API 后对它们进行了调整。当我进行这些重构时，我会定期检查应用程序以确保它能运行，并检查原始的 E2E 测试规范以确保应用程序仍能正常运行。

我做了最后一个改变。我们需要一些方法来抛出一个错误，如果有人没有传入项目的数量或者他们的项目计数是`0`。在这种情况下返回`0`是没有意义的，因为它不是大小为`0`的列表中的有效索引。

要么我们返回`null`或者错误。我想在这次重构中尝试一些新的东西，所以我们返回一个错误:

## 排版错误

因为我修改了`createRangePicker`来抛出一个错误，所以我使用函数组合创建了一个抛出错误的机制。该 API 类似于 AVA 抛出错误的方式。这样，如果在我们的函数执行之前满足了某些错误标准，我们可以简单地抛出一个错误，而不必使函数的逻辑变得复杂。

因为`errorWrapper`是可组合的，你可以把一个放在另一个里，另一个放在另一个里。在 ECMAScript 的更高版本中，您甚至可以使用 pipeline 操作符在一次跳转中垂直读取它们。这将大大提高可读性。

# 最终产品

在一大堆工作重构和编写单元测试之后，我能够使这个项目变得可维护、可测试和可靠。我不担心它在万圣节突然不工作了，并且可以继续添加 E2E 测试来验证它做了我期望的事情，只是为了增加一点点安全性。

如果你想看 AJAX refactor 和其他，请查看第 3 部分 ！

# 更多阅读

如果你喜欢你所读的，你也应该看看我关于智能家居和函数式编程的其他文章:

*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)
*   [通过轻触按钮控制物联网设备](https://medium.com/flicblog/controlling-iot-devices-with-the-flic-of-a-button-1349c81bddef)
*   函数式编程的表情爱好者指南:第一部分