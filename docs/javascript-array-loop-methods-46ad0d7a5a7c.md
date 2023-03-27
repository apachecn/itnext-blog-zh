# JavaScript —数组循环方法

> 原文：<https://itnext.io/javascript-array-loop-methods-46ad0d7a5a7c?source=collection_archive---------6----------------------->

## 数组方法 forEach、map、flatMap、筛选、缩减、排序、一些、每一个——简单的例子

![](img/778472e464c310e0c681b104ccbf0e32.png)

[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上[émile Perron](https://unsplash.com/@emilep?utm_source=medium&utm_medium=referral)拍摄

在本文中，我将向您展示如何使用不同的 JavaScript (ES6)数组方法在数组上循环。这些方法中的一些对我来说并不总是那么清楚，所以在这里它们被简化了。

> *注:这也适用于* ***打字稿***

# 环

## 1 — forEach

最简单的是`forEach`，它只是被设计成在一个数组上**循环，所以你可以使用单个项目。我们**什么都不回**。**

```
const array = ['John', 'Jeroen', 'Peter'];array.forEach((name) => {
  console.log(name);
});// John
// Jeroen
// Peter
```

## 2 —地图

`map`方法与之前的方法几乎相同。除了现在它**返回一个新的数组**，比如用；修改的项目。

```
const array = ['John', 'Jeroen', 'Peter'];const mappedArray = array.map((name) => {
   return `Hello ${name}!`;
});console.log(mappedArray);// ['Hello John!', 'Hello Jeroen!', 'Hello Peter!']
```

## **3 — flatMap**

地图的一个变体是`flatMap`，在这里**将一个或多个数组的数组转换为单个数组**，并对其进行映射。

```
const array = [['John'], ['Jeroen'], ['Peter']];const flatMappedArray = array.flatMap((name) => {
   return `Hello ${name}!`;
});console.log(flatMappedArray);// ['Hello John!', 'Hello Jeroen!', 'Hello Peter!']
```

# 最小化价值

## 4—过滤器

通过`filter`，我们**返回了一个只有特定项目的新数组**。有时我们并不需要数组中的每一个项目，我们只需要其中的一部分。

```
const array = ['John', 'Jeroen', 'Peter'];const filteredArray = array.filter((name) => {
   return name === 'Peter' || name === 'Jeroen';
});console.log(filteredArray);// ['Jeroen', 'Peter']
```

## 5 —减少

我认为最‘复杂’的是`reduce`，你可以用它将**数组改变/缩减为一个值**。

大部分时间都用在数字上。但它也可以用于字符串等。我们的第一个参数是`accumulator`，它保存我们之前的值，第二个参数包含我们正在循环的`currentValue`。

```
const array = [1, 2, 3];const reduced = array.reduce((accumulator, currentValue) => {
   return accumulator + currentValue;
});console.log(reduced);// 6
```

# 整理

## 6—排序

在某些情况下，您可能希望**对日期、字母或其他内容进行排序。**现在的`sort`法就是我们的家伙。我们需要指定两个参数，因为它将比较值。

```
const array = ['John', 'Jeroen', 'Peter'];const sortedArray = array.sort((name1, name2) => name1.lowercase() < name2.lowercase());console.log(sortedArray);// ['Jeroen', 'John', 'Peter']
```

# 情况

## 7—部分

使用`some`可以检查**数组中的一个或多个** **项目**是否满足某个条件。返回`true`或`false`。

```
const array = ['John', 'Jeroen', 'Peter'];const hasName = array.some((name) => {
   return name === 'Peter';
});console.log(hasName);// true
```

## 8—每

使用`every`可以检查**数组中所有** **项**是否满足某个条件。这也返回`true`或`false`。

```
const array = ['John', 'Jeroen', 'Peter'];const hasName = array.every((name) => {
   return name === 'Peter';
});console.log(hasName);// false
```

# 感谢您的阅读！我的 [Github](https://github.com/jeroenouw/) 。如果你觉得这篇文章有用，可以考虑看看我的其他文章:

[](https://levelup.gitconnected.com/10-advanced-typescript-questions-for-an-interview-with-answers-6f0513b1688e) [## 面试中的 10 个高级打字问题及答案

### 检查你的候选人是否有能力回答这些更高级的打字问题

levelup.gitconnected.com](https://levelup.gitconnected.com/10-advanced-typescript-questions-for-an-interview-with-answers-6f0513b1688e) [](https://levelup.gitconnected.com/learn-dart-for-flutter-by-comparing-it-to-typescript-3078ca18fe8f) [## 通过与 TypeScript 进行比较来学习 Dart(用于颤振)

### Dart 代码与 TypeScript 代码比较的基本示例

levelup.gitconnected.com](https://levelup.gitconnected.com/learn-dart-for-flutter-by-comparing-it-to-typescript-3078ca18fe8f) [](/javascript-prevent-objects-from-being-changed-d1ca82f02975) [## JavaScript —防止对象被更改

### 对象方法 freeze、seal、preventExtensions、isFrozen、isSealed、isExtendable 和 const 关键字—简单示例

itnext.io](/javascript-prevent-objects-from-being-changed-d1ca82f02975)