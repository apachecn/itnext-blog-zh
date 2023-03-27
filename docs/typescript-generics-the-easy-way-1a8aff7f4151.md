# TypeScript 泛型——The！休闲小站

> 原文：<https://itnext.io/typescript-generics-the-easy-way-1a8aff7f4151?source=collection_archive---------1----------------------->

寻找一个伟大的(仅远程)反应开发？或者只是想聊聊天？访问 LinkedIn 上的[我的个人资料](https://www.linkedin.com/in/bengrunfeld/)并问好！😃

![](img/f10408d7af7cf4ea807a9b2888c01ce6.png)

在我的 TypeScript 之旅中，最令人讨厌的部分可能是泛型。语法从一开始就把我搞糊涂了，而且这些概念并不像其他关于 TypeScript 的东西那样“自然地”有意义。

所以我做了每一个理性拖延者会做的事情，试图避免他们的地狱…这是一个悲惨失败的策略。泛型无处不在，从像 Apollo 这样常用的库，到你需要参与并帮助的小项目，所以处理它们的唯一方法是学习它们，练习使用它们，并从本质上克服你对它们的厌恶。强行打开它——没有别的办法了。

注意:我不会在本文中讨论类，所以如果你想将泛型应用于 JS 类，请参考[文档](https://www.typescriptlang.org/docs/handbook/generics.html#generic-classes)。

泛型就像变量，只不过它们不是表示值，而是表示类型。例如

```
function getAge<T>(age: T): T {
  return age;
}const myAge = getAge<number>(40);console.log(myAge);    // logs 40
```

很丑，不是吗？如果代码的可读性是最重要的，那么*这个*是一个怎样的改进呢？嗯，你的猜测和我的一样好，所以让我们一起去喝打字稿吧…😕。

那么上面的代码是怎么回事呢？首先，我们声明一个将`age`作为参数的函数，并简单地返回它。你在开头看到的尖括号`<T>`意味着这个函数采用了一种类型的`T`。因为这是一个*变量*，所以您可以继续使用它，就像使用任何其他变量一样。所以我们用它来声明`age`的类型是`T`，然后函数返回一个类型为`T`的值。

现在，`T`可以是任何有效的类型脚本[基本类型](https://www.typescriptlang.org/docs/handbook/basic-types.html)。此外，我们可以使用任何名称或字母，如`U`或`Name`。我们甚至可以使用小写字母，比如`n`，尽管惯例是使用大写字母或首字母大写的单词，例如`Key`。

然后我们继续调用函数`getAge`，现在*我们定义我们希望`T`是什么——即`number`。最后，我们退出`myAge`,然后去拿一杯冰啤酒，在意识到我们已经正式进入“中年”后，对着它静静地哭泣…😭🍺🍺🍺*

## *类型推理*

*TypeScript 有一个称为**类型推断**的特性，这意味着如果类型足够简单，您不必总是声明正在使用的类型。编译器可以算出来。所以在上面的例子中，我们实际上不需要指定`number`，因为 TypeScript 会从我们给它的输入中计算出来。也就是说，我们可以像这样调用函数，但一切都是正常的，并且是类型安全的:*

```
*const myAge = getAge(40);*
```

## *粗箭头语法*

*如果你像我一样，你自然会尽可能地使用胖箭头语法来声明函数，因为它们真的很酷。不幸的是，您需要使用一种变通方法来让 TypeScript 使用粗箭头，所以上面的函数声明是这样写的:*

```
*const getAge = <T,>(age: T): T => age;*
```

*注意到`T`后面的逗号`,`了吗？又一次，丑陋，但是必要的(奇怪的是，这正是我的前妻曾经向她的朋友们描述我的方式…😕)*

## *多类型变量——痛苦错误的教训*

*所以以上可能开始有意义了，但是不要担心，TypeScript 将很快增加一个新的复杂性层，使您保持不变。*

*带着你新找到的自信，你可能会尝试这样的事情，但结果却是直接面对一个冰冷的错误。*

```
*const divide = <T, U>(first: T, second: U): T => first / second;const result = divide<number, number>(40, 2);// ERROR: Type 'number' is not assignable to type 'T'. 'T' could be instantiated with an arbitrary type which could be unrelated to 'number'.*
```

*好的，所以返回值`T`意味着你只能返回未掺杂和未修改的`first`。嗯，还不算太糟，是吧？*

*对自己傻笑，你认为你已经想通了，并指定了一个返回值`number`，却发现 TypeScript 的人不太喜欢你，或者任何人…*

```
*const divide = <T, U>(first: T, second: U): number => first / second;const result = divide<number, number>(40, 2);// ERROR: The left-hand side of an arithmetic operation must be of type 'any', 'number', 'bigint' or an enum type.*
```

*什么？！？！但是我们在调用函数的时候指定了`number`。这不就是泛型的全部意义吗？？拥有一个可以稍后设置的变量类型。*

***解决方案***

*你需要在泛型上设置约束，因为否则你的程序会正常工作。嗯，我的意思是，否则 TypeScript 会有意义…嗯，我的意思是:TypeScript 还会检查参数在您的函数中是如何使用的(就像令人毛骨悚然的老大哥)，如果它认为让事情过于一般化可能会导致错误，那么它会生成一个错误并吐出这个哑元。*

*基本上，你必须约束泛型允许的类型，使之与编译器认为你应该在函数中使用的类型一致——这意味着无论如何它都不再是泛型了。*

*所以基本上在这种情况下使用泛型是没有意义的，因为你最终还是要设置类型，就像普通的函数定义一样。我不知道——也许这对比我更伟大的人来说是有意义的。*

```
*const divide = <T extends number, U extends number>(
    first: T,
    second: U
  ): number => first / second;const result = divide<number, number>(40, 2);*
```

*请记住，由于类型推断，您可以省略`number`定义，TypeScript 会解决这个问题。例如*

```
*const result = divide(40, 2);*
```

*仅供参考—在许多财务 API 中，由于浮点和 Javascript，您必须处理表示为字符串的数字。如`"0.000001"`。*

*TypeScript 足够聪明，它能判断出如果你在字符串上使用`parseInt`或`parseFloat`，然后对它们执行数学运算，这是合法的操作，而不是上面那个讨厌的错误。也就是说，这段代码的工作原理是:*

```
*const divide = <T extends string, U extends string>(
    first: T,
    second: U
  ): number => parseFloat(first) / parseFloat(second);const result = divide("0.000001", "0.00000005");*
```

# *通用接口*

*我们可以在接口中使用泛型来动态设置对象属性的类型。*

```
*interface KeyVal<K, V> {
    key: K;
    value: V;
  }const keyval: KeyVal<string, number> = {
    key: "MyKey",
    value: 748129,
  };console.log(keyval);*
```

*上面可以看到，`KeyVal`把`K`和`V`作为自变量(变量类型)，然后用它们来为`key`和`value`设置类型。它们变成的实际类型可以稍后在代码中指定。*

## *函数中的通用接口*

*我们也可以在接口中使用泛型来声明函数的类型。*

```
*interface GenericDivision {
    <T extends number, U extends number>(first: T, second: U): number;
  }function division<T extends number, U extends number>(
    first: T,
    second: U
  ): number {
    return first / second;
  }const divide: GenericDivision = division;console.log(divide(20, 2))      // log 10*
```

*下面是一个返回混合值对象的工作代码示例:*

```
*interface KeyVal {
    key: string;
    value: number;
  }interface GenericMakeKeyVal {
    <T extends string, U extends number>(key: T, value: U): KeyVal;
  }function makeKeyVal<T extends string, U extends number>(
    key: T,
    value: U
  ): KeyVal {
    return {
      key,
      value,
    };
  }let getKeyVal: GenericMakeKeyVal = makeKeyVal;console.log(getKeyVal("id", 12345));*
```

# *结论*

*在了解了它们之后，我必须承认我不喜欢泛型。就像我上面说的，一旦你想在一个泛型中操作任何参数，你就必须声明泛型的类型，就像你在一个普通的函数定义中一样——那么它们有什么意义呢？*

*正如 **bugcatcher9000** [在我的 StackOverflow 问题中评论](https://stackoverflow.com/questions/62980992/typescript-generics-error-left-hand-side-of-an-arithmetic-operation-must-be-of/62981068#62981068)的那样，它们最有用的功能可能是声明泛型接口。*

*也就是说，它们真的在任何地方都被使用，所以希望这篇文章能帮助你理解它们。*

*祝你好运！*