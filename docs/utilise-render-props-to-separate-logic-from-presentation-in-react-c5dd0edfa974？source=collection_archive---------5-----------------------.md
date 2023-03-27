# 在 React 中利用渲染道具将逻辑与表示分离

> 原文：<https://itnext.io/utilise-render-props-to-separate-logic-from-presentation-in-react-c5dd0edfa974?source=collection_archive---------5----------------------->

![](img/8933e4fa3854e49902a096a7f2866351.png)

反应 JS

将逻辑从你的陈述中分离出来一直是一种最佳实践。它加强了[单一责任原则(SRP)](https://en.wikipedia.org/wiki/Single_responsibility_principle) ，并使代码更容易测试。

然而，React 因为让开发人员轻易地将这两个问题混为一谈而陷入麻烦。让我们举一个来自[主页](https://reactjs.org/)的例子来解释:

```
class Timer extends React.Component {
  constructor(props) {
    super(props)
    this.state = { seconds: 0 }
  } tick() {
    this.setState(prevState => ({ seconds: prevState.seconds + 1 }))
  } componentDidMount() {
    this.interval = setInterval(() => this.tick(), 1000)
  } componentWillUnmount() {
    clearInterval(this.interval)
  } render() {
    return <**div**>Seconds: {this.state.seconds}</**div**>
  }
}
```

上面你可以看到相当简单的`Timer`类。您还可以看到该类的显示逻辑被编码到了`render()`方法中。

在`Timer`类中有`<div>Seconds: {this.state.seconds}</div>`的问题是它没有将逻辑和表示分开。

让我们把这两个问题分开:

```
const SecondsActive = ({ seconds }) => 
  <**div**>Seconds: {seconds}</**div**>;class TimerContainer extends React.Component {
  state = { seconds: 0 } tick() {
    this.setState(({ seconds }) => ({ seconds: seconds + 1 }))
  } componentDidMount() {
    this.interval = setInterval(() => this.tick(), 1000)
  } componentWillUnmount() {
    clearInterval(this.interval)
  } render() {
    return this.props.view({ seconds: this.state.seconds })
  }
}ReactDOM.render(
  <**TimerContainer** view={SecondsActive} />,
  document.getElementById('app')
);
```

通过一个简单的切换到通过`prop`渲染，我们使得`Timer` ( `TimerContainer`)类可以在许多表示组件中重用，并且使得表示组件非常容易测试！

# 阅读更多关于渲染道具和反应

*   [JSX，一年中](https://gist.github.com/chantastic/fc9e3853464dffdb1e3c)
*   [表示和容器组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
*   [反应 JS](https://reactjs.org/)
*   [渲染道具](https://reactpatterns.com/#render-callback)