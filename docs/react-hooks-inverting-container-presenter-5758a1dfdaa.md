# 反应挂钩:反转容器/演示器

> 原文：<https://itnext.io/react-hooks-inverting-container-presenter-5758a1dfdaa?source=collection_archive---------0----------------------->

![](img/5b04da3712ce6822b174dabaca70e01e.png)

照片由[夏洛特·康尼比尔](https://unsplash.com/@she_sees?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

## 或者:我是如何学会停止担忧并热爱结束的

反应钩子已经完全从基于职业的原始软泥中爆发出来。

比我酷的人已经用了几十年了。谁还会使用类呢？你还在用类？？恶心！

在他们到达后不久，我开始听到人们说“钩子取代了容器/展示者模式。”丹·阿布拉莫夫本人更新了他那篇著名的 [Medium 文章，文章以一个警告开始:](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)

> *我不建议再这样拆分你的组件了。* ***如果你觉得在你的代码库中很自然，这种模式就可以得心应手。但我已经多次看到它在没有任何必要的情况下，以近乎教条的热情被强制执行。***

我理解他关于“教条式热情”的观点，但不幸的是，他怀疑 React 中分离关注点的强大心理模型。

我的第一个问题是“钩子将如何取代容器/展示者？”好吧，事实上，我看到了。您不再需要容器*组件*，因为有状态逻辑可以包含在自定义*钩子*中。可移植的状态，这实际上很好。

HOCs？渲染道具？随着钩子应用的增加，这些可能会被淘汰。钩子给了你渲染道具的灵活性，而不需要在一个额外的组件中包装你的组件。

当然，第一次接触 React 的开发人员将不得不考虑`useEffect`和依赖数组的微妙之处，但是他们不需要记住一长串的生命周期方法。还记得你初来乍到时的反应吗？似乎每次你想出一个生命周期方法，就会有另外两个像九头蛇一样出现在它的位置上。更不用说他们的道具——“嗯，是`prevProps/prevState`还是`nextProps/nextState`？我想我会再次打开旧的开发工具。"

不管怎样，我说到哪了…容器/呈现器真正的好处是它鼓励你将关注点分开。您的容器处理状态/数据，而您的呈现者——漂亮而纯粹——持有 UI，两者之间有一条清晰的分界线。他们的关系很清楚:容器…包含…呈现者。它在 React 架构模型中创建了一个自然的层次结构。

# 颠倒过来

给我留下深刻印象的是，这种关系现在颠倒了。之前只负责 UI 的组件现在… *包含了容器*。

似乎没有一个明确的方法可以将这两者分开。这让我开始思考。

您可以拥有一个从有状态功能组件呈现的无状态表示组件:

```
// Hook
const useName = () => {
  const [name, setName] = useState('DeeDee');
  const onJoey = () => setName('Joey');
  return { name, onJoey };
}// Presenter
const GabbaGreeting = ({ name, onJoey }) => (
  <div>
    Gabba gabba hey, {name}
    <button onClick={onJoey}>Click to Joey-ify</button>
  </div>
);// Ugh.
export default const NameContainer = (props) => {
  const { name, onJoey } = useName(); return <GabbaGreeting name={name} onJoey={onJoey} />
}
```

这工作，但是`NameContainer`感觉像一个不必要的层。

# 然后就是测试。

表示组件测试起来非常简单明了——纯功能、道具输入、UI 输出。它们非常适合 TDD，尤其是对于那些刚刚开始测试优先开发的开发人员。以这种方式编写组件确实能让你明白如何(以及为什么)将你的架构分成不同的层。既然表示性组件/元素现在从组件内部获得了属性/属性，那么如何测试现在有状态的功能组件的纯粹表示性部分呢？

我想到的一个方法是 DI，在你的组件中注入一个定制的钩子，但是在尝试了一点之后，感觉很奇怪。另一个可能的解决方案是允许在你的定制钩子中设置初始值，但是这会变得复杂…

```
const useForm = ({
  initialValues,
  initialErrors,
  initialIsDirty,
  initialIsLoading,
  etc...
}) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState(initialErrors);
  const [isDirty, setIsDirty] = useState(initialIsDirty);
  const [isLoading, setIsLoading] = useState(initialIsLoading);
  etc...
}
```

# 一个新的挑战者出现了

经过一番思考后，我想到了一个解决方案，感觉它有解耦、测试和简洁的潜力。我写了一个小库， [react-hooks-compose](https://www.npmjs.com/package/react-hooks-compose) ，它可以让你将钩子连接到表示组件。

```
composeHooks({ useMyHook })(Presenter);
```

它有一个类似于`react-redux`中的`connect`的 API，将钩子的返回值映射到道具。这是我们第一个例子的样子:

```
// You still have your Hook
const useName = () => {
  const [name, setName] = useState('DeeDee');
  const onJoey = () => setName('Joey');
  return { name, onJoey };
}// You still have your Presenter
const GabbaGreeting = ({ name, onJoey }) => (
  <div>
    Gabba gabba hey, {name}
    <button onClick={onJoey}>Click to Joey-ify</button>
  </div>
);// What have we here? Say, that's pretty easy on the eyes!
export default composeHooks({ useName })(GabbaGreeting);
```

它涵盖了所有 3 个痛点:它保持了事物的解耦性，使测试变得简单明了，并消除了讨厌的额外容器组件。

# 选择你自己的冒险

我用钩子越多，我就越…上瘾(唉)。挂钩的好处远远大于缺点，我们正处于一个有趣的时期，在这个时期我们可以进行实验，找到新的做事方法。最佳做法正在出现；我们应该让它们出现，而不是强加它们。如果我们继续尝试，继续玩，新的模式就会出现。

老实说:我无视了不要完成我的项目并将它们转化为钩子的建议，并在这个过程中学到了很多。我喜欢“一切都只是函数”的一致性闭包在，`this`在外。相对于钩子和渲染道具，我更喜欢钩子的人体工程学。

也许我想多了，也许这不是问题。也许有人会想出一个更好的主意。或者其他人也感受到了同样的痛苦，而[反应-挂钩-撰写](https://www.npmjs.com/package/react-hooks-compose)正是他们一直在寻找的解决方案！我所知道的是，现在我只是感到一点分离(担忧)的焦虑。