# 逻辑赋值运算符— JavaScript

> 原文：<https://itnext.io/logical-assignment-operators-javascript-89e39e886c61?source=collection_archive---------4----------------------->

## Java Script 语言

## 使用这些有用的 JavaScript 操作符编写更少的代码

![](img/670550b0374d69a15fa29bbeb553c236.png)

[注册](https://unsplash.com/@theregisti?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

从 EcmaScript 2021 版本开始，有三个新的赋值运算符可用。准确地说是“逻辑赋值运算符”。听起来很拗口，但这只是写条件和赋值的一种速记方式。让我们开始吧。

> *注:这也适用于* ***的打字稿***

# 逻辑赋值运算符

## 和赋值&&=

我们的第一个例子是使用更广为人知的`AND`操作符，也称为`&&`。我们通过在后面添加一个`=`符号使它成为一个赋值操作符，就像这个`&&=`。如果左边和右边都是`true`，那么它返回新值‘11’。如果其中一边是`false`，那么它不返回新值，所以返回原来的`undefined`。

```
let example1 = 10;
let example2 = undefined;// Using the AND assignment:
example1 &&= 11; // logs 11
example2 &&= 11; // logs undefined// Using without the AND assignment:
example1 = example1 && 11; // logs 11 
example2 = example2 && 11; // logs undefined
```

## OR 赋值||=

另一个操作符是`OR`操作符(`||`)，同样是为了将它用作赋值，我们像这样使用它`||=`。它确实和`&&=`一模一样，但是反过来了。所以如果左边是`true`，那么返回新值。如果不是，则返回右边的值。

```
let example1 = 10;
let example2 = undefined;
let example3 = 0;// Using the OR assignment:
example1 ||= 11 // logs 10
example2 ||= 11 // logs 11
example3 ||= 11 // logs 11// Using without the OR assignment:
example1 = example1 || 11; // logs 10 
example2 = example2 || 11; // logs 11
example3 = example3 || 11; // logs 11
```

## 无效转让？？=

接下来是最后一个逻辑赋值运算符，即`Nullish`赋值(`??=`)。该操作符没有`OR`操作符严格，但作用几乎相同。参见此处解释的[差异。注意`example3`的差异。](https://medium.com/p/c24e2be6fe5e)

```
let example1 = 10;
let example2 = undefined;
let example3 = 0;// Using the Nullish assignment:
example1 ??= 11 // logs 10
example2 ??= 11 // logs 11
example3 ??= 11 // logs 0// Using without the Nullish assignment:
example1 = example1 ?? 11; // logs 10 
example2 = example2 ?? 11; // logs 11
example2 = example3 ?? 11; // logs 0
```

仅此而已！

我希望你今天学到了一些新东西。如果你觉得这篇文章有用，请点击 50 次👏按钮并关注我了解更多内容。