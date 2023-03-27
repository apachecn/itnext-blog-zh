# 使用 Vuex 存储并在 TypeScript 中键入:与 Vue 3 组合 API 兼容的解决方案

> 原文：<https://itnext.io/use-a-vuex-store-with-typing-in-typescript-without-decorators-or-boilerplate-57732d175ff3?source=collection_archive---------1----------------------->

![](img/d14977ed5cc15cc3f5825dfac4dddaaf.png)

感谢[好的免费照片](https://www.goodfreephotos.com/south-africa/cape-town/long-beach-landscape-in-capetown-south-africa.jpg.php)。

我发现了一种在 TypeScript 中使用 Vuex 存储而不丢失键入的方法。它不需要类，所以它兼容 Vue 3 和 composition API。

# 使用完全类型化的轻量级包装器

在运行时，库 direct-vuex 将在每个 getters、突变和动作周围创建一个包装器。我们可以在商店实现之外使用它们。

所以与其写:

```
store.dispatch("myAction", myPayload);
```

…我们写道:

```
store.dispatch.myAction(myPayload);
```

或者，不写:

```
store.dispatch("myModule/myAction", myPayload);
```

…我们写道:

```
store.dispatch.myModule.myAction(myPayload);
```

Getters 和 mutations 的访问方式相同:

```
store.getters.myModule.myGetter;
store.commit.myModule.myMutation(myPayload); 
```

TypeScript 可以正确推断这些包装的类型。

# 如何创建直营店

商店可以像往常一样实现。这里有一个模块:

```
*// module1.ts*export interface Module1State {
  name: null | string
}export default {
  namespaced: true as true,
  state: {
    name: null
  } as Module1State,
  getters: {
    message: state => `Hello, ${state.name}!`
  },
  mutations: {
    SET_NAME(state, newName: string) {
      state.name = newName
    },
  },
  actions: {
    async loadName({ commit }, payload: { id: string }) {
      const name = `Name-${payload.id}` *// load it from somewhere*
      commit("SET_NAME", name)
      return { name }
    },
  }
}
```

无论如何，一个小细节:当我们用老方法编写商店代码时，唯一的一件小事是必须声明`namespaced: true as true`。Direct-vuex 需要知道`namespaced`的字面类型(类型`true`或者`false`而不是`boolean`)。

现在，我们可以使用 direct-vuex 创建商店。

首先，将 direct-vuex 添加到 Vue 应用程序:

```
npm install direct-vuex
```

然后，创建商店:

```
*// store.ts* import Vue from "vue"
import Vuex from "vuex"
import { createDirectStore } from "direct-vuex"
import module1 from "./module1"Vue.use(Vuex)const { store, rootActionContext, moduleActionContext } = createDirectStore({
  modules: {
    module1
  }
})*// Export the direct-store instead of the classic Vuex store.*
export default store*// The following exports will be used to enable types in the*
*// implementation of actions.*
export { rootActionContext, moduleActionContext }*// The following lines enable types in the injected store '$store'.*
export type AppStore = typeof store
declare module "vuex" {
  interface Store<S> {
    direct: AppStore
  }
}
```

与您通常的代码相比，`new Vuex.Store(options)`被替换为`createDirectStore(options)`。事实上，`createDirectStore()`函数只是简单地为您调用`new Vuex.Store()`，然后返回一个带有包装器的直接商店和一个围绕经典 Vuex 商店的良好类型。

然后，我们从模块`vuex`中用我们的良好类型的直接存储来扩充类型`Store`。

就是这样！

direct-vuex 提供的存储已经包含了所有类型良好的状态、getters、突变和动作。

# 使用直营店

经典的 Vuex 商店仍然可以通过`store.original`属性访问。我们需要它来初始化 Vue 应用程序:

```
import Vue from "vue"
import store from "./store"
import App from "./App.vue"new Vue({
  store: store.original, *// Inject the classic Vuex store*
  render: h => h(App),
}).$mount("#app")
```

注入的店还是经典的 Vuex 店。但是它有一个`direct`属性，包含我们的良好类型存储。我们可以使用它，或者直接在每个需要它的组件中导入存储:

```
import store from "./store";
// or: const store = this.$store.direct;async function test() {
  store.state.otherModule *// Error.* store.state.module1.name *// Ok, type is 'string | null'.*
  store.state.module1.otherProp *// Error.* store.getters.module1.message *// Ok, type is 'string'.*
  store.getters.module1.otherProp *// Error.* store.commit.module1.SET_NAME("abc") *// Ok.*
  store.commit.module1.SET_NAME(123) *// Error.*
  store.commit.module1.OTHER_PROP(123) *// Error.* const result = await store.dispatch.module1.loadName({ id: "12" })
                 *// Ok, result type is: '{ name: string }'*
  store.dispatch.module1.loadName({ otherPayload: "12" }) *// Error*
}
```

如你所见，不再有`any`。整个存储区的类型都是正确的，这得益于 IDE 中的自动完成功能。

注意，direct-vuex 不是 vuex 的重写，而只是一个非常轻量级的包装器。所以，你所知道的关于 Vuex 的一切仍然适用，因为它仍然是 Vuex 在地下工作。如果您愿意，您可以通过注入的`this.$store`或`store.original`同时使用底层的 Vuex 存储。

# 使用 Direct-vuex 编写商店的实现

这是[的文件](https://github.com/paleo/direct-vuex#implement-a-vuex-store-with-typed-helpers)。

希望这个小图书馆能有所帮助。