# 不用真正尝试就能理解异步迭代器

> 原文：<https://itnext.io/understand-async-iterators-665259680044?source=collection_archive---------3----------------------->

![](img/3e5f51223477076999f308918b505bcc.png)

保罗·坎德罗拍摄的照片

没人喜欢回调。这就像是现代版的`GOTO`声明。但是我们怎样才能取代那些讨厌的东西，比如点击处理程序呢？很有可能，您已经将所有简单的语句都重构到 async/await 语句中，但是仍然存在那些讨厌的顽固分子，比如 DOM 事件、WebSockets、事件发射器和任何其他类似流的对象。当然，我们可以把这些变成可观的，但是把 1 次回调变成 3 次回调似乎并没有太大的改善。如果我可以承诺一个函数，为什么我不能迭代一个事件流？答案是新的 2018 async iterables，自从几年前首次提出以来，已经有很多旧的、令人困惑的、完全错误的信息存在。所以，通过反复试验的魔力，我写了一个助手来把点击处理程序变成 for 循环。

# 开始简单

在编程中，就像在生活中一样，懒惰通常更好。所以，当我构建新的东西时，我喜欢尽可能地懒惰，延迟编写让我思考的代码。所以，让我们写下我们想要的样子&把所有难写的东西放进一个黑盒子里:

```
*// I want to go from this* document.addEventListener('click', (event) => console.log('click')

*// to this
const* clicks = **streamify**(document, 'click')
*for await* (*const* event *of* clicks) {
  console.log('click')
}
```

到目前为止一切顺利！现在我们只需要那个神奇的`streamify`函数来实际做一些事情。我们知道它将调用`addEventListener`，它需要一个`handler`来获得`event`。因此，让我们只填写简单的部分，将困难的部分放在另一个黑盒中:

```
*function* streamify(element, listener) {
  *const* handler = (event) => { **// do magic here** }
  element.addEventListener(listener, handler)
}
```

进步！现在避开困难的事情，让我们想想如何返回一个“异步迭代器”。老实说，我真的不知道那看起来像什么，我不能只写`new AsyncIterator()`，但我知道`async`意味着`promise`，迭代器是返回多于 1 个值的东西，有点像生成器，所以让我们滚动它，返回无限的承诺。无用的函数是我的专长，所以很简单:

```
***function**** streamify(element, listener) {
  *const* handler = (event) => {}
  element.addEventListener(listener, handler)
  ***while* (*true*) {
    *yield new* Promise(resolve => resolve(event))
  }**
}
```

现在我们在有土豆的地方锄地！看起来`handler`正在得到一个`event`，而`Promise`正在产生一个`event`，所以现在我必须以某种方式将`event`从`handler`移动到`Promise`。我能想到的最愚蠢的事情就是共享一个变量，所以让我们这样做:

```
*function** streamify(element, event) {
 ***let* nextResolve**  *const* handler = (event) => {
 **nextResolve(event)**  }
  element.addEventListener(event, handler)
  *while* (*true*) {
    *yield new* Promise(resolve => {
 **nextResolve = resolve**    })
  }
}
```

我们做到了！我们有一个正常工作的整流器！当一个点击事件进来时，它用我们给它的承诺来解决。相当甜蜜！…直到有人点击*真的*快。

# 加强水流调节器

天真的解决方案只适用于缓慢的事件。如果两个事件被快速连续触发，第二个调用将被吞掉。我们不希望这样，尤其是如果我们将它用于 websockets 之类的东西。所以，管它呢，如果 1 个共享解析器是好的，那么一系列共享解析器肯定会更好！让我们按事件的顺序排列它们:

```
*function** streamify(element, event) {
 ***const* pushQueue = []**  *const* handler = (event) => {
 **pushQueue.push(event)**  }
  element.addEventListener(event, handler)
  *while* (*true*) {
    *yield new* Promise(resolve => {
 ***const* nextEvent = pushQueue.shift()**      resolve(nextEvent) // error! no nextEvent
    })
  }
}
```

不，这解决了推送问题，但是现在我们得到了一个错误，因为迭代器在一个点击事件存在之前请求了一个点击事件。该死。好吧，队列用于推送事件，为什么不用一个队列来拉取解析器呢？

```
*function** streamify(element, event) {
  *const* pushQueue = []
 ***const* pullQueue = []**  *const* handler = (event) => {
    *const* nextResolve = pullQueue.shift()
    *if* (nextResolve) {
      nextResolve(event)
    } *else* {
      pushQueue.push(event)
    }

  }
  element.addEventListener(event, handler) ***const* pullValue = () => *new* Promise(resolve => {
    *const* nextEvent = pushQueue.shift()
    *if* (nextEvent) {
      resolve(nextEvent)
    } *else* {
      pullQueue.push(resolve)
    }
  })**
  *while* (*true*) {
 ***yield* pullValue()**  }
}
```

呜！有用！现在我可以永远随心所欲地点击了！…但是如果我只想听第一对夫妇呢？本来，我会调用`removeEventListener`，但是处理函数隐藏在我们可爱的新包装器中。当流结束时，我如何删除它？好吧，我从简单的&开始，只要写出我想要的样子:

```
*// from this
let* clickCount = 0
*if* (clickCount++ > 2) {
 **document.removeEventListener('click', handler)**  console.log('Bye!')  
}

*// to this
let* clickCount = 0
*for await* (*const* event *of* clicks) {
  console.log('click', event)
  *if* (clickCount++ > 2) **clicks.return()** // this breaks the loop
}
console.log('Bye!')
```

我知道生成器值有那个神奇的`return()`方法，但是我希望我可以自己写，这样我也可以调用`removeEventListener`。我知道那个小`*`其实并不神奇；它只是告诉函数用 4 个方法将返回值包装在一个特殊的对象中。因此，让我们抛弃星号&来编写引擎实际读取的生成器(加上添加我们的小额外处理程序):

```
***function***streamify(element, event) {
  ...
  *return* {
    [Symbol.asyncIterator] () {
      *return this* },
    next: () => ({
      done, // TODO how do we calculate this?
      value: **pullValue()**
    }),
    *return*: () => {
 **element.removeEventListener(listener, handler)**
      *return* {done: *true*}
    },
    *throw*: (error) => ({done, value: Promise.reject(error)})
  }
}
```

这看起来令人望而生畏，但它实际上只是`*`免费提供给我们的一堆样板文件。在发电机出现之前的糟糕日子里，我们不得不一直写类似的蹩脚的东西。今天，这个样板文件看起来应该很熟悉，因为它和你在控制台上登录`Set`或`Map`时看到的是一样的。

让我们看看我们得到了什么:`Symbol.asyncIterator()`方法告诉世界上的其他人把它当作一个生成器，而不是一个恰好具有完全相同字段的对象。其他所有的都返回一个带有`done`字段的对象，可能还有一个`value`。如果`done`为假，我们知道我们可以期待*某种*类型的值——因为这是一个*异步*迭代器，我猜这个值将是我们之前给出的同一个`pullValue()`承诺。剩下要做的就是在调用`next()`时计算出`done`的值。既然我们知道如果调用了`throw`或`return`，那么`done`应该为真，那么让我们像在&之前一样，在外部作用域中共享变量:

```
*function* streamify(element, event) {
  ...
 ***let* done = *false*** *return* {
    [Symbol.asyncIterator] () {
      *return this* },
    next: () => ({**done**, value: pullValue()}),
    *return*: () => {
 **done = *true***element.removeEventListener(listener, handler)
      *return* {done}
    },
    *throw*: (error) => {
 **done = *true*** *return* {done, value: Promise.reject(error)}
    }
  }
}
```

…我们是`done`！我现在可以编写一个 for 循环，每次有事件进来时都进行迭代。如果我不喜欢这个值，我可以调用`throw`。当我玩够了，我可以打电话给`return`。

以下是它的整体外观:

```
*const* streamify = *function* (element, event) {
  *const* pullQueue = []
  *const* pushQueue = []
  *let* done = *false
  const* pushValue = *async* (args) => {
    *if* (pullQueue.length !== 0) {
      *const* resolver = pullQueue.shift()
      resolver(...args)
    } *else* {
      pushQueue.push(args)
    }
  }

  *const* pullValue = () => {
    *return new* Promise((resolve) => {
      *if* (pushQueue.length !== 0) {
        *const* args = pushQueue.shift()
        resolve(...args)
      } *else* {
        pullQueue.push(resolve)
      }
    })
  }

  *const* handler = (...args) => {
    pushValue(args)
  }

  element.addEventListener(event, handler)
  *return* {
    [Symbol.asyncIterator]() {
      *return this* },
    next: () => ({
      done,
      value: done ? *undefined* : pullValue()
    }),
    *return*: () => {
      done = *true* element.removeEventListener(event, handler)
      *return* {done}
    },
    *throw*: (error) => {
      done = *true
      return* {
        done,
        value: Promise.reject(error)
      }
    }
  }
}
```

# **结论**

如果你一直滚动到底部找到 GitHub repo 的链接，[这里是](https://github.com/mattkrick/asynciterify)。

在 5 分钟内，我们已经编写了一个 50 LOC 的包装器，它可以与所有存在依赖性的竞争包直接接触。在包装了我们的事件之后，我们终于可以宣布我们不再编写带有回调的代码了。2012 年的梦想来了！最棒的是，我们没有学习什么是异步可迭代*而是学习了*如何使用它。我们不必拿出函数式编程教科书来研究一个`Subject`、`Deferrable,`或`Disposable`的定义。相反，我们建造了一些实用的东西——比如上烹饪课，然后带着晚餐离开。*方式*比 CS 理论更好玩。

唯一的问题是:仅仅因为我们可以，就意味着我们应该吗？从干净代码的角度来看，也许是吧！但是性能呢？承诺天生就比回调慢，最重要的是，我们在两个队列中调用`shift()`一堆。在 Chrome 67 中运行代码，对于 100-1，000，000 个事件，它比原生回调慢 2-100 倍以上。当代码被传输到 ES5 时，速度会慢 20-1000 倍！因此，从基准的角度来看，这简直糟透了。但在现实世界中，这相当于额外增加 0.1 毫秒来处理 10 个并发事件。放弃试镜是公平的交易！