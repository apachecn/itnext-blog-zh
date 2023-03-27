# React 路由器:如何添加子路由

> 原文：<https://itnext.io/react-router-how-to-add-child-routes-62e23d1a0c5e?source=collection_archive---------3----------------------->

## 对于 react 路由器 4 及以上

![](img/4ce2e180f81d374eba0139d7f39ce175.png)

Ergyn Meshekran 在 [Unsplash](https://unsplash.com/search/photos/router?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

希望现在您已经熟悉了 react。但是，在构建单页 web 应用程序时，最重要的事情之一是理解路由。幸运的是，react 让路由变得非常简单。

当您第一次开始编写 react 应用程序时，您可能会从一堆这样的应用程序开始:

```
const NewComponent = ({shouldDisplay}) => {
  return shouldDisplay ? <div>HA!</div> : null
}
```

你可以为你的路由这么做，但是随着你的组件和应用变得越来越大，事情会变得越来越棘手。

大概可以说最好的路由库是 react-router。而不是深入研究 react-router。让我们谈谈如何让您的路由器使用子路由或子路由。如果你需要更多关于如何使用 react 路由器的信息，点击这里查看网站。

首先确保导入所需的库和组件。

```
import { 
   BrowserRouter as Router, 
   Link, 
   Route 
} 
from 'react-router-dom'
```

现在，我们首先需要的是一个组件来呈现我们的路线。

这里有一行有点复杂。在第 23 行，我使用了渲染而不是组件。这是因为我希望能够在那条路线上将道具传递到我的组件中。

我在这里的目标是有一个页面，呈现路线“/”上的着陆和一个呈现“/About”上的子路线的路线。

现在我们需要让组件着陆并四处走动。

我们会把它们做成非常简单的组件。

现在真的很简单了，如果你在 react 应用程序中运行这段代码，你会看到 route /Landing 将呈现一个包含内容的页面

```
This is a Landing Page
```

和路由/关于内容

```
About Page
```

因此，现在我们希望“关于”页面的内容有一些链接，这些链接将在页面上显示不同的内容，但也保留“关于”页面已经有的内容

我们通过使用 about 部分中 react 路由器的<route>和<link>组件来实现这一点。</route>

我将使用的另一个东西是来自 react 路由器的匹配道具。

下面是代码的样子

现在你的代码应该是完整的了。该应用程序应该呈现和关于页面的链接，以改变标题下的段落文本。

我希望你能学到一些东西！编码快乐！

如果你觉得这有帮助。给我留言或者在 [twitter](https://twitter.com/DuffinAvery) 上关注我。