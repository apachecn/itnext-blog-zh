# 箭头函数:JavaScript ES6 特性系列(第 2 部分)

> 原文：<https://itnext.io/arrow-functions-javascript-es6-feature-series-pt-2-e8c31c823392?source=collection_archive---------1----------------------->

# 什么时候函数不是函数？当它是箭头时

![](img/ce7dc680182913ecca27dbf595fcc79a.png)

由[罗马法师](https://unsplash.com/@roman_lazygeek?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/teaching?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

# 介绍

这一系列帖子背后的灵感很简单:仍然有很多开发人员认为 JavaScript 有时毫无意义——或者至少，与其他编程语言相比，JavaScript 的行为看起来很奇怪。

因为它是如此流行和广泛使用的语言，所以我想提供一些关于我经常使用的 JavaScript ES6 特性的帖子，供开发人员参考。

我们的目标是让这些文章简短，但仍能深入解释对该语言的各种改进，我希望这些文章能启发您使用 JS 编写一些真正酷的东西。谁知道呢，在这个过程中你可能会学到一些新东西。😄

> 在我的第二篇文章中，我想深入探讨箭头函数，以及它们与传统的函数声明和函数表达式有何不同。

# 函数声明

您可能以前听说过这一点，但它值得重复:在 JavaScript 中，函数是一级对象，因为它们可以像任何其他对象一样拥有属性和方法。它们与其他对象的区别在于函数可以被调用。简而言之，它们是`[Function](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function)`物体。

从现在开始，我假设您熟悉 JavaScript 中函数的一般概念，但是在我讨论箭头函数之前，有必要先讨论一下[函数声明](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)(也称为函数语句)和[函数表达式](https://developer.mozilla.org/en-US/docs/web/JavaScript/Reference/Operators/function)。

函数声明是 JavaScript 中最基本的函数语句。它定义了一个函数以及它需要运行的指定参数。这里有一个例子:

## **解析一个函数声明**

```
function multiply(number1, number2){
  return number1 * number2;
}console.log(multiply(4, 9));  // prints: 36 to the console
```

如果你在看上面的函数声明例子，下面是组成函数的内容。`multiply`是函数名，`number1`和`number2`是函数接受的两个参数，函数体`return number1 * number2;`是语句。

## 函数声明特征

开发人员在编写代码时需要记住函数声明的某些特性，因为它们总有一天会让你犯错——它们会让我们所有人犯错(包括我自己)。🙋

## **函数返回未定义，除非另有说明**

默认情况下，函数返回`undefined`。通过在正文中包含`return`关键字，您可以指定它返回的值。

**未定义的函数声明与返回值**

```
function returnsNothing(item1, item2) {
  item1 + item2;
}console.log(returnsNothing(1, 9)); // prints: undefinedfunction returnsSomething(item1, item2) {
  return item1 + item2;
}console.log(returnsSomething(1, 9)); // prints: 10
```

对于我上面的例子，`item1`和`item2`的和是从`returnsSomething` 函数返回的，而`returnsNothing`函数虽然做完全相同的加法，但当用`console.log()`调用该值时，它仅仅返回 undefined。

## **函数声明被挂起**

类似于我在上一篇[博文](/var-let-const-javascript-es6-feature-series-pt-1-fa603567809e)中讨论的变量提升，JavaScript 中的函数声明被提升到封闭函数或全局范围的顶部。这意味着，你可以在一个函数被代码声明之前使用它。

**函数声明提升与函数表达式不提升**

```
console.log(hoistedFunction()); // prints: Hello, I work even though I am called before being declaredfunction hoistedFunction() {
  return 'Hello, I work even though I am called before being declared';
}console.log(notHoisted()); // prints: TypeError: notHoisted is not a functionvar notHoisted = function() {
  return 'I am not hoisted, so I will not be found if called before my declaration';
}console.log(notHoisted()); // prints: I am not hoisted, so I will not be found if called before my declaration
```

在上面的例子中，函数`hoistedFunction()`返回它的值，不管它何时在代码中被调用，因为它是一个函数声明。

另一方面，赋给变量`notHoisted()`的第二个函数(它是一个函数表达式)没有被提升到作用域的顶部，因此如果它在函数被解析之前被调用，它会在代码中抛出一个`TypeError`，表明它不是一个函数(主要是因为编译器还不知道它)。

当考虑函数声明时，这些是你需要知道的主要事情。让我们继续讨论函数表达式。

# 函数表达式

[函数表达式](https://developer.mozilla.org/en-US/docs/web/JavaScript/Reference/Operators/function)类似于函数声明。它们仍然有名称(这次是可选的)、参数和基于主体的语句。

## 函数表达式的剖析

```
const divide = function(number1, number2){
  return number1 / number2;
}console.log(divide(15, 5)); // prints: 3
```

对于这个例子，变量`divide()`被分配给匿名函数，该函数接受参数`number1`和`number2`，并根据主体语句`return number1 / number2;`返回商。

## 功能表达特征

就像函数声明一样，函数表达式也有自己独特的定义。以下是你需要知道的关于他们的事情。

## 函数表达式可以是匿名的(也可以不是)

正如我上面简单提到的，函数表达式，因为它们被赋给了变量，所以可以省略名字，成为所谓的“匿名函数”。这是可能的，因为变量名将被*隐式地*分配给函数。

**匿名函数表达式(隐式命名在起作用)**

```
const anonymous = function() {
  return 'I do not need my own name, as I am assigned to the variable anonymous';
}console.log(anonymous()); // prints: I do not need my own name, as I am assigned to the variable anonymous
```

正如变量名所暗示的，因为它引用的函数没有名字，所以它隐式地将`anonymous()`赋给该函数。

然而，如果你想在函数体内引用当前函数，你需要创建一个*显式*命名的函数(它的名字只在函数体内是局部的)。

**命名函数表达式(工作时显式命名)**

```
var math = {
  'factit': function factorial(n) {
    console.log(n)
    if (n <= 1) {
      return 1;
    }
    return n * factorial(n - 1);
  }
};

console.log(math.factit(3)); //prints: 3; 2; 1;
```

对于这个变量`math`，您可以通过在对象外部调用`math.factit();`来调用`factorial()`函数，并传入所需的参数。在我的日常开发中，我并没有发现对这种类型的命名函数表达式有太多的需求，但是如果需要的话，知道它是可用的就好了。

**底线:**如果函数表达式的名字被省略，那么它将被赋予变量名(隐式名)。如果函数表达式的名称存在，它将是分配的函数名称(显式名称)。

## 函数表达式可以是生命

函数表达式可以作为一个生命来使用:一个[立即调用函数表达式](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)，函数表达式一旦被定义就运行。

JavaScript 引擎的这种立即执行是由匿名函数末尾的`()`触发的。

我在这里不会讲太多细节，但是对于在生命中创建的任何变量，要让外部或全局范围可以访问，匿名函数必须像函数表达式一样被赋予一个变量。

如果不是这样，并且匿名函数只是在运行时被调用，那么在函数作用域内创建的任何变量对外界都是不可见的。

**可访问的生活变量与不可访问的生活变量**

```
const cogitoErgoSum = (function () {
  const quote = "I think therefore I am";
  return quote;
})();// immediately creates the output
cogitoErgoSum; // prints: I think therefore I am(function (){
  const quote2 = "I am not outside this IIFE";
})();quote2; // prints: ReferenceError: quote2 is not defined
```

## **函数表达式不提升**

函数表达式(和箭头函数)跟在新的`let`和`const`变量关键字之后，因为它们在运行时不会被提升。

正如我在上面关于提升的函数声明一节中所演示的，函数表达式*不*提升，当执行到达函数表达式时，函数表达式被创建，并且从那时起它是可用的。

**函数声明提升与函数表达式不提升**

```
console.log(hoistedFunction()); // prints: I am a function declaration so I am hoisted to the top of the scope at run timefunction hoistedFunction() {
  return 'I am a function declaration so I am hoisted to the top of the scope at run time';
}console.log(stillNotHoisted()); // prints: TypeError: stillNotHoisted is not a functionconst stillNotHoisted = function() {
  return 'I am a function expression and therefore, hoisting does not apply to me';
}console.log(stillNotHoisted()); // prints: I am a function expression and therefore, hoisting does not apply to me
```

结果和我在函数声明中描述的一样，如果函数表达式在运行时解析之前被调用，就会抛出`TypeErrors`。就是不做。

好了，现在是时候讨论箭头功能了:ES6 最新最大的功能改进。

# 箭头函数表达式➡️

![](img/757128f99426ada0e576bb7ac5416484.png)

最基本的箭头函数语法。

[箭头函数表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)是正则函数表达式的语法紧凑替代。

## 剖析两种基本箭函数表达式

```
const basicArrow = () => {
  return 'The most basic of basic arrow functions';
}basicArrow(); // prints: The most basic of basic arrow functionsconst basicArrow2 = oneParam => 'Single line with ${oneParam } is also valid';basicArrow2('only one param'); // prints: Single line with only on param is also valid 
```

上面的两个例子`basicArrow()`和`basicArrow2()`都是箭头函数的有效例子。与所有的函数表达式一样，这两个匿名函数都是由分配给它们的变量隐式命名的。

但不同的是，关键字`function`是不必要的，相反，如果没有必需的参数，它被一组括号`()`代替，即`basicArrow2()`所需的单个参数的名称，即`oneParam`(在这种情况下不需要括号)，或者，对于任何其他数量的参数，您可以使用`(paramOne, paramTwo, paramThree, ...)`。

类似地，第一个函数在函数体内有一个普通的`return`语句，用花括号`{}`括起来，但是，如果这个语句非常简单，并且您可以将返回结果放在一行中，那么实际的`return`和花括号也可以省略，就像在`basicArrow2()`中一样。这是带有隐含 return 语句的简洁主体语法。

## 箭头功能特征

虽然箭头函数乍一看很容易识别，但它们实际上有一些奇怪的、特定的特征，开发人员需要记住这些特征。

除了简洁的语法，arrow 函数缺少 [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) 、 [arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments) 、 [super](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super) 或 [new.target](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target) 关键字。这些事实也导致了 arrow 函数的一个最大缺点:它们不适合作为方法，也不能用作构造函数。我将很快更详细地讨论这个问题。

## 较短的函数语法

在我看来，正则函数表达式的第一个也是最大的改进是 arrow 函数提供的更短、更简洁的语法。

这是一个与传统函数表达式完全相同的函数，然后再写成一个箭头函数表达式。

**传统函数表达式:**

```
var elements = [‘Hydrogen’, ‘Helium’, ‘Lithium’, ‘Beryllium’];elements.map(function(element) {
  return element.length;
});// this statement returns the array: [8, 6, 7, 9]
```

**新箭头函数表达式:**

```
var elements = [‘Hydrogen’, ‘Helium’, ‘Lithium’, ‘Beryllium’];elements.map((element) => element.length);// this statement still returns the same array: [8, 6, 7, 9]
```

看看这些，告诉我第二个是不是更容易阅读，并遵循代码在做什么。这本身就是我想尽可能使用箭头函数的最大原因。只是干净和清晰多了。

## 吊装仍然不适用

就像传统的函数表达式一样，提升仍然不适用于箭头函数。

**没有吊装，只有**

```
console.log(fish()); // prints: TypeError: fish is not a functionconst fish = () => ['perch', 'salmon', 'trout', 'bass'];console.log(fish()); // prints: [ 'perch', 'salmon', 'trout', 'bass' ]
```

如果你试图在代码中声明之前调用`fish()`变量，就会抛出`TypeError`。和以前一样，解决方案是要么将函数声明为函数声明，这样它就被提升到作用域的顶部，要么等到函数表达式之后再调用代码。

## **没有单独的“这个”**

在 arrow 函数之前，每个新函数都根据函数的调用方式定义了自己的`[this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)`值:

*   在构造函数的情况下是一个新对象，
*   `undefined`在严格模式函数调用中，
*   如果函数作为“对象方法”被调用，

另一方面，箭头函数没有自己的`this`。使用封闭词法范围的`this`值；箭头函数遵循普通的变量查找规则，从当前作用域级别开始，一直搜索到最高级别来查找变量。因此，在搜索当前范围内不存在的`this`时，一个箭头函数最终从其封闭范围内找到了`this`对象。参见下面的例子，了解不同之处。

`**This**` **作用域，显示函数声明**

```
function Person() {
  // The Person() constructor defines `this` as an instance of itself.
  this.age = 0;

  setInterval(function growUp() {
    // In non-strict mode, the growUp() function defines `this`
    // as the global object (because it's where growUp() is executed.), 
    // which is different from the `this`
    // defined by the Person() constructor.
    this.age++;
  }, 1000);
}

var p = new Person();
```

`**This**` **范围，显示箭头功能**

```
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| properly refers to the Person object
  }, 1000);
}

var p = new Person();
```

## **参数没有绑定**

除了不能访问`this`之外，arrow 函数没有自己的`[arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)` [对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)。

因为您可能没有听说过它们，`arguments`是一个函数中类似于`Array`的对象，包含传递给该函数的值。我说`Array` -like 是因为这个`arguments`有一个长度属性和索引，但是它缺少 Array 的内置方法，比如`.forEach()`和`.map()`。

因此，在这个例子中，`arguments`只是对封闭范围的参数的引用。

`**arguments**` **带箭头功能**

```
var arguments = [1, 2, 3];
var arr = () => arguments[0];

arr(); // prints: 1

function foo(n) {
  var f = () => arguments[0] + n; // foo's implicit arguments binding. arguments[0] is n
  return f();
}

foo(3); // prints: 6
```

在大多数情况下，使用 rest 参数是使用`arguments`对象的一个很好的替代方法。

**休息参数用箭头功能**代替 `**arguments**`

```
function foo(n) { 
  var f = (...args) => args[0] + n;
  return f(10); 
}

foo(1); // prints: 11
```

**rest 参数，我将在本系列的另一篇博文中讨论，是 ES6 推荐的访问和操作 arrow 函数内部的`arguments`的方法。**

## **不使用“New”作为构造函数**

**好了，最后要知道的箭头函数特性:箭头函数不能作为构造函数使用，当和`new`关键字一起使用时会抛出一个`TypeError`。**

```
var Foo = () => {};
var foo = new Foo(); // prints: TypeError: Foo is not a constructor
```

**仅此而已。这就是关于箭头函数你需要知道的。简单！😅**

# **结论**

**JavaScript 是一种非常强大的编程语言，它的受欢迎程度只会继续增加(如果年度开发者调查可信的话)。尽管有如此多的开发人员使用它，但肯定会有一些误解和知识差距，特别是随着 ES6 越来越广泛地被日常使用。**

**我的目标是阐明您日常使用的一些 JavaScript 和 ES6 语法，但可能从未完全理解其工作方式的细微差别。**

**各种类型的函数都是 JavaScript 的主食，无论是传统的函数声明或函数表达式，还是更新、更简洁的 ES6 arrow 函数。知道何时(以及如何)有效地利用每种类型函数的好处，肯定会使编写 JS 变得更容易。**

**过几周再来看看，我会写更多关于 JavaScript 和 ES6 或其他与 web 开发相关的东西，所以请关注我，这样你就不会错过了。**

**感谢您的阅读，我希望您能够更好地将箭头函数整合到您的 JavaScript 应用程序中。如果你觉得有帮助，请与你的朋友分享！**

**如果你喜欢读这篇文章，你可能也会喜欢我的其他一些博客**

*   **[Var，Let & Const: JavaScript ES6 特性系列(Pt 1)](/var-let-const-javascript-es6-feature-series-pt-1-fa603567809e)**
*   **[通过设置同步](http://Take Your VS Code Configuration Anywhere Easily with Settings Sync),将您的 VS 代码配置轻松带到任何地方**
*   **[JavaScript 国际方法](/javascript-international-methods-b70a2de09d92)**

****参考资料和更多资源:****

*   **函数声明，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Statements/function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)**
*   **函数表达式，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/web/JavaScript/Reference/Operators/function](https://developer.mozilla.org/en-US/docs/web/JavaScript/Reference/Operators/function)**
*   **Arrow Functions，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Functions/Arrow _ Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)**