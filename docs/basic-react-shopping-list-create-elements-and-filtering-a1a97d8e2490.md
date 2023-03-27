# 基本反应购物清单-创建元素和过滤

> 原文：<https://itnext.io/basic-react-shopping-list-create-elements-and-filtering-a1a97d8e2490?source=collection_archive---------2----------------------->

![](img/db419039a286487131dcd0b05e720ba3.png)

在本教程中，我将向你展示如何用过滤制作一个非常基本和简单的购物清单。

我用 codepen 写代码。

首先，您需要一个容器来呈现 React 应用程序，因此我们在 HTML 部分创建了一个 id 为“reactContainer”的 div，这可能就是您想要的:

```
<div id=”reactContainer”>

</div>
```

然后我写了一些 SAS 代码，但是我没有花太多时间。

```
body
  background: #4abdac
  width: 100vw
  height: 100vh

.top
  background: #fc4a1a
  color: white
  border: 2px solid white.wb
  background: #dfcde3
```

现在是时候开始编写一些 React 代码了。我只创建了一个组件，在一个真正的应用程序中，你应该创建一些组件。首先，我们添加构造函数和呈现方法。

```
class App extends React.Component 
{
 constructor(props)
 {
 super(props);
 this.state = {
 todosInit: [‘100 kg Meat’,’Ham’, ‘Cheese’],
 todos: [],
 };
 }

 }render() 
 {
 return (
 <div>
 <div className=”container top”>
 <div className=”row”>
 <div className=”col-lg-12">
 <h2 className=”text-center”>Shopping List</h2>
 </div>
 </div>
 </div>
 </div>
 ); 
 }

}ReactDOM.render(<App />, document.getElementById(‘reactContainer’));
```

状态 todosInit 负责呈现初始列表，而 todos 将存储所有 todos。现在，我们将添加 ComponentDidMount，用于在 todos 状态下插入 todosInit，我们将插入两个方法，一个用于管理输入字段中的状态更改，另一个用于向列表中添加新元素。

```
class App extends React.Component 
{
 constructor(props)
 {
 super(props);
 this.state = {
 todosInit: [‘100 kg Meat’,’Ham’, ‘Cheese’],
 todos: [],
 todoText: ‘’,
 message: false
 };
 this.updateTodoText = this.updateTodoText.bind(this);
 this.createTodo = this.createTodo.bind(this);
 }

 componentDidMount()
 {
 this.setState({
 todos: this.state.todosInit,
 });
 }

 updateTodoText (e)
 {
 this.setState({
 todoText: e.target.value
 });
 }

 createTodo (e)
 {
 e.preventDefault();
 this.setState({ 
 todos: […this.state.todos, this.state.todoText],
 todoText: ‘’,
 });
 }render() 
 {
 return (
 <div>
 <div className=”container top”>
 <div className=”row”>
 <div className=”col-lg-12">
 <h2 className=”text-center”>Shopping List</h2>
 </div>
 </div>
 </div>
 <div className=”container wb”>
 <div className=”row”>
 <form onSubmit={this.createTodo}>
 <div className=”col-lg-12 input-group”>
 <input type=”text”
 className=”center-block”
 placeholder=”Insert here…”
 value={this.state.todoText}
 onChange={this.updateTodoText}
 />
 <button className=”btn btn-success center-block”>Create</button>
 </div>
 </form>
 <ul>
 {this.state.todos.map((todo) => 
 {
 return (<li key={Math.floor(Math.random() * 10000) + 1}>{todo}</li>);
 }
 )} 
 {this.state.message ? <li>No search results.</li> : ‘’ }
 </ul>
 </div>
 </div>
 </div>
 ); 
 }

}ReactDOM.render(<App />, document.getElementById(‘reactContainer’));
```

在映射一个州的元素时，您需要向元素添加一个键。我只是用了一个随机数，在实际应用中你不应该用它。现在我们要写过滤部分的代码

```
 filterTodo(e)
 { 
 var updatedList = this.state.todosInit;
 updatedList = updatedList.filter((item =>{
 return item.toLowerCase().search(
 e.target.value.toLowerCase()) !== -1;
 }) );
 this.setState({ 
 todos: updatedList,
 });
 if (updatedList == 0 ) {
 this.setState({ 
 message: true,
 });
 } else {
 this.setState({ 
 message: false,
 });
 }
 }
```

在向您展示输入字段的完整代码之前，我将向您解释 filterTodo 方法的逻辑。当搜索字段的输入发生变化时，将调用此方法，并检查列表中是否存在该值。该代码使所有输入值小写，并与我们找到一些匹配，它更新 todos 状态，并在屏幕上呈现。我创建了一个消息状态，只是为了在没有搜索结果时显示消息“没有找到结果”。

现在完成了代码，你可以在[https://codepen.io/nunosoares/pen/zjWpJN](https://codepen.io/nunosoares/pen/zjWpJN)中阅读并做一些修改。可以回复，展示其他解决方案。玩得开心。

```
class App extends React.Component 
{
 constructor(props)
 {
 super(props);
 this.state = {
 todosInit: [‘100 kg Meat’,’Ham’, ‘Cheese’],
 todos: [],
 todoText: ‘’,
 message: false
 };
 this.updateTodoText = this.updateTodoText.bind(this);
 this.createTodo = this.createTodo.bind(this);
 this.filterTodo = this.filterTodo.bind(this);
 }

 componentDidMount()
 {
 this.setState({
 todos: this.state.todosInit,
 });
 }

 updateTodoText (e)
 {
 this.setState({
 todoText: e.target.value
 });
 }

 createTodo (e)
 {
 e.preventDefault();
 this.setState({ 
 todos: […this.state.todos, this.state.todoText],
 todoText: ‘’,
 });
 }

 filterTodo(e)
 { 
 var updatedList = this.state.todosInit;
 updatedList = updatedList.filter((item =>{
 return item.toLowerCase().search(
 e.target.value.toLowerCase()) !== -1;
 }) );
 this.setState({ 
 todos: updatedList,
 });
 if (updatedList == 0 ) {
 this.setState({ 
 message: true,
 });
 } else {
 this.setState({ 
 message: false,
 });

 }

 }render() 
 {
 return (
 <div>
 <div className=”container top”>
 <div className=”row”>
 <div className=”col-lg-12">
 <h2 className=”text-center”>Shopping List</h2>
 </div>
 </div>
 </div>
 <div className=”container wb”>
 <div className=”row”>
 <form onSubmit={this.createTodo}>
 <div className=”col-lg-12 input-group”>
 <input type=”text”
 className=”center-block”
 placeholder=”Insert here…”
 value={this.state.todoText}
 onChange={this.updateTodoText}
 />
 <button className=”btn btn-success center-block”>Create</button>
 </div>
 </form>
 <ul>
 {this.state.todos.map((todo) => 
 {
 return (<li key={Math.floor(Math.random() * 10000) + 1}>{todo}</li>);
 }
 )} 
 {this.state.message ? <li>No search results.</li> : ‘’ }
 </ul>
 </div>
 <input type=”text”
 className=”center-block”
 placeholder=”Filter here…”
 onChange={this.filterTodo}
 />
 </div>
 </div>
 ); 
 }

}ReactDOM.render(<App />, document.getElementById(‘reactContainer’));
```