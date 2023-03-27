# React 挂钩和上下文(没有 Redux)

> 原文：<https://itnext.io/react-hooks-context-without-redux-401235df2a68?source=collection_archive---------3----------------------->

在本文中，我将展示 React 的特性、钩子和上下文是如何实现高效简单的状态管理的。以前只有在使用 Redux 和 MobX 等状态管理库时才有可能，React 现在内置的支持使开发体验变得流畅简单。

![](img/35a6b45a467a5f0a656d638e6ffc2aa8.png)

请克隆项目以跟进:[https://github.com/amwais/react-hooks-and-context](https://github.com/amwais/react-hooks-and-context)。

主分支是最终版本，每个分支代表本教程中的一个步骤。

# ***V01:基本挂钩用法***

我已经使用了 *create-react-app* 来引导这个项目。

如您所见，我已经将 App.js 更改为功能组件，因为我们不再需要使用基于类的组件。

该文件包含应用程序的基本工作版本:一个标题、一个包含文本输入字段和添加按钮的“添加待办事项”表单，以及最终呈现的待办事项列表。这些还没有封装到单独的组件文件中，现在只是混合在一起。

这里使用了两个钩子: **useState** 和 **useReducer。**

**useState，**两者中较简单的一个，负责保持表单组件的本地状态:

```
const [ textField, setTextField ] = useState('');
```

useState 是一个钩子，您可以根据需要调用它多次，每次调用只负责存储一个状态键。我们使用数组分解从钩子中获得两个返回值:状态键的名称( *textField* )和负责修改它的函数的名称( *setTextField* )。

现在，在我们的“尚待分离的组件”表单中，我们可以很容易地使用这个本地状态来创建一个受控的输入字段:

```
<input autoFocus type="text" value={textField} onChange={(e) => setTextField(e.target.value)} />
```

注意像 *this.state.textfield* 和*this . setstate({ text:e . target . value })*这样的调用是不需要的，取而代之的是两个简洁直接的调用。

现在我们来看看 **useReducer** 。这个钩子比 **useState、**更复杂，它非常类似于 Redux 管理全局状态的方式。

让我们从创建一个缩减器开始，这不在我们的应用程序组件的范围之内:

```
const appReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, {
        text: action.payload
      }];
    case 'REMOVE_TODO':
      return state.filter((_, index) => index !== action.payload);
    case 'TOGGLE_COMPLETED':
      return state.map((item, index) => {
        if (action.payload === index) {
          return {
            ...item,
            completed: !item.completed
          };
        }
        return item;
      });
    default:
      return state;
  }
};
```

如您所见，在使用 Redux 时，通常定义缩减器的方式没有什么不同。

现在，在我们的应用程序组件中，让我们调用 **useReducer** :

```
const [state, dispatch] = useReducer(appReducer, [{
  text: 'Eat Dinner',
  completed: false
}]);
```

这就是神奇之处:不需要使用 Redux 的 connect 函数，也不需要用它包装我们的组件。相反，除了初始状态之外，我们还为钩子提供了 reducer，瞧，我们可以访问状态和调度函数了。简单。干净。简单。

例如，我们现在可以创建动作函数来调用 dispatch 函数，目标是我们的 reducer:

```
const handleDelete = (index) => {
  dispatch({
    type: 'REMOVE_TODO',
    payload: index
  });
};
```

从我们的删除按钮调用这个函数:

```
<button onClick={() => handleDelete(index)}>Delete</button>
```

# V02:带上下文的组件封装

我们应用程序的下一步将是把这三个组件中的每一个都分离到一个单独的文件中。

这是 React 中的一种常见模式，但是使用 Redux 时，它曾经是一项有些乏味的任务，因为必须创建组件，然后使用 mapStateToProps(用于将状态键映射到组件的 Props)和 mapDispatchToProps(用于将动作函数映射到组件的 props)将该组件包装在一个连接函数中。

但是现在，如果没有 Redux，我们如何从封装的组件内部调用 **dispatch** 或访问**reducer 的(全局)状态呢？**

这就是上下文 API 发挥作用的地方，它使开发 React 应用程序成为一种更愉快的体验。首先，看看这个版本中的 App.js:现在它包含了返回的 JSX、我们的钩子定义和我们的 reducer 定义(接下来也会分开)。此外，还会创建一个新的上下文:

```
export const TodosDispatch = React.createContext(null);
```

使用这个 TodosDispatch 上下文是为了向其下的任何子组件提供对 reducer 的调度功能的访问。我们是这样提供的:

```
<div className = "App" >
  <TodosDispatch.Provider value = {dispatch} >
    <Header / >
    <AddToDoForm / >
    <ToDoList todos = {state}/> 
  </TodosDispatch.Provider >
</div>
```

现在，让我们将 AddToDoForm.js 作为一个需要访问该上下文的组件的示例。首先，我们导入上下文:

```
import { TodosDispatch } from '../App';
```

接下来，我们调用 **useContext** 钩子:

```
const dispatch = useContext(TodosDispatch);
```

就这样，我们可以访问应用程序“store ”,并且可以从我们的子组件调用调度功能:

```
const handleAddToDo = (e) => {
  e.preventDefault();
  if (textField.length > 0) {
    dispatch({
      type: 'ADD_TODO',
      payload: textField
    });
    setTextField('');
  }
};
```

# V03:使用多个减速器

但是如果我们有多个 reducers，并且一些组件需要访问它们呢？保持同样的模式应该能让我们得到我们想要的。在分支中，我添加了一个错误减少器，它将作为一种全局方式来显示不同组件中的错误。我们首先创建另一个上下文:

```
export const ErrorsContext = React.createContext(null);
```

接下来，我们创建减速器:

```
const [errors, errorDispatch] = useReducer(errorReducer, {
  header: '',
  form: ''
};);
```

现在，在我们的组件树中，我们包装需要访问该上下文的组件:

```
<div className = "App" >
 <TodosDispatch.Provider value = {todosDispatch} >
  <ErrorsContext.Provider value = {[errors, errorDispatch]}>
    <Header />
    <AddToDoForm />
  </ErrorsContext.Provider> 
  <ToDoList todos = {todos}/>
  <ErrorsContext.Provider value = {errorDispatch}>
    <ErrorButtons />
  </ErrorsContext.Provider>
 </TodosDispatch.Provider> 
</div>
```

注意我们如何使用多个上下文，我们可以用任何上下文包装任何组件。

还要注意，我们附加到这个上下文的值不一定只是调度函数，也可以是一个数组:

```
<ErrorsContext.Provider value = {[errors, errorDispatch]}>
```

这就是 Header 和 AddToDoForm 能够访问实际错误减少器状态的方式。

# 延伸阅读:

[https://reactjs.org/docs/hooks-reference.html](https://reactjs.org/docs/hooks-reference.html#usestate)

[https://reactjs.org/docs/context.html](https://reactjs.org/docs/context.html)