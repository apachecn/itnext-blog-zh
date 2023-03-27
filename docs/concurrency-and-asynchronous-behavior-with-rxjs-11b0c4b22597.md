# RxJS 的并发和异步行为

> 原文：<https://itnext.io/concurrency-and-asynchronous-behavior-with-rxjs-11b0c4b22597?source=collection_archive---------2----------------------->

![](img/56ad22116c339e71bc6ea23c24063a7e.png)

照片来自 Pexels 上的 Bahram Jamalov

在最近发表的一篇关于[反应式流和多线程](https://medium.com/@abetteroliver/reactive-streams-and-multithreading-efd63b67de9a)的文章中。如果您是一名 JavaScript 开发人员，您会意识到 JavaScript 是单线程的，您可能会认为这与您无关。没那么快！那篇文章中讨论的原则也适用于 RxJS，您甚至可以在 JavaScript 中实现并发。所以我来分享一些专家知识。

注:示例使用 RxJS ≥ 6

# 调度程序

在前面提到的文章中，我展示了使用*发布*和*订阅*操作符可以让流在不同的线程上运行。那些操作符也存在于 RxJS 中(publishOn 命名为 observeOn)。如果你不能用它们来选择一根线，它们是如何工作的？它们影响流执行的时间。他们使用调度程序来完成这项工作。

简单地说，调度器控制一个工作单元何时被执行。RxJx 内置了五个调度程序，其中三个与异步执行相关:

*   Asap:调度微任务队列的执行
*   Async:调度宏任务队列的执行
*   动画帧:调度下一个动画帧的执行

如果你想了解更多关于微任务、宏任务和动画帧的知识，我推荐 YouTube 视频《事件循环的进一步冒险》。

现在我只想说，使用其中一个调度器会使流异步。要在 RxJS 的当前版本中使用它们，您可以像这样导入它们:

```
import {animationFrameScheduler, asapScheduler, asyncScheduler} from 'rxjs';
```

在像这样的旧版本中:

```
import { async } from 'rxjs/scheduler/async';import { asap } from 'rxjs/scheduler/asap';import { animationFrame } from 'rxjs/scheduler/animationFrame';
```

除了前面提到的操作符，你还可以在其他函数中传递一个调度器，比如:的

```
*of(1,2, asyncScheduler)
  .subscribe(console.log);console.log("Before or after?")//Prints
//Before or after?
//1
//2*
```

*没有调度器，“之前还是之后？”最终会被印刷出来。*

# *分叉和连接*

*一个常见的任务是从服务器获取数据，通常我们需要不止一个请求。在我们收到响应之前需要一些时间，为了更快，并行地拥有独立的请求是一个好主意。*

*由于 JavaScript 中的单线程限制，发送请求和处理响应不能并行进行(或者可以吗？)，我们不必在发送下一个请求之前等待一个响应。*

*但是如果我们在继续之前需要所有请求的响应呢？女士们先生们，我来介绍*福克加入:**

```
*import { forkJoin, from } from 'rxjs';forkJoin(
  from(fetch(someUrl)),
  from(fetch(someOtherUrl)),
)
.subscribe(console.log);//Prints
//[Response, Response]*
```

*函数 *fetch* 发出一个(AJAX)请求。来自的函数*创建一个发出响应的可观察对象。 *forkJoin* 创建一个可观察对象，等待两个响应都被接收到，并发出一个包含两个响应的数组。**

*如果你对承诺比较熟悉的话，类似于 *Promise.all():**

```
*Promise.all(
  fetch(someUrl),
  fetch(someOtherUrl)
)
.then(console.log)*
```

*现在我们已经在我们的流中加入了一定程度的并发性，我认为我们已经为真正的并行执行做好了准备。*

# *网络工作者*

*已经引入了 Web workers 来允许 JavaScript 代码在另一个线程中在后台执行。终于多线程了！*

*但是你不能用 web workers 执行任何代码。为了避免多线程的危险并保持 JavaScript 的单线程性，web workers 与其他代码分开，只能与消息通信。*

*对于那些从未使用过 worker 的人来说，worker 代码是这样的:*

```
*onmessage = function(e) {
  //do something
  const result = ...
  postMessage(result);
}*
```

*和主脚本:*

```
*const myWorker = new Worker(‘worker.js’);
myWorker.onmessage = result => {
  //do something with the result
}myWorker.postMessage("foo");*
```

*主脚本发送消息“foo ”,工人做一些事情，并使用 *postMessage* 将结果返回给主脚本，主脚本通过 *onmessage* 事件处理程序接收该结果。*

*我们如何把它用于可观测量呢？最简单的方法是使用*平面图/合并图*。*

```
*function processWithWorker(value) {
  return new Observable(subscriber => {
    const myWorker = new Worker('worker.js');
    myWorker.onmessage = result => {
      subscriber.next(result);
      subscriber.complete();
      myWorker.terminate();
    }
    myWorker.postMessage(value);
  });
}of(1,2)
.pipe(
  flatMap(processWithWorker)
)
subscribe(console.log);//prints whatever the workers return*
```

*为了简单起见，我们为源发出的每个值创建一个工人。*

*发出值，然后创建工作线程。我们向它发送一条消息，并返回一个新的可观测值。工作者一返回一个值，该可观察对象就发出一个值。请注意，我们还发出一个完成信号，并终止工作线程。*

*工作代码与主脚本并行运行，但是由于流的性质，值仍然是一个接一个地被处理。如果我们想让它们并行处理呢？如果不混淆代码的含义，那是不可能的。但是如果只有几个固定值，可以使用 forkJoin:*

```
*forkJoin(
 processWithWorker(1),
 processWithWorker(2),
}
.subscribe(console.log);*
```

*这两个值将独立于主脚本并行处理。*

# *结论*

*虽然 JavaScript 是单线程的，但 RxJS 遵循与其他反应流库相同的原则。我们可以创建异步流，拥有一定程度的并发性，web workers 甚至允许并行性。*