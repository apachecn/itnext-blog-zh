# 增强的对象文字值速记:JavaScript ES6 特性系列(Pt 6)

> 原文：<https://itnext.io/enhanced-object-literal-value-shorthand-javascript-es6-feature-series-pt-6-e00dfdc24f64?source=collection_archive---------1----------------------->

# 因为在一个对象中输入同样的东西两次是疯狂的

![](img/4640210c7263bff19126a58b9efe4b98.png)

凯文·Ku 在 [Unsplash](https://unsplash.com/s/photos/focus?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

# 介绍

这些帖子背后的灵感很简单:对于很多开发人员来说，JavaScript 并没有太大的意义——或者至少有时令人困惑。

根据维基百科的数据，截至 2017 年 5 月，JavaScript 为 1000 万个最受欢迎的网页中的近 95%提供了支持。

由于 JS 对 web 的贡献如此之大，我想提供一些我经常使用的 ES6+特性的文章和例子，供其他开发人员参考。

我们的目标是让这些文章简短、深入地解释该语言的各种改进，我希望这些文章能启发您使用 JS 编写一些真正酷的东西。谁知道呢，在这个过程中你可能会学到一些新东西。😄

> *我在这个系列中的第六篇帖子将是关于 ES2015 引入的新的对象属性和方法值简写，这是迄今为止从变量初始化对象属性和定义函数的最简洁的方法。*

# 对象:JavaScript 中的一切都是一体的

谈到 JavaScript，您可能听说过这样一句话:“JavaScript 中的一切都是对象。”函数、字符串、数组、对象——它们都是对象。也可以是别的，但是除了原语，其他的也都是 JS 里的对象。

然而，这篇文章关注的是对象对象；通常使用`new Object()`、`Object.create()`或文字符号初始化器进行初始化。

在了解 ES2015 带来的新变化之前，让我们先来看看这些变化。

**对象初始化器示例:** `**new Object()**`

```
// object initialization options// object constructor
function Pet(type, name, age, greeting) {
  this.breed = type;
  this.name = name;
  this.age = age;
  this.greeting = greeting;
  this.sayHello = function () {
    return `${this.name} says 'hello' as ${this.greeting}`;
  }
}// new Object()
const pet1 = new Pet("cat", "Felina", 3, "meow");
console.log(pet1);/* prints: Pet {
breed: 'cat',
name: 'Felina',
age: 3,
greeting: 'meow',
sayHello: [Function] } */console.log(pet1.sayHello());
// prints: Felina says 'hello' as meow
```

第一个例子展示了如何使用`new Object()`关键字创建一个对象。首先，定义一个名为`Pet`的对象构造函数，它接受许多参数，甚至有自己的方法`sayHello()`。

然后，声明变量`pet1`,调用`new Pet()`,并向其传递适当的参数。瞧——当您调用它或它的方法`pet1.sayHello()`时，一个名为`pet1`的新对象就存在了。

**对象初始化器示例:** `**Object.create()**`

```
// Object.create()
const pet2 = Object.create(pet1);
pet2.breed = "dog";
pet2.name = "Rufus";
pet2.age = 4;
pet2.greeting = "woof";console.log(pet2);
/* prints: Pet {
breed: 'dog',
name: 'Rufus',
age: 4,
greeting: 'woof' } */console.log(pet2.sayHello());
// prints: Rufus says 'hello' as woof
```

上面的第二个例子演示了如何使用`Object.create()`初始化器。此选项要求已存在的对象作为新创建对象的原型。

在这个例子中，`pet2`以第一个例子中的`pet1`为原型。然后，`pet2`给来自`pet1`的所有属性赋予新的值，它也从`pet1`继承了方法`sayHello()`。

所以当`pet2.sayHello()`被调用时，它打印出`"Rufus says 'hello' as woof"`。就这么简单。

**对象初始化器示例:对象文字** `**{}**`

```
// object literal initialization
const pet3 = {
  type: "guinea pig",
  name: "Holly",
  age: 6,
  greeting: "snuffle",
  sayHello: function() {
    return `${this.name} says 'hello' as ${this.greeting}`
  }
}console.log(pet3);
/* prints: { type: 'guinea pig',
name: 'Holly',
age: 6,
greeting: 'snuffle',
sayHello: [Function] } */console.log(pet3.sayHello());
// prints: Holly says 'hello' as snuffle// object literal property and method assignment by variables
const type = "fish";
const name = "Nemo";
const age = 2;
const greeting = "bloop";
function sayHello() {
  return `${this.name} says 'hello' as ${this.greeting}`;
}const pet4 = {
  type: type,
  name: name,
  age: age,
  greeting: greeting,
  sayHello: sayHello
}console.log(pet4);
/* prints: { type: 'fish',
name: 'Nemo',
age: 2,
greeting: 'bloop',
sayHello: [Function: sayHello] } */console.log(pet4.sayHello()); 
// prints: Nemo says 'hello' as bloop
```

到目前为止，这个最终的对象初始化是日常编码中最常用的:对象文字初始化。

有两种方法可以做到:

1.  通过声明对象并同时分配它的所有属性和方法来初始化它:就像`pet3`。
2.  通过提前声明任何变量和函数，然后将它们分配给一个新的对象，比如`pet4`。

无论你选择怎样做，这两种方法都是完全可以接受的，并且经常使用。

> 特别是对象文字，这是 ECMAScript 2015 显著改进的地方，让我们看看如何改进。

## 速记属性名

ES2015 在对象文字方面改进的第一件事是对象内属性的缩写初始化，前提是属性键与现有变量名匹配。这被大多数人称为:[简写属性名](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Property_definitions)。

**为对象属性分配变量的传统语法**

```
// traditional object property assignment
const breed = "chinchilla";
const howOld = 1;
const nickname = "Chi Chi";
const activities = "dust baths";
function funFact() {
  return `${this.nickname}'s fun fact is she likes ${this.activities}`;
}const exoticPet = {
  breed: breed,
  nickname: nickname,
  howOld: howOld,
  activities: activities,
  funFact: funFact
};console.log(exoticPet);
/* { breed: 'chinchilla',
nickname: 'Chi Chi',
howOld: 1,
activities: 'dust baths',
funFact: [Function: funFact] } */console.log(exoticPet.funFact()); 
// prints: Chi Chi's fun fact is she likes dust baths
```

以上是将变量赋给对象属性的传统方式。当属性已经与变量名匹配时，创建`exoticPet`似乎需要很多额外的输入，不是吗？

让我们在下一个例子中修复冗余。

**ES2015 将同名变量分配给对象属性的简写示例**

```
// es2015 shorthand object property assignment
const otherBreed = "alpaca";
const firstName = "Alfie";
const likes = "frolicking";
function timeSpent() {
  return `${this.firstName} enjoys ${this.likes} whenever he can.`;
}const exoticPet2 = {
  otherBreed,
  firstName,
  likes,
  timeSpent
};console.log(exoticPet2);
/* { otherBreed: 'alpaca',
firstName: 'Alfie',
likes: 'frolicking',
timeSpent: [Function: timeSpent] } */console.log(exoticPet2.timeSpent());
// prints: Alfie enjoys frolicking whenever he can.
```

如您所见，`exoticPet2`的语法比前一个例子清晰得多。由于`exoticPet2`的属性名与上面声明的变量相匹配:`otherBreed`、`firstName`等等，你可以简单地声明一次属性名，ES2015 就知道如何将同名的变量插入到对象中。

很巧妙的把戏，是吧？我真的很欣赏 ES6 改进的这个特别的细节。我经常使用它。

*注意:*注意不要为你的属性使用相同的名字，第二个属性会覆盖第一个属性。

**注意:重复的属性采用最后的赋值**

```
// duplicate properties assume the last value assignedconst exoticPet3 = { type: "parrot", type: "cockatoo"};
console.log(exoticPet3.type); // prints: cockatoo
```

## 速记方法定义

接下来我们对 ES2015 的对象文字的改进:[方法速记定义](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Method_definitions)。方法定义速记语法省略了`function`关键字和冒号，就像属性赋值一样。

**在对象中创建方法(函数)的传统语法**

```
// traditional method definitions
const person = {
  play: function() {
    return "I like to play board games with my friends.";
  },
  swim: function() {
    return "When I exercise, I like to swim laps in the pool.";
  }
};console.log(person.play()); // prints: I like to play board games with my friends.console.log(person.swim()); // prints: When I exercise, I like to swim laps in the pool.
```

在传统的 JavaScript 中，方法(附加到对象上的函数)类似于普通的对象属性，在对象上用类似于`play`或`swim`的属性名声明，然后用传统的匿名函数声明语法`function() { /* ...do something here */ };`赋值。

然后，对象的方法可以像它的属性一样被调用:`person.play()`或`person.swim()`。那里没有什么惊天动地的事情。

但是真的有人很喜欢匿名函数吗？那鸿除了不必要的代码之外，它似乎没有增加太多，如果我们现在对传统函数有简洁的箭头函数表达式，为什么不对对象方法也做一些类似的事情呢？

**ES2015 在对象中创建方法(函数)的简写**

```
// es2015 shorthand method definitions
const person2 = {
  play() {
    return "I like to play the violin in my free time.";
  },
  swim() {
    return "Swimming in the ocean is my favorite thing to do at the beach.";
  }
};console.log(person2.play()); // prints: I like to play the violin in my free time.console.log(person2.swim()); // prints: Swimming in the ocean is my favorite thing to do at the beach
```

介绍用于对象文字符号的 ES2015 方法简写。`person2`对象有两个类似于`person1`、`play()`和`swim()`的方法，正如您所看到的，对象本身的声明缺少`function`关键字，取而代之的是属性名和函数名。

更干净，更简洁，老实说，当我阅读这样编写的代码时，它对我来说比旧的匿名函数更有意义。

好了，到了最后一个 ES2015 对象文字改进的时候了:计算属性名。

## 计算属性名

从 ECMAScript 2015 开始，对象初始值设定项语法支持计算属性名。该特性允许您将表达式放在括号`[]`中，该表达式将被计算并用作属性名。

**ES2015 通过括号符号计算属性键/名称的简写**

```
// computed property names / keys
let i = 1;
const items = {
  ['item' + i++] : i,
  ['otherItem' + i++] : i,
  ['aThirdItem' + i++] : i
}console.log(items); // prints: { item1: 2, otherItem2: 3, aThirdItem3: 4 }
```

老实说，我在日常生活中从未使用过这种计算属性名功能，但是如果情况需要，知道它的存在是很好的，对吗？

仅此而已。这就是从 ES2015 开始你需要了解的 JavaScript 中的对象字面量。

# 结论

随着时间的推移，所有 JS 开发人员都已经熟悉了这种语言的许多怪癖和缺陷，但是最近，ECMAScript 委员会每年都会发布 JavaScript 语言的新更新，旨在使我们的生活更加轻松。一些更新的语法变化看起来比其他的更奇怪，但我很高兴地说，我们如何处理对象文字的改进比大多数更类似于旧的语法。

令人高兴的是，这些变化产生了简洁的对象，可以更容易地为它们分配变量、方法甚至新的计算属性名。非常棒。

我写这个博客系列的目的是深入到目前使用的 JavaScript 和 ES6 语法中我最喜欢的部分，并向您展示如何使用最好的部分来获得最大的效果。

过几周再来看看，我会写更多关于 JavaScript 和 ES6 或其他与 web 开发相关的东西。

感谢您的阅读，我希望您能够使用这种新的对象文字速记来使您自己的 JavaScript 对象在未来变得更加紧凑和易于管理。

**如果你喜欢读这篇文章，你可能也会喜欢我的其他博客:**

*   [字符串模板文字:JavaScript ES6 特性系列(Pt 5)](https://medium.com/better-programming/string-template-literals-javascript-es6-feature-series-pt-5-a40e55a5485b)
*   [Spread & Rest 参数:JavaScript ES6 特性系列(Pt 4)](/spread-rest-parameters-javascript-es6-feature-series-pt-4-c9e9f0c0228f)
*   [默认函数参数值:JavaScript ES6 特性系列(Pt 3)](/default-function-parameter-values-javascript-es6-feature-series-pt-3-bd8392a88a12)

**参考资料和更多资源:**

*   对象文字符号，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Operators/Object _ initializer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)
*   对象属性定义速记，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Operators/Object _ initializer # Property _ definitions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Property_definitions)
*   对象方法定义速记，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Operators/Object _ initializer # Method _ definitions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Method_definitions)