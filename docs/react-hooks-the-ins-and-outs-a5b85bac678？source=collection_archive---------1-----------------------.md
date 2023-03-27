# React Hooks —细节

> 原文：<https://itnext.io/react-hooks-the-ins-and-outs-a5b85bac678?source=collection_archive---------1----------------------->

> [*挂钩*](https://reactjs.org/docs/hooks-intro.html) *是让你从功能组件“挂钩”React 状态和生命周期特性的功能。*

![](img/417b79541fc0a025f7382aafb6245290.png)

# 为什么是钩子

# 我们如何重用有状态逻辑— [HOC](https://reactjs.org/docs/higher-order-components.html)

例如

```
import { withRouter } from "react-router-dom";
import { Form } from 'antd';function A(props) {
  const {
    match: { params },
    history,
    form
  } = props; return <div> test </div>;
}export default withRouter(Form.create()(A))
```

HOC 有一些缺点。

*   **包装组件的外壳**。HOC 实际上返回了一个包装了原始组件的新组件。
*   依赖关系不明确:从组件 **A** 来看，我们无法清楚地知道 **this.props.form** 来自 **Form.create** HOC， **this.props.history** 来自 **withRouter** HOC
*   命名冲突:想象一下 **Form.create** 和 **withRouter** 都注入了一个同名的道具，这很容易发生，因为它们是由两个团队正确维护的两个独立的 hoc

# 我们如何用钩子来改进

```
import { useHistory } from "react-router-dom";
import { Form } from 'antd';export default function A(props) {
  const [form] = Form.useForm();
  const history = useHistory(); return <div> test </div>;
}
```

*   不再有包装，因为它们都在同一级别
*   清除依赖关系，因为它们来自不同的钩子调用:**表单**来自**使用表单**和**历史**来自**使用历史**
*   没有命名冲突，因为它们来自不同的钩子，你可以重命名钩子返回值

# 实践中的挂钩

要实现具有**复位**功能的受控**输入**元素，我们可以这样做:

```
import React, { useState } from "react";
import { Input, Button } from 'antd';export default function App() {
  const initialValue = 'please enter something';
  const [inputValue, setInputValue] = useState(initialValue); return (
    <div className="App">
        // note the value is from e.target
      <Input value={inputValue} onChange={e => setInputValue(e.target.value)}/>
      <Button onClick={() => setInputValue(initialValue)}> Reset </Button>
    </div>
  );
}
```

有几个有状态的逻辑，我们可以抽象和恢复。

*   取值—例如，目标值
*   复位逻辑—复位至初始值

# 自定义挂钩— [使用事件目标](https://codesandbox.io/s/useeventtarget-z18i3?file=/src/useEventTarget.js)

```
import { useState, useCallback } from "react";export default function useEventTarget(initialValue = "") {
  const [value, setValue] = useState(initialValue);
  const onChange = useCallback(e => setValue(e.target.value), []); // reusable value fetch logic
  const reset = useCallback(() => setValue(initialValue), [initialValue]); // resuable reset logic return {
    value,
    onChange,
    reset
  };
}
```

上面的例子可以用定制的 useEventTarget 钩子[重构](https://codesandbox.io/s/useeventtarget-z18i3?file=/src/App.js):

```
import React from "react";
import { Input, Button } from "antd";
import useEventTarget from "./useEventTarget";export default function App() {
  const { value, onChange, reset } = useEventTarget("please enter something"); return (
    <div className="App">
      <Input value={value} onChange={onChange} />
      <Button onClick={reset}> Reset </Button>
    </div>
  );
}
```

# 精神开销

> *Hooks 不能代替类生命周期，尽管 useEffect 可以模拟生命周期的行为。Hooks 有助于重用有状态逻辑，但是它也有代价。*

假设我需要一个显示**计数**页面，该计数从零开始，每秒递增 1。

```
import React, { useState, useEffect } from 'react';export default function SetInterval() {
  const [ count, setCount ] = useState(0); useEffect(() => {
    const interval = setInterval(() => {
      console.log("interval count", count); // count is always 0
      setCount(count + 1);
    }, 1000);
    return () => {
      clearInterval(interval)
    }
  }, []); return <div> normal count is { count } </div>;
}
```

当[的代码在](https://codesandbox.io/s/setinterval-wt720?file=/src/App.js)之上时，页面上的计数将保持在 1。这是由于 **useEffect** 调用仅在挂载时运行(因为 **useEffect** 依赖项在第 14 行被设置为[])并且**计数**值在该点为 0(即我们在第 4 行用 **useState** 调用设置的初始值)。每次调用 interval 函数时， **count** 值总是为 0，因为它是一个闭包变量。因此，setCount(count + 1)将是 setCount(1)。

# [解决方案 1 —使用效果依赖关系](https://codesandbox.io/s/setinterval-with-dependency-36fb5?file=/src/App.js:0-355)

> *如果您的使用效果依赖于外部变量，请指定依赖关系*

```
import React, { useState, useEffect } from "react";export default function SetInterval() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const interval = setInterval(() => {
      console.log('interval count', count); // increment by one per run
      setCount(count + 1);
    }, 1000); return () => {
      // each time count value changes (i.e per second, clear the interval and init one with new count value
      console.log('interval clear', count); 
      clearInterval(interval);
    }
  }, [count]); return <div> dep count is {count} </div>;
}
```

# [解决方案 2 —使用状态回调](https://codesandbox.io/s/setinterval-with-prev-vsi1y)

```
import React, { useState, useEffect } from "react";export default function SetInterval() {
  const [count, setCount] = useState(0); useEffect(() => {
    const interval = setInterval(() => {
      // it doesnt matter what count was, it tells React to increment by one with previous value
      setCount(a => a + 1);
    }, 1000); return () => {
      clearInterval(interval);
    };
  }, []); return <div> prev normal count is {count} </div>;
}
```

# [解决方案 3 — useRef](https://codesandbox.io/s/setinterval-with-ref-qldj0?file=/src/App.js)

```
import React, { useRef, useState, useEffect } from "react";export default function SetInterval() {
  const [count, setCount] = useState(0);
  const ref = useRef();
  ref.current = count; useEffect(() => {
    const interval = setInterval(() => {
      console.log("interval count", ref.current);
      setCount(ref.current + 1);
    }, 1000); return () => {
      clearInterval(interval);
    };
  }, []); return <div> ref count is {count} </div>;
}
```

# 通知；注意

*   如果您想关注我的博客系列的最新消息/文章，请[【观看】](https://github.com/n0ruSh/blogs/)订阅。