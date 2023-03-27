# 组织 redux 状态的更好方法

> 原文：<https://itnext.io/a-better-way-to-organize-redux-state-6a62dd79b2df?source=collection_archive---------1----------------------->

![](img/e0ef7242f4e43d4bc7d4be7ee5fd5975.png)

[公园巡警](https://unsplash.com/@parktroopers?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/future-structure?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍照

作为 Holloway[的一名技术主管，我从理论的角度思考了很多关于软件的问题，有时它可能是纯粹的哲学或核心代数。在软件工程面试中，数学问题经常被用来测试分析能力。但不仅仅是在面试中，你需要进行数学思考。我发现在写一行代码之前花几天时间思考和分析一个问题是更明智的做法。](https://www.holloway.com)

几年前，在 Stripe 的一次工程职位面试中，我收到了一个关于构建一个在超时后删除值(或使它们不可访问)的数据存储的问题。这是一个美丽的谜题，因为它可以通过许多方法来解决，大多数方法都可行，但只有少数方法是好的。

这个难题在我的记忆中找到了一个位置，在我不时回来反思的事情之间。

突然，我意识到能够为键值存储中的记录设置生存期(或者根本不删除它们)，改变了我们组织和处理 redux store state 之类的东西的方式。例如，我们可以忘记在存储状态下覆盖记录。

从事数据库工作的人可能不会理解我的兴奋。DB 里没有`overwrite`这种东西。你可以`write`、`update`或`delete`一次记录。但是 redux 状态有点不同。

首先是客户端，客户端的 app 都是有状态的，所以你需要有一个状态来描述要渲染哪些数据和实际要渲染的数据。当你需要为你的应用程序保存这样的数据时，Redux 非常方便。

一些随机的 todo 应用程序可能有这样的存储状态:

很明显，我们需要把所有的东西都放进一个商店里，来描述 app 目前可以渲染什么。并在需要时获取新数据，例如解析路由状态，或者加载数据以在弹出窗口中显示任务细节。

但是，如果需要在用户单击链接之前加载数据，或者如果我们需要缓存一段时间的数据，并减少在数据刚刚加载时的重新获取，该怎么办呢？

如果一个使用示例中状态的应用程序需要在后台获取额外的任务，它将导致立即呈现这些任务。这可能不是我们想要的行为。

## 分离数据和状态

我发现的关于 redux 状态的困惑是，它经常被用来在同一个地方同时存储数据记录和 app 的状态。如果我们想在 redux 中添加一条未来需求的记录，这意味着我们也要更新应用程序的状态。

这使得缓存或预取在没有黑客攻击的情况下几乎不可能。

最理想的解决方案可能是将数据和状态完全分离，并将数据记录存储在 indexedDB 中的某个地方，并实现类似于`indexedDBConnect`的东西来向 React 组件提供数据。或者发明其他方法。但这是一个更深入的话题*(顺便说一下，这里有一篇关于 indexedDB 的好文章:* [*如何使用 IndexedDB 构建渐进式 Web 应用*](/indexeddb-your-second-step-towards-progressive-web-apps-pwa-dcbcd6cc2076) *)* ，在这篇文章中，我想谈谈如何组织`redux` store 以更好的方式存储状态和数据。

我们需要将数据记录与应用程序的状态分开。虽然听起来很简单，但需要一些时间来吸收。我试着用一个例子来解释。

让我们看看 todo 应用程序的状态，看看什么是状态，什么是记录。

1.  *路线状态:*

```
route: { url: '/' }
```

`route`是一个州，没有记载。它说明了当前的 URL 是什么，仅此而已。

2.*用户状态:*

```
user: { id: "user-1", name: "Peter", },
```

`user`更有趣一点。它声明谁是当前用户以及用户模型有哪些字段。所以这里的状态是`currentUser`，记录是`user`对象。

分离的状态和记录如下所示:

```
users: { 
  current: "user-1", 
  "user-1": { id: "user-1", name: "Peter" },
}
```

3.*任务*:

```
tasks: {    
  list: [      
    {        
      id: "task-1", 
      title: "finish to-do app with a new framework",        
      state: "in-progress",      
     }, {        
      id: "task-2",        
      title: "write on medium about it",        
      state: "not-started",      
     },    
   ],  
}
```

这里的状态是任务 id 和任务顺序:

```
tasks: { 
  currentTasks: [
    "task-1",
    "task-2"
  ],

  "task-1": {        
    id: "task-1", 
    title: "finish to-do app with a new framework",        
    state: "in-progress",      
  }, 
  "task-2": {        
    id: "task-2",        
    title: "write on medium about it",        
    state: "not-started",      
  },
}
```

以及重组后的整个 redux 状态:

```
{
  route: { url: '/' }, users: { 
    current: "user-1", 
    "user-1": { id: "user-1", name: "Peter" },
  }, tasks: { 
    currentTasks: [
      "task-1",
      "task-2"
    ],

    "task-1": {        
      id: "task-1", 
      title: "finish to-do app with a new framework",        
      state: "in-progress",      
    }, 
    "task-2": {        
      id: "task-2",        
      title: "write on medium about it",        
      state: "not-started",      
    },
  },
}
```

状态和数据记录是分开的。我们可以根据需要预取尽可能多的任务记录，或者在不删除任务记录的情况下呈现任务的子集。我们不必担心 redux 中的数据过多，我们可以在超时之后删除值。取决于你从故事开始就解决谜题的方式。

## 结论

这在理论上看起来不错，实际上，在实践中也是如此。霍洛韦阅读器应用程序的 redux store 就是这样组织的。它允许为 popovers 特性同时渲染多条路线，或者在渲染指南之前预取大量 JSON 对象，而不用担心混乱的存储状态和许多其他事情，包括性能优化，但这是未来故事的主题。