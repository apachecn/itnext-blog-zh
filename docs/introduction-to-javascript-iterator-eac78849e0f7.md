# JavaScript 迭代器简介

> 原文：<https://itnext.io/introduction-to-javascript-iterator-eac78849e0f7?source=collection_archive---------3----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

# [符号](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

ES6 引入了一种叫做*的新类型符号*。*符号*函数返回类型为*符号*的值。

```
const symbol1 = Symbol();
const symbol2 = Symbol('hi');console.log(typeof symbol1); //symbolconsole.log(symbol3.toString()); //Symbol(foo)// each symbol value created by Symbol function is unique
console.log(Symbol('foo') === Symbol('foo'));  // false// Symbol itself is a function
console.log(typeof Symbol); //function
```

# [迭代器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)

一个*迭代器*是一个提供 *next* 方法的对象，该方法返回序列中的下一项。该方法返回一个具有两个属性的对象:*完成*和*值*。对于一个要成为*可迭代*的对象，它必须有一个带有 *Symbol.iterator* 键的 function 属性，它为每个调用返回一个新的*迭代器*。

```
// Array has build-in iteration support
function forOf(arr) {
    for(let i of arr) {
        console.log(i);
    }
}/**
 * for...of loops is based on iterator
 * so forOf implementation above is basically:
 */
function iterating(arr) {
    let iterator = arr[Symbol.iterator](); //get the iterator for the array
    let next = iterator.next();
    while(!next.done) {
        console.log(next.value);
        next = iterator.next();
    }
}
```

# 使对象可迭代

*对象*没有内置迭代支持。

```
let obj = {a: 'b', c: 'd'}
for(let i of obj) {
    //Uncaught TypeError: obj is not iterable
    console.log(i);
}
```

为了使*对象*可迭代，我们必须将 *Symbol.iterator* 添加到实例或原型中。

```
Object.defineProperty(Object.prototype, Symbol.iterator, {
    value: function() {
        let keys = Object.keys(this);
        let index = 0;
        return {
            next: () => {
                let key = keys[index++];
                return {
                    value: `${key}-${this[key]}`,
                    done: index > keys.length
                }
            }
        };
    },
    enumerable: false
});let obj = {a: 'b', c: 'd'}
for(let i of obj) {
    console.log(i); // a-b c-d
}
```

# [发电机](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)

函数*声明定义了一个*生成器函数*，其返回值为*生成器对象*。*生成器对象*符合*迭代器协议*。注意*发生器函数*本身就是一个函数。

```
//generator function
function* generatorFn() {
    yield 1;
    return 2;
}// generator object - iterator
let generatorObj = generatorFn();
let nextItem = generatorObj.next(); console.log(typeof generatorFn); //function
console.log(typeof generatorObj); //object
console.log(typeof generatorObj.next); //function
console.log(nextItem); //{value: 1, done: false}
```

因此，为了使*对象*可迭代，我们可以用*生成器函数*定义它的 *Symbol.iterator* 。

```
Object.defineProperty(Object.prototype, Symbol.iterator, {
    value: function*() {
        let keys = Object.keys(this);
        for(let key of keys) {
            yield `${key}-${this[key]}`;
        }
    },
    enumerable: false
});let obj = {a: 'b', c: 'd'};
for(let kv of obj) {
    console.log(kv); // a-b c-d
}
```

# 在实践中

有了这种技术，我们可以使定制数据类型可迭代。

```
class Group {
    constructor() {
        this._data = [];
    } add(it) {
        this._data.push(it);
    } delete(it) {
        let index = this._data.indexOf(it);
        if(index >= 0) {
            this._data.splice(index, 1);
        }
    } has(it) {
        return this._data.includes(it);
    } [Symbol.iterator]() {
        let index = 0;
        return {
            next: () => ({
                value: this._data[index++],
                done: index > this._data.length
            })
        };
    } static from(iterable) {
        let group = new Group();
        for(let item of iterable) {
            group.add(item);
        }
        return group;
    }
}let group = Group.from(['a', 'b', 'c']);
console.log(group)for (let value of group) {
    console.log(value);
}console.log([...group]);
```