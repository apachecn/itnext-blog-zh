# 创造简单 javascript 事物的乐趣

> 原文：<https://itnext.io/the-joy-of-creating-simple-javascript-things-983d9a1c9aee?source=collection_archive---------4----------------------->

**TLDR；**有时候，我发现自己有一个想法，修理小东西是非常无聊的。我知道这不是真的，这更像是一种情绪状态，大多数时候都有办法获得乐趣。我想分享一个小故事。

![](img/79023f3cea043a290f9f784d5f54bb08.png)

照片由[马克·泰格霍夫](https://unsplash.com/@tegethoff?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/zen-garden?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

## 代码

前一段时间我遇到了代码。这让我感觉就像当我快要碰到一个坑时一样，我无法避免。然后我听到撞击声，责怪自己不够专心。

该代码创建 DOM 元素并在鼠标移动事件时更新节点的内容。最后一句话的整个描述对我来说就像在加油站抽烟:DOM 更新+鼠标移动事件是你一般要分开的东西，至于分成[不同执行上下文](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)的例子。

代码看起来就像这样

没什么不好的。有用！这不是真正的应用程序瓶颈，虽然可能有点慢。你可以注意到有时候。在这种特殊情况下，这没什么大不了的，但我仍然觉得自己碰到了一个坑——这可能会让你在未来付出代价，但却是可以避免的。

## 修补坑洞

作为一名优秀的前端工程师，我知道每当我需要一些东西(比如[左填充](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/)，嘿)时，我应该去`npm`或者回到过去的`bower`，甚至是 jQuery.com，复制粘贴一些闪亮的金属插件。

我这样做了，并找到了十几个小库，用于从 JSON 或其他一些简洁的方式创建 DOM 元素。

我可以将这个库添加到`package.json`中，重写代码，并为拥有更少的技术部门和新的依赖而感到高兴。这太无聊了。

我宁愿多花一点时间，回忆或学习一些东西，找点乐子，为什么不呢？

所以我从编写一个简单的帮助器来创建 DOM 节点开始。

创建 DOM 元素的助手

用我的新助手重写了代码:

现在看起来好多了，但这还不够。它仍然不是很高效，而且需要多次更新 DOM，[，这很昂贵](https://www.reddit.com/r/javascript/comments/6115ay/why_do_developers_think_the_dom_is_slow/)。我不喜欢它的样子，所以在下一次迭代中，我开始重新编写代码，然后更新助手来匹配我需要的东西。

好看一点的

我知道我需要 dom [片段](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment)来做更少的 dom 更新，当你想大量更新 DOM 并保持 UI 快速时`requestAnimationFrame`很方便。

再次更新了`element`助手，并做了一个计划外的更改，使这个助手更好:

在第 8-11 行，如果一个孩子是一个数组，那么它看起来像另一个`element`调用的参数。这个小小的变化引发了代码的另一次迭代:

很好！

现在我觉得不错。简短而简单，更好的是，现在我有了助手，它可以在应用程序的任何地方使用，并且不到 20 行代码。

我经历了几次反复才达到那个程度。可能我会在这上面多花两倍的时间，来修复可以用其他方法修复的代码，用一个库，或者我可以在第一次迭代后就停下来。

但是，我想说的是，拥有一个我写的工作的、简单的和漂亮的代码是一件给我带来快乐的事情，这样做很有趣。或许对你来说也是如此。

—

这个故事假设你知道一些 javascript 和一些关于 DOM 的知识。如果你是这门语言的新手，没有任何链接可以让你学习一些关于 javascript 和前端的知识，我对此感到很抱歉

*   棒极了的 javascript —如果你刚刚开始学习，向下滚动到阅读部分
*   [牛逼列表的前端部分](https://github.com/sindresorhus/awesome#front-end-development)
*   关于前端工程的更多信息[——有一段时间没有更新，但是列出了许多需要知道的重要信息，可能值得一读。](https://github.com/dypsilon/frontend-dev-bookmarks)