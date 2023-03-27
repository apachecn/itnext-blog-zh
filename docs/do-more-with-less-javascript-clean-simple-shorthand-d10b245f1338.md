# 用更少的 JavaScript 做更多的事——简洁明了的简写

> 原文：<https://itnext.io/do-more-with-less-javascript-clean-simple-shorthand-d10b245f1338?source=collection_archive---------0----------------------->

## 清理您的 JAVASCRIPT

## 我们不要把事情复杂化。使用普通 JavaScript 表达式的简写替代方法提高可读性。

![](img/8e093fe1d32e76801a7475acf9b87f01.png)

您正在编写肮脏、混乱的代码(影响您代码库的每一个区域)。

# 与认知超载作斗争

编写干净的代码并不仅仅意味着用更漂亮的格式，遵循最佳实践，或者留下一些好的注释，给你的代码实际上正在发生的事情一些线索。它包括为那些愿意阅读、试图理解和使用你的代码的人写代码，并带有一定的**同理心**。这可能意味着选择**替代方法**或**工具**，根据**如何适应其环境**来实现相同的目标。

## 问自己一些问题

*   清楚发生了什么吗？(你努力实现的目标)
*   **读起来是不是让人望而生畏？**(好像你要再喝点咖啡)
*   你有没有让它看起来比实际更复杂？(你是不是工程过度了)

> 干净的代码减少了认知负荷，更有效地传达了意图，并减少了跟上逻辑所需的时间，以**扩展**、**改进**或**添加**到功能中。

## 改变你的习惯

有时候，你写的代码是完美的**有效的**、**可用的**和**有点可读的**，但是你可能绕过了解决问题的道路。也许那是因为你还没有努力开始使用 [**更新的语法**](http://es6-features.org/#Constants) ，或者你还停留在你的工作方式和你的**思维过程中。**

也许是时候通过创造新习惯来改变你的习惯了。用解决相同问题的简写替代方案来代替它们。这需要不断地问自己问题，直到你习惯了新的方法。

将要讨论的速记表达式有其局限性。它们**并不总是正确的选择**，就像任何工具一样。**然而**，它们是有效的语言特性&语法。

你可能对使用的一些代码有强烈的感觉。如果是这样的话，我想听听！

> 如果你有一个**配置良好的** [**linter**](https://eslint.org) 那么你可以**包含** [**规则**](https://www.npmjs.com/package/eslint-config-airbnb) 来帮助你用**首选语法**进行编写。

让我们从一个简单的开始

# 简化您的真实性检查

## 充分发挥“真实”的潜力

不用使用复杂的`if`条件来检查一个值是否是真的，你可以简单地**使用变量本身作为条件**。然而，你需要知道真值背后的逻辑。

*   将评估为**真值**，如果不是`false`、`0`、`0n`、`""`、`null`、`undefined`和`NaN`。

## 以前

## 在...之后

# 三元运算符

## 停止使用这么多 if-else 语句

[三元运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)经常被用作 **if 语句**的快捷方式。简单来说就是**问一个问题**(条件)，如果答案是**是**(真)或**否**(假)，你就执行一些代码。

因此，**语句有三个部分**:要求值的**条件**、真值表达式**和假值表达式**。

```
condition ? exprIfTrue : exprIfFalse
```

## 以前

## 在...之后

## 当三元运算符变得混乱时

对于单个表达式，建议使用三元运算符**。您可以使用它们来执行基于条件的代码:`condition ? executeThis() : executeOther()`或分配给变量`const num = condition ? 3 : 5;`。**

当**在一个表达式中引入多个三元运算符**时，就会变得**混乱**。比如说…

乍看之下，这种逻辑可能并不清晰，但你希望它清晰！为了你**未来的自己**和你**的同行**。

当意图开始变得不清楚时，你应该**将它们**分解成单个语句或者使用 **if-else** 语句。

## 奖金(省略‘else’)

正如你从上面的 **if-else，**你可以在`if`语句中省略 else if`return`。`return`将导致函数提前返回，使`else`成为多余的(因此，逻辑不需要它来按预期工作)。

# 短路评估

## (变量赋值真的需要 if-else 吗)

[短路评估](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators)是一个条件，除了**优先级**之外，所使用的**逻辑运算符**决定语句是否应该提前返回值(短路)。它允许你阻止**不必要的代码执行。**

*   `(some falsy expression) && *expr*`被短路评估到 **falsy** 表达式；
*   `(some truthy expression) || *expr*`被短路评估为**真值**表达式。

## 以前

## 在...之后

## 额外收获:非布尔值

短路评估，当与**非布尔值**一起使用时，可以返回非布尔值。

*   &&操作员短路到第一个错误值**值**
*   ||操作员短路至第一真值**值**

然而，要小心引入复杂性。当您引入**多个**逻辑运算符时，您就开始**牺牲短路评估**的好处了。这是滋生**简单得令人恼火的**(但容易被忽略)错误的温床。

这是由于 [**运算符优先于**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence) 。值得注意的是，由于**优先于**，将首先计算`&&`，除非**将单独的逻辑运算符条件放在括号**中。

# 箭头功能

一个 [**箭头函数表达式**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators) 是一个常规函数的**语法紧凑替代**。它没有自己与`this`、`arguments`、`super`或`new.target`的绑定。

使用箭头函数使得我们的代码**更加简洁**，**简化了**和`this`的作用域。

## 以前

## 在...之后

带参数和不带参数的箭头函数的多种用法

# 环

## 使用`forEach` [阵列原型法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)

它允许你**迭代元素**并一次操作一个。通过**回调**参数提供对每个元素的操作。它看起来更干净，也更容易推断出意图。

## 以前

## 在...之后

**然而**的一个缺点是，你**不能提前终止** `**forEach**` **执行**，就像你可以用常规`for..of`语句一样。如果你需要提前终止的可能性，那么这个**不是正确的工具**。

> 与其他 for 循环相比，`forEach()`的性能可能会有所不同。测试表明`[forEach](https://github.com/dg92/Performance-Analysis-JS)` [明显比](https://github.com/dg92/Performance-Analysis-JS)慢，但是对于简单的任务，应该没有真正的性能问题。

# 传播算子

[扩展操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)允许 iterable 在有**零个或多个参数**的地方扩展，比如**数组**中的元素或者**对象**的键值对。

## 以前

## 在...之后

# Rest 运算符

[rest 操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)允许不确定数量的参数，表示为一个数组。

## 以前

## 在...之后

# 默认对象值(使用扩散)

当给对象赋值时，有时你想检查它们是否**真**(已经定义并且不为空等)。

如果它们是假的，你可能想用 T21 回到默认值。您可以使用**扩展操作符实现这一点。**第二次扩展具有**覆盖已经定义的对象值**的能力，否则保留`defaultValues`。

因此，默认值被用作第一个扩展的**，以便第二个扩展**覆盖第一个**。**

## 以前

## 在...之后

# 倒计时时

这只是一个简单的 for 循环 i **的替换，遍历一个 iterable 的每个索引**。使用具有递减值`val--`的`while`循环将**倒计数到零**(其本身为假)以结束循环。

看起来**更清晰**(对我来说)和**容易理解**你正在迭代的东西乍一看。

## 以前

## 在...之后

> 在 javascript 中找到 while 循环的用法很不错(我不怎么用它们)。

# 按位非运算符

![](img/87d0cb42cab12e008a71d781d079e913.png)

## 检查值是否在数组中

**位非**运算符可与`indexOf`语句一起使用，以**检查值是否存在于`Array`或`String`中的**。

我们通常会检查`indexOf`操作的返回值是否大于`-1`，如果`indexOf`返回 **0 或更高，波形符`~`将返回 true。**

> 按位非背后的逻辑是`~n == -(n+1)`。

## 以前

## 在...之后

这种更简单的语法也可以用`Array.prototype.find`和`Array.prototype.includes`来实现。

# 自调用匿名函数(IIFE)

当存在我们希望在浏览器解析时立即执行**的代码时，和/或如果我们不想让**用**冲突变量** s 污染名称空间**时，我们将使用`IIFE`(立即调用的函数表达式)。它本质上引入了另一个作用域。**

如果您需要在声明一个函数后立即执行它，这是非常有用的。它将**中的声明+执行组合成一个简洁的表达式。**

## 以前

## 在...之后

您可以使用**任意** [**一元运算符**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Unary) 代替`!`自行执行一个函数。

> ***一元运算符包括+、-、*** ***~***

## 要记住的重要一点

这些简写表达意在使代码**更简单**和**更清晰**。

因此，将多个速记表达式一起使用是反作用于和**初衷**的。

## 一些其他资源，以进一步你的速记游戏

[](https://www.sitepoint.com/shorthand-javascript-techniques/) [## 25+ JavaScript 速记编码技术- SitePoint

### 这确实是任何 JavaScript 开发人员的必读之作。我写了这本速记 JavaScript 编码指南…

www.sitepoint.com](https://www.sitepoint.com/shorthand-javascript-techniques/)  [## ECMAScript 6:新功能:概述和比较

### 编辑描述

es6-features.org](http://es6-features.org/#Constants) 

# 请随意指出我错过的任何东西。任何问题，我都会尽力帮忙！