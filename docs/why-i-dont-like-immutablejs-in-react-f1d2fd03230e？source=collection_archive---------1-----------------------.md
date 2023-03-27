# 为什么我不喜欢反应中的不变量

> 原文：<https://itnext.io/why-i-dont-like-immutablejs-in-react-f1d2fd03230e?source=collection_archive---------1----------------------->

![](img/8257f284815a45220fab12f0c7b50a78.png)

我不认为我需要向唱诗班宣讲 javascript 应用程序中的**突变**有多邪恶，以及 ImmutableJS 如何通过将数据封装到具有私有属性的对象中来帮助消除它。

这的确是一个非常好的主意。它迫使你不能变异任何东西，**在整个应用**！

我个人喜欢。但是，说到反应，我发现有点(或者非常)没必要。为什么让我们从不变量试图解决什么问题开始。

# 不变的

因为真理的唯一来源是 JS 中的圣经，允许改变价值观可能是灾难性的。例如，我有一个从服务器请求的客户机数据列表，我将它传递给两个组件。组件 A 改变它，但是组件 B 不知道它。更糟糕的是，原始的客户端列表永远消失了，直到您用另一个远程请求重新设置它。

ImmutableJS 是如何将所有东西包装(或封装)到它自己的映射或对象中的，就像 Java 一样。另外，它使所有属性都是私有的，并且只提供 getters 和 setters。好就好在，setters 不设置，他们用你想要的新值返回一个全新的对象！所以基本上你不能改变原始对象的任何值，一旦它被 ImmutableJS 包装。

## React 是如何关联的？

在 React 中你要处理的只是**道具**和**状态**。让我们快速回顾一下。

**props** :只读，从父节点传递。即使你可以给一个道具赋予不同的值，它也不会改变视图，它不会*。这一点非常重要。*

***状态**:局部变量，或者我应该称之为常量。原因是，给**状态**赋值也没有任何意义。它不更新视图，并且值只在您的作用域内局部改变(在您更新它的函数内)。*

*好吧，如果你还在猜测我想说什么，那么你可能会继续读下去。*

*作为一名经验丰富的 React 开发人员，除了一些实验之外，我从未使用过，也没见过给**道具**或**状态**赋值。如果你正在做，在你被抓住之前，停止！虽然，我真的相信，每个 React 开发人员在他们的 React 旅程中，或者旅程本身，会很快停止这样做。*

*这就是我的观点:在 React 中，你永远不要给**道具**或**状态**赋值，只有它们才是重要的。*

> *你在生命周期之外创建的其他变量呢？*

*我的问题是:它们重要吗？*

*恐怕不行。如果你试图用一个变量来更新视图，而不是那两种情况，你会很纠结，最终会很沮丧。否则，您就没有使用 React 的功能。*

*尽管 React 对道具或状态赋值没有限制，但你不会喜欢做，不会想做，也不会做。*

# *功能的*

*React 本身是功能性的。为什么*

*每个 React 组件实际上都是一个函数。道具和状态都包在里面。如果你想从一个孩子身上改变一个父母的状态，他们唯一的办法就是使用一个函数(当然，你是在设置一个新的状态)。*

> *但是 ImmutableJS 提供了全功能的 API。*

*我完全同意，这也是你应该做的。有很多图书馆可以帮助你。我会推荐 [RamdaJS](http://ramdajs.com/) 。*

*它提供的功能比你想象的要多，而且都是纯功能性的。你可以做 currying，组成，函子和许多许多令人兴奋的功能。如果你想了解更多细节，请观看下面的视频。*

*是的，你必须在你的代码中使用另一个库，就像你正在使用的库太多了。但这是唯一的缺点，我的朋友。老实说，我不认为这是骗局。*

# *与 Redux 一起使用*

*我读过一些关于 ImmutableJS 与 Redux 配合得有多好的文章。*

*我不是说它不是，但是真的，你需要吗？*

*Redux 中的减速器也完全可用。Reducers 总是返回一个新状态。您可能希望只在 reducers 中使用不变量，以避免传入的**状态**发生变异。但是对于它返回的状态，真的需要让它成为不可变的对象吗？答案上面提到了。*

# *没有永恒*

## *首先，调试愉快！*

*对于 ImmutableJS，我必须一次又一次地点击，才能在控制台中展开包装好的数据。如果只是一个简单的对象嵌套列表，我无法简单的找出里面是什么。*

*有一些 devtools 可以帮助你将数据恢复到正常的*状态，尽管如此，我还是不喜欢那种无法获得数据的感觉。**

## *ES6 特性和更多功能*

*我喜欢 ES6 的功能。使用 ImmutableJS 有点糟糕。而不是使用部分赋值，你需要一直调用 *get* 。*

```
*// ES6
const {profile, image, age, gender} = this.props.client;// ImmutableJS
const profile = Immutable.get(this.props.client, 'profile');
const image = Immutable.get(this.props.client, 'image');
const age = Immutable.get(this.props.client, 'age');
const gender = Immutable.get(this.props.client, 'gender');*
```

*和扩展运算符:*

```
*// Spread operator
return {
  ...state,
  profile: R.map(mapClientToProfile, state.client),
  updated: true,
  someOtherValue: value
}// ImmutableJS
return merge(state, {
  profile: ImmutableJS.get(state, 'client').map(mapClientToProfile),
  updated: true,
  someOtherValue: value
})*
```

*有人认为使用 spread 运算符的性能不如 *merge* 函数有效(在这里运行一些测试[)。而语法方面，spread 运算符和调用函数没有太大区别，所以 function 赢在这一点上。正如你所看到的，ImmutableJS 正试图转移到 functional，然而，它仍然不太正确。(看上面的视频，真的很说明问题！)](https://www.measurethat.net/Benchmarks/Show/579/1/arrayprototypeconcat-vs-spread-operator)*

*总而言之，在 React 中，更新视图的机制设计得很好，可以让你开始学习如何避免直接变异，并做更具功能性的事情，ES6 也提供了非常好的功能来帮助你忘记变异。*

*改为纯函数是避免代码突变的另一种方法，RamdaJS 就是一个很好的工具。ImmutableJS 还提供了一些功能，如*映射*、*过滤*和*查找*，但很常见的是，人们解开一个不可变的对象，进行复杂的映射，然后再包装回去。为什么一开始就要包装它们呢？它增加了数据的复杂性，而不是简化数据。*

*对我来说，如果你真的理解突变有多糟糕，并愿意使用库来避免它，ImmutableJS 只是选项之一，而且肯定不是最好的。有些人用它来防止其他开发人员做错误的事情，这是可以理解的，但你知道，总有更友好的方式。*