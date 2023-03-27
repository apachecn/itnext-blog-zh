# 如何用 RxJS 添加 autosave 的终极指南💾

> 原文：<https://itnext.io/the-ultimate-guide-on-how-to-add-autosave-with-rxjs-45b89cd23edc?source=collection_archive---------1----------------------->

![](img/b572e19c6079239d06655f49f0e71f08.png)

照片由[https://unsplash.com/@sandrokatalina](https://unsplash.com/@sandrokatalina)拍摄

## 这是游戏计划，✔

1.  首先，我们需要监听输入字段的变化。[**from event**](https://www.learnrxjs.io/learn-rxjs/operators/creation/fromevent)**操作符非常适合监听 DOM 事件，比如输入更改。您可以向它传递一个元素和一个事件，比如 key up**

**2.限制请求数 w/ **去抖时间**🏀。使用反跳时间意味着我们只在用户停止输入一段时间后才发送请求。**

**如果没有反跳时间，请求将按字母触发。例如，假设你正在输入 *{…"c"…..一个。“t”}。*在用户暂停输入后，我们应该使用 [**去抖时间**](https://www.learnrxjs.io/learn-rxjs/operators/filtering/debouncetime) 稍微等待一下，而不是不必要地发送每个字母的请求。**

**3.防止重复请求 w/[**distinctUntilChanged**](https://www.learnrxjs.io/learn-rxjs/operators/filtering/distinctuntilchanged)**。**使用 distinctUntilChanged 意味着我们将在发送请求之前比较当前值和以前的值。**

**如果没有[**distinctUntilChanged**](https://www.learnrxjs.io/learn-rxjs/operators/filtering/distinctuntilchanged)，如果我们不小心，可能会发送重复的请求。例如，假设您正在键入{ " **cat** " ………。"猫"..“猫”}。第一次输入“cat”时，会发出保存请求。现在，如果你输错了“catr ”,按 backspace 键，你将再次以“cat”结束。🐈。不需要为“猫”发送 2x 请求。为了确保只发出一个请求，使用[**distinctUntilChanged**](https://www.learnrxjs.io/learn-rxjs/operators/filtering/distinctuntilchanged)**。****

**4.✨使用**切换图**取消之前的发射，只监听 latest✨.**

**如果没有 switchmap 的这种取消效果，您可能会遇到一些奇怪的情况。网络是疯狂的，不能保证第一个请求会在第二个之前被收到。这就是为什么我们应该取消任何以前的请求，只监听最新的值。**

**5.订阅更改并获得自动保存结果。👌**

**✌️ **结论****

**嗯，那是相当神奇的✨.Rxjs 很棒吧？👍**

**如果你喜欢这篇文章，请关注、分享或评论。我定期发表文章，并期待在你变得更伟大的旅程中帮助你。**

****最终代码:****

**【https://stackblitz.com/edit/typescript-ymbl3b?embed=1】T2&file = index . ts**

# **结论:**

# **感谢阅读。如果你学到了什么，点击跟随按钮！**