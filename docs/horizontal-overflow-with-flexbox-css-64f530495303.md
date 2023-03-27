# 使用 Flexbox CSS 的水平溢出

> 原文：<https://itnext.io/horizontal-overflow-with-flexbox-css-64f530495303?source=collection_archive---------1----------------------->

![](img/f9b19ad49703ba2868bda9f58bfb34f0.png)

乔安娜·科辛斯卡在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

做一名网络开发人员充满了意想不到的挑战。

耶。

我只是想为我的画廊组件创建一个水平的图像容器，并完成它。

所以我想出了一个基本的布局:

第一相

干净利落。
但是……唉！
缺少容器的右填充！容器膨胀时，似乎出了什么差错。

搜索互联网(+[规格](https://www.w3.org/TR/css-flexbox-1/))并没有给我一个答案，为什么会发生这种情况。

不过，它给了我一些变通办法:

## 1.在伪元素后使用::命令

因为看起来没有考虑容器的端侧填充(在 LTR 方向上是正确的填充)，所以我们可以添加一个具有所需的相同填充值的伪元素，并且我们将获得正确的填充。

## 2.使用包装 div

另一个解决方法是用另一个 div 包装我们的容器，它将被用作可滚动容器。

我们将把我们的`flex-container`的`padding`换成`margin`，并给它一个`display: inline-flex`，这样`flex-container`将随着它的内容而增长。

*注意:
这个解决方案可能不如使用* `*::after*` *伪元素有效，因为* `*hover*` *事件不适用于* `*margin*` *，所以你会丢失* `*flex-container*` *中的那个区域。*

# 结论

我给了你两个选择来解决这个问题。

比起最后一个，我更喜欢第一个，因为你没有失去任何优势。这将迫使您保持`flex-container`和`::after`填充同步，但这是一个很小的代价(使用 LESS 和 SASS 等预处理程序，您可以将值赋给一个变量)。

请求:
如果你真的明白为什么会发生这种情况，并能给我指出原因，这将是一个很大的帮助，因为 2 天的搜索没有给我一个满意的答案，可以在这里分享。