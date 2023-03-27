# 在 react 中处理大型列表和表格

> 原文：<https://itnext.io/handling-large-lists-and-tables-in-react-238397854625?source=collection_archive---------0----------------------->

![](img/69f99d6c9836318987827f4776feee04.png)

想象一种常见的情况:您有一个包含大量数据的表，或者一个包含许多项的列表。如果您在 react 中已经遇到过这种情况，您应该知道组件在屏幕上实际呈现之前会有一段等待时间，然后它会立即呈现。

```
class ParentComponent extends React.Component {
  render() {
    return <div>
      ...
      {this.props.items.map( item =>
        <ChildComponent {...item}/>
      )}
    </div>
  }
} 
```

一旦 ParentComponent 接收到 props.items，它首先将其全部添加到虚拟 DOM，然后检查真实 DOM 的哪一部分需要协调，然后一次性更新它。您看到的延迟是 react 在进行协调之前处理大量项目所需的时间。

我们想要的是能够显示已经加载到虚拟 DOM 中的项目，同时等待其他项目加载。要做到这一点，我们需要尽快呈现我们的 ParentComponent(让它在屏幕上可见)，然后逐步添加项目。让我们扩展我们的 ParentComponent:

```
constructor(props) {
  super(props); this.state = {
    items: this.props.items.slice(0, 1)
  }
}
...
render() {
  return <div>
    ...
    {this.state.items.map( item =>
      <ChildComponent {...item}/>
    )}
  </div>
}
```

我们添加了一个构造函数，并在其中定义了 state.items，这是我们刚刚更新的 render()方法中实际映射的内容。我们确保 state.items 是一个只有一个元素的数组。现在我们需要一种方法来逐个更新 state.items 和添加其他项。通过使用这种方法，render 方法将更频繁地触发(props.items.length times ),但它将立即显示所有加载的项目，这正是我们想要的。

```
recursive = () => {
  setTimeout(() => {
    let hasMore = this.state.items.length + 1 < this.props.items.length;
    this.setState( (prev, props) => ({
      items: props.items.slice(0, prev.items.length + 1)
    }));
    if (hasMore) this.recursive();
  }, 0);
}
```

编辑:如果您担心在递归函数中调用 setState，即调用太多次，请注意:

1.  调用 setState 是更新视图的正确方法
2.  根据您的需要，您可以一次加载多个项目。因此，如果您有一个包含 4 个项目的网格，您可以一次加载 4 个项目。
3.  setState 可以将几个调用批处理在一起，从而提高效率。
4.  setTimeout 将执行放在堆栈的末尾，因此如果用户与已经呈现的组件交互，他的交互将优先于呈现其余的项目。这提高了 UX，因为用户可以开始工作，而不必等待一切都出现。
5.  做性能测试，看看它是否减慢或加快你的应用程序
6.  这是有用的，如果你已经有所有的项目在前面。如果你正在抓取，你需要使用不同的策略
7.  提取成分就不会有副作用。做这件事的组件在它的状态下不应该做任何事情。我把它做成一个临时的。

这是一个递归函数，它会一直调用自己，直到长度相同。每次迭代将向 state.items 数组添加一个元素。我们添加了 setTimeout 将其放在堆栈的末尾，这样我们只在渲染完成后更新状态，从而提供更好的用户体验。我们只需要调用这个函数:

```
componentDidMount() {
   this.recursive();
}
```

瞧啊。

编辑:也有可能在 render 方法中分割 props，在递归中只更新一个 state.counter，但是我没有得到性能上的差别。

编辑 2:这段代码已经过测试，证明对于包含 10 到 1500 个 react 元素的列表和集合非常有用，挂载时间很长(2 到 3 毫秒)。您也可以将 react-virtualized 作为一种外部解决方案