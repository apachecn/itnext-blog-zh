# Var，Let & Const: JavaScript ES6 特性系列(第 1 部分)

> 原文：<https://itnext.io/var-let-const-javascript-es6-feature-series-pt-1-fa603567809e?source=collection_archive---------2----------------------->

# 让我们从头开始…

![](img/760087da8bd112730fabff4efd137aee.png)

苏珊·霍尔特·辛普森在 [Unsplash](https://unsplash.com/search/photos/building-blocks?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

# 介绍

这些帖子背后的灵感很简单:对于很多开发人员来说，JavaScript 是一个谜——或者至少是不太了解。

再加上 ES6，以及 ECMAScript 委员会现在计划每年发布的更新，有很多信息需要及时了解。

抱歉，我控制不了自己——我从遇见你妈妈的时候就爱上了巴尼。

我想提供一些关于我经常使用的 JavaScript ES6 特性的帖子，供开发人员参考。

我们的目标是让这些文章简短、深入地解释该语言的各种改进(以及如何使用它们)，我希望这些文章能启发您使用 JS 编写一些真正酷的东西。谁知道呢，在这个过程中你可能会学到一些新东西。😄

> 对于我的第一篇文章，从`var`、`let`和`const`开始是有意义的:JavaScript 的构建块。

# 定义变量

`[var](https://en.wikipedia.org/wiki/JavaScript_syntax#Variables)`是最初的 JavaScript 语法:用 OG 的方式声明一个变量，并可选地将其初始化为一个值。在 ES6 成为一种语言之前，它就已经存在了。

在 JavaScript 中，变量没有预定义或附加的类型，这意味着任何值都可以存储在任何变量中。一些例子:

## **给变量赋值**

```
var one = 1; // the integer 1 has just been assigned to the variable onevar two = 'two'; // the string 'two' has been assigned to the variable twovar paige = {
  gender: female,
  hairColor: red,
  eyeColor: blue
};
// the object containing a person's traits has been assigned to the variable paige
```

上面所有的`var`都是 JavaScript 允许将任何值赋给任何变量的有效例子，不像 Java 这样的强类型语言，Java(直到最近)要求所有变量在编译前声明和类型化。

关于`var`要知道的另一件事是，不管它是全局作用域还是局部作用域，这些变量都可以随意重新声明和更新。看看这个完美的例子。

## **重用不同值类型的相同变量**

```
var greet = ‘Hello there’;console.log(greet); // prints: ‘Hello there’greet = trueconsole.log(greet); // prints: truegreet = 11;console.log(greet); // prints: 11greet = welcome = (name) => { 
  return `Hi ${name}. Welcome!`
};console.log(greet("Paige")); // prints "Hi Paige. Welcome!"
```

尽管我重用了现有的变量`greet`，JavaScript 没有抛出任何错误，不管变量是字符串、布尔值、整数——见鬼，甚至是函数。

## 变量声明、词法范围和提升

在使用`var`时，有一些事情需要注意(这已经让很多开发人员犯了错误，包括我自己)。这些类型的变量是“词汇作用域”的，并且受制于所谓的“提升”

这在执行中意味着一个`var`类型的变量，不管它是在代码块中的什么地方定义的，都将在代码执行前被拉到其作用域的顶部。例如，如果你写了几行 JavaScript，然后声明了`var tree = 'elm';`，在运行时，在后台，`var tree`将被“提升”到它被定义的作用域的顶部。

它可能直到几行之后才被赋值给`'elm'`的值，但是根据 JavaScript 引擎，这个变量将会存在。这里有一个例子来说明这是如何工作的，以及它可能意味着什么。

**JavaScript 中的变量声明与变量赋值**

```
var greeter; // the JS engine knows greeter is a variable, but it doesn't know what value greeter hasconsole.log(greeter); // currently, greeter prints: undefinedgreeter = “say hello”; // now, greeter's been assigned to the value 'say hello'console.log(greeter); // prints: 'say hello'
```

词法范围在这里也发挥了作用。不要太深入 JavaScript 中词法范围的细节(因为那是一篇完整的 *other* 博客文章)，单词“词法”指的是这样一个事实，词法范围使用变量在源代码中声明的位置来确定该变量在哪里可用。

这可以归结为，如果一个变量是在全局范围内创建的，它就被提升到全局范围的顶部，如果这个变量是在一个函数内创建的，它就被提升到函数的顶部，以此类推。

下面是 JavaScript 引擎运行时变量提升和词法作用域的例子。

**基于词法范围的变量提升**

```
var something; // declaration is hoisted to the top of the global scopeconsole.log(something); // prints: undefinedfunction doAnotherThing() {
  var anotherThing; // declaration is hoisted to the top of the function's lexical scope but not immediately assigned a value var thisOneThing = 'a defined value'; // this variable is hoisted and then immediately defined function useAnotherThing(anotherThing) {
    console.log(`The value of another thing: ${anotherThing}`);
  } function useThisOneThing(thisOneThing) {
    console.log(`The value of this one thing: ${thisOneThing}`)
  } useAnotherThing(anotherThing); // prints: "The value of another thing: undefined" because the variable wasn't assigned a value until AFTER the function useAnotherThing() ran anotherThing = 'an undefined value'; // finally the variable, initialized earlier, is assigned a value useThisOneThing(thisOneThing); // prints: "The value of this one thing: a defined value" because the variable was assigned a value before the function useThisOneThing() ran
}something = 'that thing';console.log(something); // prints: "that thing"console.log(doAnotherThing()); // see printouts above next to inner functions
```

在上面的例子中，我展示了变量`something`和`anotherThing`在被声明后没有被立即赋值，当`console.log()`被调用时它们是`undefined`。一旦它们的值在脚本中被进一步设置，并且再次调用`console.log()`，它们就将它们的值打印到控制台。

然而，变量`thisOneThing`在初始化后立即被初始化并赋值，这意味着当使用它的函数`useThisOneThing()`运行时(这也是 JavaScript [闭包](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)工作的一个例子)，它打印出字符串`“The value of this one thing: a defined value.”`

关于`var`和它是如何工作的，我想这就足够了。还有上百万的其他教程也在讨论这个问题。现在，让我们来看看令人兴奋的新 ES6 变量语法，以及这些类型的变量有什么不同。

# 让

`[let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)`最终确定，并随着 2015 年 ES6 的发布正式引入广泛使用。在引入的两个新变量类型中，它是一对变量中更灵活和短暂的。

## 让我们来看看 Var 的相似之处

虽然`let`是一个全新的变量类型，但它确实与`var`有一些共同之处。例如:

*   当`let`被初始化时，可以选择在初始化时给它赋值，
*   `let`也是一种变量类型，可以在运行时变异和/或重新分配给不同的数据类型。

相似之处也就到此为止了。

## 让我们分歧到`Var`

好了，这就是有趣的地方，`let`开始与`var`不同。

## **块范围&变量声明和赋值**

`let`是语句范围内的块，或使用它的表达式，不像`var`，它定义一个全局变量，或局部变量到整个函数，而不管块的范围。

底线:`let`存在于(通常)由像 so `{}`这样的花括号定义的代码块中。

另外，`let`的值可以更新，但不能重新声明为同一作用域内的新变量，而`var`类型的变量可以。下面有一个例子展示了`let`和`var`的区别。

**基于块范围**的 `**let**` **的有效赋值**

下面是一个在同一个文件中使用关键字`let`给变量`greeting`赋值的有效例子，但是在不同的块范围内。

```
let greeting = “say Hi”; // one variable declared in the global scopeif (true) {
  let greeting = “say Hello instead”; // another variable of the same name declared within this inner, block scope console.log(greeting); // prints: ”say Hello instead”
}console.log(greeting); // prints: ”say Hi”
```

**由于块范围**，无效的 `**let**` **赋值**

由于这个`let greeting`变量在全局范围内被赋值，它不能被重新声明并在相同的范围内被重新赋值。如果出现这种情况，它会在控制台中抛出一个`SyntaxError`。

```
let greeting = “say Hi”;let greeting = “say Hello instead”; //SyntaxError: Identifier ‘greeting’ has already been declared
```

用`var`做同样的事情不会产生这样的错误。

```
var fruit = "banana";console.log(fruit);  // prints: "banana"var fruit = "pear";console.log(fruit); // prints: "pear"
```

## **可变提升(或无提升)和全局对象**

吊装不适用于`let`。与`var`不同的是，`let`类型的变量在解析器对其定义求值之前不会被初始化为`undefined` *。*

如果你试图在代码的初始化部分之前使用一个`let`变量，你会得到一个`ReferenceError`抛出。

**引用** `**let**` **的错误，因为它在解析**之前被调用

```
console.log('test 0', test)let test; // prints: ReferenceError: test is not defined
```

在程序和函数的顶层，`let`与`var`不同，当在最顶层作用域中声明时，也不在全局`window`对象上创建属性。这不一定是一件坏事，这意味着对全局名称空间的污染更少，我们都可以从中受益，但这也意味着您不能使用经常被误解的`this`属性来访问上述全局变量。

**没有带有**和`**let**`的全局对象属性

```
var x = 'global';let y = 'global';console.log(this.x); // prints: "global"console.log(this.y); // prints: undefined
```

好了，我认为关于`let`已经足够了，是时候转向更严格、更不可变的变量类型`const`了。

# 常数

`[const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)`出现在与 2015 年`let`相同的 ES6 版本中。但是`let`提供了更大的灵活性和更少的永久性变量，`const`提供了更多的刚性和永久性。

到目前为止，它也是三种变量类型中最不同的一种。

## Const 的定义特征

这个变量声明与 let 共享一些东西。

*   `const`也是一样，仅限于块范围界定，
*   当在全局范围内创建时，它不会在`window`对象上创建属性，
*   当它被声明时，它也必须被赋予一个值，
*   并且在运行时不会被提升。

## 块作用域

Const 和`let`有相同类型的块作用域，但是一旦`const`被用来声明一个变量，这个值就不能通过重新赋值或者在这个作用域内重新声明来改变。

`**const**`T20【重新分配类型错误】并重新声明同一范围内的语法错误

```
const pet = "dog";console.log(pet);  // prints: "dog"pet = "cat"; // TypeError: Assignment to constant variableconsole.log(pet);const person = "Sean";const person = "George"; // SyntaxError: Identifier 'person' has already been declared
```

将现有的`pet`变量指向不同的值或者试图在相同的范围内重新声明`person`都会导致控制台出错。

但是考虑到块范围，下面的例子没有任何问题。

**使用**T5 对变量进行有效块范围界定

```
const pet = "dog";console.log(pet);  // prints: "dog"if (true) {
  const pet = "cat"; // TypeError: Assignment to constant variable
  console.log(pet);  // prints: "cat"
}const person = "Sean";console.log(person); // prints: "Sean"function meet() { 
  const person = "George"; 
  console.log(`Meet ${person}`);
}console.log(meet()); // prints: "Meet George"
```

## 不变性(某种程度上)

`const`创建一个对值的只读*引用。尽管这个声明的警告是值本身是可变的，但是`const`一旦被赋值，就不能被重新赋值。*

如果`const`被分配给一个对象，该对象内的属性仍然可以被更新和改变。

`**const**` **对可更新对象的引用**

```
const greeting = {
  message: ‘Hello there’,
  person: ‘Paige’
};console.log(greeting); // prints: { message: ‘Hello there’, person: ‘Paige’ }greeting.message = ‘Hi there’;console.log(greeting); // prints: { message: ‘Hi there’, person: ‘Paige’ }
```

## 同时声明、初始化且无全局对象

像`const`这样的全局常量**而不是**会成为`window`对象的属性，不像`var`变量。常数的初始值设定项也是必需的；也就是说，您必须在声明它的同一个语句中指定它的值(这是有意义的，因为它以后不能更改)。

`**const**` **不会用对象属性污染全局范围**

```
var x = 'global';const y = 'global';console.log(this.x); // prints: "global"console.log(this.y); // prints: undefined
```

如果`const`在初始化时没有赋值，它将抛出一个`SyntaxError`。同样，如果一个脚本试图在解析器到达变量之前调用它，它将抛出一个 ReferenceError，就像`let`一样，因为`const`不需要提升，并且在赋值之前不能设置为`undefined`。

**声明前调用变量或赋值初始化时出错**

```
console.log(fruitTree); // ReferenceError: fruitTree is not definedconst fruitTree = "fig tree";const tree; // SyntaxError: Missing initializer in const declaration
```

这也是关于`const`的所有信息。

# 结论

JavaScript 语言已经存在了 20 多年，甚至 ES6 在 2015 年就已经问世，但即使如此，关于它仍然有很多误解和知识差距。

本系列文章的目的是消除这些误解，让您更好地理解 JavaScript，并介绍一些您可能每天都在使用但没有完全掌握其细微差别的 ES6 语法。

因为 JavaScript 中的一切都是围绕变量构建的，所以从`var`、`let`和`const`开始似乎是一个合适的地方。

过几周再来看看，我会写更多关于 JavaScript 和 ES6 或其他与 web 开发相关的东西，所以请关注我，这样你就不会错过了。

感谢阅读，我希望你学到了一些关于 JS 最基本的构建模块的新东西:不起眼的变量。如果你觉得有帮助，请与你的朋友分享！

**如果你喜欢读这篇文章，你可能也会喜欢我的其他博客:**

*   [JavaScript 国际方法](/javascript-international-methods-b70a2de09d92)
*   [使用 ES6 在 JavaScript &中析构深度嵌套的对象，避免未定义的错误破坏你的代码](/using-es6-to-destructure-nested-objects-in-javascript-avoid-undefined-errors-that-break-your-code-612ae67913e9)
*   [JavaScript 的异步/等待与承诺:大辩论](/javascripts-async-await-versus-promise-the-great-debate-6308cb2e10b3)

**参考资料和更多资源:**

*   Var，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Statements/var](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var)
*   Let，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Statements/let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
*   Const，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Statements/const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
*   JavaScript 语法，维基百科:[https://en.wikipedia.org/wiki/JavaScript_syntax](https://en.wikipedia.org/wiki/JavaScript_syntax)