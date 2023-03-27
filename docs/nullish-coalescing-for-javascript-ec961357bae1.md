# JavaScript 的无效合并

> 原文：<https://itnext.io/nullish-coalescing-for-javascript-ec961357bae1?source=collection_archive---------4----------------------->

![](img/c277ebab3a14eea8f35fd7d236247171.png)

Nullish 合并提案已经被移到第 3 阶段，也就是说，很快它将被添加到 JS 标准中，让我们看看它如何帮助我们。

你检查过多少次一个变量是否是`null`？不是`undefined`、`''`或`false`，而只是`null`，我通常会为此添加一个 if 条件`variable === null`，我已经这样做了无数次。

考虑下面的代码

```
letcounter;
if(*response.*data === null) counter = 1;
else counter = *response.*data;
```

如果我们可以轻松地做到这一点，而不需要那么多代码来检查它是否是`null`会怎么样。无效合并就是这样做的。这是它的外观和工作原理。

```
letcounter= *response.data* ??1;
// *now if data is 0 counter value is 0* // *if it is null or undefined we set it to 1;*
```

因此，只有当值为`undefined`或`null`时，才会使用默认值。

```
result = actualValue ?? defaultValue
```

让我们看看当我们使用逻辑 OR 运算符时会得到什么。

```
letcounter= *response.data* ||1;
// *here even if the value is 0, that is the value is defined* // *we still get 1, which we don't want.*
```

概括地说,“无效聚结”本质上是

```
a ?? b
a !== undefined && a !== null ? a : b
```

# 当前状态和如何使用

您可以查看 [ECMAScript 下一个兼容性表](http://kangax.github.io/compat-table/esnext/#test-nullish_coalescing_operator_(??)),以了解。？支持运算符。

巴别塔有插件`@babel/plugin-proposal-nullish-coalescing-operator`

*原载于 2019 年 11 月 29 日*[*https://solankiamit.com*](https://solankiamit.com/nullish-coalescing-for-javascript)*。*