# React 优化技巧

> 原文：<https://itnext.io/react-optimization-tips-224c66b4b30d?source=collection_archive---------2----------------------->

![](img/8cf9e93c184563fc9b7e4963f0d60faa.png)

在 React 中，当组件的状态或道具改变时，它会重新渲染。虽然 state 表示内部数据，props 表示外部数据是有意义的，但情况并非总是如此。有一些技术可以防止组件不必要的渲染。让我们看一下例子。

# 总是重新渲染

```
let count = 0;class Button extends React.Component {
  render = () => {
    const { color } = this.props
    count++;
    return (
      <React.Fragment>
        <span> Render count: { count } </span>
        <button style={{ color }}> text </button>
      </React.Fragment>
    )
  }
}class App extends React.Component {
  state = {
    color: 'red'
  } showRedButton = () => {
    this.setState({
      color: 'red'
    })
  } showGreenButton = () => {
    this.setState({
      color: 'green'
    })
  } render() {
    const { color } = this.state;
    const { showRedButton, showGreenButton } = this;
    return (
      <React.Fragment>
        <Button color={color}/>
        <button onClick={showRedButton}> red </button>
        <button onClick={showGreenButton}> green </button>
     </React.Fragment>)
  }
}ReactDOM.render(<App/>, document.getElementById('root'))
```

[在线代码示例](https://codepen.io/n0rush/pen/MZwwZr)

每次我们点击按钮(或者是`red`或者是`green`),`count`增加，表明`Button`组件正在重新渲染。然而，`Button`唯一依赖的道具就是`color`。如果`color`保持不变，渲染是不必要的。

目标:如果我们一直按下`red`按钮，我们希望`count`保持不变，因为我们不想重新渲染`Button`组件，因为`color`保持不变。

# 技巧 1:shouldcomponentdupdate

```
let count = 0;class Button extends React.Component {
  shouldComponentUpdate(nextProps) {
    if(this.props.color === nextProps.color) {
      return false;
    }
    return true
  } render = () => {
    const { color } = this.props
    count++;
    return (
      <React.Fragment>
        <span> Render count: { count } </span>
        <button style={{ color }}> text </button>
      </React.Fragment>
    )
  }
}class App extends React.Component {
  state = {
    color: 'red'
  } showRedButton = () => {
    this.setState({
      color: 'red'
    })
  } showGreenButton = () => {
    this.setState({
      color: 'green'
    })
  } render() {
    const { color } = this.state;
    const { showRedButton, showGreenButton } = this;
    return (
      <React.Fragment>
        <Button color={color}/>
        <button onClick={showRedButton}> red </button>
        <button onClick={showGreenButton}> green </button>
     </React.Fragment>)
  }
}ReactDOM.render(<App/>, document.getElementById('root'))
```

[ShouldComponentUpdate 代码示例](https://codepen.io/n0rush/pen/KbpdGr)

# 技巧二:反应。纯组件

# 它是如何工作的

```
let count = 0;class Button extends React.PureComponent {
  render = () => {
    const { color } = this.props
    count++;
    return (
      <React.Fragment>
        <span> Render count: { count } </span>
        <button style={{ color }}> text </button>
      </React.Fragment>
    )
  }
}class App extends React.Component {
  state = {
    color: 'red'
  } showRedButton = () => {
    this.setState({
      color: 'red'
    })
  } showGreenButton = () => {
    this.setState({
      color: 'green'
    })
  } render() {
    const { color } = this.state;
    const { showRedButton, showGreenButton } = this;
    return (
      <React.Fragment>
        <Button color={color}/>
        <button onClick={showRedButton}> red </button>
        <button onClick={showGreenButton}> green </button>
     </React.Fragment>)
  }
}ReactDOM.render(<App/>, document.getElementById('root'))
```

[PureComponent 代码示例](https://codepen.io/n0rush/pen/JwdYVG)

# 银弹？

`PureComponent`基本上是内置`shouldComponentUpdate`挂钩的`Component`。似乎我们应该在任何情况下使用`PureCompnent`,因为它有本地渲染优化？不完全是。`PureComponent`正在使用浅相等进行道具和状态比较。

```
class TodoList extends React.PureComponent {
  render() {
    const { todos } = this.props;
    return (<ul>
      {
        todos.map((it, index) => <li key={index}> {it} </li>)
      }
    </ul>)
  }
}class App extends React.Component {
  inputRef = React.createRef() state = {
    todos: []
  } onAdd = () => {
    let { todos } = this.state;
    const text = this.inputRef.current.value;
    todos.push(text);
    this.setState({
      todos
    });
  } render() {
    const { todos } = this.state;
    const { onAdd, inputRef } = this
    return (
      <React.Fragment>
        <input ref={inputRef} type='text' placeholder='enter todo'/>
        <button onClick={onAdd}> add todo </button>
        <TodoList todos={todos}/>
      </React.Fragment>
    )
  }
}ReactDOM.render(<App/>, document.getElementById('root'))
```

[抗纯组分](https://codepen.io/n0rush/pen/wRKKZp)

在本例中，`TodoList`永远不会重新渲染，因为`todos`(这是来自`App`的状态，并作为道具传递给`TodoList`)总是具有相同的引用。

# 如何让它发挥作用？

```
class TodoList extends React.PureComponent {
  render() {
    const { todos } = this.props;
    return (<ul>
      {
        todos.map((it, index) => <li key={index}> {it} </li>)
      }
    </ul>)
  }
}class App extends React.Component {
  inputRef = React.createRef() state = {
    todos: []
  } onAdd = () => {
    let { todos } = this.state;
    const text = this.inputRef.current.value;
    this.setState({
      todos: [...todos, text]
    });
  } render() {
    const { todos } = this.state;
    const { onAdd, inputRef } = this
    return (
      <React.Fragment>
        <input ref={inputRef} type='text' placeholder='enter todo'/>
        <button onClick={onAdd}> add todo </button>
        <TodoList todos={todos}/>
      </React.Fragment>
    )
  }
}ReactDOM.render(<App/>, document.getElementById('root'))
```

[纯组件参考](https://codepen.io/n0rush/pen/wRKMMj)

它现在的工作原理是，我们现在每次调用`setState`时都创建一个新的`todos`数组

# React.memo —用于功能组件

# 它是如何工作的

```
let count = 0;const Button = React.memo(function(props) {
  const { color } = props
  count++;
  return (
    <React.Fragment>
       <span> Render count: { count } </span>
       <button style={{ color }}> text </button>
      </React.Fragment>
    )
})class App extends React.Component {
  state = {
    color: 'red'
  } showRedButton = () => {
    this.setState({
      color: 'red'
    })
  } showGreenButton = () => {
    this.setState({
      color: 'green'
    })
  } render() {
    const { color } = this.state;
    const { showRedButton, showGreenButton } = this;
    return (
      <React.Fragment>
        <Button color={color}/>
        <button onClick={showRedButton}> red </button>
        <button onClick={showGreenButton}> green </button>
     </React.Fragment>)
  }
}ReactDOM.render(<App/>, document.getElementById('root'))
```

# 通知；注意

*   如果您想关注我的博客系列的最新消息/文章，请[【观看】](https://github.com/n0ruSh/blogs/)订阅。