# Javascript 中的有效函数参数

> 原文：<https://itnext.io/effective-function-parameters-in-javascript-1c4f2f4602ee?source=collection_archive---------3----------------------->

![](img/d1e87ff64996ce61bf42136c22297139.png)

关于编程和编程风格的最重要的书籍之一是一本名为《干净的代码》的书。在这本书里，有一个至关重要的提示是我一直试图遵循的:**函数参数的最大数量应该是 3 个，你应该总是尽量使用 2 个或更少的**。最初我并没有意识到为什么这很重要，但是随着我继续写大量的 Javascript，我开始意识到这一点。

假设我们定义了下面的函数，我们有六个潜在的参数值。

```
const addToInventory = function(itemName, category, brokerName, sellingCost, initialCost, inStock) {// Some code goes here};
```

从函数定义的角度来看，这非常有意义，因为每个参数都有变量。这些变量命名得很好，它们在函数代码中有一定的意义。然而，我们在执行函数时会遇到清晰度的问题以及设置默认参数的问题。调用这个函数可能看起来像这样

```
addToInventory("Awesome Thing", "Toys", "Chester McChester", 12, 6, false);
```

如果你和我一样，那么如果你回头再看这段代码，你会对这些值的含义感到困惑。“12”应该是什么？谁是切斯特·麦克彻斯特？为什么会有随机的“假”传入？

现在让我们假设这些参数中的一些是可选的，我们可以像这样调用函数

```
addToInventory("Awesome Thing", "Toys");
```

就清晰度而言，还不算太差……但是当你想设定初始成本时会发生什么呢？你需要设置代理名和销售成本吗？或者你可以这样做

```
addToInventory("Awesome Thing", "Toys", undefined, undefined, 6);
```

好像设置没有上下文的参数值已经够混乱的了，抛出随机的“未定义”只是增加了额外的一层“这里发生了什么事？”

# 将参数值包装在对象中！

让我们重构“addToInventory ”,改为使用一个对象来保存我们所有的参数。我将使用析构来展示如何轻松地解包 params 对象的值。如果你没有用过析构，那么我建议你去[https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Operators/destructing _ assignment # Object _ destructing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring)上了解一下。

```
const addToInventory = function(params) {const {itemName, category, brokerName, sellingCost, initialCost, inStock} = params;// Some code goes here}; // Or you could you could destructure right 
// in the function parameter declarationconst addToInventory = function({itemName, category, brokerName, sellingCost, initialCost, inStock}) {// Some code goes here};
```

现在让我们看看调用这个函数会是什么样子。

```
const params = {
  itemName: "Awesome Thing",
  category: "Toys",
  brokerName: "Chester McChester",
  sellingCost: 12,
  initialCost: 6,
  inStock: false
};addToInventory(params);// And with optional valuesconst params = {
  itemName: "Awesome Thing",
  category: "Toys",
  initialCost: 6
};addToInventory(params);
```

嘣。现在，当我稍后再回到这段代码时，我将大致了解这些值的含义，因为对象键可以为我提供指导。我明白“棒极了的东西”是项目名称，而不是其他随机值。我们也不需要以任何特定的顺序设置对象值，我们可以跳过可选值。

那么下一个问题是，对于违约你会怎么做？

# ES6 中的默认值

我在前面的例子中使用了析构，因为它有一个额外的好处。析构允许我们为从参数中提取的每个变量设置默认值(你可以在[https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Operators/destructioning _ assignment # Default _ values _ 2](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Default_values_2)了解更多信息)。

让我们看看如何使用析构来设置默认值。

```
const addToInventory = function(params) {const {itemName = 'someName', brokerName, sellingCost, initialCost = 0, inStock = true} = params;// Some code goes here};
```

使用这种技术，itemName 将默认为“someName”，initialCost 将默认为 0，inStock 将默认为“true”。

使用析构默认值对于对象来说是非常好的，但是我们最初说过，当使用一两个参数时，对象是不必要的。当我们不使用一个对象时，设置默认值怎么样？

## ES6 默认参数

我不能写一篇关于参数的文章而不提到 ES6 默认参数。如果你有三个以上的参数，你应该使用一个对象。如果没有，那么你应该检查一下 [ES6 的默认参数。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)

过去，Javascript 程序员经常不得不这样设置默认值

```
const myFunc1 = function(param1, param2) {
  if(param1 === undefined) {
    param1 = myDefault1;
  }
  if(param2 === undefined) {
    param2 = myDefault2;
  }
  // Some code goes here
};// Or the more concise and riskyconst myFunc2 = function(param1, param2) {
  param1 = param1 || myDefault1;
  param2 = param2 || myDefault2;
  // Some code goes here
};
```

这经常让开发人员感到恼火，因为其他语言有设置默认参数的简单方法。幸运的是，ES6 添加了默认参数，现在您可以这样做了。

```
const myFunc = function(param1 = myDefault1, param2 = myDefault2) {
    // Some code goes here
};
```

默认参数有助于摆脱乏味的编程，并有助于远离关于 Javascript 中的 falsy 值的[垃圾站火灾。](https://youtu.be/v2ifWcnQs6M?t=19m43s)

# ES5 中的默认值

我想为那些受困于支持旧浏览器和不能使用 [Babel 的人添加一个快速部分。](https://babeljs.io/)

如果你没有使用过 [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) ，那么我强烈建议你去看看。MDN 的官方总结称。

> “Object.assign()”方法用于将所有可枚举自身属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。

这是一种微妙但强大的方法，因为您可以轻松地合并对象，进行克隆，并根据源的顺序设置对象优先级。

让我们看看用‘addtoinventory’函数设置默认值会是什么样子。

```
const addToInventory = function(params) {
  params = Object.assign({
    itemName: "",
    category: "Other",
    brokerName: "General",
    sellingCost: 0,
    initialCost: 0,
    inStock: false
  }, params);

  const {itemName, category, brokerName, sellingCost, initialCost, inStock} = params; // Some code goes here};
```

现在，初始的“目标”被设置为所有我们想要的缺省参数，传入的 params 对象根据需要覆盖缺省值。如果您的 defaults 对象位于函数之外(比如在全局配置对象中)，您甚至可以将一个空对象设置为目标，并将默认值设置为源。

**注意，目标会发生变异，所以你不应该将全局对象设置为目标参数。**

```
const addToInventory = function(params) {
  params = Object.assign({}, myDefaults, params); const {itemName, category, brokerName, sellingCost, initialCost, inStock} = params; // Some code goes here};
```

但是如果我想使用所有的默认值呢？Object.assign 的一个妙处是它会忽略未定义的源。这意味着在上面的函数中，params 对象可以作为“undefined”传入，Object.assign 仍然可以正常工作。

# 摘要

基本上，你可以做的是清理你的函数参数并有效地设置默认值

*   将函数参数的数量保持在 3 个或更少
*   如果需要 3 个以上的参数值，请使用对象
*   如果您使用一个对象并且可以使用 ES6，请使用析构来设置默认值
*   如果您使用对象而不能使用 ES6，请使用 Object.assign 轻松设置默认值
*   如果没有使用对象作为参数，并且可以使用 ES6，请使用 ES6 默认参数
*   如果您没有使用对象作为参数，并且不能使用 ES6，您只需检查值是否未定义

希望这有所帮助！