# 如何在 React 中进行 AJAX API 调用

> 原文：<https://itnext.io/how-to-make-ajax-api-calls-in-react-8ee7b8ee539d?source=collection_archive---------5----------------------->

![](img/7d6cadab144da06d53b471edc64339c3.png)

在本教程中，我们将介绍在 React 中使用 AJAX 的不同方法，以及在哪里调用 AJAX？`componentDidMount()`vs`componentWillMount()`vs ES6 类构造函数。

通过一个简单的例子，我们将了解如何通过不同的机制，如 **Axios** 库、 **XMLHttpRequest** 或现代浏览器的 **fetch** API，使用 React 发出 AJAX 请求或 API 调用(GET、POST、PUT 和 DELETE)来获取、创建、更新和删除数据。

React 是一个视图库，用于构建用户界面，而不是一个完整的框架，例如 Angular 或 AngularJS。在 MVC 架构中，React 代表了**V**view 部分。因此，如果你以前使用过客户端框架，你会注意到缺少许多抽象，比如进行 HTTP 调用的服务(相当于 AngularJS 中的 **$http** )。

所以——如果你问发送 AJAX 调用的反应相当于什么？没有！

你不应该认为这是这个库的弱点，因为 React 不应该处理通常由框架处理的所有任务。React 的全部目的是使用来自 **props** 和 **state** (通常从 API 服务器获取)的数据来呈现无状态组件(没有数据的转储组件)和有状态组件。

那么如何从远程 HTTP 服务器获取数据呢？或者如何进行 API 调用？

从外部库到浏览器 API，您有太多的选择。您所要做的就是根据您的需求选择合适的解决方案。

# 选择正确的 HTTP 调用机制

有许多具有不同特性的库，可以用来从远程服务器获取数据。

*   如果你觉得使用 JavaScript Promises 很舒服，你可以使用 Axios。
*   如果你喜欢使用前沿标准，你可以使用`fetch` api
*   您也可以使用 jQuery，但是不建议为了进行 API 调用而包含整个库。
*   可以使用 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest) 接口。

现在让我们看一个实际的例子，使用浏览器的 fetch api。

我们将创建一个简单的 React 应用程序，它向 Reddit 服务器发送 API 调用来获取一些 subreddit 帖子。

所以继续创建一个新的 React 项目。我正在使用和推荐 **create-react-app** ，因为它让你从配置 WebPack 的麻烦中解脱出来，让你快速生成一个启动项目来构建你的应用。

如果您决定这样做，您可以使用 npm 来安装 **create-react-app** ,其中包含:

```
npm install -g create-react-app
```

现在，您可以使用以下内容生成一个新的 React 项目:

```
create-react-app react-ajax-demo 
cd react-ajax-demo 
npm start
```

# 如何使用 Fetch API？

根据 [Mozilla MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

> *Fetch API 提供了一个 JavaScript 接口，用于访问和操作 HTTP 管道的各个部分，例如请求和响应。它还提供了一个全局 fetch()方法，该方法提供了一种简单、逻辑的方法来通过网络异步获取资源。*

现在让我们从 Reddit 获取一些数据。在`App.js`中，更新代码以反映这些变化。

```
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      posts: []
    };
  }  
  fetchFirst(url) {
    var that = this;
    if (url) {
      fetch('[https://www.reddit.com/r/](https://www.reddit.com/r/)' + url + '.json').then(function (response) {
        return response.json();
      }).then(function (result) {

        //console.log(result.data.children);

        that.setState({ posts: result.data.children, lastPostName: result.data.children[result.data.children.length - 1].data.name });

        console.log(that.state.posts);
      });
    }
  }  
  componentWillMount() {

      this.fetchFirst("reactjs");

  }    
  render() {

    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">React AJAX Example</h1>
        </header>
        <p className="App-intro">
          <ul>
            {this.state.posts.map(post =>
              <li key={post.id}>{post.title}</li>
            )}
          </ul>          
        </p>
      </div>
    );
  }
}

export default App;
```

就在`fetchFirst()`方法中，我们发送一个 GET 请求，使用下面的代码从`'https://www.reddit.com/r/' + url + '.json'`获取数据:

```
fetch('[https://www.reddit.com/r/](https://www.reddit.com/r/)' + url + '.json').then(function (response) {
        return response.json();
}).then(function (result) {
    console.log(result.data.children);
});
```

API 很简单，唯一需要的参数是资源的 URI。在我们的例子中，它是一个 JSON 文件，但也可以是任何类型的资源，如图像或其他类型。

如果没有额外的参数，默认情况下`fetch(url)`发送一个 **GET** HTTP 请求。

还可以通过在第二个参数中显式指定方法名来调用其他 HTTP 方法，如 POST、PUT 或 DELETE。例如，下面是我们如何用`fetch()`发送 POST 请求

```
var form = new FormData(document.getElementById('login-form'));
fetch("/login", {
  method: "POST",
  body: form
});
```

# 在哪里发出 AJAX 请求:componentWillMount vs componentDidMount？

React 应用程序是一组组件，包括根组件、父组件、顶级组件和子组件(以及子组件的子组件),它们可以被可视化为一个树形结构。React 社区中出现了许多问题——如何处理数据？数据如何从获取数据的组件流向其他组件？最重要的是，在哪里放置获取数据的代码？

在前面的例子中，我们调用了来自`componentWillMount()`生命周期事件的数据获取逻辑，当组件将要挂载时，在组件的第一次渲染之前调用。

有许多地方可以获取数据，例如:

*   component render()方法:这是个坏主意！您应该避免在组件的`render()`方法中放置任何异步代码，即改变应用程序状态或导致副作用的代码。
*   生命周期事件`componentDidMount()`:React 文档推荐的地点。
*   生命周期事件`componentWillMount()`:根据 React 文档，不推荐。
*   ES6 中组件的构造函数:它被认为是反模式！但是可以像`componentWillMount()`一样使用。

因此，让我们来看看获取数据时需要遵循的最佳实践。

# 正在获取 componentDidMount()事件中的数据

这是最简单的方法。我们在`componentDidMount()`而不是`componentWillMount()`从远程服务器获取数据，这样我们可以避免`fetch()` API 的任何副作用。

## componentDidMount()与 componentWillMount()

在组件第一次渲染之前调用了`componentWillMount()`方法。这可能会让任何人误以为这是获取数据的最佳地点。但事实并非如此！因为 fetch 调用是异步的，这意味着调用可能不会在组件呈现之前返回，从而导致组件呈现空数据。您可以采取一些预防措施，例如设置初始状态，以便组件在数据到达并触发另一个呈现周期之前能够正确呈现。

`componentDidMount()`方法在组件第一次渲染后被调用，所以这是你可以安全放置任何获取数据、有副作用或操作 DOM 的异步代码的地方。

*   当您采用这种方法时，您已经正确地设置了初始状态，以处理尚不可用的数据或未定义的状态。
*   对于服务器端渲染，`componentWillMount()`将被服务器调用两次，又被客户端调用一次，因此获取数据的请求将被发送两次，从而与服务器进行不必要的往返。另一方面，`componentDidMount()`在客户端只被调用一次。

我们的示例将更新为类似这样的内容:

```
componentDidMount() {
    this.fetchFirst("reactjs");
}
```

## componentWillMount() vs 构造函数

在 ES6 React 中，构造函数就在类被实例化之前被调用，所以问题是当使用 JavaScript 2015 (ES6)类时，你能放弃`componentWillMount()`而支持类构造函数吗？

显然，`componentWillMount()` lifecycle event 只有在您使用 React with old JavaScript(即 ES5) through，`React.createClass()`方法时才需要，因为您无法定义构造函数。因此，我们可以假设可以使用构造函数来代替装载后生命周期事件。但那不是 100%正确！因为如果在你的组件构造函数中有副作用，而不是在`componentWillMount()`事件中，React 会抛出警告。因此，如果您的代码需要更新其他组件中的状态，最好避免使用构造函数。我们之前也看到了 React 开发者不鼓励`componentWillMount()`vs`componentDidMount()`。最后一个结论:对任何不会产生副作用的初始化代码使用构造函数，比如状态更新，否则使用`componentDidMount()`。在 React repository 上甚至还有一个公开的问题，即[反对挂载后事件](https://github.com/facebook/react/issues/7671)。

# 通过 props 传递数据

另一种方法是将数据从父组件传递给子组件。但是当然父组件也应该避免前面的获取数据的方法，那么在哪里呢？父组件可以使用 React 路由器钩子(比如 onEnter()钩子)来获取数据，然后通过 **props** 将它们传递给子组件。

在我们的简单示例中，我们只有一个组件，但这可以很容易地分解为如下内容:

Fetch.js

```
export default {
fetchFirst: function(url){
    fetch('[https://www.reddit.com/r/](https://www.reddit.com/r/)' + url + '.json').then(function (response) {
            return response.json();
    }).then(function (result) {
         return result.data.children;   
    }); 
}  
}
```

RedditPost.js

```
import React from "react";

class RedditPost extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      title: props.title,
      id: props.id
    }
  }

  componentWillReceiveProps(nextProps) {
    this.setState(nextProps);
  }

  render() {
            <li key={this.state.id}>{this.state.title}</li>
      );
    }
  }
}

RedditPost.propTypes = {
  id: React.PropTypes.number,
  title: React.PropTypes.string
};

RedditPost.defaultProps = {
  id: null,
  title: ""

};

export default RedditPost;
```

App.js

```
import fetchFirst from "./Fetch";

import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      posts: []
    };
  }  

  componentDidMount() {

      fetchFirst("reactjs").then((posts)=>{
          this.state.posts = posts;
      });

  }    
  render() {

    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">React AJAX Example</h1>
        </header>
        <p className="App-intro">
          <ul>
            {this.state.posts.map(post =>
              <RedditPost {...post}/>
            )}
          </ul>          
        </p>
      </div>
    );
  }
}

export default App;
```

# 使用 Redux 状态管理器

第三种方法是使用像 Redux 这样的状态管理器。在这种情况下，可以从代码中的任何地方获取数据，然后存储在一个全局存储中，供所有其他组件使用。

# 结论

AJAX 允许获取数据，然后更新用户界面，而无需刷新页面。这是当前现代应用程序的要求。客户端框架结合了服务来轻松地进行 HTTP 调用，但是对于 React 这样的库，你需要依赖其他库，或者更好的是，使用浏览器标准:旧的接口 **XMLHttpRequest** 非常复杂(但是你可以将它封装在函数中或者使用 jQuery 这样的外部库)，或者现代浏览器的 **fetch** API(如果你想支持旧的浏览器，你可以使用 polyfills)。

*原载于*[*www.techiediaries.com*](https://www.techiediaries.com/react-ajax/)*。*