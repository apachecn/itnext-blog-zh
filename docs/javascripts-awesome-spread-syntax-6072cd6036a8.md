# Javascript 令人敬畏的扩展语法[…]

> 原文：<https://itnext.io/javascripts-awesome-spread-syntax-6072cd6036a8?source=collection_archive---------4----------------------->

![](img/c2c15db0551280f23248253562790987.png)

Linux 的创始人伊努斯·托沃兹(inus Torvalds)曾经说过，优秀的程序员不担心代码，而是担心数据结构和它们之间的关系(T2)。

如果是这样的话，那么好的编程语言提供了很好的工具来管理数据结构和它们之间的关系——从而帮助程序员专注于重要的事情，而不是为实现的细节操心。

Javascript 最近增加的**扩展语法**就是一个很好的例子。

Spread 语法是一个非常有用的声明工具，现在可以处理两种数据结构类型: **iterables** 和 **object literals** 。

> Spread 语法允许 iterable or 在需要零个或多个参数(用于函数调用)或元素(用于数组文字)的地方进行扩展。
> 
> Spread 语法允许一个**对象文字**在需要零个或多个键值对的地方被**扩展**。

## 但是什么是可重复的呢？

一个 *iterable* 对象是允许你通过使用***(iterable 的 var 值){ }*** 循环迭代其所有值的任何对象。

它通过返回一个为其 [Symbol.iterator](https://www.javascripture.com/Symbol#iterator) 属性生成一个[迭代器](https://www.javascripture.com/Iterator)的**函数**来实现这一点。

在 Javascript 中，某些对象类型像 [**数组**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) **，** [**映射**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) **，** [**集合**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) 和 [**字符串**](https://www.w3schools.com/jsref/jsref_obj_string.asp) ，有[内置的可迭代对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#Built-in_iterables)具有默认的迭代行为。

例如，一个**数组**:

```
const arr = [‘a’, ‘b’, ‘c’];**for** (const x **of** arr) {   // *here's the* ***for..of*** *statement*
  console.log(x);
}
```

这会产生:

```
abc
```

在引擎盖下，***' for(const x of arr)'***来自上面的片段执行以下操作:

```
const iterator = arr[Symbol.iterator]();
let current = iterator.next();
while (!current.done) {
  console.log(current.value);
  current = iterator.next();
}
```

**字符串**本质上只是 [Unicode](https://en.wikipedia.org/wiki/Unicode) 字符的集合，大多数数组概念都适用于它们。因此字符串也有一个内置的迭代器。字符串的[默认迭代器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/@@iterator)逐个返回字符串的字符**。**

```
const stringExample = 'hi';
typeof stringExample[Symbol.iterator]; // "function"const iterator = stringExample[Symbol.iterator]();iterator + ''; // “[object String Iterator]”
iterator.next(); // { value: "h", done: false }
iterator.next(); // { value: "i", done: false }
iterator.next(); // { value: undefined, done: true }**for**(const element **of** stringExample) {
  console.log(element)   *// produces 'h' and 'i';*
}
```

## **什么是对象字面量？**

**JavaScript 对象文字是一个用逗号分隔的键-值对列表，用花括号括起来。**

```
const myObject = {
    sProp: 'some string value',
    numProp: 2,
    bProp: false
};
```

**循环语法中的**for…也可以用于对象文字，以迭代其**可枚举属性**(即键值对)。这个循环包括从原型链继承的属性。****

```
let obj = {
  key1: "value1",
  key2: "value2",
  key3: "value3"
};for (const key in obj) {
  console.log(`${key} = ${obj[key]}`); // 'key1 = "value1"' etc..
}
```

**好了，现在我们有了一个关于 iterables 和 object literals 的句柄。**

**重申一下，扩展语法*扩展了* iterables 和 object literals，这里应该有零个或多个参数/元素。**

**让我们看看这样的例子:**

```
function sum(x, y, z) { *//expects three arguments, returns their sum*
  return x + y + z;
}const numbers = [1, 2, 3];
console.log(sum(**...numbers**));
```

**在这个例子中，我们已经将数组传递给函数 ***sum*** 。这个函数需要三个参数。Spread 语法扩展了数组的 iterable 元素，并将结果(三个独立的数组元素)传递给函数。**

```
const someString = "hi";
[...someString] // ["h", "i"]
```

**在上面的例子中，扩展语法**将字符串**扩展成它的可迭代的组成部分(每个字符)。**

# **实际使用的扩展语法示例**

**让我们看一些有用的扩展语法的例子。**

## **1.克隆阵列**

```
const origArr = [1,2,3];// traditional ways of cloning arrays ⬇
const copy = [].concat(origArray); 
const copy2 = origArray.slice();
const copy3 = Array.from(origArr);// using the spread operator ⬇
const copy4 = [**...origArr**]; *// much more succinct!*
```

## **2.克隆对象文字**

**使用从一个提供的对象文字到一个新对象的可枚举属性的扩展克隆**。****

```
const obj1 = { foo: 'bar', x: 42 };
const obj2 = { foo: 'baz', y: 13 };const clonedObj = { **...obj1** }; 
*// Object { foo: 'bar', x: 42 }*const mergedObj = { **...obj1**, **...obj2** }; 
*// Object { foo: 'baz', x: 42, y: 13 }*
```

> ****注意，通过传播进行克隆总是肤浅的！****

**这意味着，如果原始属性值之一是一个**对象**，那么克隆将引用同一个**对象**；*它不递归克隆对象内的对象*:**

```
**const** original = { prop: {} };
**const** clone = {...original};
**console**.log(original.prop === clone.prop); 
*//returns true, as cloned property points to same object as original*original.prop.foo = 'abc';
**console**.log(clone.prop.foo); // abc 
// ⬆ *(cloned property points to same object as original!)*
```

## **3.串联数组**

```
const arr1 = [1,2,3];
const arr2 = [4,5,6];*// The old way:* ⬇
const arr3 = arr1.concat(arr2);*// With spread syntax this becomes:* ⬇
const arr4 = [...arr1, ...arr2];*// Merging two arrays:*
const parts = ["shoulders", "knees"];
const lyrics = ["head", .**..parts**, "and", "toes"];
*//* lyrics = *["head", "shoulders", "knees", "and", "toes"]*
```

## **4.在另一个数组的开头插入一个数组**

**[*array . prototype . un shift()*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift)常用于在现有数组的**开头**插入一个数组的值。如果没有扩展语法，这将按如下方式完成:**

```
let arr1 = [0, 1, 2];
let arr2 = [3, 4, 5];
Array.prototype.unshift.apply(arr1, arr2) 
*// arr1 is now [3, 4, 5, 0, 1, 2]*
```

**使用扩展语法，这变成:**

```
let arr1 = [0, 1, 2];
let arr2 = [3, 4, 5];
arr1 = [**...arr2, ...arr1**]; 
*// arr1 is now [3, 4, 5, 0, 1, 2]*
```

## **5.从基元数组中移除任何重复项**

```
const arr = [1, 1, 3, 4, 5, 6, 6, 7, 8, 4, 3, 5, 4, 3, 2, 8, 7, 5];
const unique = [**...new Set(arr)**]; *// [1, 3, 4, 5, 6, 7, 8, 2]*
```

## **6.使用数学函数和一般的函数调用**

**Spread 可以用作参数或函数，函数**可以接受任意数量的参数**。**

```
let numbers = [9, 4, 7, 1];
Math.min(**...numbers**); // 1
```

**Math 对象的函数集是 spread 操作符作为函数的唯一**参数的完美例子。****

**当你想使用数组的元素作为函数的参数时，通常使用[*function . prototype . apply()*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)*。扩展语法使这变得多余。***

```
*function myFunction(x, y, z) { }
var args = [0, 1, 2];
**myFunction.apply(null, args);***// With spread syntax the above can be written as:*function myFunction(x, y, z) { }
var args = [0, 1, 2];
myFunction(**...args**);*
```

## ***7.条件对象属性***

```
*const getUser = (emailIncluded) => {
  return {
    name: ‘John’,
    surname: ‘Doe’,
    **...emailIncluded && { email : 'john@doe.com' }**
  }
}const user = getUser(true);
console.log(user); 
*// outputs { name: “John”, surname: “Doe”, email: “john@doe.com” }*const userWithoutEmail = getUser(false);
console.log(userWithoutEmail); 
*// outputs { name: “John”, surname: “Doe” }**
```

## ***8.生成从 0 到 99 的数字数组***

```
*const arr = [**...Array(100)**].map((_, i) => i);*
```

***spread 操作符循环遍历数组*、*并为每个数组元素创建一个新的**索引键**，从 0 开始迭代递增 1，索引处的值*未定义*。***

***结果是*能够映射*。在示例中，array [map](https://www.w3schools.com/jsref/jsref_map.asp) 函数返回每个数组元素的索引键值，最终返回一个从 0 到 99 的整数数组。***

## ***参考***

 ***[## 可迭代 JavaScript API

### JavaScript Iterable 对象的交互式 API 参考。可迭代对象是任何返回函数的对象…

www.javascripture.com](https://www.javascripture.com/Iterable)*** ***[](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) [## 迭代协议

### ECMAScript 2015 的几个新增内容不是新的内置或语法，而是协议。这些协议可以是…

developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) [](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) [## 扩展语法

### Spread 语法允许在零个或多个…

developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) [](https://www.geeksforgeeks.org/javascript-spread-operator/) [## JavaScript | Spread 运算符- GeeksforGeeks

### Spread 运算符允许 iterable 在应该有 0+个参数的地方扩展。它主要用于可变的…

www.geeksforgeeks.org](https://www.geeksforgeeks.org/javascript-spread-operator/) [](https://codeburst.io/a-simple-guide-to-destructuring-and-es6-spread-operator-e02212af5831) [## 析构和 ES6 扩展运算符的简单指南

### JavaScript 到 ES6 版本的发展带来了一系列新的工具和实用程序。这些工具允许我们…

codeburst.io](https://codeburst.io/a-simple-guide-to-destructuring-and-es6-spread-operator-e02212af5831) [](http://2ality.com/2016/10/rest-spread-properties.html) [## ES2018:静止/扩散属性

### 关于|捐赠|订阅|存档|搜索|原因 ML | ES2018

2ality.com](http://2ality.com/2016/10/rest-spread-properties.html) [](/heres-why-mapping-a-constructed-array-doesn-t-work-in-javascript-f1195138615a) [## 这就是为什么映射一个构造的数组在 JavaScript 中不起作用

### 以及如何正确地去做

itnext.io](/heres-why-mapping-a-constructed-array-doesn-t-work-in-javascript-f1195138615a) [](https://davidwalsh.name/spread-operator) [## Spread 运算符的 6 大用途

### 感谢 ES6 和 Babel 之类的东西，编写 JavaScript 已经变得非常动态，从新的语言语法到…

davidwalsh.name](https://davidwalsh.name/spread-operator) [](https://medium.com/front-end-weekly/es6-magical-stuffs-spread-syntax-in-depth-afdd0118ebd0) [## ES6 神奇的东西—深入传播(…)语法

### 一个经典问题

medium.com](https://medium.com/front-end-weekly/es6-magical-stuffs-spread-syntax-in-depth-afdd0118ebd0)***