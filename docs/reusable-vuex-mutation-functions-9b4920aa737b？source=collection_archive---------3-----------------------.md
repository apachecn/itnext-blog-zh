# 可重复使用的 Vuex 变异函数

> 原文：<https://itnext.io/reusable-vuex-mutation-functions-9b4920aa737b?source=collection_archive---------3----------------------->

![](img/56ecd7a4a55e1c1545b5e5cc5de18053.png)

[照片由泽维尔·罗宾](https://flic.kr/p/T5HHt7)在 Flickr 上拍摄

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Freusable-vuex-mutation-functions-9b4920aa737b%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

## 更新

下面描述的`vuex`助手函数模式现在作为一个 npm 模块可用， [vuex-intern](https://gitlab.com/rhythnic/vuex-intern) 。

## 介绍

[Vue.js](https://vuejs.org) 是我这些天去的前端框架。比起 React 组件模型，我并不太喜欢组件模型，但是 Vue 的中央状态管理工具 Vuex 与 Vue 组件的集成带来了非常好的开发者体验。这篇文章是关于我如何在 Vuex 中进行突变的。

这篇文章对那些熟悉 Vuex 的人来说更有意义，但对那些不熟悉的人来说，Vuex 中的突变是一个改变状态的函数。变异函数接收状态和要进入状态的值。一般来说，这些都是简单的函数。我开始注意到它们中的很多看起来都是一样的，比如设置一个值，或者向列表中添加一个条目，所以我写了一些状态变异函数，可以用区分这些相似函数的信息来配置(定制)它们。

# 例子

其中最简单的叫做‘集’。它只是设置了 state 的根级别属性。

第一个例子是没有状态变异函数时的样子。

```
const state = {
  prop1: null,
  prop2: null
}const mutations = {
  setProp1 (state, val) {
    state.prop1 = val
  },
  setProp2 (state, val) {
    state.prop2 = val
  }
}
```

让我们重构使用一个状态赋值函数。

```
const set = key => (state, val) => {
  state[key] = val
}const mutations = {
  setProp1: set('prop1'),
  setProp2: set('prop2')
}
```

我们已经减少了一些乏味的样板代码，同时不再需要测试我们的变异函数(假设使用的状态变异函数已经过测试)。

## 使用状态变异器的示例 Vuex 模块

这里有一个例子，包括我经常使用的变异函数。实现在下面的例子中。

```
const state = {
  primitive: null,
  bool: false,
  list: [],
  users: []
}const getters = {
  // get the user by their id
  userById: findByKey('users', '_id')
}const mutations = {
  setPrimative: set('primative'),
  toggleBool: toggle('bool'),
  pushToList: pushTo('list'),
  replaceUser: replaceRecordInList('users', '_id')
}const actions = {
  ...
  fetchUser ({ commit }, userId) {
    fetch(`https://example.com/users/${userId}`)
      .then(res => res.json())
      .then(user => commit('replaceUser', user))
  }
}
```

## 履行

这些`vuex`助手函数被收集在一个名为 [vuex-intern](https://gitlab.com/rhythnic/vuex-intern) 的 npm 模块中。请到那里查看帮助器函数的完整列表和实现，并导入模块使用它们。