# 设计 SolidJS:悬念

> 原文：<https://itnext.io/designing-solidjs-suspense-f4e92c625cb5?source=collection_archive---------2----------------------->

## React 不是唯一能够停止时间的库。

![](img/43f04669f4d093529798ddbe91d50379.png)

来自 pixabay.com 的自然水滴

[*SolidJS*](https://github.com/ryansolid/solid) *是一个高性能的 JavaScript UI 库。本系列文章深入探讨了设计该库的技术和决策。使用 Solid 不需要了解这些内容。今天的文章主要关注 Solid 的异步渲染方法。*

悬念和并发模式被许多人吹捧为 web 开发的未来。React 正在开发的这些特性将彻底改变 web 应用程序的设计方式。但这些都是复杂的话题，虽然[丹·阿布拉莫夫](https://medium.com/u/a3a8af6addc1?source=post_page-----f4e92c625cb5--------------------------------)的一系列[推文](https://twitter.com/dan_abramov/status/1120971795425832961)有助于促进对 React 这些功能意图的理解，但我仍然觉得普通公众还没有理解。更重要的是，我认为他们没有办法将这种方法与其他方法进行比较。因此，我觉得我在为所有事物的反应式库开发相同的特性集时，遇到了一些关键的见解。伙计们，这里没有虚拟世界。因此，当我探索 Solid 解决方案的细微差别时，我希望同时提供一些关于实际情况的见解。

# 异步渲染与细粒度反应

异步渲染、时间分片以及并发模式在某个时候都是这个特性的名称。命名的进展相当有见地地展示了 React 团队对他们正在探索的特性的进展。我要说的是，这是一个正在解决的虚拟 DOM 问题。从反应式库的角度来看，这并不特别令人印象深刻。然而，并发模式不仅仅是简单的情况。

从最基本的意义上来说，这是从允许 React 中断渲染并在合理的时候继续开始的。React 这样的虚拟 DOM 库的问题在于它是基于自顶向下的协调的。这意味着无论何时发生变化，所有下游组件都会重新呈现。有一些优化可以防止这种情况(比如`shouldComponentUpdate`)，但是在初始渲染时，这不是你可以优化的，因为工作需要发生。因此，在渲染过程中，基于某种状态(如 if 语句)阻止部分渲染并触发预定的计时器仍然会导致大量工作。

然而，同样具有反应性的库像固体却不是这样。没有和解。它可以从停止的地方继续工作。一种想法可能是，React 16 和 Fiber 引擎只是给了虚拟 DOM 类似于反应库的粒度。但这不是什么新鲜事。KnockoutJS 早在 2010 年就有这种能力。

# 调度和动画基准

如果这是故事的结尾，我想没有人会谈论这个。第一个 React Fiber 演示展示了 React 调度工作的新能力，允许 React 15 无法比拟的流畅动画。最受欢迎的是 Sierpinski 三角演示。一般来说，基准测试强调了对时间片和这种性质的演示的误解。你看，在这之后，一堆库做了他们的尝试，他们中的许多人给出了比 React 16 更好的结果，添加了更多的三角形等等…但这证明不了什么。

所有这些演示都有两个问题。首先，该测试旨在展示在高负载下的适度降级，而不是最大化性能。我们在一个使用 DOM 的浏览器中。这不是你游戏引擎里的 C 程序。当所有事情都被优化时，操作 DOM 的成本比在 JavaScript 中做的任何事情都要高 100 倍。

计划是一种少做工作的练习。因此，为这件事沾沾自喜还为时过早。简单来说，它要慢一些。我们在基准测试中已经看到了这一点。请求动画帧会减慢将大表格呈现到屏幕上的速度。最终，调度会带来更多的开销，但好处来自于有选择地保持 UI 的部分更具响应性。但这意味着对这种东西进行基准测试是相当武断的。如果 60fps 是你的上限，你可以在 16 毫秒的帧中写很多垃圾，但这没什么关系。然而，分割渲染，做更多的 DOM 操作，总的来说更昂贵。JS 框架基准测试在 DOM 操作的成本上是一个苛刻的老师。在成为有争议的[最快的 JavaScript UI 框架](https://medium.com/javascript-in-plain-english/how-we-wrote-the-fastest-javascript-ui-frameworks-a96f2636431e)的道路上，我已经被教导过很多次了。

第二，我们仍然受到浏览器的约束。每个人都有相同的调度技巧。`setTimeout`、`requestAnimationFrame`、`Promise.resolve`等等……演示中所有的 React does 都叫`requestIdleCallback`。它可能不是一个通用的 API，但它是一个本地 API。我做了一个可靠的实现，将状态传播封装在同一个处理程序中，很快它就像 React 一样工作了。不过如果你看看这个 GitHub 问题:[https://GitHub . com/claudio pro/react-fiber-vs-stack-demo/issues/3](https://github.com/claudiopro/react-fiber-vs-stack-demo/issues/3)我很快意识到这个实现并不理想。使用功能较弱设备的人注意到，React Fiber 从不更新圆圈中的数字。我能够在 Chrome 中通过 CPU 节流轻松重现这一点。这不是 React 的错，只是 Chrome 的`requestIdleCallback`实现选择了如何处理负载。那时，我意识到这不是那么简单，React 实现才是。我用 Solid 建立了一个稍微复杂一点的调度机制，可能有点不太流畅，但是保证了更新。这是微不足道的，因为固体已经暴露了这种控制的能力。

[](https://github.com/ryansolid/solid-sierpinski-triangle-demo) [## ryansolid/solid-Sierpinski-三角形-演示

### 由 React Fiber demo 推广。我强烈建议打开 Chrome 调试器的“性能”标签，调节 CPU…

github.com](https://github.com/ryansolid/solid-sierpinski-triangle-demo) 

# 并发模式

因此，很明显，这些基础只是将虚拟 DOM 的动画性能与反应式库更好地结合起来。事实上，像 Inferno 这样超级高效的虚拟 DOM 库可能没有它也能做得很好。我敢肯定，任何人想出的令人惊叹的时间切片演示都会很容易地被一个反应库匹配，如 Solid 或甚至 Svelte。但这都是题外话。这不是性能的问题，而是体验的问题，并发模式比糟糕的动画演示要酷得多。

让并发模式令人惊叹的不一定是真正的并发，记住浏览器中的 JavaScript 大多是单线程的。虽然已经显示出有希望的结果，但是卸载到工作线程的实验还没有在这个领域中进行，因为它们不能在 DOM 上工作。同样，DOM 操作的成本使得聪明的工作委托在大多数情况下成为一种开销。并发模式允许维护多个呈现状态，以便在我们等待的时候可以更快地进行呈现，而最终用户不会知道。并发模式中的“并发”是 React 可以向您显示一个内容，同时在屏幕外呈现下一个内容。这里的潜力是巨大的。它从这里描述的缓冲开始，这就是悬念的全部，但也可以转移到一些很酷的事情，如 SSR 渲染的部分流水合或对多种可能的未来进行预测渲染(我希望有人将此称为量子渲染)。这是罕见的和有趣的，我知道我想有坚实的东西。

# 焦虑

如果你一直在跟踪，你会发现像 Solid 这样的细粒度反应库已经具备了支持这种内置的所有基础，而不需要昂贵的重写。因此，当 React 实验分支出现时，我已经能够跟上它，因为该模型基本上是固体的自然状态。然而，有一些关键的区别。当然，最大的问题是，Solid 是一个反应库。如果你有兴趣的话，在这个过程中开发解决方案有点像 FPGA 编程。Solid 的反应式系统是基于同步反应式编程的，所以我感觉有点像是在编程设计最终用户将使用的硬件。发展发生在 3 个阶段，每一个建筑都建立在以前的基础上。

## 1.后备基础

简单的悬念

这是你可能熟悉的简单的悬念演示。当代码读取尚未加载上游占位符的异步加载值时，会呈现回退，而不是显示当前部分呈现的组件。这与第一节中提到的简单方法的关键区别在于它是非阻塞的。条件性不继续渲染树，而悬念允许渲染继续，让更多的异步请求触发。

React 通过我见过的最聪明的 JavaScript 技巧之一实现了这一点，它承诺立即中断渲染，稍后再继续渲染。Solid 的实现并不那么鼓舞人心。因为 Solid 无论如何都会在屏幕外渲染(更多细节见 [JSX 文章](https://medium.com/@ryansolid/designing-solidjs-jsx-50ee2b791d4c)),我们只是利用了上下文 API。Solid 继续在屏幕外渲染组件，当它命中异步加载函数时，无论是`lazy`包装的组件还是`loadResource` for any promise，它都会进行上下文查找以找到`Suspense`提供者，并通知它增加一个计数器，以便当它完成渲染并返回树连接时(记住 JSX 从里到外执行),`Suspense`提供者组件返回回退而不是渲染的子组件。当异步请求完成时，计数器递减，当达到零时，真实的孩子被交换到视图中。没什么更复杂的了。没有像电路那样简单地进行缓冲的起止渲染。

## 2.过渡

我想大多数反应式库可以很轻松地实现这样的悬念，而不需要对它们的库做任何核心修改。但这还不是悬念如此强大的原因之一。真正的强大之处在于，在初始加载之后，您可以呈现下一个状态，同时继续显示上一个状态。标签页之间的图片切换。当然，你可以显示你的回退，但是当你看到页面卸载时，总感觉有点像后退。相反，即使技术上响应速度较慢，保留以前的内容，甚至避免回退状态，感觉也会更快。

React 包装 state setter 的方法是第一个可以用同样的方式处理 Solid 的方法。因为 Solid 是同步执行的，所以包装更新并临时全局提升配置以供所有下游暂挂执行读取允许暂挂在第三保持状态下工作。在这种状态下，我们推迟显示回退。对于简单的情况，这是可行的，但是很快你就会遇到一个有控制流的场景。悬念边界内的一些条件视图或循环迭代。他们会自己切换。这是有问题的，因为 Solid 不是自顶向下的。任何下游更新都可以独立执行，即使我们在悬念占位符处持有当前子视图。鉴于悬念只是一个上下文 API，控制流的处理相当容易。

暂记标签

结果是用`useTransition`来包装状态更新，并将`awaitSuspense`传递给控制流组件，我们可以保持当前呈现的视图，直到过渡超时，这种方式非常类似于 React 中的悬念过渡。

## 3.悬念列表

在这一点上有点反高潮，但正如你可能猜到的，我在这里更多地使用了上下文 API。这一次允许子悬念和悬念列表元素注册它们自己的机制，然后基于规则控制这些子元素的呈现。我希望有比这更多的东西，但同样的非阻塞方式，我们只是连接规则。

暂停列表

试着重新加载这个例子几次。根据首先加载的是帖子还是有趣的事实，你会看到它的加载方式有所不同。这个`SuspenseList`总是按照模板顺序显示元素(`forwards`，并折叠回退，只显示当前加载的元素。

# 结论

所以你有它。在反应库中实现的暂停。它的实现方式与 React 完全不同，但却获得了相似的结果。React 也有一些不可用的好处:

*   Solid 的版本渲染 DOM 的速度更快，所以当悬念最终解决时工作量更少。
*   它在层次上是非阻塞的，所以如果需要的话，嵌套的加载可以同时触发，因为我们不会通过抛出一个承诺来中断渲染，我们只是在准备好之前不会将节点附加到 DOM。没有嵌套和懒惰组件等问题..
*   Solid 的惰性道具评估允许孩子们在不知道悬念的情况下触发悬念，因为他们需要数据。你不需要重写你的组件来处理悬念。

老实说，我很想看看我还能做些什么。我们只是触及了表面。

> 特别感谢 React 团队，感谢他们不断拓展 Web UI 开发的边界。他们的表面模型(而不是内部)非常接近一个细粒度的反应库，这使得做这样的工作比我想象的更容易。

[](https://github.com/ryansolid/solid) [## 瑞安固体/固体

### 一个用于构建用户界面的声明式、高效且灵活的 JavaScript 库。—瑞安固体/固体

github.com](https://github.com/ryansolid/solid) [](/designing-solidjs-reactivity-75180a4c74b4) [## 设计固体:反应性

### 2019 年前端开发的热点按钮话题。这很大程度上归功于苗条而富有的哈里斯…

itnext.io](/designing-solidjs-reactivity-75180a4c74b4) [](https://medium.com/javascript-in-plain-english/designing-solidjs-immutability-f1e46fe9f321) [## 设计固体:不变性

### 反应式状态管理可以既是不可变的又是最高效的吗？

medium.com](https://medium.com/javascript-in-plain-english/designing-solidjs-immutability-f1e46fe9f321) [](https://medium.com/@ryansolid/designing-solidjs-jsx-50ee2b791d4c) [## 设计固体:JSX

### 为什么虚拟 DOM 产生的语法也是反应式 UI 库的最佳语法？

medium.com](https://medium.com/@ryansolid/designing-solidjs-jsx-50ee2b791d4c)