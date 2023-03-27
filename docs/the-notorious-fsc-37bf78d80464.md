# 臭名昭著的 FSC

> 原文：<https://itnext.io/the-notorious-fsc-37bf78d80464?source=collection_archive---------3----------------------->

我很幸运有时间开始学习如何反应。对于任何建筑商来说，工具箱里都有一项令人惊叹的技术。在完成了[教程](https://reactjs.org/tutorial/tutorial.html)并自己做了一点试验后，我拿起了一本罗宾·维鲁奇的 [*学习反应之路*](https://www.robinwieruch.de/the-road-to-learn-react) 。如果你对 React 感兴趣，你可能也读过——如果没有，帮自己一个忙，去看看吧！就是反应出你不知道的[*JS*](https://github.com/getify/You-Dont-Know-JS)对于 Javascript 的意义。

我有一个坏习惯，就是忘乎所以，发表过于密集(阅读:太长)的博客文章。作为我 React 之旅的一部分，我的一个目标是将我学到的重要东西写在博客上…一边写一边改进我的帖子。每个人都应该有足够的知识来变得有趣，但要比一部中篇小说短。我们走着瞧。🙂

今天我想谈谈 React 中不吉利的命名“FSC”或*功能性无状态组件*。不是从专家的角度，而是作为和你一起学习它们的人。它们是什么？你应该什么时候使用它们？我们将回答这些问题，并在此过程中重构一个简单的组件…

![](img/4f0c3c075d4ed42e5ab02dc48c1a615b.png)

像 FSC 这样的城市不必不透明！

# 什么，什么时候，为什么？

我们来拆开“FSC”——*功能性*、*无状态*、*组件*。原来名字里有很多东西。与 *ES6 类组件*构建在 [ES6 类](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)之上不同，FSC 是使用…等等… *函数*组成的！嗯，好吧。我们一直在使用函数，这并不可怕。所以我们有两种表示 React 组件的方式…我们如何选择一种呢？嗯，课程是 ES6 的，所以它们一定更好，对吗？请继续阅读！

特别是有了像 [create-react-app](https://github.com/facebook/create-react-app) 这样为你处理传输的现代工具，我认为可以肯定地说*你可以用功能组件做任何你不能用类组件*做的事情。你经常会看到完全基于类的简单的无状态教程。这样做的一个好处是*一致性*——组件有一个共同的外观和感觉，所以你可以说在浏览代码时有更少的认知负荷。我认为在某些情况下，模板化或生成样板文件也更容易。

反之则不然… *你可以用类组件做 FSCs 不支持的事情*。线索就在名字里:*无状态*。FSC 只能访问[属性](https://reactjs.org/docs/components-and-props.html)，不维护[本地状态](https://reactjs.org/docs/state-and-lifecycle.html) ( `this.state`)。这也意味着您失去了对生命周期方法的访问(FSC 本身，或其返回的 JSX，实际上是 render 方法)。

正如有人可能认为一致的组件方法使代码更容易阅读一样，另一方面是*FSC 需要更少的代码行*(更少的潜在错误)并且在浏览代码时呈现更少的“噪音”。如果你的应用程序需要状态，你将失去一些“一致性”，因为你仍然需要一个或多个类组件。

在我看来，过多考虑性能是不成熟的优化，但大型项目可能会看到将类组件重构为 FSC 的好处——如果不需要状态，就可以避免管理其生命周期的潜在开销，还可能看到更小的包大小。

请注意，即使在 FSC 中，您仍然可以在接收`props`和返回 JSX 之间工作。您可以在您的 FSC 中添加自定义功能或其他代码，它们只是不会像`constructor()`、`componentDidMount()`、`componentWillUnmount()`等那样与[组件生命周期](https://reactjs.org/docs/state-and-lifecycle.html)中的特定阶段相关联。

# 变得真实

一个片段抵得上一千个博客，所以让我们重构一个简单的组件……[*React*](https://reactjs.org/docs/thinking-in-react.html)中的思考是一个有趣的练习，它会带你将一个模拟组件变成一个真实的组件。在[的示例解决方案](https://codepen.io/gaearon/pen/BwWzwm)中，一切都被一致地实现为 ES6 类。教学的明智选择(甚至可能是您的个人偏好)，但是让我们将一个组件重构为 FSC 来巩固上述理论。

一个简单的 React ES6 类组件。

我是初学者，这是教程中的一个示例组件，由比我聪明的人编写……正如你所料，它已经很容易阅读了。当我们重构的时候，保持开放的心态，试着**想象我们观察到的任何小收益在一个大项目中被放大**。

在我们开始处理代码之前，问一个问题总是好的，“**这是重构的好候选吗？”**根据我们目前所知，我们看到`ProductCategoryRow`没有引用`this.state`或任何生命周期方法。很好，看起来我们可以安全地把它变成一个函数:

与 FSC 的组件相同。

它的工作原理是一样的，现在我们只是接收`props`作为函数参数。除了那个小的改变，我们旧的渲染方法变成了我们的 FSC。这已经减少了需要考虑的代码行，但是 ES6 语法糖帮助我们消除了更多的视觉混乱。组合 [*箭头函数*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 、 [*析构赋值*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) 和 [*简洁体*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#Function_body) 产生一个将注意力吸引到输入和输出的简化函数:

ES6 语法糖

属性在函数签名中被析构，所以如果你传递更多的参数，只需添加参数...默认值也可以。从技术上来说，你可以选择简洁的黄金，去掉回车的括号，但是这会让一些语法高亮变得疯狂。😱

我们的 FSC 去 11！

# 结论

![](img/7e9765eb37f09fda8bc36feb69190530.png)

FSCs 可以简化你的代码。

如果你和我一样是 React 新手，希望这有助于澄清什么是 FSC，什么时候它们是合适的，以及如何使用它们。我们从*十二*行代码中取出一个非常简单的组件到*七* ( *> 40%缩减*)，这就给出了添加功能的一行代码(传入`children`)。当设想如何简化围绕生产代码的测试来匹配时，这真是令人大开眼界！ES6 语法确实提高了可读性和可维护性，并且由于有了像 [create-react-app](https://facebook.github.io/create-react-app) 和 [babel](https://babeljs.io) 这样的工具，大多数人都可以使用它。

一个尺寸肯定不能适合所有… *在现实世界中，你可能会有结合了 ES6 类和功能性无状态组件的应用*。对于初学者或中等规模的项目，FSCs 可能是一个不成熟的优化。你需要知道取舍，并为工作选择合适的工具。对于某些人来说，一个更简洁的代码库，用更少的代码行来隐藏错误，可能就足以证明重构的合理性。