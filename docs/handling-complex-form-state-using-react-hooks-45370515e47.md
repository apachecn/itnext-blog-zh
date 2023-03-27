# 使用 React 钩子处理复杂的表单状态

> 原文：<https://itnext.io/handling-complex-form-state-using-react-hooks-45370515e47?source=collection_archive---------0----------------------->

一些优点的替代方法

![](img/be373847515f7929d3fcc30a1b4ba142.png)

大家都上钩了吗？[形象积分——免费代码营](https://cdn-media-1.freecodecamp.org/images/1*0MgGEfZfLO91g1Oa2h3ebQ@2x.png)

所以，距离 **React hooks** 发布已经有一段时间了。从表面上看，每个人都为他们着迷。嗯，我明白。因为我也是你们中的一员。钩子让我着迷！

钩子允许我们创建更小的、可组合的、可重用的、更易管理的 React 组件。

有时你可能使用钩子来管理表单状态，使用 **useState** 或 **useReducer。**

现在，让我们考虑一个场景，其中您必须管理一个具有多个表单输入的复杂表单状态，表单输入可以是几种不同的类型，如*文本、数字、日期*输入。表单状态甚至可以有嵌套信息，例如，用户的地址信息有自己的子字段，如 *address.addressLine1* 、 *address.addressLine2* 等。

也许你还需要根据当前状态更新表单状态，比如切换按钮。

现在，如果您对每个单独的表单字段使用 **useState** ，那么您就能够基于当前状态计算新的状态。

```
const [modalActive, updateModal] = useState(false)
.
.
.
// new state based on previous
updateModal(prev => !prev)
```

但是，如果你有太多的单个表单字段，比如 100+，( **YESS！！。我管理着 100 多个表单字段**，这种方法并不友好。

想象一下！！..

```
const [firstName, setFirstName] = useState('')
const [middleName, setMiddleName] = useState('')
const [lastName, setLastName] = useState('')
.
.
.
.
```

编写不同的使用状态，然后为每个字段使用单独的更新函数是不实际的。

所以，我们的另一个选择是钩子， **useReducer。**

让我们看一个例子。

```
const initialState = {
  firstName: '',
  lastName: ''
};function reducer(state, action) {
  switch (action.type) {
    case 'firstName':
      return { firstName: action.payload };
    case 'lastName':
      return { lastName: action.payload };
    default:
      throw new Error();
  }
}function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);return (
    <>
      <input
        type="text"
        name="firstName"
        placeholder="First Name"
        onChange={(event) => {
          dispatch({
           type: 'firstName',
           payload: event.target.value
          })
        }}
        value={state.firstName} />
      <input
        type="text"
        name="lastName"
        placeholder="Last Name"
        onChange={(event) => {
          dispatch({
           type: 'lastName',
           payload: event.target.value
          })
        }}
        value={state.lastName} />
   </>
  );
}
```

诶！！，不好。

你不可能为 reducer 中的那些 **n** 数量的表单字段编写每个用例。

但是 **useReducer** 中使用的 reducer 函数只是一个普通的函数，返回更新后的状态对象。所以，我们可以做得更好。

```
function reducer(state, action) {
  // field name and value are retrieved from event.target
  const { name, value } = action

  // merge the old and new state
  return { ...state, [name]: value }
}
```

现在，这看起来是一个更好更干净的减速器。

但是……这不允许我们在用回调函数调用 update 函数时根据当前状态计算新的状态。就像我们能做的一样..

```
this.setState((prev) => ({ isActive: !prev }))// orconst [modalActive, updateModal] = useState(false)
.
.
.
updateModal(prev => !prev)
```

此外，如何更新嵌套状态，如 *address.addressLine1，address.pinCode.*

好吧！！。关于使用不太理想的方法来管理复杂的表单状态，我们已经讨论了很多。

让我向你展示解决方案。

哒哒！！

所以，这是处理这种复杂表单场景的完整源代码。

我稍微解释一下减速器( **enhancedReducer** :P)的功能。

reducer 函数接收两个参数，第一个参数是更新前的当前状态。当您调用 **updateState / dispatch** 函数来更新减速器状态时，会自动提供该参数。reducer 函数的第二个参数是您用来调用 **updateState** 函数的值，它不必是{ type: 'something '，payload: 'something' }形式的典型 *redux 操作对象*。它可以是任何东西，数字、字符串、对象甚至函数。

这就是我们正在利用的。如果 **updateArg** 是一个函数，我们用当前状态调用它来计算新的状态。我们从这个函数返回的任何对象都成为我们的新状态。

如果 updateArg 是一个普通的旧 Javascript 对象，那么有两种情况。

1- **该对象没有 _path 和 _value 属性—** ，因此是一个普通的更新对象，就像我们给 this.setState 的一样。因此，您可以用一个包含您想要更新的状态片段的新对象来调用 **updateState** ，它会将其与旧对象合并，并返回新状态。

2- **对象具有 _path 和 _value 属性—** 当使用具有这两个属性的对象调用 updateState 函数时。我们将此视为特殊情况，其中 **_path** 表示嵌套的字段路径。以字符串形式，例如:“address.pinCode”或表示路径的数组[“address”，“pinCode”]。

但是，我们如何使用这样的路径表示来更新对象中的嵌套字段呢？。我们使用**lodash’**s**set**方法。它接受两种路径形式作为 update 和 object 的有效输入。

```
set(objectToUpdate, path, newValue)const state = {
  name: {
   first: '',
   middle: '',
   last: ''
  }
}// and to update, for eg: first name.
// both ways of path are correct.set(state, 'name.first', 'Aditya')
set(state, ['name', 'first'], 'Aditya')
```

但是， **set** 方法就地改变对象，并且不返回新的副本，但是在 React world 中，变化检测依赖于**不变性**，一个新的数据副本，在内存中有一个新的位置。

因此，为了绕过这一点，我们使用 **immer，**以一种易于使用的形式帮助处理 Javascript 对象的不变性。

```
import produce from 'immer'produce(state, draft => {
  set(draft, _path, _value);
});
```

**从产生**函数 immer 将对象作为它的第一个参数，在我们的例子中是当前状态，它的第二个参数是一个函数，它接收对象的**草稿副本**以进行变异，无论您在草稿状态下在这个函数内部修改什么，都是在**副本**上完成的，而不是实际输入对象的**状态**就地完成。然后自动返回**新对象**和**更新后的**数据。

这是我们的增强型减速器:D

仅仅

```
yarn add lodash immer
```

尽情享受吧。

gist 中的例子可以进一步细化，在 **enhancedReducer** 中处理更多的边缘情况，并且可以通过映射表单规范对象来缩短表单字段代码，以动态地创建它，并减少代码重复和其他一些事情。

有些读者可能对这种方法有不同的看法。所以，我们可以随时讨论。

也许你们中的一些人会有一个问题，如果我们这么想复制 **this.setState** ，那么为什么不也有 setState 第二个参数回调函数，在状态更新后执行一些操作。嗯，这还不够明确！！！。我们将采用先做这个，后做那个的方法。我们将一步一步地告诉代码，如何做某事。而不是简单的告诉它做什么。我会使用 **useEffect** 而不是回调函数和 how to do 方法，因为那是声明性的，*对变化做出反应。*

声明式和命令式，函数式编程是另一个话题，我会在以后分享。

资源—

 [## Lodash 文档

### 编辑描述

lodash.com](https://lodash.com/docs/4.17.15#set) [](https://github.com/immerjs/immer) [## immerjs/immer

### 通过简单地修改“年度突破”的当前树获胜者，创建下一个不可变的状态树…

github.com](https://github.com/immerjs/immer)