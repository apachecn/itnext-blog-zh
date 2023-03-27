# 用放大和反应挂钩创建可重用的抽象

> 原文：<https://itnext.io/creating-reusable-abstractions-with-amplify-and-react-hooks-97784c8b5c2a?source=collection_archive---------1----------------------->

![](img/0abfd909a610f5e17079a46676176f85.png)

Amplify 可能是创建应用程序原型的最佳库之一。随着规模的扩大，您发现自己在很多地方都需要它的模块，因此您直接在大多数组件中导入它们。与此同时，React 开发人员构建代码的方式发生了有趣的转变。随着钩子的引入，核心概念，如将组件分为表示和容器，正在失去优势，而一种更全面的方法开始接管，其中容器和表示部分被集成到一个单一的实体中。

本文将利用这一新趋势和定制挂钩的概念，以便在 React 和外部服务之间就它们相互通信的方式划出一条界限。换句话说，*你的 React 组件真的需要知道你在使用 Amplify 吗？*

# 设置场景

我见过的大多数关于 Amplify 和 React 的教程，一般都是直接在 React 组件中导入`Auth` & `API`这样的模块。这类似于直接在组件中编写数据获取逻辑，通过导入正在使用的 HTTP 库(即`axios`)并在生命周期方法(或`useEffect()`钩子)中执行请求。

尽管这个绝对没有任何问题，但有人可能会认为这可能会在未来造成一些小问题。你看，随着应用程序的增长，业务需求、代码的复杂性以及——很可能——为其做出贡献的人也在增长。将数据提取层抽象出来可以带来很多好处，比如减少样板文件、提高可读性、将 API 相关的逻辑集中在一个地方，以及帮助其他开发人员(以及未来的你)保持应用程序结构的一致性。

同样的原理也适用于 Amplify 的模块。我们以`Auth`模块为例。许多组件将需要访问当前经过身份验证的用户，这又转化为在这些组件中调用`Auth.currentAuthenticatedUser`。如果我们能够抽象出`Auth`模块的实现方式，并公开一个自定义接口来隐藏我们在幕后使用 Amplify 的事实，会怎么样？我说的是[外观模式](https://en.wikipedia.org/wiki/Facade_pattern)的一个简单实现。虽然有很多方法可以实现这一点，但我认为一个很好的解决方案是通过 React 钩子，因为它们最大的卖点之一是代码可读性的降低。考虑到这一点，让我们直入主题。

# 用钩子屏蔽放大器的授权模块

为了举例说明上面所描述的内容，我将尝试通过创建一个钩子来屏蔽 Amplify 的`Auth`模块的扩展公共接口，该钩子将只暴露一个项目子集，供我们的 React 组件与之交互。如前所述，这里的目标是添加一个层，只暴露我们的应用程序需要什么，而不让 React 知道**它是如何在幕后实现的。**

我们首先需要将用户实例存储在某个地方。为此，我们将通过以下方式创建一个环境:

```
import React from 'react';// originally the user is going to be `null`
const UserContext = React.createContext(null);
```

之后，我们需要一种向应用程序提供该值的方式，这通常通过`UserContext.Provider`组件来处理。但是我们缺少的是在应用程序加载后立即检查用户是否已经通过身份验证。为了解决这个问题，我们将为`UserContext.Provider`创建一个自定义包装器，作为它的“控制器”，而不是向现有的顶级 React 组件添加 api 调用。该组件将安装在树的顶部，并执行所有需要的副作用:

```
import React from 'react';/* The UserContext was previously defined */const UserProvider = (props) => {  
  const [user, setUser] = React.useState(null);     // fetch the info of the user that may be already logged in
  React.useEffect(() => {   
    Auth.currentAuthenticatedUser()
     .then(user => setUser(user))      
     .catch(() => setUser(null));  
  }, []); // make sure other components can read this value
  return (
    <UserContext.Provider value={user}>
      {props.children}
    </UserContext.Provider>
  )
}
```

为了读取这个值，其他组件需要访问它。让我们通过一个自定义挂钩来实现这一点:

```
import React from 'react';/* The UserContext & UserProvider were previously defined */const useUser = () => {  
  const context = React.useContext(UserContext); if(context === undefined) {    
    throw new Error('`useUser` must be within a `UserProvider`');  
  }  
  return context;
};
```

我们缺少的最后一点是认证功能。我们的组件可以访问用户，但不能登录或注销用户，因为它们实际上不知道如何登录或注销用户。让我们通过扩展我们先前定义的`UserProvider`来解决这个问题，这样它不仅返回`user`实例，还返回一组与之相关的附加函数:

```
import React from 'react';const UserProvider = (props) => {  

  ...
  ... const login = (usernameOrEmail, password) =>
   Auth.signIn(usernameOrEmail, password)
     .then(cognitoUser => setUser(cognitoUser)); const logout = () => 
   Auth.signOut()
     .then(() => setUser(null)); return (
    <UserContext.Provider value={{ user, login, logout }}>
      {props.children}
    </UserContext.Provider>
  )
}
```

现在，我们的 React 组件将可以访问*登录* & *注销*功能。如果我们将上述所有内容组合在一起，并添加一些额外的优化，最终的代码将类似于以下内容(请务必阅读注释):

“使用用户”挂钩的实现，

在分析我们从中得到了什么之前，我想展示一下这些模块是如何被使用的会很好。看看下面的一些例子:

如何使用“useUser”挂钩的示例

我们在这里实现的是成功地将 Amplify 与 React 解耦，方法是将 Amplify 的扩展认证 api 映射到简单的自定义函数，并且不依赖于 Amplify 内部实现它们的方式。他们的公共接口将保持不变，即使 Amplify 的 api 发生变化，这意味着它会发布一个突破性的变化，你只需要在一个地方更新代码。此外，如果您决定 Cognito 不适合您，您可以放心地用类似于`Auth0`的东西替换它，而不会有太多麻烦，因为(再次)您只是在一个文件中编辑代码。这也意味着其他开发人员不需要知道或理解身份验证是如何工作的，这使得他们的加入速度更快。编写测试也变得更容易，因为我们正在执行的依赖注入的这种变化(通过我们的定制 auth 接口)，使得编写不受外部包变化影响的快速&弹性模拟变得容易。这意味着您不必模仿 Amplify 本身，而是模仿您公开的定制认证接口。

另一方面，使用钩子可以让你快速编写和替换组件，减少组件代码的数量，增加可维护性和可读性，并允许项目间的重用。后者可能是最重要的好处之一，因为您现在可以拥有一个可重用的认证接口，可以在您的所有项目和团队之间共享。我们的定制`useUser`钩子还允许您在将公开的函数提供给消费者之前向它们添加额外的逻辑，所以不要将这个钩子看作是一个简单的`useContext`调用的包装器，而是一个由提供者持有的状态选择器。

尽管我在这篇文章中试图保持简单，但是根据您的需要，您也可以扩展这个解决方案，以包括其他方法，如`signUp`、`updateUser`、`deactivateUser`等。只要确保它小而简单。如果它变得太麻烦而无法维护，您总是可以将逻辑分成多个钩子，以便使它更加模块化。例如，您可以用一个`useUser`钩子来处理与用户相关的内容，用一个`useAuth`钩子来处理与认证相关的任何事情(在这种情况下，您还需要创建两个不同的提供者)。

# 结束语

这篇文章的目的是看你如何创建抽象，这些抽象不仅可以在不同的项目中重用，也可以在不同的模块中重用。您可以轻松扩展这个解决方案，并将其用于像`Storage`或`Analytics`这样的模块。像这样的抽象增加了代码的模块化可读性，同时使维护、测试和代码互换变得轻而易举，几乎没有额外的开销。到目前为止，它对我来说效果很好，我很想听听你的想法。

干杯:)

***附言*** 👋 ***嗨，我是***[***Aggelos***](https://aggelosarvanitakis.me)***！如果你喜欢这个，考虑一下*** [***在 twitter 上关注我***](https://twitter.com/AggArvanitakis) ***并与你的开发者朋友分享这个故事*😀**