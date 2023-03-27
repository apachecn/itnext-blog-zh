# 可重用组件的规则

> 原文：<https://itnext.io/the-rules-of-reusable-components-2b221c138bcf?source=collection_archive---------1----------------------->

![](img/ac30e75f867852a7efeb1da755f50298.png)

我花了很多时间开发新的可重用 UI 组件。

这意味着在开发过程中，我倾向于提取放在共享库或样式指南中的 UI 组件。

这些组件将在整个项目中重复使用。当集成到功能中时，通常具有不同的样式或布局。

一路上，我学到了一些帮助我节省时间和痛苦的规则。

## 1.保持布局流畅

我数不清一个组件有多少次是固定宽度的，这影响了一个响应式的设计。这意味着我必须进去摆弄现有的道具，这些道具是为网站中的另一个功能设置的。不太好。

通过保留组件`100% width`,您将布局责任传递给了父组件。

例如，使用`CardOne`比使用`CardTwo`要容易得多。因为`CardOne`会在屏幕尺寸变化时响应父列的变化。

```
const CardOne = ({ title }) => (
  <div style={{ padding: "20px 30px", background: "white" }}>{title}</div>
);

const CardTwo = ({ title }) => (
  <div style={{ width: 600, padding: "10px", background: "white" }}>
    {title}
  </div>
);

const App = () => (
  <div className="row">
    <div className="col-md-6">
      <CardOne title="I'm Fluid" />
    </div>
    <div className="col-md-6">
      <CardTwo title="I will break this layout" />
    </div>
  </div>
);
```

## 2.允许传递额外的道具

原来，您的按钮组件需要一个特定的数据属性来与您正在使用的库一起工作，以实现一个特定的特性。可惜`Button`组件只允许`className`、`children`和`onClick`。

我们如何解决这个问题？

[现代 ES6 JavaScript 允许你传播函数参数和对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)。这可以用来为组件提供道具。

```
// Before -> <button className="button">Click me</button>

const Button = ({ children, className, onClick }) => (
  <button onClick={onClick} className={`${styles.button} ${className || ""}`}>
    {children}
  </button>
);

// After -> <button className="button" data-theme="dark">Click me</button>

const Button = ({ className, ...props }) => (
  <button className={`${styles.button} ${className || ""}`} {...props} />
);

// Example

const App = ({ onActivate }) => (
  <SpecificFeature>
    <Button data-theme="dark" onClick={onActivate}>
      Click me
    </Button>
  </SpecificFeature>
);
```

# 3.把逻辑往上推/保持沉默

阻碍可重用性的最大问题之一是特定于某个特性的有状态逻辑是在可重用组件中实现的。

假设你有一个`Dropdown`组件。点击链接时在导航中使用的。

```
const DropdownItem = ({ className, ...props }) => (
  <li className={`dropdown-item ${className || ""}`} {...props} />
);class Dropdown extends React.Component {
  state = { toggled: false }; toggle = () => this.setState(state => ({ toggled: !state.toggled })); render() {
    return (
      <div className="navbar-link">
        <span onClick={this.toggle}> {this.props.title} </span>
        {this.state.toggled && (
          <ul className="dropdown"> {this.props.children} </ul>
        )}
      </div>
    );
  }
}const Navbar = () => (
  <div className="navbar">
    <div className="navbar-brand">Example</div>
    <ul className="navbar-menu">
      <Dropdown title="My Account">
        <DropdownItem>One</DropdownItem>
        <DropdownItem>Two</DropdownItem>
      </Dropdown>
    </ul>
  </div>
);
```

上面的问题是:

1.  `Dropdown`处理打开和关闭下拉菜单的逻辑。这使得在更复杂的场景中重用`Dropdown`变得更加困难(例如:只有当请求成功时才打开下拉菜单)。
2.  `Dropdown`具有与`Navbar`组件相关的标记。

如果我们重构`Dropdown`组件来提升逻辑。我们最终得到一个更加可重用的`Dropdown`组件。

```
const DropdownItem = ({ className, ...props }) => (
  <li className={`dropdown-item ${className || ""}`} {...props} />
);const Dropdown = ({ className, ...props }) => (
  <ul className={`dropdown ${className || ""}`} {...props} />
);class Navbar extends React.Component {
  state = { showUserOptions: false }; toggleOptions = () =>
    this.setState(state => ({
      showUserOptions: !state.showUserOptions
    })); render() {
    return (
      <div className="navbar">
        <div className="navbar-brand">Example</div>
        <ul className="navbar-menu">
          <div className="navbar-link">
            <span onClick={this.toggleOptions}>My Account</span>
            {this.state.showUserOptions && (
              <Dropdown>
                <DropdownItem>One</DropdownItem>
                <DropdownItem>Two</DropdownItem>
              </Dropdown>
            )}
          </div>
        </ul>
      </div>
    );
  }
}
```

# 摘要

*   保持组件布局流畅。
*   允许额外的道具传入组件。
*   尽可能地让逻辑远离它们。