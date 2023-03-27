# bigint-money:一个做货币数学的 NPM 软件包

> 原文：<https://itnext.io/bigint-money-an-npm-package-for-doing-currency-math-be041aeec8df?source=collection_archive---------3----------------------->

![](img/fa5a2422dd0e30e3bb640a64436d6edb.png)

不久前，我对用金钱和浮点问题做数学感到困惑。经过大量的研究和朋友们的帮助，我明白了很多人在我之前已经做过的事情:使用浮点确实不好。你可能会遇到舍入问题，并且真的想使用“精确数学”。

之后，我意识到我想使用新的 Ecmascript [bigint](https://github.com/tc39/proposal-bigint) 类型作为表示金钱的基础。这是最近版本的 Node.js 和 Typescript 支持的，这也是我需要的地方。

但是有一个问题，`bigint`类型只保存“整数”,不做定点十进制数学。我在 NPM 上四处寻找可以为我抽象这个的东西，但是找不到一个库。

所以我花了几天时间编写了一个简单的名为`bigint-money`的 npm 库，它封装了`bigint`类型。它是用打字稿写的，而且是开源的，所以任何人都可以使用:[https://www.npmjs.com/package/bigint-money](https://www.npmjs.com/package/bigint-money)。

主要特点:

# 基准

NPM 上的大多数“钱”库只使用 2 位数字进行精确计算，或者使用 Javacript 的“数字”,很快就会溢出。

我找到的唯一可比较的图书馆是[大款](https://www.npmjs.com/package/big-money)。如果您的 Javascript 环境还不支持`bigint`，这可能是最好的选择。

我的简单基准计算一个有 100 万条目的分类帐。

```
 bigint-money | big-money
ledger       816 ms | 43.201 ms 
%             100 % | 5294 %
```

基准脚本可以在[库](https://twitter.com/evertp/status/1088147830529802242)中找到。

我想强调的是，这个(相当好的)结果不是我的功劳。图书馆很快，因为`bigint`很快。

我确实需要自己实现一些功能，实际上我觉得更擅长编写快速代码的人可能会比我做得更好。

我想，特别是`moneyValueToBigInt`和`bigintToFixed`内部函数可能会被比我聪明得多的人优化。

# 例子

它通常是这样工作的:

```
// Creating a money object.import Money from 'bigint-money';
const foo = new Money('5', 'USD');
```

也可以用一个数字创建一个新的货币对象

```
const foo = new Money(5, 'USD');
```

但是，如果你给它传递一个不安全的数字，比如一个浮点数，就会抛出一个错误:

```
const foo = new Money(.5, 'USD'); // UnsafeIntegerException
```

一旦有了 Money 对象，就可以使用`toFixed()`输出一个字符串。

```
const foo = new Money('5', 'USD');
console.log(foo.toFixed(2)); // 5.00
```

# 算术

可以在上面用`.add()`和`.subtract()`:

```
const foo = new Money('5', 'USD');
const bar = foo.add('10');
```

所有的货币对象都是不可变的。调用这些函数不会改变原始值:

```
console.log(foo.toFixed(2), bar.toFixed(2)); // 5.00 1.00
```

您还可以传递货币对象来进行加减运算:

```
const startBalance = new Money(1000, 'USD');
const salary = new Money(2000, 'USD');
const newBalance = startBalance.add(salary);
```

如果您尝试添加不同货币的货币，将会出现错误:

```
new Money(1000, 'USD').add( new Money( 50000, 'YEN' ));
// IncompatibleCurrencyError
```

除法和乘法:

```
// Division
const result = new Money(10).divide(3);// Multiplication
const result = new Money('2000').multiply('1.25');
```

# 比较对象

现在，货币对象只有一个比较功能。

```
const money1 = new Money('1.00', 'EUR');money.compare(2); // Returns -1
money.compare(1); // Returns 0
money.compare(0); // Returns 1
money.compare('0.01'); // returns -1
money.compare(new Money('1.000005', 'EUR')); // returns 1// throws IncompatibleCurrencyError
money.compare(new Money('1', 'CAD'));
```

其思想是，如果对象小于传递的对象，则返回-1。如果相等，则返回 0，如果传递的值较大，则返回 1。

这使得排序变得容易:

```
const values = [ new Money('1', 'USD'), new Money('2', 'USD') ]; values.sort( (a, b) => a.compare(b) );
```

# 分配

当把钱平分时，可能会损失一便士。例如，在三个人之间分配 1 美元时，每个人得到 0.33 美元，但还有多余的 0.01 美元。

allocate 函数将一个货币值分成均匀的部分，但是剩余的部分按循环方式分配。

```
const earnings = new Money(100, 'USD');
console.log( earnings.allocate(3, 2); ); // Results in 3 Money objects:
// 33.34
// 33.33
// 33.33
```

分割债务(负值)也像预期的那样起作用。

allocate 函数的第二个参数是精度。基本上就是你感兴趣的位数。

对于美元和大多数货币，这是 2。它需要传递这个参数，因为 Money 对象无法猜测所需的精度。

*最初发表于*[T5【evertpot.com】](https://evertpot.com/bigint-money-typscript-lib/)*。*