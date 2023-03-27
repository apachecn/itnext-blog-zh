# 遍历对象的方法

> 原文：<https://itnext.io/ways-to-iterate-through-objects-19c64526bb56?source=collection_archive---------11----------------------->

![](img/417b79541fc0a025f7382aafb6245290.png)

# 相关基本概念

# 属性访问器

```
let obj = {
  a: 'b'
};// dot notation
console.log(obj.a); // 'b'
// bracket notation
console.log(obj['a']); // 'b'
// through prototype chain
console.log(obj.toString, obj.toString === Object.prototype.toString); // ƒ toString() { [native code] } true
```

# 检查对象中是否存在属性

*   运算符中的[:如果指定的属性在指定的对象*或其原型链*中，则返回 true。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in)
*   [hasOwnProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) :返回一个布尔值，表明对象是否将指定的属性作为自己的属性(相对于继承它)。

```
let obj = {
  a: 'b'
};console.log('a' in obj); //true
console.log('toString' in obj); //true
console.log(obj.hasOwnProperty('a')); //true
console.log(obj.hasOwnProperty('toString')); //false
```

# [object . define property](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)&&[object . define properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)—直接在对象上定义新的或修改现有的属性，返回该对象。

```
let obj = {
  a: "b"
};Object.defineProperties(obj, {
  c: {
    value: 'd'
  },
  e: {
    value: 'f'
  }
});
console.log(obj); // {a: "b", c: "d", e: "f"}
```

## [获取器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)和[设置器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set)

我们可以使用 getters 和 setters 来生成计算属性。例如

```
let people = {
  firstName: 'michael',
  lastName: 'zheng',
  get fullName() {
    return `${this.firstName} ${this.lastName}`
  },
  set fullName(val) {
    [this.firstName, this.lastName] = val.split(' ');
  }
}console.log(people.firstName, people.lastName, people.fullName);
//"michael", "zheng", "michael zheng"
people.fullName = 'hello world';
console.log(people.firstName, people.lastName, people.fullName);
//"hello", "world", "hello world"
```

# 有三种方法可以迭代对象

```
let student = {
  name: "michael"
};Object.defineProperties(student, {
  age: {
    enumerable: false,
    value: 18
  },
  grade: {
    value: 6,
    enumerable: true
  },
  sex: {
    value: "male",
    enumerable: false
  }
});Object.prototype.x = "inherited";
```

在上面的例子中，我们创建了一个对象*学生*。*学生*具有以下属性:

*   *名称*:自枚举属性
*   *年龄*:自身不可枚举财产
*   *等级*:自枚举属性
*   *性别*:自身不可枚举的财产

以及原型链中的自定义属性:

*   *x* :可枚举

# [for…in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) 遍历一个对象的可枚举属性，包括原型链。

```
for (let prop in student) {
  console.log(prop); //'name', 'grade', 'x'
}for (let prop in student) { // self properties only
  if (student.hasOwnProperty(prop)) {
    console.log(prop); // 'name', 'grade'
  }
}
```

# [Object.keys](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) 返回一个给定对象的属性名数组，只遍历自身可枚举的属性。(即不包括原型链)

```
console.log(Object.keys(student)); // [‘name’, 'grade']//check whether is plain object:
Object.keys(student).length === 0; //falseObject.keys({}).length === 0; //true
```

# [object . getownpropertymanames](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames)返回直接在给定对象上找到的所有自身属性(包括不可枚举的属性)的数组。

```
// will not iterate through prototype chain
console.log(Object.getOwnPropertyNames(student)); // [‘name’, ‘age’, 'grade', 'sex']
```

# 总结

methods through prototype chain enumerate only for…in yyobject . keysnyobject . getownpropertymanamesnn

# 通知；注意

*   读书笔记系列如果想关注最新的新闻/文章，请[【观看】](https://github.com/n0ruSh/blogs/)订阅。