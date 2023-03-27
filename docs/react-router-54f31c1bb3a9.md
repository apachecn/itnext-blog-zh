# React Router:一个用于多页面导航的 React 库。

> 原文：<https://itnext.io/react-router-54f31c1bb3a9?source=collection_archive---------0----------------------->

![](img/0d0c7058d7cb2c19c5b784d084300d03.png)

费伦茨·阿尔马西在 [Unsplash](https://unsplash.com/s/photos/react-js?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

根据 [React 路由器文档](https://reactrouter.com)，React 路由器是一组导航组件的集合，它们以声明的方式与您的应用程序组合在一起。

在本文中，我们将学习如何使用 React Router(版本 5)在 React 应用程序中呈现多个组件(页面),就像任何多页面应用程序一样。但是首先，让我们讨论一些关于 React 的事情。

# React 如何呈现页面

React 作为一个单页应用程序工作。创建一个 React 应用程序后，看看公共文件夹，你会看到一个 index.html 文件。在文件的内容中有一个 id 为“root”的

。这个根来自我们的根组件，默认情况下是 src 文件夹中的 App.js 组件(您可以检查 src 文件夹中的 index.js 文件，以查看 App.js 组件是如何被设置为根组件的)。因此 React 所做的是查看 App.js 组件内部，然后将它在那里看到的任何内容呈现给 DOM。

# 安装 React 路由器

> 在安装 React 路由器之前，您应该在您的计算机上安装 Node.js，以便我们可以使用 npm。可以在 [Node.js 网站下载最新版本。](https://nodejs.org)

现在要安装 React 路由器，打开项目文件夹 中的终端 ***并运行下面的命令:***

**npm 安装 react-router-dom**

这将安装 React 路由器使用的所有软件包。安装完成后，我们就可以开始了。

# 创建多个组件

在我们开始之前，让我们创建三个组件:主页，关于和联系组件。

主页组件:

```
function Home() {
    return (
        <div>
            <h1>This is the homepage component</h1>
        </div>
    )
}

export default Home
```

关于组件:

```
function About() {
    return (
        <div>
            <h1>This is the About page component</h1>
        </div>
    )
}

export default About
```

接触组件:

```
function Contact() {
    return (
        <div>
            <h1>This is the Contact us page component</h1>
        </div>
    )
}

export default Contact
```

请注意，这不是充当根组件的 App.js 组件。我们将使用根组件来创建我们所有的路由。

# 导入特征和组件

让我们首先从 react-router-dom 导入一些我们将使用的特性，并导入我们所有的组件。所有这些都将在根组件中完成。

```
import {BrowserRouter as Router, Route, Switch, Link} from 'react-router-dom'; 
import Home from './Home';
import About from './About';
import Contact from './Contact';

function App() {
  return (
    <Router>
      <div className="App">

      </div>
    </Router>
  );
}

export default App;
```

这里有四点需要说明: **BrowserRouter、Route、Switch** 和 **Link。**

**浏览器路由器**

从上面的代码片段可以看出， **BrowserRouter** 被导入为**路由器**；如果你理解模块在 JavaScript 中是如何工作的，那么你就会知道那里发生了什么。您可以查看带下划线的文章，以更好地了解 [ES6 模块和应用](https://medium.com/codex/es6-modules-and-application-34a40365cd43?source=your_stories_page-------------------------------------)。现在，其他所有内容都将嵌套在<路由器>T21/路由器>中。

**路线**

Route 将用于嵌套我们想要导航到的每个组件，还包括组件的路径/URL。

**开关**

Switch 很容易理解。它只是让 React 一次渲染一个组件。简单吧？请注意，将为每个组件渲染放置在开关之前的任何组件。当你有一个用来放置链接的导航栏时，这是必要的。

**链接**

链接类似于 HTML 锚标记()。它用来告诉你点击的元素将会指向哪里。当您单击一个链接元素时，它将查找具有相应路径的组件，然后将用户导航到该组件/页面。这样做的好处是，当您请求另一个页面时，您的页面不会重新加载，因此您的网页会以令人难以置信的速度提供服务。不准重装！

现在，让我们使用这些功能并连接所有组件。

# 使用 React 路由器

```
import {BrowserRouter as Router, Route, Switch, Link} from 'react-router-dom'; 
import Home from './Home';
import About from './About';
import Contact from './Contact';

function App() {
  return (
    <Router>
      <div className="App">
         <Switch>

            <Route exact path="/">
              <Home/>
            </Route>

            <Route exact path="/About">
              <About/>
            </Route>

            <Route exact path="/Contact">
              <Contact/>
            </Route>

          </Switch>
      </div>
    </Router>
  );
}

export default App;
```

上面的代码片段总结了该片段之前解释的所有内容。组件嵌套在中，并给出一个路径。如果您注意到，有一个额外的属性，它就是**精确的**属性。精确属性有助于反应与路线中提供的精确路径相匹配的渲染路径。如果没有包含确切的属性，那么当你点击一个链接元素时，它会查看你的所有路径，并呈现它看到的第一个匹配。所以如果你有两个路径，“/create”和“createMore”，前者总是会被渲染；当你点击后者时，React 会查看两个路径，并假设“createMore”中的“create”与“create”相同。使用 **exact，**它必须在渲染前查看完整的路径名。

Home 组件的路径也不同于带有“/”的其余组件。第一次加载 React 应用程序或网站时，任何路径为斜线的路径都将首先被渲染。我们将使用 Home 组件创建链接，将用户引导到其他组件。

```
import { Link } from 'react-router-dom';

function Home() {
    return (
        <div>
            <h1>This is the homepage component</h1>

            <Link to="/About">Click me to visit the About page</Link>

            <Link to="/Contact">Click me to visit our Contact page</Link>
        </div>
    )
}

export default Home
```

如上所述，当浏览器第一次加载时，将呈现 Home 组件。现在我已经在 Home 组件中包含了链接，允许用户访问其他组件。不要忘记从 react-router-dom 导入链接。“to”属性指定了一个路径，如果该路径匹配任何一个<route>路径，那么嵌套在该路径中的组件将被渲染到 DOM 中。</route>

这只是 React 路由器的基础，还有更多的功能，你可以查看[文档](https://reactrouter.com/)。

我有更多的教育性文章给你。关注我并订阅我的时事通讯，以便在我发表文章时将文章发送到您的电子邮件中。您可以在评论框中提出您的问题和建议。感谢您的阅读。