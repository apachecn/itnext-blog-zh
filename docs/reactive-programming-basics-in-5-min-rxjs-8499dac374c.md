# 5 分钟反应式编程基础(RxJS)

> 原文：<https://itnext.io/reactive-programming-basics-in-5-min-rxjs-8499dac374c?source=collection_archive---------5----------------------->

# 主要反对意见

这篇文章是在这个[教程](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)的启发下写的。我发现理解开头非常有用，特别是因为反应式编程的入门水平很高，尤其是 RxJS 的 api。描述这样的材料会引导你进行大量的反思，以了解你想要与读者分享什么新的东西，而不是重复已经遍布互联网的材料。

# 反应式编程的本质

什么是反应式编程？这是一个基于异步数据流的程序。在流下意味着开发过程中的一切可能:来自输入控件的事件、鼠标事件、变量、动作、数据结构、数据请求和响应。你只是宣布对这些流的反应，它们会协调所有的反应。

作为时间轴中有序事件序列的流可以发出三种不同类型的数据:数据有效载荷、错误和流结束信号。您可以在每种类型的数据上附加自己的处理程序，或者只关注数据负载。

订阅过程是监听特定流上的新事件。观察者是我们定义为对某些流事件的反应的函数。信息流本身是一种可观察的东西，我们在倾听新的事件。(这是一种观察者模式，在之前的[文章](https://medium.com/@ruslanmalogulko/easy-patterns-observer-63c832d41ffd)中描述过)。

流的主要好处是它们避免了内存泄漏，并且从内存消耗的角度来看非常便宜。将几个流加入到新的流中，过滤流事件，将一个流作为输入传递给另一个流，从另一个流中排除一个流的结果，这些都非常容易。

通常流用下一种方式描述:

```
---d-----d---d----x------d--|->d - emitted values
x - error
| - end signal of current stream
-> - timeline itself
```

# 您可以对流执行的主要功能

一般来说，在许多反应式库中，你可以找到许多已经实现的函数来帮助编排流。让我们来描述其中的一些。

**map** :基于当前流数据创建另一个流。(例如，对于每个发出的值，我们希望在单独的流中发出 1):

```
clickStream: ---a---b------c-----d--->
                   map(item => 1)
mapStream:   ---1---1------1-----1--->
```

**filter** :在过滤当前流的基础上创建另一个流(过滤发射值):

```
clickStream: ---3---2------4-----1--->
              filter(item => item > 2)
filterStream:---3----------4--------->
```

**扫描**:聚合所有之前发出的值，并作为函数`g(aggregatedValue, currentValue)`在流上产生值；

```
clickStream: ---2---2------3-----1--->
           scan((sum, item) => sum += item)
scanStream:  ---2---4------7-----8--->
```

**间隔**:创建流，以给定的间隔和开始延迟发出值；

```
interval(startDelay, intervalTime):
intervalStream:  -0-1-2-3-4-5-6-7-8-(.....)-->
```

**buffer** :创建按照某种标准(例如时间限制)缓冲发出值的流；

```
clickStream: ---2-2-------3------1-2------3--2------->
                     buffer(interval(1000))
scanStream:  ------[2,2]-----[3]-----[1,2]-----[3,2]->
```

**merge** :将两个或多个单独的流合并成一个结果流，从所有合并的流中发出数据；

```
clickStream:   ---c-c-------c------c-c------c--c------->
keyDownStream: -------k---k----k----k---k--------k-----> merge(slickStream, keyDownStream)
mergedStream:  ---c-c-k---k-c--k---ckc--k---c--c-k----->
```

反应式编程提高了代码的抽象级别，因此您可以专注于事件的相互依赖性，而不必经常摆弄大量的实现细节。今天，网络应用程序更加动态，有许多实时事件 UI 应该以某种方式作出反应。这是一个很好的想法，将这样的关注分成流，并忘记在应用程序上传播交互逻辑的地狱。

# 使用示例

让我们考虑一个不太复杂的解决 ui 问题的例子。我们在页面上有一些输入字段，这个输入应该在没有交互的 3 秒钟内阻塞(没有按键或鼠标移动)。只有在页面上单击三次鼠标后，用户才能继续编辑该字段。基于 RxJS 库的实现，它有非常丰富的 api，可以用于很多很多用例，甚至更多。但是对于当前的例子，我们只需要基本的，如上所述。

```
// let's create streams from native page events with 
// fromEvent functionconst moveSource = fromEvent(document, "mousemove");
const keyDownSource = fromEvent(document, "keydown");
const clickSource = fromEvent(document, "click");// let's get a link to page body, input control and prompting 
// text which is only available in edit modeconst body = document.querySelector("body");
const input = document.querySelector("input");
const prompt = document.querySelector("#prompt");// so, once our streams defined as a sources we can manipulate 
// them in needed way and subscribe to operate with resultclickSource
  .pipe( // collect all manipulations into the pipe
    buffer(interval(1000)), // buffer clicks per 1 sec
    filter(x => x.length > 2) // filter them by amount more than 2
  )
  .subscribe(x => {
      // describe what should happen when triple and more 
      // clicks happened (enable input control, focus that 
      // field and display edit prompt) input.disabled = false;
      input.focus();
      prompt.style.display = 'inline';
});// collect mouse keydown eventskeyDownSource
  .pipe(
    // and merge current stream with mouse move events stream merge(moveSource), // switchMap resets mapping function right after new stream 
    // value  emitted and waits 3 sec to block ui by passing 
    // to subscriber 'inactive' value switchMap(() =>
      interval(3000).pipe(
        mapTo('inactive')
      )
    )
  )
  .subscribe(x => { // in subscriber we checking if value came as 'inactive' 
    // and after that disabling input control and make prompt 
    // hidden to user. console.log(x);
    if (x === "inactive") {
      input.disabled = true;
      prompt.style.display = 'none';
    } 
  });
```

# 到目前为止还不错(而不是总结)

反应式方法是伟大而复杂的。但是它需要完成从命令式思维到陈述式思维的转换。在很多情况下，这看起来像是开销，但是你可以决定你的应用程序是否有很多需要复杂管理的事件。在任何情况下，关注分离都是重要且必要的…为什么不从系统中的事件开始呢？:)干杯