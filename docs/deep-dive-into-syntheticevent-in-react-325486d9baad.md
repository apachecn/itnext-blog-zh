# 深入探究 React 中的合成事件

> 原文：<https://itnext.io/deep-dive-into-syntheticevent-in-react-325486d9baad?source=collection_archive---------0----------------------->

![](img/417b79541fc0a025f7382aafb6245290.png)

> `*SyntheticEvent*` *是一个包装器，它是 React 事件系统*的一部分

# 为什么它有用

*   跨浏览器:通过`nativeEvent`属性包装浏览器的本地事件，并在顶层提供统一的 api 和一致的行为
*   更好的性能:事件通过冒泡[委托](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events)给文档。[注意 React 17 中删除了事件池](https://reactjs.org/blog/2020/08/10/react-v17-rc.html#no-event-pooling)

# 它是什么样的

每个`SyntheticEvent`对象都有以下属性

```
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget // the element that the handler is registered
boolean defaultPrevented
number eventPhase
boolean isTrusted
DOMEvent nativeEvent
void preventDefault()
boolean isDefaultPrevented()
void stopPropagation()
boolean isPropagationStopped()
void persist()
DOMEventTarget target // the element that the handler is triggered
number timeStamp
string type
```

# 当本地和合成事件混合时

合成事件被委托给`document`节点。因此，本地事件首先被触发，并且事件冒泡到 doucment，之后合成事件被触发。

```
import React, { useEffect } from "react";export default function App() {
  useEffect(() => {
    document.addEventListener('click', e => {
      // @4 stop parent native propogation
      // e.stopPropagation();
      console.log('[document] native dom event triggered');
    });
    const parentDiv = document.getElementById('parent');
    parentDiv.addEventListener('click', e => {
      // @1 stop parent native propogation
      // e.stopPropagation();
      console.log('[parent div] native dom event triggered');
    });
  }, []); const onParentClick = e => {
    // @2 stop parent propogation
    // e.stopPropagation();
    console.log('[parent div] synthetic event triggered');
  }; const onChildClick = e => {
    // @3 stop child propogation
    // e.stopPropagation();
    console.log('[child button] synthetic event triggered');
  }; return <div id="parent" onClick={onParentClick}>
    <button onClick={onChildClick}>child</button>
  </div>;
}
```

点击各种设置的`child`按钮的输出将是:

## 不停止发布—输出:

```
[parent div] native dom event triggered
[child button] synthetic event triggered
[parent div] synthetic event triggered
[document] native dom event triggered
```

本机 dom 事件是在组件挂载后调用时注册的。

## 停止在父节点的本机节点(即@1)上的传播—输出:

```
[parent div] native dom event triggered
```

没有冒泡到文档，就好像文档上没有发生单击事件= >没有触发合成事件

## 停止对父合成事件(即@2)的提交—输出:

```
[parent div] native dom event triggered
[child button] synthetic event triggered
[parent div] synthetic event triggered
[document] native dom event triggered
```

## 停止对孩子的合成事件(即@3)的建议—输出:

```
[parent div] native dom event triggered
[child button] synthetic event triggered
[document] native dom event triggered
```

## 停止对孩子的合成事件(即@4)的查询—输出:

```
[parent div] native dom event triggered
[child button] synthetic event triggered
[parent div] synthetic event triggered
[document] native dom event triggered
```

# 要点

*   如果您想在所有 React 处理程序之后接收事件，请在`document`上监听。在任何其他地方监听，以便在 React 处理程序之前接收
*   React 事件处理程序将始终在本机捕获处理程序之后执行

[代码](https://codesandbox.io/s/synthetic-event-ruv3f?file=/src/App.js)

# 通知；注意

*   如果您想关注我的博客系列的最新消息/文章，请[【观看】](https://github.com/n0ruSh/blogs/)订阅。