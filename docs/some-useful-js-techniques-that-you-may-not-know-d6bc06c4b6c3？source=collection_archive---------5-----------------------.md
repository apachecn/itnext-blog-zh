# 一些你可能不知道的有用的 JS 技巧

> 原文：<https://itnext.io/some-useful-js-techniques-that-you-may-not-know-d6bc06c4b6c3?source=collection_archive---------5----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

我最近读完了 [JavaScript 启蒙](https://www.amazon.com/JavaScript-Enlightenment-Library-User-Developer/dp/1449342884)。这本书更多的是关于 JavaScript 的基础知识，适合初学者。以下是我从这本书得到的好处和延伸。

# 数学

JavaScript 开发人员通常将构造函数名大写，以区别构造函数和普通函数。因此，一些开发人员可能会将*数学*误认为函数，因为其名称是大写的，而*数学*实际上只是一个对象。

```
console.log(typeof Math); // object
console.log(typeof Array); // function
```

# 功能

有两种方法可以构造函数:

*   函数声明/表达式

```
function sum(x, y) {
    return x + y;
}
console.log(sum(3,4)); //7
```

*   [函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)构造函数

```
let multiply = new Function("x", "y", "return x * y;");
console.log(multiply(3,4)); //12
```

在开发中，我们经常需要使用[调用](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)或者[应用](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)来切换函数上下文。例如

```
function print(){
    console.log(this.a);
}print.call({a: 'hello'}); //hello
```

有些面试问题会问为什么 *print* 不定义 *call* 为其属性但是 *print.call()* 不会抛出错误。因为 *print* 是来自*函数*构造器的实例，所以它继承了 *Function.prototype* 到[原型链](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)中定义的所有方法。

```
print.call === Function.prototype.call
```

# 如何进行数组检查

[typeof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof) 可用于确定原始数据类型的类型。但是它不能区分数组和对象。

```
console.log(typeof [1,2]); //object
console.log(typeof {}); //object
```

有几种方法可以做数组检查:

*   API: [Array.isArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray)

```
console.log(Array.isArray([1,2])); //true
```

*   [toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)

```
console.log(Object.prototype.toString.call([1,2])
            .toLowerCase() === '[object array]'); //true
```

注意这里我们必须使用[object . prototype . tostring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)和 [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 来切换调用上下文，因为 [Array.prototype.toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toString) 被覆盖。

```
console.log(Object.prototype.toString.call([1,2])); //[object Array]
console.log([1,2].toString()); //1,2
```

*   [实例 of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof)

```
[1,2] instanceof Array === true;
```

[object . prototype . tostring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)不适用于自定义数据类型。在这种情况下，我们可以使用的[实例来确定类型。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof)

```
class Person {}let mike = new Person();
console.log(Object.prototype.toString.call(mike)); // [object Object]
console.log(mike instanceof Person); // true
```

# 未定义 vs 空

# 未定义-无值

有两种情况下变量是未定义的。

*   变量已定义，但未赋值。

```
let s;
console.log(s); //undefined
```

*   变量根本没有定义。

```
let obj1 = {};
console.log(obj1.a); //undefined
```

# null 特殊值

```
let obj2 = {a: null};
console.log(obj2.a); //null
```

如果我们的目标是过滤掉未定义和 null，我们可以使用带有双等号的*弱比较*(即==)。

```
console.log(null == undefined); //true
let arr = [null, undefined, 1];
let fil = arr.filter(it => {
    return it != null;
});
console.log(fil); //[1]
```

# 参考

[JavaScript 启蒙](https://www.amazon.com/JavaScript-Enlightenment-Library-User-Developer/dp/1449342884)

# 通知；注意

*   读书笔记系列如想关注最新消息/文章，请[【观看】](https://github.com/n0ruSh/the-art-of-reading)订阅。