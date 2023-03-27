# 我编码快 40 年了，为什么还在做傻事？

> 原文：<https://itnext.io/ive-been-coding-for-almost-40-years-so-why-do-i-still-do-stupid-things-6ea458b8edff?source=collection_archive---------6----------------------->

![](img/c04f4c3243dfdbaf73cdf73a23b2c218.png)

堪培拉的阴影:作者拍摄的照片。

## 我刚刚发现了一个 bug，这个 bug 一度让我怀疑自己对 Javascript 基础的理解。

SS**ymptom**:我正在调用一个函数`getCommonData`，它是从一个名为`feedScanner`的模块中加载的。调用`getCommonData()`应该返回实际数据。在单元测试中是这样，但是，在实践中，它会返回到它的初始化状态。

```
// This fails as it thinks the scanner has not started.const commonData = getCommonData();
console.debug('commonData', commonData);
```

我 **调查**:看来我的预感是对的，有**两个** `feedScanner`模块！WTF。

```
*** Hey I’m a feedscanner ***
*** Hey I’m a feedscanner ***
```

> Javascript 导入被缓存了，为什么我会看到两个 T4 模块？

I调查:它**一定**与`webpack`配置有关。在这个项目中它是定制的和复杂的。

> 奇怪的是，也许这一直在发生，但我现在才注意到，因为现在是我第一次依赖这些数据。这可能是真的吗？

也

> 我只是重写了 feedScanner 是如何被触发的，这样它就不会启动，直到听到一条消息(通过`window.postMessage`)说一些‘公共’数据已经被捕获并准备好供`feedScanner`使用。

到

> 也许我真的不理解 Javascript `import`系统，或者至少不理解`Babel`对我的代码做了什么，以及它可能会带来哪些奇怪的副作用。

很多移动部件和很多已经改变了。

太容易成为替罪羊了`Webpack`。

我 **调查**:再挖。还是怪我的`Webpack`配置。摆弄我的`Webpack`配置中的东西，比如:

```
optimization: {
 runtimeChunk: ‘single’
}
```

这是疯狂的巫术，并迅速给了我四个单独的`feedScanners`！

## FML。

# ❄︎ — — — — — — — — — — — — — — — — — — — — ❄︎

我 **开始**考虑*陌生人*关于为什么可能有两个 feedScanners 的理论。我重写了很多相邻的代码。
我睡在这个问题上。我醒来时有了一个想法。

> 可能不是`Webpack`。

# ❄︎ — — — — — — — — — — — — — — — — — — — — ❄︎

# T 何的问题

## 简单地说。

当你解决一个问题时，没有比实现的洪流更好的了。唉，最终的结果是，你可以在一堆证据中增加一条，证明你实际上是一个多么愚蠢的人。

在接收空数据的文件中:

```
import { getCommonData } from ‘src/content/feedscanner’;
```

在所有其他与`feedScanner`模块相关的文件中:

```
import { start } from ‘src/content/feedScanner’;
```

`Webpack` **正在**加载*两个* `feedScanner`模块——但这既不是`Webpack`的错，也不是我的配置的错。这是更平淡无奇的东西。

T **he** 正确的拼写应该是‘feedScanner’但是 macOS 作为一个*非装箱*文件系统没有注意到。但是`Webpack`关心大小写，并假设它们是两个不同的模块。然而，它从同一个文件中提取了它们，所以它们是相同的代码，但是对`Webpack`来说，它看起来像是不同的文件，对最终的 Javascript 来说，它们是两个独立的模块。

# 吸取的教训

F **或第一百次** : macOS 不是一个区分大小写的文件系统，这并不总是由`Webpack`或你的`Webpack`配置造成的。

记住，伙计们，斯特金法则和其他任何东西一样适用于你自己。

感谢阅读。

—

像这样但不是订户？你可以通过[davesag.medium.com](https://davesag.medium.com/membership)加入来支持作者。