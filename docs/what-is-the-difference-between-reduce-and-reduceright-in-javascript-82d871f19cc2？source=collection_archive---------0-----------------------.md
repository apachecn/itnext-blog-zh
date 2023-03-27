# JavaScript 中 reduce 和 reduceRight 有什么区别？

> 原文：<https://itnext.io/what-is-the-difference-between-reduce-and-reduceright-in-javascript-82d871f19cc2?source=collection_archive---------0----------------------->

![](img/a3afb278579820a452b88e8a6e08300c.png)

伊利亚·波什科夫在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

您是否想过这两个函数在 JavaScript 中是如何工作的？首先，我将演示内置函数，然后，我们将创建自己的 reduce 和 reduceRight 函数。言归正传！

我创建了一个列表并声明了一个函数，该函数为列表中的每个元素加 1，并在每次迭代中返回一个新列表，但是我对同一个列表应用了两个不同的函数，第一个是 reduce 函数，第二个是 reduceRight 函数。你知道除了函数名还有什么区别吗？

reduce 函数从左到右，当它到达最后一个元素时，该函数应用于该元素，在下一次迭代中将返回累加器。当使用 reduceRight 函数时，迭代从右向左进行，当到达最后一个元素时，累加器将为空，并向后运行，因此实际上，第一个元素是最后一个元素，这与反转列表并应用 reduce 函数是相同的。

因此，上表中 reduce 函数返回[5，4，3，2]，reduceRight 函数返回[2，3，4，5]。为了更好地说明这一点，我们将构建自己的函数。让我们从 reduce 开始:

上面的 reduce 函数接收三个参数，第一个是列表，第二个是累加器，第三个是 reduce 函数。对于每一次迭代，累加器都被建立起来，直到到达列表的末尾，当没有元素时，函数返回累加器。换句话说，如果我们想对一个列表[1，2，3，4]应用 reduce，其中 reduce 函数是(acc，element) => [element + 1，…acc]，迭代将如下所示:

减少右功能将与前面略有不同:

上面的 reduce right 函数接收同样的三个参数，第一个是列表，第二个是累加器，第三个是 reduce 函数。对于每一次迭代，它都转到下一个元素，直到到达列表的末尾，然后开始向后运行。当没有元素时，它返回累加器。

通常，您将使用 reduce 函数，而不是 reduceRight 函数。我希望现在你已经清楚这两种功能的区别了！

快乐编码:)