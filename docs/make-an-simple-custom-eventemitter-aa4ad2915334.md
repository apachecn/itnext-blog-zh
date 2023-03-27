# 制作一个简单的自定义事件发射器

> 原文：<https://itnext.io/make-an-simple-custom-eventemitter-aa4ad2915334?source=collection_archive---------6----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

# 思想

最近我一直在读一本关于 JS 异步的书 [Async JavaScript](https://www.amazon.com/Async-JavaScript-Responsive-Pragmatic-Express-ebook/dp/B00AKM4RVG) ，JS event 是这个问题的一个有用的解决方案。为了更深入地理解事件是如何工作的，我创建了一个定制的 EventEmitter，它包含了[节点 EventEmitter](https://nodejs.org/api/events.html) 的大部分工作功能。[源代码](https://github.com/n0ruSh/the-art-of-reading/blob/master/javascript/Async%20Javascript/event.js)不超过 60 行。

# 一般想法

总的想法是用一个对象( *this.handlers* )来保存从事件名称(类型:字符串)到其相关侦听器/处理程序(类型:数组<函数>)的映射。当每个事件被触发时，遍历相关的侦听器/处理程序并执行它们。

```
class Emitter {
    constructor(){
        /**
         * keep mapping information。
         * e.g. 
         *   {
         *      'event1': [fn1, fn2],
         *      'event2': [fn3, fn4, fn5]
         *   }
         */
        this.handlers = {};
    }
}
```

# 关于这些方法的一些细节

# on —事件绑定

```
on(evt, handler) {
    this.handlers[evt] = this.handlers[evt] || [];
    let hdl = this.handlers[evt];
    hdl.push(handler);
    return this;
}
```

为了简单起见，我们在绑定处理程序时不检查重复项。也就是说，如果你对同一个函数调用上的*两次，那么当事件被触发时，它就会被调用两次。该方法返回 *this* 以允许方法链接.*

# 关-取消事件绑定

```
removeListener(evt, handler) {
    this.handlers[evt] = this.handlers[evt] || [];
    let hdl = this.handlers[evt];
    let index = hdl.indexOf(handler);
    if(index >= 0) {
        hdl.splice(index, 1);
    }
    return this;
}
```

请注意，这里我们在解除函数绑定时将函数引用与严格比较进行了比较。JavaScript 中的函数通过它们的引用进行比较，就像对象比较的工作方式一样。

```
function f1() {
    console.log('hi');
}function f2() {
    console.log('hi');
}let f3 = f1;
console.log(f1 === f2); //false
console.log(f1 === f3); //true
```

# 一次-绑定，但只能触发一次

```
once(evt, handler) {
    this.handlers[evt] = this.handlers[evt] || [];
    let hdl = this.handlers[evt];
    hdl.push(function f(...args){
        handler.apply(this, args);
        this.removeListener(evt, f);
    });
    return this;
}
```

它与方法上的*类似。但是我们需要用另一个函数包装处理程序，这样一旦处理程序被执行，我们就可以移除绑定，以实现**只被触发一次**。*

# 发出—触发事件

```
emit(evt, ...args) {
    this.handlers[evt] = this.handlers[evt] || [];
    let hdl = this.handlers[evt];
    hdl.forEach((it) => {
        it.apply(this, args);
    });
    return this;
}
```

当一个事件被触发时，找到所有相关的处理程序(即 this.handlers[evt])并执行它们。

# eventNames —获取具有活动(即非空)处理程序的已注册事件的列表

```
eventNames() {
    return Object.keys(this.handlers).reduce((arr, evt) => {
        if(this.listenerCount(evt)) {
            arr.push(evt);
        }
        return arr;
    }, []);
}
```

这里我们不简单地返回所有 *this.handlers* 的键，因为一些事件可以与一些处理程序绑定，然后再移除它们。在这种情况下，事件名称作为有效键存在于 *this.handlers* 中，但没有活动的处理程序。例如

```
let server = new Emitter();
let fn = function(){};
server.on('connection', fn);
server.removeListener('connection', fn);
server.handlers.connection; //[]
```

因此，我们需要过滤掉句柄为空的事件。这里我们使用了 [Array.prototype.reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce?v=example) 来使代码更简洁。在很多情况下， *reduce* 都是有用的，比如计算一个数组的和:

```
function sumWithForEach(arr) { // with forEach
    let sum = 0;
    arr.forEach(it => {
        sum += it;
    })
    return sum;
}function sumWithReduce(arr) { // with reduce
    return arr.reduce((sum, current) => {
        return sum + current;
    })
}
```

# 参考

*   [异步 JavaScript](https://www.amazon.com/Async-JavaScript-Responsive-Pragmatic-Express-ebook/dp/B00AKM4RVG)

# 通知；注意

*   如果您想了解最新的新闻/文章，请点击[【观看】](https://github.com/n0ruSh/the-art-of-reading)订阅。