# Go 的新排序算法:pdqsort

> 原文：<https://itnext.io/gos-new-sorting-algorithm-pdqsort-822053d7801b?source=collection_archive---------0----------------------->

![](img/04a1f94ee9392ec06db5d1119508e01d.png)

*照片由* [*马库斯·斯皮斯克*](https://unsplash.com/@markusspiske?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) *上* [*下*](https://unsplash.com/s/photos/different-sizes?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

我一直觉得 Go 的标准库非常容易阅读。标准库的一部分包括自包含的概念，不需要太多的背景知识就可以深入了解。当我读到 Go 的排序算法已经从[变成了](https://github.com/golang/go/commit/72e77a7f41bbf45d466119444307fd3ae996e257)一种叫做“pdqsort”的东西时，我想去看看并了解一下它会很好。

在这篇文章中，我们将通过研究以前的排序算法和新的排序算法来做到这一点。

到目前为止，Go 依靠其标准库中的 [Quicksort](https://en.wikipedia.org/wiki/Quicksort) 。你已经可以在这里继续检查它的实现[了](https://github.com/golang/go/blob/release-branch.go1.18/src/sort/sort.go#L231)但是我还会提供相关的链接。

快速排序算法的核心工作原理如下:

1.  选择一个枢轴
2.  将小于轴心的项目排列在轴心的“左侧”,将大于轴心的项目排列在“右侧”
3.  对于透视的两侧，从#1 开始递归地应用相同的逻辑

完成这些步骤后，我们应该最终得到一个经过排序的集合。这在许多数据集上运行良好，我们得到了一个很好的排序算法，平均复杂度为 o(n logn)*⁴*。然而，在某些情况下，它不能很好地执行，并且最终具有二次复杂度。这意味着它的最坏情况性能为 O(n ^ 2)⁵ )。Quicksort 表现不佳的这些情况大多与已排序或几乎 sorted⁶的数据集有关。

虽然我提到 Go 最初使用了 Quicksort，但 Go 标准库使用了一些技巧来避免陷入 Quicksort 会产生最差性能或相当性能的情况。

阅读源代码，其中一些如下:

*   如果集合大小为“use⁸小 enough"⁷[shellsort](https://en.wikipedia.org/wiki/Shellsort)
*   如果递归深度太大，切换 to⁹ [堆排序](https://en.wikipedia.org/wiki/Heapsort)
*   试着选择一个好支点⁰

正如你可能已经注意到的，Go 的快速排序实现并不是算法的简单形式，而是一种叫做 [Introsort](https://en.wikipedia.org/wiki/Introsort) 的混合形式。

现在，我们来谈谈新的排序算法“pdq sort”——模式战胜快速排序。看名字，听起来还是 Quicksort。对更改的描述概括了在某些最坏的情况下，它在哪些方面比以前的版本产生了更好的性能。

让我们从这里的[开始](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/sort.go#L48)，看看有什么变化&留了下来。

当集合大小很小时，我们仍然使用到[插入排序](https://en.wikipedia.org/wiki/Insertion_sort)，在某些情况下[退回到](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L77-L81)堆排序。用于[选择枢轴](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L256-L261)的方法也类似于前面的实现。

与前面的实现不同，pdqsort 在遇到不同的模式时，试图通过移动事物来避免最坏的情况:

*   如果我们正在处理一个不能以平衡方式分区的分区⁴，我们将[移动](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L238)以避免将来出现这种不平衡的分区。此外，不是退回到纯粹基于递归深度的堆排序，而是使用不平衡分区的数量来切换到回退。
*   [处理](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L99-L104)集合中可能已经排序或反向排序的部分
*   [处理等于支点的](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L108-L112)元素，以避免将来不必要的交换

在介绍 pdqsort 算法的[文章](https://arxiv.org/pdf/2106.05123.pdf)中有更多的内容。这些是我通过阅读源代码和文章可以发现的主要区别。新算法的实现似乎有更多的活动部分，尽管代码是以一种易于理解的方式组织的。

这有助于我理解发生了什么变化。我希望这能很好地概述这一变化以及它试图解决的问题。

*1。截至 Go 1.18，这仍未发布，预计将在未来版本中登陆*

*2。Pivot 是在给定集合中排列其他项目时要保留在原位的项目*

*3。排序的概念可能因被排序的项目而异。这些可能是整数或更复杂的数据结构。*

*4。时间复杂度*的解释见 [*本*](https://en.wikipedia.org/wiki/Big_O_notation)

**5。n 平方的阶。**

*6。那可能包含许多重复的元素——参见“ [*荷兰国旗问题*](https://en.wikipedia.org/wiki/Dutch_national_flag_problem)*

**7。不到 13**

***8。参见* [*此处*](https://github.com/golang/go/blob/release-branch.go1.18/src/sort/sort.go#L215-L222) 的相关代码**

***9。参见* [*此处*](https://github.com/golang/go/blob/release-branch.go1.18/src/sort/sort.go#L234-L242) *中递归切换到 Heapsort 的深度是如何计算的***

**10。见 [*本*](https://en.wikipedia.org/wiki/Quicksort#Choice_of_pivot) *解释选择一个好支点的困难和不同的方法。这些大多也可以在 Go 的源代码中找到:* [*避免*](https://github.com/golang/go/blob/release-branch.go1.18/src/sort/sort.go#L110) *整数溢出，* [*挑选*](https://github.com/golang/go/blob/release-branch.go1.18/src/sort/sort.go#L111-L118)*[*第九*](https://en.wikipedia.org/wiki/Median#Ninther) *"作为支点****

**11。参见相关文章 [*此处*](https://arxiv.org/pdf/2106.05123.pdf) *对算法更透彻的解释。***

***12。变化的作者* [*包括*](https://github.com/golang/go/commit/72e77a7f41bbf45d466119444307fd3ae996e257) *一个比较，表明它比以前的排序切片算法快大约 10 倍，并且从来没有比以前的版本慢很多。***

***13。虽然之前的算法用的是*[*Shellsort*](https://en.wikipedia.org/wiki/Shellsort)*，但都是非常相似的算法。***

***14。这是通过检查枢轴的任一侧(参见*[*1*](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L120)&[*2*](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L124)*)来完成的，以查看我们最终是否有两个分区，其中一个是* [*比另一个*小得多的](https://github.com/golang/go/blob/72e77a7f41bbf45d466119444307fd3ae996e257/src/sort/zsortinterface.go#L118)**