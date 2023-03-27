# 减速器组织——更进一步

> 原文：<https://itnext.io/reducer-organization-taking-a-step-further-4aba7af755d6?source=collection_archive---------4----------------------->

![](img/2c2681d33417f647384d3d06d14adc92.png)

>我现在有了一个闪亮的新博客。阅读这篇文章的最新更新[https://blog . goncharov . page/reducer-organization-taking-a-step-further](https://blog.goncharov.page/reducer-organization-taking-a-step-further)

我们将概述 Redux/NGRX 应用程序中减速器在过去两年中的发展。从普通的`switch-case`开始，到通过键从对象中选择一个缩减器，最后解决基于类的缩减器。我们不仅要讨论如何做，还要讨论为什么做。

> *如果你有兴趣在 Redux / NGRX 中处理太多的样板文件，你可能想看看这篇文章*[](https://medium.freecodecamp.org/yet-another-guide-to-reduce-boilerplate-in-your-redux-ngrx-app-3794a2dd7bf)**。**
> 
> **如果你已经熟悉了从 map 技术中选择一个缩减器，考虑直接跳到基于类的缩减器。**

# *普通开关盒*

*所以让我们来看看在服务器上异步创建一个实体的日常任务。这次我建议我们描述一下如何创造一个新的绝地。*

*老实说，我从未在生产中使用过这种减速器。我的理由有三点:*

*   *`switch-case`介绍了一些紧张点，泄漏的管道，我们可能会在某些时候忘记及时修补。如果不立即做`return`，我们可能总是忘记添加`break`，我们可能总是忘记添加`default`，我们必须添加到每个减速器。*
*   *`switch-case`有一些样板代码本身没有添加任何上下文。*
*   *`switch-case`是 O(n)[种](https://stackoverflow.com/questions/4442835/what-is-the-runtime-complexity-of-a-switch-statement)种。这本身不是一个可靠的论点，因为 Redux 无论如何都不是很有表现力，但它让我内心的完美主义者抓狂。*

*[Redux 的官方文档](https://redux.js.org/recipes/reducing-boilerplate#generating-reducers)建议采取的合乎逻辑的下一步是通过键从对象中选择一个减速器。*

# *通过关键点从对象中选择减速器*

*这个想法很简单。每个状态转换都是状态和动作的函数，并有相应的动作类型。考虑到每个动作类型是一个字符串，我们可以创建一个对象，其中每个键是一个动作类型，每个值是一个转换状态的函数(一个缩减器)。然后，当我们收到一个新的动作时，我们可以通过键从该对象中选择一个所需的缩减器，即 O(1)。*

*这里最酷的事情是`reducerJedi`内部的逻辑对于任何缩减器都保持不变，这意味着我们可以重用它。甚至还有一个名为 [redux-create-reducer](https://github.com/kolodny/redux-create-reducer) 的小库，它就是做这个的。它使代码看起来像这样:*

*很漂亮，是吧？尽管这仍然有一些警告:*

*   *如果是复杂的减速器，我们必须留下大量的注释来描述这个减速器做什么以及为什么。*
*   *巨大的减速器图很难读懂。*
*   *每个减速器只有一个对应的动作类型。如果我想让同一个减速器运行几个动作呢？*

*基于类的减速器成了我在黑夜王国里的光棚。*

# *基于类的缩减器*

*这一次，让我从这种方法的原因开始:*

*   *“类”方法将是我们的缩减器，方法有名字，这是一个有用的元信息，我们可以在 90%的情况下放弃注释。*
*   *“Class”方法可以被修饰，这是一种易于阅读的声明方式来匹配动作和 reducers。*
*   *我们仍然可以使用一个引擎盖下的行动图来获得 O(1)复杂度。*

*如果这听起来像是一个合理的理由列表，让我们开始吧！*

*首先，我想定义一下我们想要得到的结果。*

*现在，当我们看到我们想要达到的目标时，我们可以一步一步来。*

# *第一步。@动作装饰师。*

*这里我们想做的是接受任意数量的动作类型，并将它们作为元信息存储起来，供类的方法以后使用。为此，我们可以利用[反射-元数据](https://github.com/rbuckton/reflect-metadata) polyfill，它为[反射](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)对象带来了元数据功能。之后，这个装饰器会把它的参数(动作类型)作为元数据附加到一个方法上。*

# *第二步。从一个缩减器类中创建一个缩减器函数*

*正如我们所知，每个 reducer 都是一个纯粹的函数，它接受一个状态和一个动作，并返回一个新的状态。好吧，类也是一个函数，但是没有`new`ES6 类不能被调用，我们必须用一些方法从类中产生一个实际的缩减器。所以我们需要以某种方式改变它。*

*我们需要一个函数来处理我们的类，遍历每个方法，收集动作类型的元数据，构建一个 reducer 映射，并从 reducer 映射中创建一个最终的 reducer。*

*下面是我们如何检查一个类的每个方法。*

*现在，我们希望将收到的集合处理成一个 reducer 映射。*

*所以最终的函数可能是这样的。*

*我们可以像这样把它应用到我们的`ReducerJedi`类中。*

# *第三步。融合在一起。*

# *后续步骤*

*以下是我们遗漏的内容:*

*   *同一个动作对应几个方法怎么办？目前的逻辑不处理这个。*
*   *我们可以加上 [immer](https://github.com/mweststrate/immer) 吗？*
*   *如果我使用基于类的动作呢？我怎么能传递一个动作创建者，而不是一个动作类型？*

*所有这些额外的代码样本和例子都包含在 [reducer-class](https://github.com/aigoncharov/reducer-class) 中。*

*我必须说为 reducers 使用类并不是一个原创的想法。很久以前，@amcdnl 提出了令人敬畏的 [ngrx-actions](https://github.com/amcdnl/ngrx-actions) ，但是看起来他现在专注于 [NGXS](https://github.com/ngxs/store) ，更不用说我想要更严格的类型和与特定角度逻辑的解耦。[这里列出了 reducer-class 和 ngrx-actions 之间的主要区别。](https://github.com/aigoncharov/reducer-class#how-does-it-compare-to-ngrx-actions)*

> *如果你喜欢为你的 reducers 使用类的想法，你可能会喜欢为你的 action creators 做同样的事情。看看[*flux-action-class。*](https://github.com/aigoncharov/flux-action-class)*

*希望你找到了对你的项目有用的东西。请随时将您的反馈传达给我！我非常感谢任何批评和问题。*

*UPD #1:看看 reddit 上的这个讨论，听听支持和反对这种方法的其他观点*