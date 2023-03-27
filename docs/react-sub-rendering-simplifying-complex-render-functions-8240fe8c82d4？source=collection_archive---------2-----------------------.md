# 反应子渲染:简化复杂的渲染功能

> 原文：<https://itnext.io/react-sub-rendering-simplifying-complex-render-functions-8240fe8c82d4?source=collection_archive---------2----------------------->

![](img/4446862539089b0b586f9dd66fc2b355.png)

你有没有构建过一个带有渲染函数的 React 组件，这个渲染函数有一大堆杂乱的三元语句，用来根据你的组件状态决定渲染什么？别担心，这种事我们谁都会遇到的。

在这篇文章中，我将教你一个简单的技巧，你可以在有状态 React 组件中使用它来清理复杂的渲染逻辑。我称这种技术为“子渲染”。它来源于将大的复杂函数分成更小的简单子函数的过程。就像你把函数分成子函数一样，你也可以把渲染函数分成子渲染方法。

# 你想什么时候做？

假设我们有一个 React 组件，它使用外部 API 来获取结果列表。该组件在启动 API 调用之前，将在装载状态下启动。当被调用时，它应该返回一个带有结果的响应。但是这个 API 非常不稳定，所以如果出现错误，您还需要显示一些内容来处理错误。也有可能没有结果。在这种情况下，响应将是一个空数组。

该组件将有三种状态。这些状态是`isLoading`、`results`和`hasError`。该组件还将利用其他地方定义的四个子组件(超出了本文的范围)。这些子组件是`<Loading />`、`<Results />`、`<NoResults />`和`<Error />`。除了`<Results />`组件，这些子组件都不需要任何道具。这只需要一个`results` prop，它将是存储在主组件的`results`状态中的响应有效负载。

希望这有意义。让我们来谈谈不同的状态，以及它们应该如何影响最终呈现的内容。

## 正在加载

`isLoading`是一个布尔值，我们将其初始化为`true`，因为当组件第一次挂载时，我们还没有获取结果。因此，在结果返回(满或空)或 API 调用出错之前，它一直保持为真。当这为真时，我们将渲染`<Loading />`组件。

## 结果

假设 API 调用没有任何问题，我们应该会收到一个结果列表。因为它是一个列表，我们将它初始化为一个空数组(`[]`)。我们这样做是因为我们希望 API 响应总是一个数组。在没有结果的情况下，我们的响应负载中仍然会有一个空数组。假设端点不再处于`isLoading: true`状态，我们将在`<Results />`组件中呈现结果。如果响应负载是一个空数组，那么我们将转而呈现`<NoResults />`组件。

## 哈泽罗

我们有`hasError`布尔值，以防 API 调用过程中出现错误。这在组件状态下被初始化为`false`，只有在 API 调用失败时才被设置为 true。如果`hasError`为真，我们将渲染`<Error />`组件。

# 运行中的渲染功能

好了，关于我们要做什么，我们已经说得够多了。现在让我们开始行动吧！

![](img/98c9954811517cae2c548bc9c76833b6.png)

在这个例子中，我将省略一些代码，如处理组件状态更改的导入和 API 逻辑。因为这里的重点是解释渲染逻辑。

实现此呈现逻辑的最常见方式是使用嵌套的三元语句来实现，如下所示:

```
class MyComponent extends Component {
  state = {
    hasError: false,
    isLoading: true,
    results: [],
  }   componentDidMount() {
    // implement API call logic here
  } render() {
    const { hasError, isLoading, results } = this.state; return (
      <Container>
        {isLoading ? <Loading /> :
          <>
            {hasError ? <Error /> :
              <>
                {results.length === 0 ? <NoResults /> :
                  <Results results={results} />
                }
              </>
            }
          </>
        }
      </Container>
    );
  }
}
```

首先，如果你不熟悉`<>...</>`语法。这是 React 片段的速记语法，你可以在我写在这里的博客文章[中读到关于它们的所有内容。](https://www.barrymichaeldoyle.com/fragment)我还偷偷加入了一个`<Container>`组件，作为一个样式包装器，让我们的组件稍微复杂一点，以便演示。

我不知道你怎么想，但我发现这种逻辑乍一看很难理解。当你在写它的时候，它是非常有意义的，但是两天后回顾它，你可能会质疑你的理智。如果你不明白，我把算法写成一个小片段，帮助你掌握。

```
if (isLoading) {
  return <Loading />;
} if (hasError) {
  return <Error />;
} if (results.length === 0) {
  return <NoResults />;
}
return <Results results={results} />;
```

现在记住，当编译器遇到`return`语句时，它会离开函数。这就是我不写`else if`声明的原因。这实际上是一个惊人的解决方案，我们可以用它来代替上面使用的复杂的嵌套三元逻辑。

**但是我们上面嵌套的三元逻辑被包裹在一个** `**<Container />**` **组件中所以我们不能这样做。现在我们该怎么办？**

这里有两种可能的选择。

## 选项 1:将所有的逻辑从包装器移到它自己的组件中

将包装器中的逻辑移到它自己的组件中是一项非常简单的任务。唯一令人恼火的是，您必须将所有的状态属性传递给组件。但是即使这样做也不需要太多的努力，这要感谢我们奇妙的 ES6 能力，它允许我们散布道具。最后，这样做将为您留下以下简单的渲染函数:

```
render() {
  return (
    <Container>
      <ComponentRenderer {...this.state} />
    </Container>
  );
}
```

然后，您会将您的`<ComponentRenderer>`无状态功能组件写成这样:

```
const ComponentRenderer = ({ hasError, isLoading, results }) => {
  if (isLoading) {
    return <Loading />;
  } if (hasError) {
    return <Error />;
  } if (results.length === 0) {
    return <NoResults />;
  }
  return <Results results={results} />;
}
```

这是一个非常好的解决方案。我推荐它而不是我们最初的解决方案，因为从程序员的角度来看，它更容易逻辑阅读。它将渲染内容的逻辑移出了 JSX，这肯定比将所有内容都放在 JSX 更好。

但是还有另一种解决方案，它不需要创建一个全新的组件。

## 选项 2:实现子渲染

在我开始谈论选项 1 切线之前，我确实是从子渲染开始这篇文章的。所以让我们回到正题！我将首先向您展示完整的示例，因为它几乎不言自明:

```
class MyComponent extends Component {
  state = {
    hasError: false,
    isLoading: true,
    results: [],
  } componentDidMount() {
    // implement API call logic here
  } renderResults() {
    const { hasError, isLoading, results } = this.state; if (isLoading) return <Loading />;
    if (hasError) return <Error />;
    if (results.length < 1) return <NoResults />;
    return <Results results={results} />;
  } render() {
    return (
      <Container>
        {this.renderResults()}
      </Container>
    )
  }
}
```

告诉过你这是不言自明的。如果你理解你的 JavaScript 和 React 基础，那么这应该是有意义的。

为了好玩，我把`results.length === 0`改成了`results.length < 1`。本质上，它们做的是同样的事情，但是我确信做一个比做另一个有一些可笑的微小的性能提升。见鬼，现在我受到启发去研究并写下它。好吧，我又忘乎所以了。

这个解决方案的优点是——就像选项 1 一样——它非常干净。您不需要创建一个全新的组件。最好的部分，你不需要传递任何参数！`renderResults`事件处理程序已经可以通过`this`访问组件的`state`。如果你读过我关于[绑定这个](https://www.barrymichaeldoyle.com/bind-this)的帖子，这是一个不需要绑定到`this`的事件处理程序的很好的例子。

## 选项 3:在每个单独的子组件中处理渲染逻辑

大声喊出来，Julius Koronci 对我的原始帖子发表了评论，指出了这个技巧。谢谢你帮我扩展这篇文章！

简化渲染逻辑的另一种方法是将该逻辑移到需要渲染的上下文中。像这样:

```
class MyComponent extends Component {
  state = {
    hasError: false,
    isLoading: true,
    results: [],
  }

  componentDidMount() {
    // implement API call logic here
  }render() {
    const { hasError, isLoading, results } = this.state;return (
      <Container>
        <Loading isLoading={isLoading} />
        <Error hasError={hasError} />
        <Results {...this.state} />
      </Container>
    )
  }
}
```

在这种情况下，我们“渲染”每个子组件。不完全是。我们现在做的是将渲染逻辑移到子组件级别。例如，通过向子组件发送`isLoading`道具来呈现`<Loading>`子组件。在`<Loading>`子组件中，我们将使用`isLoading`道具来决定渲染什么。如果`isLoading`为`true`，返回加载组件内容。如果`isLoading`为`false`，则返回`null`，即不渲染任何内容。

该加载组件将被实现为无状态功能组件，如下所示:

```
const Loading = ({ isLoading }) => {
  if (isLoading) {
    return <LoadingContent />;
  }return null;
}
```

现在，是的，你可以把它变成一个三进制语句，让它适合一两行。但是重要的是保持代码的可读性，以便为您自己和以后遇到这个函数的其他开发人员节省时间。

关于这一点的另一个注意是关于如何实现`<Results />`组件有一些变化。我必须将所有状态值传递给它，因为它仍然需要检查是否基于`isLoading`和`hasError`状态进行渲染。如果`isLoading`或`hasError`为真，则返回`null`。

然后，您仍然需要将逻辑向下移动到该组件中，以便不呈现任何结果。这最终把我们带回到最初的问题上。或者你可以重复自己的做法，在主组件中留下`<NoResults />`，也根据`isLoading`、`hasError`和`results`的长度来渲染。

尽管如此，这仍然是清理主要组件的可行解决方案。但在我看来。与其说这是一个*收拾你的烂摊子*的解决方案，不如说是一个*移动你的烂摊子*的解决方案。这可能会影响组件的可重用性，基本上是选项 1 的解决方案的进一步分裂。

# 结论

根据这篇文章的标题，你可能已经猜到我会选择选项 2 中讨论的子渲染方法。我更喜欢它，因为它让事情变得简单，所有内容都在一个地方易于阅读。呈现逻辑的范围是主要组件的状态。因为状态决定了应该呈现什么，所以将呈现逻辑保持在同一级别是有意义的。我非常喜欢将组件拆分成更小的组件。但前提是它能让事情看起来更容易理解。

关于 React 组件中的子渲染，我要说的就是这些。一旦你在实践中看到它，它是相当自明的。我希望你从这篇文章中学到了一些新东西。如果你喜欢它，那么一定要喜欢它，为它鼓掌并分享它！

这篇文章的灵感来自于[Code Complete:A Practical Handbook of Software Construction，第二版](https://amzn.to/2LxtXDo)。这本书总是激励我变得更好。变得更好是很棒的，所以如果你还没有变得更好，那就去看看吧！

你可以在[脸书](https://www.facebook.com/barrymichaeldoyle)、[推特](https://twitter.com/barrymdoyle)、[媒体](https://medium.com/@barrymdoyle)、 [YouTube](https://www.youtube.com/barrymichaeldoyle?sub_confirmation=1) 上关注我，或者甚至在 [LinkedIn](https://www.linkedin.com/in/barry-michael-doyle-11369683/) 上与我联系，以保持对我发布的新内容的更新。

最后，在下面的评论中让我知道你在处理渲染逻辑时更喜欢做什么。我还没有真正弄清楚在这里做事情的最佳实践方式是什么。但我确实相信，无论你选择哪条路，都应该是一致的！

*原载于 2018 年 9 月 6 日*[*www.barrymichaeldoyle.com*](https://www.barrymichaeldoyle.com/sub-rendering/)*。*