# TypeScript 中的点自由样式是什么？

> 原文：<https://itnext.io/whats-point-free-style-in-typescript-39337000c8cb?source=collection_archive---------4----------------------->

## TypeScript/JavaScript 中的函数编程

![](img/3652333a13a30f6176f5dd7419a4d6f2.png)

# 什么是无积分

单点自由风格基本上是你写代码，但不明确地在代码中提供参数。

这在需要函数的回调中尤其有用。我将向您展示 TypeScript 中的示例，它在 JavaScript/ES6 中也是一样的。

想象一下你有一份财务交易清单。

```
interface Transaction {
 amount: number;
}
```

现在你想得到一个正交易的列表。

通常您可以这样做:

```
public getPositiveTransactions(transactions: Transaction[]) {
  return transactions.filter((t: Transaction) => t.amount > 0));
}
```

或者您可以通过将函数名传递给过滤器回调来实现无点风格。

```
public getPositiveTransactions(transactions: Transaction[]) {
  return transactions.filter(this.isPositive);
}private isPositive(transaction: Transaction) {
  return transaction.amount > 0;
}
```

# 积分免费的好处

point free 的好处是让你的代码**易于阅读**。从长远来看，当你写任何代码时，可读性应该是最重要的。这里有一句我最喜欢的关于“可读性”的名言:

> 程序应该是写给人们阅读的，只是顺便给机器执行的。

但是最初看起来没有什么好处，因为我们基本上只是用函数名替换了一个内联函数实现。

```
transactions.filter((t: Transaction) => t.amount > 0));vstransactions.filter(this.isPositive);
```

但是当需求变得更加复杂时，您将开始看到好处。

现在你有了一个新的需求，说我们需要得到一个有“大”金额的交易列表，这个“大”金额在某处被定义为一个常数。

所以通常你只需要更新你的函数来改变内联函数。

```
private const BIG_AMOUNT = 10;  

public getBigTransactions(transactions: Transaction[]) {
  return transactions.filter((t: Transaction) => t.amount > this.BIG_AMOUNT);
}
```

这是可行的，但是现在你的内部函数实现依赖于一个“外部”常量，这对一个函数来说是不好的。我们希望我们的函数是“纯”的，这意味着所有的输入都被提供给函数。

让我们来看看如何以点自由的风格做到这一点:

```
private const BIG_AMOUNT = 10;public getBigTransactions(transactions: Transaction[]) {
  return transactions.filter(this.moreThan(this.BIG_AMOUNT));
}private moreThan(amount: number) {
  return (transaction: Transaction) => transaction.amount > amount;
}
```

注意代码是" **fluent** "读起来:如果超过 BIG_AMOUNT，我们就要过滤。

# 小心“这个”

如果我把上面的代码改成下面的，会有用吗？

```
private const BIG_AMOUNT = 10;public getBigTransactions(transactions: Transaction[]) {
  return transactions.filter(this.moreThanBigAmount);
}private moreThanBigAmount(transaction: Transaction) {
  return transaction.amount > this.BIG_AMOUNT;
}
```

答案是:不会，不会因为“这个”而行不通。

对于 TypeScript/JavaScript 中的“this”我就不多赘述了，但是“this”基本上取决于调用者。而在我们这里的例子中，当在`filter`回调中调用`this.moreThanBigAmount`时，调用者就是“window”。

我们可以在定义函数时通过`bind`到`this`来解决这个问题，但是我们应该避免这样做。

关键是，如果我们试图用函数式编程来编程或思考，我们需要显式地将所有依赖项定义为函数输入。我们应该避免在任何作为第一类对象传递的函数中使用“this”。

# 积分免费是如何运作的

所以自由点函数的工作原理是:

1.  在需要函数的地方提供函数名。
2.  在大多数情况下，该功能需要**部分应用。**(参见上面的`moreThan`功能)

如果你不熟悉**部分应用的**功能，或者技术名称“**奉承**，我会在下一篇文章中解释。

*本文属于 TypeScript/JavaScript 中* [*函数式编程系列。*](https://medium.com/@hamxiaoz/functional-programming-in-typescript-javascript-d9d79663bc4)