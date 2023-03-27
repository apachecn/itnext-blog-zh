# 了解反应键

> 原文：<https://itnext.io/understanding-react-key-18787854489a?source=collection_archive---------0----------------------->

> *键给 react 元素一个稳定的* `*identity*`

# 没有关键字的数组中的警告

```
import React from 'react';function App() {
  const persons = [‘mike’, ‘jason’, ‘sharky’];
  return <ul>
    {persons.map(p => {
      return <li>{p}</li>
    })}
  </ul>
}
```

React 会抛出一个警告，上面的代码是:`Warning: Each child in a list should have a unique "key" prop.`

# 用什么钥匙？

按键应该是`unique`和`stable`

# 指数？也许吧，但只是`under some circumstances`

数组索引在数组中是唯一的。

```
import React, { useState } from 'react';function App() {
  const persons = [‘mike’, ‘jason’, ‘sharky’];
  return <ul>
    {persons.map((p, index) => {
      return <li key={index}>{p}</li>
    })}
  </ul>
}
```

但是，如果项目的顺序/数量可能改变，数组索引不是`stable`。

```
import React, { useState } from 'react';function App() {
    const [persons, setPersons] = useState(
    ['mike', 'jason', 'sharky']
  ); const onAdd = () => {
    setPersons(['fishman', ...persons])
  }
  return <>
    <button onClick={onAdd}> Add fishman on top </button>
    <ul>
      {persons.map((p, index) => {
        return <li key={index}><input type="checkbox"/>{p}</li>
        })}
      </ul>
  </>
}
```

[array-index-as-key—code sandbox](https://codesandbox.io/s/staging-resonance-vy7eq)

勾选`mike`然后点击添加按钮将导致`mike`的检查状态消失

# `Stable`键

通常我们会为来自后端/数据库的每个记录提供一个唯一且稳定的属性。

```
import React, { useState } from ‘react’;export default function App() {
  const [persons, setPersons] = useState(
    [
      {id: 'uywoejk', name: 'mike'},
      {id: 'woeioqj', name: 'jason'},
      {id: 'eljlkqd', name: 'sharky'}
    ]
  ); const onAdd = () => {
    setPersons([{
      id: 'wuioeioe', name: 'fishman'
    }, ...persons])
  }
  return <>
    <button onClick={onAdd}> Add fishman on top </button>
    <ul>
      {
        persons.map((p, index) => {
          return <li key={p.id}><input type="checkbox"/>{p.name}</li>
        })
      }
    </ul>
  </>
}
```

[稳定键—代码沙盒](https://codesandbox.io/s/gallant-visvesvaraya-0b3lo)

# 使用键卸载组件

用例:我有一个选择和输入组件。每次选择选项改变时，将输入重置为默认状态，

```
import React, { useState } from “react”;export default function App() {
  const [option, setOption] = useState("");
  return (
    <>
      <select onChange={e => setOption(e.target.value)}>
        <option key="a">A</option>
        <option key="b">B</option>
      </select>
      <input defaultValue="this is default"/>
      <input key={option} defaultValue="this is default" />
    </>
  );
}
```

# 结论

*   `key`是 react 元素的唯一标识符
*   `key`应该是`unique`和`stable`。在不同渲染阶段具有不同`key`的 react 元素将被视为不同的元素。旧的将被卸载，并创建一个新的。
*   仅当数组项目的数量/顺序在整个应用程序生命周期中保持不变时，才使用数组索引作为键。
*   始终倾向于使用后端/数据库唯一标识符 id like 属性作为`key`

# 通知；注意

*   如果您想关注我的博客系列的最新消息/文章，请[【观看】](https://github.com/n0ruSh/blogs/)订阅。