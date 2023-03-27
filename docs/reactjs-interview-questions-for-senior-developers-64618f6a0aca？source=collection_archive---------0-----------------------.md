# ReactJS 针对高级开发人员的面试问题

> 原文：<https://itnext.io/reactjs-interview-questions-for-senior-developers-64618f6a0aca?source=collection_archive---------0----------------------->

![](img/f02eb73425c22d3519e00a4c4f0503aa.png)

https://unsplash.com/photos/wawEfYdpkag

如果一个房间里有几个已经使用 React 年以上的开发人员，他们很可能会问你类似“道具和状态的区别”这样的问题。

相反，人们希望看到你对常见的 ReactJS 模式、众所周知的陷阱、如何重构和测试你的组件等等的了解。

你可以和 React 一起工作多年，实际上对一些不太实际的问题没有意见，这很好。然而，如果你有一个面试，那么有一个观点是非常重要的。为什么？嗯，首先，能够回答面试官的问题感觉很好，也表明了你对这个话题的兴趣。

# Q1。为什么我们既有受控输入又有不受控输入？

## 面试官想要看到的:对基本反应概念的深刻理解。

好吧，这更像是一个热身问题。更重要的是给出一个完整的答案。

**一个受控输入**接受它的当前值作为一个属性，以及一个改变该值的回调。这是一种“反应方式”:

```
<input type="text" value={value} onChange={this.handleChange} />
```

不受控制的输入使用 DOM API 在内部存储自己的状态。

这里不提供`value`和`onChange`句柄，但是我们使用 [ref](https://reactjs.org/docs/refs-and-the-dom.html) ():

```
<input type="text" ref={this.textInput} />
```

现在，您可以通过以下方式访问输入数据:

```
this.textInput.current.value
```

你的面试官想听到更多细节:使用不受控制的组件有什么好处吗，有什么性能差异吗？

就我个人而言，我从未使用过不受控制的输入，但是如果你正在学习 React 或者你必须集成 React 和 non React 代码，那么这可能是必要的。

值得一提的是，您的数据(状态)和 UI(输入)总是与受控输入方法同步，这意味着您必须更新组件的状态，这将触发 [React 协调过程](https://reactjs.org/docs/reconciliation.html)。

而对于不受控制的元素就不需要这样了——只需将值保存在输入 DOM 元素中。

# Q2。为什么我们需要一个密钥属性？给出一个坏键导致错误的例子。

## 面试官想看的:关于 React 内部运作的一些见解。

答案基于[官方 React 对账流程文件](https://reactjs.org/docs/reconciliation.html)。

存在具有 O(n)时间复杂度的经典差分算法，其可用于创建反应元素树。但这意味着显示 1000 个元素需要**十亿次**比较。

相反，React 实现了一个启发式的 O(n)算法，并假设开发人员可以用一个`key`道具提示哪些子元素在不同的渲染中可能是稳定的。

坏钥匙怎么办？嗯，如果你决定让你的孩子可移动，索引可能是一个非常糟糕的键。[看看这个演示](https://codesandbox.io/s/react-example-qfp5s)。尝试在第二个输入中键入一些内容，然后删除第一个。但是你仍然可以看到第二个的价值，为什么呢？

因为你的钥匙不稳定。删除后，您的第三个孩子的键等于 3，现在的键等于 2。这和现在反应不是同一个元素。它会将其与错误的 DOM 元素进行匹配，该元素之前有一个等于 2 的键(它保留了我们在第二个输入中键入的值)。

# Q3。React 事件系统与 DOM 事件有何不同。使用 stopImmediatePropagation()。

## 面试官想看的:不同 JavaScript 事件模型的一般知识和日常实践经验。

回答这个问题的最好方法是先简要解释一下像冒泡和捕获这样的 DOM 事件概念是如何工作的。

简而言之，使用冒泡，事件首先被最内部的元素捕获和处理，然后传播到外部元素。

通过捕获，事件首先被最外面的元素捕获，然后传播到内部元素。

查看这篇文章，了解更多信息和演示。

在那之后，最好提一下 React 使用了**事件委托模式**——而不是给它们每个都分配一个处理程序——我们在它们的共同祖先上放了一个处理程序，可以用`event.target`访问那个元素。

然后很明显地解释了`stopImmediatePropagation`与传统`stopPropagation`方法的不同之处:由于事件委托模式，它防止了同一个事件的其他侦听器被调用。

# Q4。如何防止组件重渲染？

## 面试官想看到的:更多关于反应的知识，你关心表现。

这是最常被问到的问题之一。值得一提的是:

1.  **shouldcomponentdupdate()**—默认情况下返回“真”。如果您知道哪些属性必须触发更新，您可以覆盖。
2.  **PureComponents** —它们之间的区别在于`[React.Component](https://reactjs.org/docs/react-api.html#reactcomponent)`没有实现`[shouldComponentUpdat](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)e`方法，而`React.PureComponent`用一个浅属性和状态比较实现了它。
3.  **React.memo** —与前一个相同，但它与功能组件一起工作。

作为后续:什么时候使用 Component 而不是 PureComponent？

在 modern React 中，99%的情况下使用 PureComponents。但是，如果使用 Redux 选择器，通常需要显式指定传入的属性更改，以取消即将发生的重新渲染，从而防止 UI 抖动。在这种情况下，使用组件是合适的。

如果你对这篇[深度阅读](http://jamesknelson.com/should-i-use-shouldcomponentupdate/)有些模糊，那就来看看吧。

# Q5。举一个 HOC 的例子。

## 面试官想看到的:你熟悉一个众所周知的反应模式。

*HOC* —高阶分量是接受一个分量并返回一个新分量的函数。这是一种在 React 组件之间共享代码的技术。

假设我们有一个简单的按钮组件:

```
const Button = props => <button {...props}>Hello</button>;
```

我们想为按钮组件创建一个红色边框的特设。让我们使用我们的定义来创建它:

```
const withRedBorder = Component =>
  (props) => (
    <Component {...props} style={{ border: "1px solid red" }} />
  );
```

现在我们可以创建一个 RedButton 组件:

```
const RedButton = withRedBorder(Button);// and use it as:
<RedButton />
```

使用特设模式的库: [Redux connect](https://react-redux.js.org/api/connect) ，[compose](https://github.com/acdlite/recompose)。

# Q6。什么是渲染道具？

## 面试官想看到的:你熟悉第二种众所周知的反应模式。

*渲染道具* —当一个组件接受一个返回 React 元素的函数并调用它，而不是实现它自己的渲染逻辑。

这是 React 组件之间共享代码的另一种技术。

一个例子:

```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

使用渲染道具模式的库:[反应路由器](https://reacttraining.com/react-router/web/api/Route/render-func)，[降档](https://github.com/paypal/downshift)。

**一个后续**:那 HOC vs 渲染道具呢。哪款比较好，什么时候用？这是一个颇有争议的话题，在这篇[文章](https://www.richardkotze.com/coding/hoc-vs-render-props-react)中你可以找到一些赞成和反对的意见。

# **Q7。对组件的单元测试和集成测试进行反应。**

## 面试官想看什么:了解测试组件的不同方法。

值得一提的是[酶](https://airbnb.io/enzyme/)和[反应测试库](https://github.com/testing-library/react-testing-library)。

**React 测试库**提供了一个干净简单的 API，专注于“像用户一样”测试应用程序。这意味着 API 返回 HTML 元素，而不是用 Enzyme 中的浅层渲染来反应组件。这是一个编写集成测试的好工具。

**Enzyme** 仍然是一个有效的工具，它提供了一个更复杂的 API，可以让你访问组件的属性和内部状态。为组件创建**单元测试**是有意义的。

看看这个关于 react-testing-library 的简洁视频。

# **Q8。你最喜欢的钩子是什么，如何实现？**

## 面试官想看的:钩子的实际用法和对其工作原理的理解。

尽管钩子仍然是新的 API，但是很多人已经在生产中使用它了，他们希望你了解它。

让我们重新创建一个`useWindowSize`——它是一个易读且非常简单的钩子:

```
import { useState, useEffect } from 'react';const useWindowSize = () => {
  const getSize = () => ({
    width: window.innerWidth,
    height: window.innerHeight
  }); const [size, setSize] = useState(getSize); useEffect(() => {
    const handleResize = () => setSize(getSize());
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []); return size;
}
```

用法:

```
const App = () => {
  const size = useWindowSize(); return (
    <div>
      {size.width}px / {size.height}px
    </div>
  );
}
```

您可能会想到一些问题，如:

*1。有必要调用你的函数* `*useWindowSize*` *吗，直接调用 getWindowSize 怎么样？*

是的，如果没有它，我们将无法自动检查钩子的[规则的违反情况，因为我们无法判断某个函数内部是否包含对钩子的调用。](https://reactjs.org/docs/hooks-rules.html)

*2。如果我们把* `*useEffect*` *中的* `*[]*` *论点去掉，会管用吗？*

是的，但是我们会在每次渲染时调用`useEffect`钩子，这可能会导致性能问题。

*3。如果我们在* `*useWindowSize*` *中处理窗口大小调整，React 如何知道何时重新渲染* `*App*` *组件？*

当你在自定义钩子内部调用`setSize`时，React 知道这个钩子用在了`App`组件中，并将重新渲染它。

有一些反作用钩子的魔法。看看[为什么 React 挂钩依赖于调用顺序？](https://overreacted.io/why-do-hooks-rely-on-call-order/)文章了解更多信息。

*4。如何让这个钩子为服务器端渲染做好准备？*

大概是这样的:

```
import { useState, useEffect } from 'react';const useWindowSize = () => {
  const isClient = typeof window === 'object';  
  const getSize = () => ({
    width: isClient ? window.innerWidth : undefined,
    height: isClient ? window.innerHeight : undefined
  }); const [size, setSize] = useState(getSize); useEffect(() => {
    const handleResize = () => setSize(getSize());
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []); return size;
}
```

# Q9。上下文 API。重新渲染了多少？

## 面试官想看的:可以说是 React 中最重要的东西——如何管理数据流。

首先，值得一提的是，与 Redux 不同，上下文**没有自己的状态，**相反，它只是一个管道，通常从另一个组件的状态读取数据。

此外，有了上下文 API，即使你使用 PureComponent 或 React.memo ，也可以很容易地重新渲染比你需要的多得多的内容。

在下面的例子中，我们每次点击按钮都会在控制台中看到一条新消息:

```
import React, { useState, useContext } from "react";
import ReactDOM from "react-dom";const ProfileContext = React.createContext();const App = () => {
  const [val, setIncrement] = useState(0);
  return (
    <div>
      <Profile userInfo={val} />
      <button onClick={() => setIncrement(val + 1)}>Click</button>
      <p>value: {val}</p>
    </div>
  );
};class Profile extends React.PureComponent {
  state = {
    name: "Alex"
  }; setName(name) {
    this.setState({ name });
  } render() {
    const { name } = this.state;
    const { setName } = this; return (
      <ProfileContext.Provider
        value={{
          name,
          setName
        }}
      >
        <Logger />
      </ProfileContext.Provider>
    );
  }
}const Logger = React.memo(() => {
  const { name } = useContext(ProfileContext);
  console.log("Logger rerendering");
  return null;
});ReactDOM.render(<App />, document.querySelector("#root"));
```

在上面的例子中，我们的`**Logger**` **组件是一个纯组件，但是我们仍然每次都重新渲染**。这是因为我们使用了上下文。尝试注释掉 Logger 组件中的第一行，现在它不会重新呈现。

这是因为上下文使用引用标识来确定何时重新呈现。让我们改变我们的`Profile`组件来使它工作:

```
class Profile extends React.PureComponent {
  state = {
    name: "Alex",
    setName(name) {
      this.setState({ name });
    }
  }; render() {
    return (
      <ProfileContext.Provider value={this.state}>
        <Logger />
      </ProfileContext.Provider>
    );
  }
}
```

现在它像预期的那样工作了。顺便说一句，如果你知道如何让它和钩子一起工作——把它贴在评论区吧！

# Q10。从类到功能组件的迁移。

## 面试官想看到的:你知道如何使用钩子重构 ReactJS，并且你可以展示出来。

使用功能组件和挂钩，您可以覆盖所有类组件用例，例如:

1.  带有 useState 挂钩的类组件状态
2.  带有 useEffect 挂钩的类组件生命周期方法
3.  额外的好处:更好的抽象和定制挂钩

让我们看看下面的例子，并使用钩子重构它:

```
class App extends React.Component {
  state = {
    value: localStorage.getItem("info") || ""
  }; componentDidUpdate() {
    localStorage.setItem("info", this.state.value);
  } onChange = event => {
    this.setState({ value: event.target.value });
  }; render() {
    const { value } = this.state;
    return (
      <div>
        <input value={value} type="text" onChange={this.onChange} />
        <p>{value}</p>
      </div>
    );
  }
}
```

由于[类字段声明](http://class fields)和[类箭头函数](https://javascriptweblog.wordpress.com/2015/11/02/of-classes-and-arrow-functions-a-cautionary-tale/)，这已经很紧凑了。

替换`setState`和`componentDidUpdate`的第一次重构可能是这样的:

```
const App = () => {
  const val = localStorage.getItem("info") || "";
  const [value, setValue] = useState(val);
  const onChange = event => setValue(event.target.value); useEffect(() => localStorage.setItem("info", value), [value]); return (
    <div>
      <input value={value} type="text" onChange={onChange} />
      <p>{value}</p>
    </div>
  );
};
```

只有 13 行代码，而不是具有完全相同功能的 19 个类。但是我们可以更进一步:我们可以很容易地在这里提取和重用一个自定义钩子:

```
const usePersistentStorage = key => {
  const val = localStorage.getItem(key) || "";
  const [value, setValue] = useState(val); useEffect(() => localStorage.setItem(key, value), [key, value]); return [value, setValue];
};
```

现在我们可以在我们的`App`中使用这个:

```
const App = () => {
  const [value, setValue] = usePersistentStorage("info");
  const onChange = event => setValue(event.target.value); return (
    <div>
      <input value={value} type="text" onChange={onChange} />
      <p>{value}</p>
    </div>
  );
};
```

只有 10 行代码，比以前缩短了两倍。在真正的应用程序中，您可以提取更多信息。

# 在你搞定它之前。

请记住，给出一个清晰而宽泛的答案是至关重要的。

例如，人们不会要求你在白板上写下 react 组件的单元测试，但是缺乏对不同 react 组件测试方法的解释会导致更多的问题。

而且，感觉越来越多的采访集中在钩子上。使用[这个网站](https://usehooks.com)来练习。

# 就是这样！

如果你有任何问题或反馈，请在下面的评论中告诉我。

## 如果这有用，请点击拍手👏下面扣几下，以示支持！⬇⬇ 🙏🏼