# 构建 React 应用程序第 3 部分—组件生成

> 原文：<https://itnext.io/building-react-app-part-3-component-generation-455e77d7b1c6?source=collection_archive---------7----------------------->

如果你直接在这里登陆，请阅读第一部分和第二部分。

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fbuilding-react-app-part-3-component-generation-455e77d7b1c6%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

![](img/07fa028afb480edc2dd3e134eb2008a7.png)

[第 1 部分:为什么做出反应？](/building-react-app-part-1-why-react-1a4ecdaffd6e)

[第二部分:架构样板](/building-react-app-part-2-architecture-boilerplate-683b992089a6)

*对于像我一样绝对热爱 React 并一直致力于此的人，跳过第 1 部分直接跳到第 2 部分。*

在第 2 部分中，我们讨论了样板代码的体系结构，以及如何设置环境和代码体系结构，以便立即创建漂亮的、可伸缩的客户端应用程序。

这看起来很奇特，但是当我们讨论的时候，有一件事困扰着我。每次我需要给我的应用程序添加一个新的视图时，都要遵循一个繁琐的过程，包括创建许多文件，在基本配置文件中输入条目等等。让我们详细讨论一下..

# 如何添加另一个视图

它简单但麻烦。我们现在需要**订单**视图。

以下是步骤

1.  在视图目录中创建一个子目录`orders`。
2.  使用与 home 相似的映射创建`index.js`，定义状态，将其映射到 props。
3.  创建类似于家的`reducer.js`，这次将全局状态对象名称替换为`orders`
4.  创建`actions.js`并定义一个执行第一个交互任务的函数，例如:fetchOrders。
5.  创建`orders.js`并定义父布局，在挂载时执行 fetchOrder 动作等。
6.  创建`orders.css`并定义视图特定的样式

现在我们已经完成了文件和目录的创建。让我们将视图连接到我们的应用程序

**步骤 1:** 在`app/reducers.js`中创建一个名为 orders 的条目，并提供组件 reducer 对象。

```
**export const** reducers = combineReducers({
  ...
  orders: orderReducer
});
```

**第 2 步:**在`app/routes.js`中创建一个条目，将订单组件映射到`/orders`路线。

```
**export default** () => {
  **return** (
    <Switch>
      ...
      <Route exact path="/orders" component={Orders}/>
      ...
    </Switch>
  );
}
```

![](img/84a4a340362628817e1d141bf14db0cf.png)

恭喜，您的新视图现已连接，可以在`http://localhost:3000/orders`访问

> 可怜的……

就像我说的，简单但麻烦。即使对我来说，我也曾多次面对这个问题。考虑到这一点，我有了一个想法，为什么不自动化这个繁琐的过程，让生活变得更容易。

受 android 代码生成的启发，你可以通过 ide 菜单中的一个选项生成一个新的活动(视图),所有文件都被创建，条目在配置文件中生成。我甚至看到一些 npm 模块在做类似的工作，但是仍然有一些不足之处。

现在是我们之前离开的**发电机**目录。

# 发电机

这不是火箭科学。它的简单模板转换成样板文件，所以你不用花时间制作文件和条目。这里不需要解释。让我们直接进入它。尝试

```
yarn generate orders
```

见证奇迹的发生。这个简单的命令将完全执行我们之前添加视图的步骤。在视图内创建目录，创建基本文件，然后通过在减速器和路径中创建条目来连接视图。

![](img/be995c0878065c2ed49fe492d36fc09f.png)

不是那样的。如果您需要通过`/myorders`路径访问订单视图，该怎么办？尝试

```
yarn generate orders -r myorders
```

> 瞧啊。！再一次…

-r 代表路线。所以指挥结构

```
yarn generate <COMPONENT_NAME> -r <ROUTE_NAME>
```

路线名称是可选的。如果未指定，则将创建和映射与组件同名的路由。

这个系列到此结束。我希望这一切在某种程度上有所帮助。我已经用这个样板设置工作了一段时间，结果非常好。

链接到样板文件库。非常欢迎反馈和改进。

[](https://github.com/ankitsharma6466/react-boilerplate) [## ankitsharma 6466/react-样板文件

### 样板代码帮助您开始使用 ReactJs 开发高度优化的 PWA。

github.com](https://github.com/ankitsharma6466/react-boilerplate) 

如果你喜欢这个，并且认为我在这里做得很好，那么请不要忘记鼓掌。你的赞赏很有价值，让我有动力。

干杯！！