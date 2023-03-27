# 使用 React 挂钩的响应背景图像🍍

> 原文：<https://itnext.io/responsive-background-images-using-react-hooks-941af365ea1f?source=collection_archive---------1----------------------->

*了解新的 React 效应——以及现实生活中的状态挂钩。*

处理全屏背景图像可能会很困难。我经常发现很少有全屏图像在横向和纵向上都很好看，所以我喜欢根据视口宽度使用不同的图像。通常我会让 CSS 中的媒体查询来处理网站的响应，但有时图像 URL 只在 JavaScript 中可用，例如，如果通过 Ajax 请求加载。

面对 React 应用程序中的这些问题，我决定尝试新的 [Hooks API](https://reactjs.org/docs/hooks-overview.html) 来解决它们。有了 React 钩子，我们既可以订阅视窗大小的变化，又可以根据屏幕的宽度设置合适的背景图像！🎉

在我的 React 应用程序中，我选择了一个适合桌面屏幕的背景图像和另一个适合移动设备的背景图像，按照这样的设计:

![](img/eb46f527fa703bbb62ed09f8cbf539cf.png)![](img/ca5b09f0bdf4e45988e2a375dc20c779.png)

窄屏幕使用适合纵向的图像，宽屏幕使用横向图像

下面是我的 React 组件。注意这是一个 f*functional 组件，*因为钩子不能与类组件一起工作。小于 650 像素的屏幕将使用适合移动设备的图像。更宽的屏幕将使用桌面图像。

我也有一些基本的 CSS 来使背景图像全屏显示并定位文本:

当组件第一次呈现时，即使没有挂钩，这也很好，但是如果我开始重新调整窗口大小，即使低于 650px 断点，图像 url 也不会改变。为了获得这个期望的副作用，我们需要通过使用[效果钩子](https://reactjs.org/docs/hooks-effect.html)将一个监听器连接到窗口的调整大小事件。

# 效果挂钩

效果挂钩`useEffect`，让我们在 React 功能组件中执行副作用。有了这个钩子，我只需要添加一次事件侦听器，它就会在每次组件呈现时运行。它基本上相当于一个类组件的`componentDidMount` 和`componentDidUpdate`生命周期方法的组合。下面是我的应用程序组件内部的效果:

```
useEffect(() => {        
  window.addEventListener('resize', handleWindowResize);         
});
```

我还想在组件卸载时删除事件侦听器。在类组件中，这将在`componentWillUnmount`方法中完成。有了效果钩子，我们可以将所有与副作用相关的代码放在一个地方——因为`useEffect`的返回值充当了一个清理方法。看起来是这样的:

```
useEffect(() => {        
  window.addEventListener('resize', handleWindowResize); return () => {            
    window.removeEventListener('resize', handleWindowResize);  
  }  
});
```

默认情况下，效果挂钩在组件每次挂载和更新时都会运行。但这在我的情况下是不必要的。为了解决这个问题，我们将使用第二个参数 useEffect，这是一个依赖关系数组。因为我对我的钩子没有依赖性，所以我将一个空数组作为第二个参数传递，这使得副作用只在组件装载和卸载时运行，这正是我们想要的:

```
useEffect(() => {        
  window.addEventListener('resize', handleWindowResize); return () => {            
    window.removeEventListener('resize', handleWindowResize);      
  }  
}, []); //empty array makes side effect only run on mount and unmount
```

# 州钩

在我的`handleWindowResize` 函数中，我现在想用新的值`window.innerWidth`更新组件的状态。但是打住，这是一个功能组件，那么不重构到一个类组件，怎么改变它的状态呢？答案是:与[状态挂钩。](https://reactjs.org/docs/hooks-state.html)

状态挂钩允许我们甚至在功能组件中使用组件状态。用`useState` 声明状态变量的方式如下:

```
const [windowWidth, setWindowWidth ] = useState(window.innerWidth);
```

数组中的第一项`windowWidth`，相当于类组件中的`this.state.windowWidth` 。第二项`setWindowWidth`，是 setter 函数，相当于类组件中的`setState({ windowWidth: window.innerWidth })`。`useState`的唯一参数是特定状态变量的初始值，在我的例子中是窗口的初始宽度。

最后，这就是`handleWindowResize`函数的样子，我在我的`useEffect`钩子内部声明了它。它调用 setter 函数来更新`windowWidth`状态:

```
useEffect(() => {
  const handleWindowResize = () => {
    setWindowWidth(window.innerWidth);
  } window.addEventListener('resize', handleWindowResize); return () => {            
    window.removeEventListener('resize', handleWindowResize);      
  }  
}, []);
```

***Edit*** *:在上面的代码部分中，看起来 setWindowWidth 是 useEffect 钩子的依赖项，因为它是在钩子外部声明的。虽然它确实在技术上是一个依赖项，并且我们可以在我们的依赖数组中指定它，但我们实际上并不需要这样做，因为 useState 中的值被 React 保证是静态的* [*。*](https://overreacted.io/a-complete-guide-to-useeffect/)

我的 React 组件现在看起来像这样，同时使用了状态挂钩和效果挂钩:

现在，每次调整窗口大小时，状态变量`windowWidth`都会更新！🎉

# 定制挂钩

应用程序中可能还有其他组件需要了解窗口宽度。为了避免在多个组件中编写相同的代码，我们可以提取处理副作用和有状态逻辑的代码，并创建一个[定制钩子](https://reactjs.org/docs/hooks-custom.html)。自定义钩子只是一个调用其他 React 钩子的 JavaScript 函数。根据 React 文档，函数的名称应该以“use”开头。我决定调用我的自定义钩子`useWindowWidth`,现在它开始工作了:

# 裁决

当我第一次看到 React Hooks 的文档时，我不确定它们在我的日常工作中会有多大用处，但是现在我已经开始在项目中引入它们，尤其是效果 Hook 变得非常有吸引力。我喜欢与一个副作用相关的所有代码都保存在同一个地方，而不是分散在不同的生命周期方法中。与创建具有相同逻辑的类组件相比，要编写的代码要少得多。语法对我来说还是有点怪，但我猜这只是偏好和习惯的问题。

希望随着钩子的使用越来越频繁，我会发现更多的好处和用例。如果没有，至少知道我们现在可以选择在 React 中处理状态和副作用时使用功能组件或类组件是件好事。