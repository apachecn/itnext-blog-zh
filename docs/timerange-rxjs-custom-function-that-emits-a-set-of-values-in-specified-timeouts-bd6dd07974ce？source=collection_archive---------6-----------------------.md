# ' timeRange' — RxJS 自定义函数，在指定的超时时间内发出一组值

> 原文：<https://itnext.io/timerange-rxjs-custom-function-that-emits-a-set-of-values-in-specified-timeouts-bd6dd07974ce?source=collection_archive---------6----------------------->

*为什么，如何，在哪里对于喜欢 RxJS 的人:-)*

![](img/69d2257aea74f930b95c5d201d1286e4.png)

定制 RxJS(图片由[鬼祟肘](https://unsplash.com/@sneakyelbow?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)拍摄)

```
**Prerequisites: I assume you know RxJS and have some practice with it. If you are new to it or want to learn some advanced topics - watch my video-course "**[**Hands-on RxJS for Web development**](https://www.udemy.com/course/hands-on-rxjs-for-web-development/)**" on Udemy.**
```

# 为什么

在我的开发实践中，我经常会遇到必须在指定的超时时间内发出一些值的情况。

例如:

1.  要显示然后隐藏一些闪屏消息:

![](img/e79262c21343f56427ee40735f593879.png)

管理启动画面消息显示/隐藏超时

2.做一些好看的动画

![](img/6143137f7658be47f2119ca9f8bf20ac.png)

管理外观超时

3.或者甚至只是发出单元测试的值(是的，我知道[弹珠](https://github.com/ReactiveX/rxjs/blob/master/docs_app/content/guide/testing/marble-testing.md)，但是有时我只是想检查最终值，句号:)

![](img/8e154883b510571d5cda1ff649e224cf.png)

带有去抖功能的自动完成的简单代码——要求提供带有要测试的特定时序的值

那么如何实现呢？好，让我们回顾一些场景:

# 如何实施？

## 丑陋的方式

好的，最简单的方法——运行许多 *setTimeout 的*，就像这样:

![](img/3ac0c39da20b6d161dcdc97aa38f19ef.png)

好吧，还不错，但如果我想在不同的延迟中显示更多的元素呢？

![](img/5fbf95c6f8001c83f684690cbb16cda6.png)

看起来又老又丑。我们能做得更好吗？

## 丑陋的方式，第 2 部分

现在让我们使用 RxJS 并尝试实现它:

![](img/c35af5d060a42497f85e5571b7aae148.png)

该代码不允许绝对延迟(当下一次发射相对于开始/订阅时刻延迟时)。

为了实现绝对延迟方法(所有延迟都是针对订阅/开始时刻指定的)，我们可以使用[***merge map***](https://rxjs.dev/api/operators/mergeMap)而不是[***concat map***](https://rxjs.dev/api/operators/concatMap):

![](img/0bc487ed42b43847cfb42706bb8fb6ca.png)

看起来有很多代码，延迟也是硬编码的。我们可以做得更好。所以让我们更进一步！

## 编辑选择方法

好的，我们已经知道硬编码值/延迟是不可重用的，我们也希望能够以*相对*(相对于以前的排放延迟)和*非相对/绝对*(相对于订阅时刻延迟)的方式进行调度。考虑到以上所有因素，我实现了一个定制的**时间范围**函数。

![](img/495468d459226403da59d6864cfeca0f.png)

自定义**时间范围**功能。

**好处:**

1.  看起来又漂亮又紧凑。
2.  允许指定绝对和相对延迟。

好，现在让我们展示我之前提到的三个用例。

# *在哪里使用*

**#1 闪屏消息控制示例**

这里是它的实现代码:

![](img/3c70dcfeb783fee2f908335bf238a10a.png)

我们在这里做什么？

1.  发射延迟为 1000 和 2000 的两个值(值在这里不起任何作用，实际上只有计时重要)
2.  第一个值将显示带有消息的 splash 元素。
3.  第二次将隐藏它(使用*. class list . toggle(' show ')*)

它是这样工作的:

![](img/e79262c21343f56427ee40735f593879.png)

你可以在这里的一个密码栏里玩[。](https://codepen.io/kievsash/pen/ZEzVWKa?editors=0010)

**#2 动画一些元素后续出现**

代码如下:

![](img/5675be0e5a0aa385e17ed7b714cbe5d6.png)

我们只是抓取一组元素，然后在每个可观测的发射范围内用指定的超时时间逐一显示它们。

结果如下:

![](img/6143137f7658be47f2119ca9f8bf20ac.png)

你可以在这里查看代码[。](https://codepen.io/kievsash/pen/YzKdqzd?editors=0110)

**#3 模拟时间范围内单元测试代码的输入参数。**

![](img/521e032a9507f3a256804810b0126ccb.png)

工作原理:

1.  我们用一个异步返回值' 42 '的函数模拟 ajax.get(模拟网络请求)
2.  则 se 去抖时间= 15ms。
3.  之后，调用 *getSearchResult* 并模仿*输入$* 作为参数并订阅(以开始该过程)。
4.  *输入$* 将发出三个值，延时分别为 1 毫秒、5 毫秒、30 毫秒。由于 debounceTime 设置为 15ms，因此只有第二个和第三个值会导致调用网络请求(在我们的例子中是 Ajax . get)；

和结果:

![](img/e9a2b7b2a620f661ef5ca520c533d990.png)

代码和测试可以在[这里](https://codepen.io/kievsash/pen/ePoRxg?editors=0010)找到。

[![](img/e39daa364c50029b206bdc057d2b3487.png)](http://eepurl.com/gHF0av)

## 结论

我在我的[***rxjs-toolbox***](https://github.com/kievsash/rxjs-toolbox)lib 中添加了 ***timeRange*** 函数。暂时还不大。你可以在这里了解更多:[*【fork join】完成进度*](https://blog.angularindepth.com/rxjs-recipes-forkjoin-with-the-progress-of-completion-for-bulk-network-requests-in-angular-5d585a77cce1)

有自己感兴趣的 RxJS 解决方案？请在评论中提及它们！

你喜欢这篇文章吗？然后 [**发推特说说吧**](https://clicktotweet.com/fEdKA) 和关注我 [**推特**](https://twitter.com/El_Extremal) 🤓**！**

想进一步了解 RxJS？试试我的视频课程“[动手 RxJS](https://www.udemy.com/course/hands-on-rxjs-for-web-development/) ”。它既包含初学者的全部内容，也包含更高级的主题。

干杯。