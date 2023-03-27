# Valtio 代理状态如何工作(普通部分)

> 原文：<https://itnext.io/how-valtio-proxy-state-works-vanilla-part-585ee38bd080?source=collection_archive---------2----------------------->

## 向可变状态添加不变性

![](img/6a257e4fe50dd3fd24e0d88d35f95ee8.png)

# 介绍

Valtio 是一个全局状态库，主要用于 React。它最初被建模为与 [useMutableSource](https://github.com/reactjs/rfcs/blob/master/text/0147-use-mutable-source.md) API 相匹配。然而，事实证明这是一个新的 API，它为可变状态增加了不变性。

什么是不可变状态？JavaScript 不支持作为语言的不变性，所以它只是一个编码契约。

```
const immutableState1 = { count: 0, text: 'hello' };// update the state
const immutableState2 = { ...immutableState1, count: immutableState1.count + 1 };// update it again
const immutableState3 = { ...immutableState2, count: immutableState2.count + 1 };
```

有些人可能对这种模式很熟悉，也可能对其他人来说是新的。我们总是在不修改现有对象的情况下创建一个新对象。这允许我们比较状态对象。

```
immutableState1 === immutableState2 // is false
immutableState2 === immutableState3 // is false// decrement count
const immutableState4 = { ...immutableState3, count: immutableState3.count - 1 };console.log(immutableState4); // shows "{ count: 1, text: 'hello' }"
console.log(immutableState2); // shows "{ count: 1, text: 'hello' }"// however their references are different
immutableState2 === immutableState4 // is false
```

不可变状态的好处是您可以将状态对象与`===`进行比较，以了解其中的任何内容是否可以更改。

与不可变状态相反，可变状态是 JavaScript 对象，在更新时没有任何契约。

```
const mutableState = { count: 0, text: 'hello' };// update the state
mutableState.count += 1;// update it again
mutableState.count += 1;
```

与不可变状态不同，我们改变状态并保持相同的对象。因为 JavaScript 对象本质上就是可变的，所以可变状态更容易处理。可变状态的问题是不可变状态好处的另一面。如果你有两个可变的状态对象，你需要比较所有的属性，看看它们是否有相同的内容。

```
const mutableState1 = { count: 0, text: 'hello' };
const mutableState2 = { count: 0, text: 'hello' };const isSame = Object.keys(mutableState1).every(
  (key) => mutableState1[key] === mutableState2[key]
);
```

这对于嵌套对象是不够的，而且键的数量可以不同。你需要所谓的 deepEqual 来比较两个可变对象。

对于大型对象，deepEqual 不是很有效。不可变对象在这里大放异彩，因为这种比较既不依赖于对象的大小也不依赖于对象的深度。

因此，我们的目标是在可变状态和不可变状态之间架起一座桥梁。更准确地说，我们希望从可变状态自动创建不可变状态。

# 检测突变

代理是一种捕获对象操作的方法。我们使用`set`处理程序来检测突变。

```
const p = new Proxy({}, {
  set(target, prop, value) {
    console.log('setting', prop, value);
    target[prop] = value;
  },
});p.a = 1; // shows "setting a 1"
```

我们需要跟踪对象是否发生了变异，所以它有一个版本号。

```
let version = 0;
const p = new Proxy({}, {
  set(target, prop, value) {
    ++version;
    target[prop] = value;
  },
});p.a = 10;
console.log(version); // ---> 1
++p.a;
console.log(version); // ---> 2
```

这个版本号是针对对象本身的，并不关心哪个属性被更改。

```
// continued
++p.a;
console.log(version); // ---> 3
p.b = 20;
console.log(version); // ---> 4
```

由于我们现在可以跟踪突变，下一步是创建一个不可变的状态。

# 创建快照

我们称可变状态的不可变状态为快照。如果我们检测到突变，即版本号改变时，我们会创建一个新的快照。

创建快照基本上就是复制一个对象。为了简单起见，让我们假设我们的对象不是嵌套的。

```
let version = 0;
let lastVersion;
let lastSnapshot;
const p = new Proxy({}, {
  set(target, prop, value) {
    ++version;
    target[prop] = value;
  },
});
const snapshot = () => {
  if (lastVersion !== version) {
    lastVersion = version;
    lastSnapshot = { ...p };
  }
  return lastSnapshot;
};p.a = 10;
console.log(snapshot()); // ---> { a: 10 }
p.b = 20;
console.log(snapshot()); // ---> { a: 10, b: 20 }
++p.a;
++p.b;
console.log(snapshot()); // ---> { a: 11, b: 21 }
```

`snapshot`是一个创建快照对象的函数。值得注意的是，快照对象仅在调用`snapshot`时创建。在那之前，我们可以做尽可能多的突变，这只会增加`version`。

# 订阅

在这一点上，我们不知道突变何时发生。通常情况下，如果状态发生变化，我们会想做些什么。对此，我们有订阅。

```
let version = 0;
const listeners = new Set();
const p = new Proxy({}, {
  set(target, prop, value) {
    ++version;
    target[prop] = value;
    listeners.forEach((listener) => listener());
  },
});
const subscribe = (callback) => {
  listeners.add(callback);
  const unsubscribe = () => listeners.delete(callback);
  return unsubscribe;
};subscribe(() => {
  console.log('mutated!');
});p.a = 10; // shows "mutated!"
++p.a; // shows "mutated!"
p.b = 20; // shows "mutated!"
```

组合`snapshot`和`subscribe`允许我们连接可变状态来做出反应。

我们将在另一篇文章中介绍 valtio 如何使用 React。

# 处理嵌套对象

到目前为止，我们的例子是简单对象，其属性是原始值。实际上，我们希望使用嵌套对象，这是不可变状态的好处。

嵌套对象看起来像这样。

```
const obj = {
  a: { b: 1 },
  c: { d: { e: 2 } },
};
```

我们也想使用数组。

Valtio 支持嵌套对象和数组。如果您对它是如何实现的感兴趣，请查看源代码。

[](https://github.com/pmndrs/valtio) [## GitHub - pmndrs/valtio:💊Valtio 为 React 和 Vanilla 简化了代理状态

### npm i valtio 使代理状态变得简单 valtio 把你传递的对象变成一个自我感知的代理。你可以做出改变…

github.com](https://github.com/pmndrs/valtio) 

# 结束语

在这篇博文中，我们在示例中使用了简单的代码。该实现做了更多的事情来处理各种情况。它仍然是最低的。

实际的 API 非常接近示例代码。下面是 TypeScript 中粗略的类型定义。

```
function proxy<T>(initialObject: T): T;function snapshot<T>(proxyObject: T): T;function subscribe<T>(proxyObject: T, callback: () => void): () => void;
```

在这篇文章中，我们讨论了 valtio 的香草部分。希望尽快写下反应部分。

*原载于 2021 年 8 月 27 日*[*【https://blog.axlight.com】*](https://blog.axlight.com/posts/how-valtio-proxy-state-works-vanilla-part/)*。*