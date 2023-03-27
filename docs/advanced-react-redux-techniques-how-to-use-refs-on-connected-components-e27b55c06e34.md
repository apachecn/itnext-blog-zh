# 高级 React/Redux 技术|如何在连接的组件上使用参考

> 原文：<https://itnext.io/advanced-react-redux-techniques-how-to-use-refs-on-connected-components-e27b55c06e34?source=collection_archive---------2----------------------->

在本文中，我将带您了解如何在 React 组件上使用 refs，该组件使用 React-Redux API 连接到 Redux 存储。

但是首先，关于参考文献的一点背景…

![](img/3d004682a0d1510ae00665a5a3ca0387.png)

不…不是那种裁判。

## 什么是裁判？

如果你没有使用过引用，我强烈推荐你阅读一下 [React 文档](https://reactjs.org/docs/refs-and-the-dom.html)来理解它们是如何工作的以及它们的用途，但是简而言之，引用是一种存储对组件实例上的对象的*引用*的方式。

它们可以用来存储对 DOM 元素或另一个类组件实例的引用，所以让我们快速浏览一下这两个用例。

## 在 DOM 元素上使用引用

```
class VideoPlayer extends React.Component { constructor() {
    super();
    this.state = {
      isPlaying: false,
    }
    this.handleVideoClick = this.handleVideoClick.bind(this);
  }

  handleVideoClick() {
    if (this.state.isPlaying) {
      this.video.pause();
    }
    else {
      this.video.play();
    }
    this.setState({ isPlaying: !this.state.isPlaying });
  }

  render() {
    return (
      <div>
        <video
          ref={video => this.video = video}
          onClick={this.handleVideoClick}
        >
         <source
            src="some.url"
            type="video/ogg"
         />
        </video>
      </div>
    )
  }
}
```

在这个代码片段中，我创建了一个非常基本的视频播放器组件，每当您单击视频时，它就会播放和暂停视频。因为我需要使用 HTML5 video 元素上的 play()和 pause()方法来实现这一点，所以我在 component 实例上存储了一个对该元素的引用。这样，我可以根据需要在单击处理程序中访问这些方法。很漂亮，对吧？

## 在其他零部件实例上使用参照

有时，您会希望在另一个组件实例上使用 ref，而不是在 DOM 元素上使用 ref。我最近在 React 内部需要使用 [Highcharts](https://www.highcharts.com/) 时遇到了这个用例。根据他们网站上的 Highcharts 博客,你可以使用他们的库在 React 中创建图表:

```
class HighchartComponent extends React.Component {
  componentDidMount() {
    this.chart = new Highcharts[this.props.type || 'Chart'](
      this.props.container, 
      this.props.options
    );
  }

  componentWillUnmount() {
    this.chart.destroy();
  }

  render() {
    return (
      <div id={this.props.container}></div>
    )
  }
}
```

如您所见，由 Highcharts 生成的图表实例存储在 HighchartsComponent 实例中。但是，如果您想在这个组件之外访问图表(以及允许您操作图表的所有方法)，该怎么办呢？比方说，我们想要创建一个按钮，当点击它时，使用内置的 Highcharts API 来销毁图表。

这就是裁判可以派上用场的地方。

```
const highchartsOptions = {
  // ...highcharts options
};class Container extends React.Component {
  constructor() {
    super();
    this.handleButtonClick = this.handleButtonClick.bind(this);
  } handleButtonClick() {
    if (this.highchartsComponent.chart) {
      this.highchartsComponent.chart.destroy();
    }
  }

  render() {
    return (
      <div>
        <HighchartsComponent
          container="chart"
          options={highchartsOptions}
          ref={highchartsComponent =>
           this.highchartsComponent = highchartsComponent
          }
        />
        <button onClick={this.handleButtonClick}>
          Destroy Chart
        </button>
      </div>
    )
  }
}
```

在这个代码片段中，HighchartsComponent 上的 ref 允许我在父容器组件上存储一个对该已挂载组件实例的引用。这就是我通过调用按钮点击处理程序中的 destroy()方法来访问它的 chart 属性并利用 Highcharts API 的方法。

## 在连接的元件上使用参照

好吧…那么通过 React-Redux connect() API 连接到 Redux store 的组件呢？应该够简单了吧？

让我们继续将前一个示例中的 high charts 组件连接到一个假想的 Redux 商店。

```
connect()(HighchartsComponent);
```

然后…所有东西都坏掉了。😭

这不再起作用的原因是 connect()是一个更高阶的组件，它接受您提供给它的任何组件，并返回一个全新的组件。因此，highcartscomponent 上的 ref 现在指向连接的组件实例，而不是 highcartscomponent 实例。如果我们试图访问图表属性，它自然会返回未定义。

那么我们能做什么呢？幸运的是，React-Redux API 提供了一种获取包装组件实例的方法(在本例中，是我们的 HighchartsComponent)。您所要做的就是在 connect()调用中将 withRef 选项设置为 true:

```
connect(null, null, null, { withRef: true })(HighchartsComponent);
```

然后，您可以使用 getWrappedInstance()方法来…您猜对了…获取包装的实例。现在，我们之前的例子看起来像这样:

```
class Container extends React.Component {
  constructor() {
    super();
    this.handleButtonClick = this.handleButtonClick.bind(this);
  } handleButtonClick() {
    if (this.highchartsComponent.chart) {
      this.highchartsComponent.chart.destroy();
    }
  }

  render() {
    return (
      <div>
        <HighchartsComponent
          container="chart"
          options={chartOption}
          ref={connectedComponent =>
            this.highchartsComponent =
            connectedComponent.getWrappedInstance();
          }
        />
        <button onClick={this.handleButtonClick}>
          Destroy Chart
        </button>
      </div>
    )
  }
}
```

一切正常！🎉

**更新**:从`react-redux` > = v6.0 开始，`withRef`和`getWrappedInstance`被弃用，取而代之的是`forwardRef` [选项](https://react-redux.js.org/api/connect#forwardref-boolean)，这简化了事情，因为您不再需要调用 getWrappedInstance 来获得包装的组件实例。

![](img/c55590dcb69b2baff779fb1a8e2815e8.png)

完整代码，请查看 [JSfiddle](https://jsfiddle.net/jacobworrel/anqjxq95/30/) 。

1.  引用不适用于功能组件，因为它们没有实例。
2.  React 明确警告不要过度使用 refs，因为它们避开了典型的自顶向下的数据流，但是我发现它们在我需要访问第三方 API 的情况下特别有用。