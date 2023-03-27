# 深入研究 JS 异步性

> 原文：<https://itnext.io/deep-dive-into-js-asynchronicity-54d06783ec0e?source=collection_archive---------5----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

# 单线程 JavaScript

JavaScript 有一个基于[事件循环](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)的并发模型。在处理任何其他消息之前，先完全处理每条消息。在对程序进行推理时，这提供了一些很好的属性，包括这样一个事实，即无论何时函数运行，它都不能被抢占，并且将在任何其他代码运行之前完全运行(并且可以修改函数操作的数据)。

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fdeep-dive-into-js-asynchronicity-54d06783ec0e)

```
runYourScript(); 
while (atLeastOneEventIsQueued) {
    fireNextQueuedEvent();
};
```

让我们用 setTimeout 作为一个简单的例子:

```
let start = +new Date;
setTimeout(function Task1(){
    let end = +new Date;
    console.log(`Task1: Time elapsed ${end - start} ms`);
}, 500); //500ms later，Task1 will be inserted into event queue// single thread
setTimeout(function Task2(){
    let end = +new Date;
    console.log(`Task2: Time elapsed ${end -start} ms`);
    /**
     * use while loop to delay the completion of the function for 3 seconds
     * this will block the execution of the next function in event loop
     * i.e. looping through the event queue has to happen after the main thread finishes its task
     */
    while(+new Date - start < 3000) {}
}, 300); //300ms later，Task2 will be inserted into event queuewhile(+new Date - start < 1000) {} //main thread delay completion for 1 second
console.log('main thread ends');
//output: 
//main thread ends
//Task2: Time elapsed 1049 ms
//Task1: Time elapsed 3000 ms
```

[setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout) 会将第一个参数(类型函数)放入事件队列。下面是上面代码中发生的情况:

*   300 毫秒后，Task2 被插入事件队列。
*   500 毫秒后，Task1 被插入事件队列。
*   1 秒钟后，主线程结束，控制台上显示“主线程结束”。
*   当主线程完成手头的工作时，它将在事件队列中循环。事件队列中的第一个函数是 Task2(因为它首先被插入)。因此，获取并执行 Task2，得到控制台输出“Task2: Time elapsed 1049 ms”。您看到的运行时间可能会有所不同，因为 setTimeout 并不完全处理函数的执行。它所做的只是将函数放入事件队列中，并有您设置的延迟。函数何时执行取决于事件队列中的排队函数，以及主线程的状态(是否有剩余的任务在运行)。
*   任务 2 完成后执行任务 1。

# 什么是异步函数

JavaScript 中的异步函数通常可以接受函数类型的最后一个参数(通常称为*回调*)，当函数完成时，回调将被插入事件队列。由于回调在事件队列中，所以该函数是非阻塞的。异步函数可以保证下面的单元测试将总是通过:

```
let functionHasReturned = false; 
asyncFunction(() => {
    console.assert(functionHasReturned); 
}); 
functionHasReturned = true;
```

注意，并不是所有带有回调参数的函数都是异步的。例如 [Array.prototype.forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) 是同步的。

```
let before = false;
[1].forEach(() => {
    console.assert(before); 
}); 
before = true;
```

# 异步函数中的异常

由于回调函数被放在事件队列中，并在以后用它自己的调用上下文执行，所以用 try-catch 机制包装异步函数将无法捕获回调中的异常。

```
try {
    setTimeout(() => {
        throw new Error('callback error'); 
    }, 0);
} catch (e) {
    console.error('caught callback error');
}
console.log('try-catch block ends');
```

在这个例子中，回调中抛出的异常没有被我们自己的程序捕获到(即 catch 块——控制台上没有显示“捕获的回调错误”)。我们可以用浏览器中的 [window.onerror](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onerror) 和节点中的[process . uncaughtexception](https://nodejs.org/api/process.html#process_event_uncaughtexception)来捕捉这些未捕捉到的异常。

如果我们想故意捕捉错误，就像我们应该一直做的那样，我们可以在回调中这样做。例如

```
let fs = require('fs'); 
fs.readFile('abc.txt', function(err, data) {
    if (err) {
        return console.error(err); 
    }; 
    console.log(data);
});
```

# 参考

*   [异步 JavaScript](https://www.amazon.com/Async-JavaScript-Responsive-Pragmatic-Express-ebook/dp/B00AKM4RVG)

# 通知；注意

*   如果你想了解最新的新闻/文章，请点击[【观看】](https://github.com/n0ruSh/the-art-of-reading)订阅。