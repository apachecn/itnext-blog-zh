# 用中介模式解耦 Vuex 模块

> 原文：<https://itnext.io/decouple-vuex-actions-with-the-mediator-pattern-58a8879de1b4?source=collection_archive---------1----------------------->

![](img/4b5ca2c4594dd412d17f0f07b8e6af18.png)

我更愿意称之为“导体”模式。我觉得更有说服力，更准确。指挥聆听管弦乐队的各个部分，并在适当的时候触发其他部分。调解人通过推回对立的力量来达成妥协。调解人的定义本身就包含冲突。我们的准则是和谐的，不冲突的☮.但是这种模式被称为[中介模式](https://en.wikipedia.org/wiki/Mediator_pattern)。为了避免混淆，让我们坚持这一点。

一旦应用程序包含足够多的特性，Vuex 模块将不可避免地进行交互。Vuex 模块有 3 种交互方式:

1.  一个动作从另一个模块调度一个动作。
2.  一个动作引发一个事件。另一个模块订阅该事件并调度一个动作。
3.  一个动作引发一个事件。中介订阅该事件。中介从另一个模块调度一个动作。

**选项 1 —“行动 A”派遣“行动 B”**

使用这个选项看起来像下面的例子。认证之后，我们需要获取一些数据。

```
...
authenticate ({ dispatch }, credentials) {
  return callAuthenticationEndpoint(credentials)
    .then(result => {
      commit('setToken', result.token)       
      return dispatch('service/fetchData')
        .then(() => result)
    })
}
```

在这三个选项中，我认为选项 1 是最常见的。这是最直接的方法，也是唯一不需要事件总线的方法。它最符合我们通常思考代码的方式，从其他函数调用函数。问题是“验证”操作并不关心您的数据需求。authenticate 动作没有做好一件事，而是在动作中产生了副作用。现在你对这个动作的测试也必须考虑到副作用。注意，我们还需要额外的代码来返回认证的结果，而不是获取的数据。错误处理呢？我们必须假设错误是在“fetchData”操作中捕获的，否则就在这里捕获它。

**选项 2 —模块监听模块**

```
// auth moduleauthenticate ({ dispatch }, credentials) {
  return callAuthenticationEndpoint(credentials)
    .then(result => {
      commit('setToken', result.token)       
      eventBus.emit('authenticated', result)
      return result
    })
}// service moduleinitialize ({ dispatch }) {
  eventBus.on('authenticated', () => dispatch('fetchData'))
}
fetchData () {
}
```

选项 2 稍微好一点，但是涉及到很多开销。您必须将事件总线导入每个模块，并从动作中发出一个事件。它还需要某种初始化动作，因为一旦我们在一个模块文件中，我们就不能访问 dispatch 方法，除非从另一个动作中。所以在代码的其他地方，需要调度这个初始化操作。

如果此时您在问，“如果我将事件监听器注册移动到另一个文件会怎么样”，那么您已经发现了中介模式，或者您已经知道了中介模式。

**选项 3 —中介模式**

```
// auth moduleauthenticate ({ dispatch }, credentials) {
  return callAuthenticationEndpoint(credentials)
    .then(result => {
      commit('setToken', result.token)       
      eventBus.emit('authenticated', result)
      return result
    })
}// service modulefetchData () {
}// mediator fileeventBus.on(
  'authenticated',
  () => store.dispatch('anotherModule/fetchData')
)
```

现在有了中介，模块不再需要初始化动作来注册事件监听器。从这个小例子中很难看出效果，但是我们已经分离了这些模块。服务模块仍将耦合到 auth 模块，因为它需要令牌来认证请求，但这是偶然的。通常，通过使用中介模式，您可以完全解耦您的模块。

仍然有额外的开销。模块之间是解耦的，但是每个模块都耦合到事件总线，所以如果你能够在另一个应用中重用一个模块，你也需要事件总线。我会说这没问题，因为它强制执行了一个有用的模式，但是在 Vuex 中，我们可以通过使用 Vuex 作为事件总线来拥有我们的蛋糕并吃掉它。

**使用 Vuex 作为事件总线**

Vuex 的 API 包括:

1.  store.subscribeAction 监听操作
2.  store.subscribe —倾听突变

这些 API 不允许您指定想要监听哪个动作。相反，您需要在处理程序中设置一个 switch 语句，根据动作类型进行分支。

```
// store-moderator.jsexport default function configureModerator (store, router) {
  // listen to mutations
  store.subscribe(({ type, payload }, state) => {
    switch (type) {
      case 'auth/setToken':
        return store.dispatch('service/fetchData')
    }
  }) // listen to actions
  // note: doesn't not wait for the result of async actions
  store.subscribeAction(({ type, payload }, state) => {
    switch (type) {
      case 'auth/signOut': return router.push('/auth/signin')
    }
  })
}
```

当我可以的时候，我试着去听突变而不是行动。通常，一个动作会导致一个突变。通过听突变，你可以知道这个动作什么时候结束。如果你监听这个动作，在这个动作被调用之后，在它有机会完成之前，这个处理程序会被调用。

现在模块内部不需要事件总线，因为 Vuex 是事件总线。如果我们看看我们的模块，它们彼此完全解耦，也从外部事件总线解耦。

```
// auth moduleauthenticate ({ dispatch }, credentials) {
  return callAuthenticationEndpoint(credentials)
    .then(result => {
      commit('setToken', result.token)
      return result
    })
}// service modulefetchData () {
}
```