# RxJS-Toolkit:方便的日常 RxJS 工具，带有可管道化的操作器

> 原文：<https://itnext.io/rxjs-toolkit-convenient-everday-rxjs-utilities-with-pipeable-operators-a3ba09d22cf3?source=collection_archive---------2----------------------->

![](img/c6aafabd184abc4ca50ea957406e9af5.png)

> rxjs-toolkit 现已无人维护，并作为现有技术保留下来，以供将来参考。请注意，有些运算符在 IE11 中不起作用。10/5/19

这是`rxjs-toolkit`的开始，这是一个基于自定义[管道操作符](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)的小型实用程序库，专注于可重用性和日常使用。目前只有一小部分可用，随着时间的推移，会有更多的可用。

通过您喜欢的节点包管理器安装`rxjs-toolkit`:

```
npm i rxjs-toolkit
yarn add rxjs-toolkit
```

下面是一些操作符，可以帮助您入门:

# tapLog

当谈到调试`RxJS`流时，通常最好的工具之一是使用`tap`操作符并记录通过的信号。在`rxjs-toolkit`，有一个叫做`tapLog`的运营商。

# ignoreFalsySignals

有时，不需要的`null`或`undefined`值可能会被压入`Observable`流。我们可以在`map`中使用分支逻辑来缓解这些信号，或者使用`filter`操作符只让真信号通过。这是针对这种情况的`ignoreFalsySignals`操作符:

这里有一个更合理的情况，当你有一个来自一个函数的流设置，这个函数正在调用一个 API:

在上面的例子中，我们保证了信号不是虚假的，但是我们没有保证`signal.foo`的真实存在。我们可以用`propsAreTruthy`来解决这个问题。

# propsAreTruthy

当一个对象在流中行进时，有时它对于给定的属性没有真值。我们可以使用`propsAreTruthy`操作符来检查是否所有的顶级属性都是真值。如果不是，信号将被映射到`false`:

与检查所有顶级属性的默认设置不同，可以通过传递逗号分隔的字符串列表来检查特定属性和嵌套属性:

# composable:props are true+ignoreFalsyValues

为了让我们的生活变得简单一点，我们可以一起使用这两个助手。

*   检查对象属性的真实性
*   如果我们关心的对象属性为 falsy，则忽略该信号
*   保持水流畅通，让下一个信号通过

尽情享受吧！更多运营商即将推出:)