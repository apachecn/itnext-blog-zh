# JavaScript 基础:数组和对象方法

> 原文：<https://itnext.io/javascript-fundamentals-array-object-methods-1f3f4adc025c?source=collection_archive---------12----------------------->

![](img/43c74842f3af4cd02d2a2284381b7a49.png)

在本文中，我们将看看一些非常有用的数组和对象方法。当我们着手操作数组和对象时，这些方法的雄辩将帮助我们编写非常干净和可读的代码。

🤓*想跟上网络发展的步伐吗？*
🚀想要将最新消息直接发送到您的收件箱吗？
🎉加入一个不断壮大的设计师&开发者社区！

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# `Object.assign()`

这种方法赋予我们将对象组合在一起的能力。

***例如:***

将两个独立的对象合并为一个:

```
const objectOne = {
  firstName: 'Santa'
}const objectTwo = {
  lastName: 'Claus'
}const objectCombined = Object.assign(objectOne, objectTwo);// objectCombined is: { firstName: 'Santa', lastName: 'Claus' }
```

*注意:*你也可以使用对象扩展语法——我们将在本文后面看到！

# Object.create()

此方法将创建一个新对象，使用现有对象作为新创建对象的原型。

***举例:***

```
let newObject = Object.create(obj);console.log(newObject);  
//{}newObject.name = “William”;console.log(newObject.speak());// My Name is William and this is year 2019
```

在我们的例子中 ***obj*** 是创建 ***新对象*** 的原型。所以通过继承，它可以使用我们原型的属性。这就是为什么我们可以使用 ***speak()*** 方法，而不用在 ***newObject*** 中声明它。

# Object.entries()

在这里，我们可以创建一个包含对象的键/值对的数组。本质上，它将对象转换为数组的数组。

***例如:***

```
let person = {
  name:”William”,
  age:30
}let entries = Object.entries(person);console.log(entries);//[ [ 'name', 'William' ], [ 'age', 30 ] ]
```

# Object.keys()

该方法返回给定对象的 ***键*** (或属性标签)的数组。

***举例:***

```
const seasonalColors = {
  winter: 'blue',
  spring: 'green',
  summer: 'yellow',
  fall: 'brown'
}const types = Object.keys(seasonalColors);// 'types' is equal to ["winter", "spring", "summer", "fall"]
```

# Object.values()

该方法返回给定对象的 ***值*** 的数组。

***举例:***

```
const seasonalColors = {
  winter: 'blue',
  spring: 'green',
  summer: 'yellow',
  fall: 'brown'
}const colors = Object.values(seasonalColors);// 'colors' are equal to ["blue", "green", "yellow", "brown"]
```

# Object.freeze()

您可以使用此方法来防止修改现有的对象属性，或者向对象添加新的属性和值。本质上，函数**冻结**对象的任何进一步改变(键或值)。

***举例:***

冻结对象以防止`name`属性被更改。

```
const frozenObject = {
  name: 'Batman'
}Object.freeze(frozenObject);frozenObject.name = 'Superman';// frozenObject will remain equal to { name: 'Batman' }
```

# `Object.seal()`

这个方法阻止任何新的属性被添加到一个对象中，但是它仍然允许现有的属性被改变。

***举例:***

密封一个对象以防止添加`isBetter`属性。

```
const sealedObject = {
  name: 'Batman'
}Object.seal(sealedObject);sealedObject.name = 'Superman';
sealedObject.isBetter = true;// sealedObject will be equal to { name: 'Superman' }
```

# `.map()`

使用这种方法，我们可以通过操作另一个数组中的值来创建一个新的数组。然后返回新数组。

***举例:***

创建一个将每个数字乘以 *10* 的数组:

```
let arr = [1,2,3,4];let multiply10 = arr.map((val, i, arr) => {
  return val *10;
});multiply10 = [10,20,30,40]
```

*注:*同**。map()** 我们只是定义我们想要发生的事情&返回它——不需要循环！

# `.filter()`

使用这种方法，我们根据数组中的元素是否满足特定条件来创建一个新数组。

***举例:***

创建一组幸运数字(数字> 3):

```
const allNumbers = [1, 2, 3, 4, 5, 6];const luckyNumbers = allNumbers.filter( num => num > 3);// luckyNumbers will be equal to [4, 5, 6]
```

# `.reduce()`

这个方法将把数组中的所有项减少到一个值。它对计算总数非常有用。返回值可以是任何类型(对象、数组、字符串、数字)。

***举例:***

将一个数组中的所有数字相加:

```
const numbers = [10, 20, 20];const total = numbers.reduce( (accumulator, currentValue) => accumulator + currentValue);// total will be equal to 50
```

# `.forEach()`

使用这种方法，我们可以对给定数组中的每一项应用一个函数。

***举例:***

```
const poets = ['Ginsberg', 'Plath', 'Yeats'];poets.forEach( poet => console.log(poet) );// 'Ginsberg'
// 'Plath'
// 'Yeats'
```

# `.some()`

该方法检查数组中的任何**项是否通过给定的条件。一个很好的用例是检查用户权限。**

***例如:***

检查数组中是否至少有一个`'teacher'`:

```
const classReady = ['student', 'student', 'teacher', 'student'];const containsTeacher = classReady.some( element => element === 'teacher');// containsTeacher will equal true
```

# `.every()`

这个方法与`.some()`非常相似，但是它将检查**数组中的所有**项目是否都通过了一个条件。

***例如:***

检查**的所有**评级是否等于或大于 3 星。

```
const ratings = [4, 5, 4, 3, 4];const goodOverallRating = ratings.every( rating => rating >= 3 );// goodOverallRating will equal true
```

# `.includes()`

使用这种方法，我们可以检查一个数组是否包含某个值。这就像`.some()`，然而它不是寻找一个通过的条件，而是检查给定数组中的一个特定值。

***例如:***

检查数组是否包含带有字符串`‘no’`的项目。

```
const responss = ['yes', 'maybe', 'no', 'yes'];const includesNo = responses.includes('no');// includesNo will equal true
```

# `Array.from()`

此方法基于另一个数组或字符串创建一个数组。然而，更常见的是使用`.map()`方法。

***举例:***

从字符串创建数组:

```
const newArray = Array.from('abcde');// newArray will equal ['a', 'b', 'c', 'd', 'e']
```

创建一个数组，使另一个数组中的每一项的值加倍。

```
const doubledValues = Array.from([2, 4, 6], number => number * 2);// doubleValues will equal [4, 8, 12]
```

# 阵列传播

我们可以使用扩展操作符(…)来扩展数组。这允许我们扩展数组中的元素。当把许多数组连接在一起时，这非常有用。

***举例:***

组合两个给定的数组。

```
const arrayOne = [1, 2, 3];
const arrayTwo = [4, 5, 6];const combinedArrays = [...arrayOne, ...arrayTwo];// combinedArrays is equal to [1, 2, 3, 4, 5, 6]
```

我们也可以使用 spread with `.slice()`来删除一个数组元素*而不改变原始数组*:

```
const companies = ['mercedes', 'boeing', 'starbucks', 'honda'];const transport = [...companies.slice(0,2), ...companies.slice(3)];// transport will equal ['mercedes', 'boeing', 'honda']
```

# 对象扩展

我们可以扩展一个对象，以允许添加新的属性和值而不发生变化(创建一个新对象)。它还可以用于将多个对象组合在一起。

***例如:***

添加新的对象属性和值，而不改变原始对象:

```
const originalObject = {
  name: 'Jonathan',
  city: 'Toronto'
};const newObject = {
  ...originalObject,
  occupation: 'Chef'
}// newObject is equal to
// { occupation: 'Chef', name: 'Jonathan', city: 'Toronto' }
```

***你准备好让你的 JavaScript 技能更上一层楼了吗？今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*JavaScript 精通完全指南*](https://gum.co/mastering-javascript) *带你从零到英雄！***

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

我们走吧！我们已经看到了许多数组和对象的方法，包括:`.assign()`、`.create()`、`.entries()`、`.keys()`、`.values()`。`freeze()`、 `.seal()`、。`map()`、`.filter()`、`.reduce()`、`.forEach()`、`.some()`、`.every()`、`.includes()`、`.from(),`、&数组/对象展开语法。😅

掌握这些方法将极大地提高代码的可读性。以及在操作数组和对象时，为您提供一些很好的入门技巧！

我希望这篇文章对你有用！可以[跟着我](https://medium.com/@timothyrobards?source=post_page---------------------------)上媒。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在[我的个人博客](http://www.easeout.co)上找到。

我目前正在构建我的[自由职业者完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果你对它感兴趣，你可以[注册，当它可用时会通知你](https://easeout.eo.page/news)👍

感谢阅读🎉