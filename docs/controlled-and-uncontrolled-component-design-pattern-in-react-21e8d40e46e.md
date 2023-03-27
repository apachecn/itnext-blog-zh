# React 中的受控和非受控组件设计模式

> 原文：<https://itnext.io/controlled-and-uncontrolled-component-design-pattern-in-react-21e8d40e46e?source=collection_archive---------3----------------------->

![](img/417b79541fc0a025f7382aafb6245290.png)

# 不受控制的

> *`*Uncontrolled*`*组件是在内部存储和维护其自身状态的组件。**

*   *一个`[ref](https://reactjs.org/docs/refs-and-the-dom.html)`用来在你需要的时候找到它的当前值。*
*   *[React](https://reactjs.org/docs/uncontrolled-components.html) 不推荐这种模式，但是当开发人员只关心组件的最终状态而不是中间状态时，这种模式很有用。以下是以不受控方式实施的`Switch`的[示例](https://codepen.io/n0rush/pen/RORrRa)。*

```
*class Switch extends React.Component {
  constructor(props) {
    super(props);
    // maintain its own state
    this.state = {
      checked: false
    };
  } // expose state data for parent component to access
  get value() {
    return this.state.checked;
  } toggle = () => {
    this.setState((prevState) => {
       return {
         checked: !prevState.checked
       }
    })
  } render() {
    // check status is maintained in own state
    const { checked } = this.state;
    let classNames = ['switch']; if (checked) {
      classNames = [...classNames, 'checked']
    }
    return (
      <div>
        <button
          className={classNames.join(' ')}
          onClick={this.toggle}/>
      </div>
    );
  }
}class Wrapper extends React.Component { ref = React.createRef(); getValue = () => {
    alert(this.ref.current.value);
  } render() {
    return <React.Fragment>
      <Switch ref={this.ref}/>
      <button
        onClick={this.getValue}
        style={{ marginTop: 20 }}
      >
        Switch Status
      </button>
    </React.Fragment>
  }
}ReactDOM.render(
  <Wrapper />,
  document.getElementById('root')
);*
```

# *受控的*

> **受控组件通过 props 获取其当前值，并通过回调通知变化。父组件通过处理回调和管理自己的状态以及将新值作为道具传递给受控组件来“控制”它。**

*以下是以受控方式实现的`Switch`的[示例](https://codepen.io/n0rush/pen/jRreVJ)。*

```
*class Switch extends React.Component {
  constructor(props) {
    super(props);
  } toggle = () => {
    // callback passed in from parent
    const { checked, onChange } = this.props;
    onChange(!checked)
  } render() {
    // check status is maintained by props from parent component
    const { checked } = this.props;
    let classNames = ['switch']; if (checked) {
      classNames = [...classNames, 'checked']
    }
    return (
      <div>
        <button
          className={classNames.join(' ')}
          onClick={this.toggle}
        />
      </div>
    );
  }
}class Wrapper extends React.Component { state = {
    checked: false
  } getValue = () => {
    alert(this.state.checked);
  } // when check status changes, maintain it in parent state
  onChange = (checked) => {
    this.setState({
      checked
    })
  } render() {
    return <React.Fragment>
      <Switch
        checked={this.state.checked}
        onChange={this.onChange}
      />
      <button
        onClick={this.getValue}
        style={{ marginTop: 20 }}
      >
        Switch Status
      </button>
    </React.Fragment>
  }
}ReactDOM.render(
  <Wrapper />,
  document.getElementById('root')
);*
```

# *混合的*

*一个`Mixed`组件允许以`Uncontrolled`或`Controlled`方式使用。它接受其初始值作为一个道具，并将其置于状态。然后，它通过[组件生命周期](https://reactjs.org/docs/react-component.html)对 props 更改做出反应，以将状态更新与 props 同步。*

*[混合模式下的切换示例](https://codepen.io/n0rush/pen/NmrOao)*

```
*class Switch extends React.Component {
  constructor(props) {
    super(props);
    // maintain check status in own state
    this.state = {
      checked: props.checked || false
    };
  } get value() {
    return this.state.checked;
  } static getDerivedStateFromProps(nextProps, currentState) {
    // react to props change and sync state accordingly
    // only in controlled mode
    if(nextProps.hasOwnProperty('checked') &&
      (nextProps.checked != currentState.checked)) {
      return {
        checked: nextProps.checked
      }
    }
    return null;
  } toggle = () => {
    const { onChange, checked } = this.props;
    const { checked: checkedInState } = this.state
    if (!this.props.hasOwnProperty('checked')) {
      // no checked prop, uncontrolled
      this.setState({
        checked: !checkedInState
      })
    }
    onChange && onChange(!checkedInState)
  } render() {
    const { checked } = this.state;
    let classNames = ['switch']; if (checked) {
      classNames = [...classNames, 'checked']
    }
    return (
      <div>
        <button
          className={classNames.join(' ')}
          onClick={this.toggle}/>
      </div>
    );
  }
}class Wrapper extends React.Component { ref = React.createRef(); state = {
    controlledChecked: false,
  } controlledOnChange = (checked) => {
    this.setState({
      controlledChecked: checked;
    })
  } getUncontrolledValue = () => {
    alert(this.ref.current.value);
  } getControlledValue = () => {
    alert(this.state.controlledCheck);
  } render() {
    return <React.Fragment>
      <div>
        Uncontrolled: <Switch ref={this.ref}/>
        <button
          onClick={this.getUncontrolledValue}
          style={{marginTop: 20}}
        >
          Uncontrolled Switch Status 
        </button>
      </div>
      <hr/>
      <div>
        Controlled:
        <Switch
          checked={this.state.controlledChecked}
          onChange={this.controlledOnChange}
        />
        <button
          onClick={this.getControlledValue}
          style={{marginTop: 20}}
        >
          Controlled Switch Status
        </button>
      </div>
      <hr/>
    </React.Fragment>
  }
}ReactDOM.render(
  <Wrapper />,
  document.getElementById('root')
);*
```

# *通知；注意*

*   *如果您想关注我的博客系列的最新消息/文章，请[【观看】](https://github.com/n0ruSh/blogs/)订阅。*