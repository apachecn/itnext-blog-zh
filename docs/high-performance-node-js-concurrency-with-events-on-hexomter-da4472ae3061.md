# 具有本机“事件”包的高性能 Node.js 并发

> 原文：<https://itnext.io/high-performance-node-js-concurrency-with-events-on-hexomter-da4472ae3061?source=collection_archive---------2----------------------->

![](img/29e473eea8a30d768ab45f6f258d8eda.png)

自然，JavaScript 被认为是一种单线程编程语言，以事件循环实现作为主要驱动引擎。然而，Node.js 并不完全如此，因为他们实际上是基于`libuv`库实现线程池 API 的，这是 Node.js 访问其他操作系统 API(如网络、文件系统等)的核心库...

LibUV 实现本身具有这种本机操作系统级线程池，它与线程间的非阻塞消息传递 API 相结合，提供了应用程序作为单线程运行的完整模拟，但实际上它被划分为多个线程，通常是 2 个 CPU 内核。

# 异步任务的问题

通常每个`async/await`任务或承诺，一般来说，作为一个单独的线程池任务执行(实际上是井任务队列)。这意味着，为了从中获取结果，任务线程应该向主线程发送一个异步消息，并从 Promise 中解析结果。这个并发的结果其实就是 Node.js 本身的全部力量。

```
// This "possibly" works in one of the Threads in a pool
async function getWebContent(url: string) {
   // dummy example :) but it shows simple Promise use case
   const { data } = await axios.get(url);
   return data;
}

// This Works in Main Event Loop Thread
(async () => {
   const html = await getWebContent('https://google.com');
   // Do something smart with this HTML code!!
   // OR JUS Log it
   console.log(html);
})();
```

让我们考虑下面的情况，你必须同时请求 100 个 URL，然后对合并的结果做一些事情，或者对下一个 100 个 URL 进行另一个循环，等等。也许你必须写一些承诺循环来检查 URL 的 fire 承诺，并等到所有 100 个都完成，就像这个例子一样。

```
async function someAsyncFunction(url: string) {
   // Some Async Stuff
}

(async () => {
   const urls = ['url1', 'url2',...];
   const results = await Promise.all(urls.map(url => someAsyncFunction(url)));
   console.log(results);
})();
```

这导致并发地运行所有的异步函数，但是同时，你要等到所有的异步函数都完成之后才能继续你的工作。这并不一定是一件坏事，事实上，如果有必要，这被认为是旋转大量异步作业的最佳方式。但是，如果您不需要立即得到所有结果，但是您必须使用全线程池并发来获得更好的应用程序性能，那么有一种更好的方法来实现这样的结果！正如你已经猜到的是`EventEmitter`:)

# 事件发射器的力量

如果你认为事件是 Node.js 中的异步操作，那你就错了！EventEmitter 同步！是的，你没看错，事件订阅者本身被认为只是为特定情况(事件名称)触发的回调函数的数组，但在 Node.js EventEmitter 中，它使用核心 LibUV 事件循环周期来传递事件和执行回调，这意味着当你发出一个事件时，它将被添加到 LibUV 的事件触发堆栈中，以便在有同步时间可用于该操作时被触发。

要点是 EventEmitter 是一种理想的方式，可以并发地旋转异步操作，然后从该异步操作再次发出一个事件来运行另一个异步任务。使用这种技术，您不必等到 promise 被解析，相反，您将获得一个包含您需要的数据的事件回调，作为一个额外的好处，它将您的事件回调扩展到其他线程池成员，释放巨大的并发优势。

```
import EventEmitter from 'events';

// Just initiating new Event stack
const baseEvent = new EventEmitter();
const urls = ['url1', 'url2',...];
const responses = [];

async function runTask({url}) {
  const {data} = await axios(url);
  baseEvent.emit('response', {html: data});
}

baseEvent.on('request', runTask);

function handleResponse({html}) {
  responses.push(html);

  if (responses.length === urls.length) {
     baseEvent.emit('end', responses);
  }
}

baseEvent.on('response', handleResponse);

function doSomething(responses) {
   // DO something with responses here
}

baseEvent.once('end', doSomething);

urls.map(url => baseEvent.emit('request', {url}));
```

这看起来很难看，很难想象大量的代码比经典的`Promise.all`运行得更快，我更喜欢这种情况，因为你可以通过发出新的事件来拥有真正并发的东西，例如处理单独的响应，或者通过发出新的 URL 作为事件来保持 URL 队列的更新。

# EventEmitter 的问题

我会从一开始就说 EventEmitter 并不是在 Node.js 中实现高效并发的灵丹妙药，但在某些情况下它确实工作得很好。然而，它也有问题！

**出现事件死循环的问题。**这在大型代码库中很常见，你最终会发出一个事件，而这个事件又会引发一个初始事件。你必须记住，EventEmitter 是同步操作，每个`.emit()`函数都会在订阅的回调函数上运行一个同步循环并调用它们，这意味着通过事件发射循环，你会得到同样的无限递归问题。

**保持并发级别的问题。**当你运行 N 次异步操作时，它会在线程池中生成一个承诺队列，这意味着如果你发出一个事件(这是同步的), N 次队列以同样的方式增长。对于 Node.js，它可能以内存溢出崩溃或其他意外错误结束。

**订户太多的问题。默认情况下，EventEmitter 希望我们将订阅者的数量保持在尽可能低的水平，因为在每个订阅者上，它都在回调上运行同步循环，这会阻塞整个事件循环。最初的限制是每个事件只有 25 个订阅者，这对于一个普通的应用程序来说是完全可以的，但是您可以根据需要增加这个数字。拥有大量数据的主要缺点是 CPU 性能成本。到目前为止，对我来说最好的用例是调用`.off(name, callback)`函数，它从事件订阅者中删除给定的回调，这样当你不需要任何新的回调执行时，只需从订阅者列表中删除它，并保持订阅者数量较低。**

# 谁需要这些东西

大多数应用程序没有这种用例，这就是为什么 EventEmitter 不那么受欢迎，当你需要这样的东西时，你可能只是通过一些递归/回调地狱设计来实现它。但是如果你考虑一个抓取网页(像 hexomter.com 的[或者发送大量通知(电子邮件、Slack 等)的应用，你必须充分利用 CPU 性能，并且能够扩展到所有内核。](https://hexometer.com)

使用`cluster`模块有一些与此类似的实现，它使用相同的执行流和消息基础通信来创建单独的整个节点进程，但是维护单独的进程是非常昂贵的，尤其是如果您考虑到使用 EventEmitter 可以共享相同的全局变量而不考虑并行使用的话。使用`cluster`模块，您必须通过同步共享变量(如果有)来维护主进程/线程状态。

这也可以用于拥有一个基于 Express 的 Node.js 服务器，只是为了在主请求处理程序响应请求时保持不同的应用程序部分执行。这有助于不用担心上下文内存释放，当您希望继续运行某个任务，但又希望在实际任务完成之前向用户发送响应时，有时会发生这种情况。对于 Node.js 应用程序来说，这可能是你能想到的最糟糕的场景，但是我想给出一个通用的观点。

# 结论

EventEmitter 并不适合每个应用程序用例，您肯定可以用一个定制的实现来替换它，但最重要的是要记住 EventEmitter 与 LibUV 的 events 绑定在一起，后者是 Node.js 的主要事件循环引擎。

目前，我们有一个完整的网络废弃工具(【hexometer.com/broken-links】T4)建立在这个原则之上，这比最初的基于承诺的实现要快得多。

如果你喜欢这篇文章，请鼓掌并分享！👏👏👏