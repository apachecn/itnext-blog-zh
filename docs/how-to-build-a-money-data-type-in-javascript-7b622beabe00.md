# 如何用 JavaScript 构建货币数据类型🔊

> 原文：<https://itnext.io/how-to-build-a-money-data-type-in-javascript-7b622beabe00?source=collection_archive---------4----------------------->

## 浮点陷阱的替代方案

![](img/c179d1e6e9c94762fd869d37d4b59d5c.png)

一场[喷火](https://en.wikipedia.org/wiki/Fire_breathing)表演中巨大火焰的画面。背景很暗。火从右边来，一个隐藏在屏幕外的人执行技术([来源](https://commons.wikimedia.org/wiki/File:Flame_of_fire.jpg))

听听音频版！

上次我写了一个[逐步的例子](https://medium.com/@fagnerbrack/the-missing-practical-step-by-step-test-driven-development-a7140ca4b71)，关于如何使用 JavaScript 将[从内向外测试驱动开发](https://8thlight.com/blog/georgina-mcfadyen/2016/06/27/inside-out-tdd-vs-outside-in.html)应用到一个问题上。那个帖子使用了`Number`类型来表示美元。

根据 [JavaScript 规范](https://es5.github.io/#x8.5),`Number`类型是一个“[双精度 64 位二进制格式 IEEE 754](https://en.wikipedia.org/wiki/Double-precision_floating-point_format#IEEE_754_double-precision_binary_floating-point_format:_binary64) 值。您不应该使用 IEEE 754 的实现来表示任何需要精确的数据。钱是其中之一。

有[也有](http://wiki.c2.com/?FloatingPointCurrency)多[内容](https://husobee.github.io/money/float/2016/09/23/never-use-floats-for-currency.html)出来[有](https://stackoverflow.com/questions/3730019/why-not-use-double-or-float-to-represent-currency)关于[这个主题](https://spin.atomicobject.com/2014/08/14/currency-rounding-errors/)。因此，这篇文章通过编码示例展示了如何创建一个能够正确处理货币类型的模型

> 您不应该在 JavaScript 中使用“数字”类型来表示钱

放债人杰克开始使用[上一篇文章](https://medium.com/@fagnerbrack/the-missing-practical-step-by-step-test-driven-development-a7140ca4b71)中的函数计算一些贷款。在某个时刻，他试图计算贷款金额“2585 美元”结果是“$52.73”后跟“999999999999”

杰克对此很不高兴。五年前，他的朋友为一个黑手党女王做会计。一天，她发现一个账户上有一个奇怪的分数，于是起了疑心。

她解雇了他。就像…用真火！

好吧，杰克现在不是为黑手党工作。然而，他不喜欢在某些情况下计算可能产生不正确的结果。

你需要修复它。

> 杰克不喜欢火

在《企业应用架构模式》一书中，马丁·福勒[为货币价值的表示提供了一个解决方案](https://martinfowler.com/eaaCatalog/money.html)。他建议你可以创建一个模型，让[提供给](https://hackernoon.com/affordance-in-software-design-12cc0d9d2721)一些算术运算。但是，代码在“amount”之上运行计算，amount 是一个内部非小数属性，不受浮点问题的影响。

您可以将可视化表示从逻辑中分离出来，并以分钟为单位运行所有的数学运算。

如果您将分表示为非小数的`Number`,您就不需要担心浮点问题。稍后，您可以将`Number`转换为`String`，并根据您想要输出的货币决定将[小数点分隔符](https://en.wikipedia.org/wiki/Decimal_separator)放在哪里。

一个[文本文件](https://gist.github.com/FagnerMartinsBrack/0f1f2a8d06fb708dd74dc78e4cd9d3aa)包含不同货币中小数点的位置和样式的例子。

鉴于这是一个一般的编程问题，而不是一个非常具体的问题领域，编程语言至少应该提供开箱即用的解决方案。

不幸的是，JavaScript 没有小数或货币类型，我不建议你花时间[重新发明](https://en.wikipedia.org/wiki/Not_invented_here)它。有一些库是值得研究的，比如 [dinero.js](https://github.com/sarahdayan/dinero.js) 。

不过，对于这篇文章，让我们从头开始创建一个`Money`类型。

根据“[支付美元的利息](https://github.com/FagnerMartinsBrack/jack-the-moneylender/blob/d0de7c4f03fc3c85af62a392a747a9bff9f6f0c7/index.html#L9-L17)”函数，唯一必要的操作是减法、乘法和“大于”没有必要花时间去开发你现在不需要的[启示](https://hackernoon.com/affordance-in-software-design-12cc0d9d2721)。

[函数“支付美元利息”的 JavaScript 代码](https://github.com/FagnerMartinsBrack/jack-the-moneylender/blob/d0de7c4f03fc3c85af62a392a747a9bff9f6f0c7/index.html#L9-L17)如果你想知道这是怎么来的，看看[之前的帖子](https://medium.com/@fagnerbrack/the-missing-practical-step-by-step-test-driven-development-a7140ca4b71)。该代码使用本机运算符进行减法、乘法和“大于”比较。

新模型的构造器接受一个带有美元符号(“$1.00”)的`String`。此时不需要支持多种货币，所以使用这种符号是安全的。

此外，新模型要求消费者始终指定两个十进制数字“. 00”，即使值是整数，如“$5.00”，而不是省略数字，如“$1”。这样逻辑就更简单了。这个问题比较容易推理。

```
Money('$0.01');
Money('$0.10');
Money('$1.00');
Money('$10.00');
Money('$12.45');
```

减法运算减去以美分为单位的值。稍后，它会生成可视化表示，并在必要时添加负号前缀。

为了产生直观的表示，您将小数点放在倒数第二位。如果小数点位置在第一个数字之前结束，用零填充缺少的字符。

[名为“至小数点”的功能的代码](https://gist.github.com/FagnerMartinsBrack/e7c07a64aa2538cc137b405a94cc71e5)该算法显示了如何将小数点放在正确的位置。

乘法比减法更难。将美元乘以美分有意义吗？也许吧。让我们试试。

***注:*** *参见* [*此评论*](https://medium.com/@fagnerbrack/the-reason-it-wouldnt-make-sense-to-multiply-dollar-amounts-against-other-types-of-dollar-amounts-8bef9dff364) *了解为什么将美元乘以美分没有意义的更多信息。*

为了解决杰克的[利息问题](https://medium.com/@fagnerbrack/the-missing-practical-step-by-step-test-driven-development-a7140ca4b71)，你需要计算 1.00 美元乘以 0.09 美元。因此，在乘法因子中使用其他美元值而不是 a `Number`似乎是合理的。

因为你将两个美元值乘以美分，如果你将 2 美元乘以 2 美元，你将得到 40000 美分。这表示为$400.00，但结果应该是$4.00。

如果你把计算结果除以 100，你会得到 400 美分的正确输出。但是，您可以通过从乘法结果中去掉最后两位来避免数学除法。

[一个文本文件](https://gist.github.com/FagnerMartinsBrack/69d14b8468e709ee980a54889a1b8dec)，显示以分为单位的乘法运算，以及如何将最右边的数字切掉。

当然，这两种解决方案都是以损失超过 2 位小数的精度为代价的。

[代码](https://gist.github.com/FagnerMartinsBrack/bae566c2df9a4f851cbcddcc82dcdcd0)显示乘法逻辑的实现。

您可以看到没有进一步优化的最终结果，在“Jack，The Moneylender”存储库的新提交(T7)中，包括测试(T10)。

我使用 TDD 来产生这个代码，就像我为 Jack 做的第一个交付物[一样。然而，这篇文章并不是要告诉你如何做 TDD 这是为了理解如何解决浮点数缺乏精度的问题。](https://medium.com/@fagnerbrack/the-missing-practical-step-by-step-test-driven-development-a7140ca4b71)

> 有许多方法来表示金钱。其中之一就是用美分来表示。

没有什么可以阻止你提高精度和扩展到美分以上，比如支持[百万货币](https://en.wikipedia.org/wiki/Mill_(currency))或[比特币(satoshis)](https://en.wikipedia.org/wiki/Bitcoin#Units) 的部分。为了做到这一点并保持向后兼容性，假设丢失的数字的值为零。如果要从 satoshis 降尺度到美分，代码也不会破；只有你会失去超过 2 位小数的精度。

浮点数是一个陷阱。他们正确地拟合了问题；直到他们没有。到那时，再做任何改变都可能为时已晚。

不要用 JavaScript `Number`类型的分数表示来表示钱。事实上，不要对任何需要精度的东西使用浮点。

可怕的事情发生时，你不需要和黑手党扯上关系。如果你不关心软件设计，那么以后可能就要有人来灭火了。

甚至可以是自己。

感谢阅读。如果你有一些反馈，请通过 [Twitter](https://twitter.com/FagnerBrack) 、[脸书](https://www.facebook.com/fagner.brack)或 [Github](http://github.com/FagnerMartinsBrack) 联系我。

感谢 [Jon Eaves](https://joneaves.wordpress.com/) 、 **Stephan Classen** 和 [Jay Bazuzi](http://jay.bazuzi.com/) 对这篇文章的深刻见解。