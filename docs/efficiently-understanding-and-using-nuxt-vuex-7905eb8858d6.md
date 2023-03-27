# 高效理解和使用 Nuxt + Vuex

> 原文：<https://itnext.io/efficiently-understanding-and-using-nuxt-vuex-7905eb8858d6?source=collection_archive---------0----------------------->

[Nuxt.js](http://nuxtjs.org/) 是一个很棒的框架。我真的很喜欢它在配置方法上的惯例。在 JS 世界中，这令人耳目一新，并且在开发过程中节省了大量时间。因此，在文档和真实世界的使用上还存在一点差距。不再拖延，让我们深入了解 Nuxt 和 Vuex 如何协同工作。

## 为什么我的应用需要这个？

函数式编程的一个特点是编写小函数来做好一件事。我们经常在紧迫的截止日期或快速解决热点问题的巨大压力下误入歧途。但在某些时候，这又回来困扰我们，并创造了一个机会，重新审视不再满足其目的的设计。也许在我们的职业生涯中，我们不止一次地发现一个变量变异是另一个事件或函数调用的副作用，是错误的来源？

Vuex 提供了处理应用程序状态的单向高速公路。一个外行人的思考方式是说“管理全局变量，不让你直接改变它们”。在这结束的时候，我希望你能掌握所有的知识，大喊“错了！”那句话。

为了更清楚地说明这一点，让我们来看一下架构:

![](img/dc1854559ae64e840fc3307b01a53782.png)

Vuex 工作原理的彩色图表

Vuex 是绿色虚线框中的所有内容。您将在下面看到我们的 Vuex 商店中有一个状态、一个突变和一些动作。当你在 Nuxt 的`store`文件夹中创建一个新文件时，这个文件可能会包含一些或所有这些东西。当一个组件正在渲染时，它可以访问 Vuex 状态，反过来又可以用于该组件运行所需的任何功能。一旦呈现，它可以分派或改变存储，因此导致使用该存储的任何其他内容更新。

## 模块与单店

老实说，[官方](https://nuxtjs.org/guide/vuex-store)文档对于应该如何配置 Vuex 非常混乱。我的观点是，如果你有三个以上的 API 调用或状态管理项来处理模块化方法是必需的。除了在一个 Nuxt 项目开始时快速地让一些东西工作之外，单一存储方法变得和使用全局变量一样痛苦。

知道模块是最好的方法，文档建议在`store`目录中构建一个`index.js`文件。除非你打算用一些花哨的插件配置 Vuex 或者改变它的配置，`index.js`不应该存在于`store`文件夹中。相反，每个“小部件”都需要自己的文件。

## 状态

State 是一个简单的函数，它返回存储在其中的对象。通常我会构建类似于`store/cars.js`的东西

```
export const state = () => ({
  list: [],
  car: {}
})
```

这给了我一个存放我收藏的汽车的地方，以及一个存放“当前”汽车的地方。

在组件 html 中，你可以使用`$state.cars.list`访问它，在 javascript 中通过`this.$state.cars.list`访问它。如果它是只读的，那就没问题。如果您想在表单中使用它，或者使用 v-model，那么它将无法工作。有多讨厌对吧？但是如果你再看一下上面的图片，这实际上很有意义。如果你可以直接改变状态，那么你可以跳过让你程序的其他部分知道它已经改变的部分。为了使用深点符号路径，你需要使用`mapState`助手。

```
computed: {
  ...mapState({
    cars: state => state.cars.**list**,
    car: state => state.cars.car
  })
}
```

尽可能让自己的状态保持平坦是非常重要的。处于某个状态的深度嵌套对象会失去反应性。例如，如果你的州包含类似`store.state.cars.car.color`的东西，颜色的突变将需要被强制重新加载。最好为单个`car` 制作一个新的商店模块。我倾向于从使用`state.things.thing`模式开始，然后当我发现我需要在这个东西上做很多操作时，重构到`state.thing`。然后，您的状态将包含`thing`的属性，将属性恢复到它们的反应状态。这是因为如果 Vuex 必须遍历对象键的深层链，那么性能会受到严重影响，并且可能会出现永无止境的循环。

## 吸气剂

可以把 getters 看作是存储的计算属性。它们应该读取模块的当前状态并返回一些东西。它应该给它所期望呈现的东西赋予一些意义。意思是全名非常重要！

```
totalCars: state => {
  return state.cars.length
},
blueCars: state => {
  return state.cars.filter(car => car.color === "blue")
}
```

Getters 在您的视图中始终是只读值。但是，它们可以与计算属性结合使用，并且您可以提供一个自定义设置器:

```
computed: {
  blueCars: {
    get() { return this.$getters['cars/blueCars'] },
    set(car) { this.$store.dispatch('cars/paintBlue', car)
  }
}
```

虽然有一个 mapGetters 助手，但我从来没有对它有很大的需求。大多数情况下，组件会完成这项工作。这并不是说不要使用它，只是更有可能从存储中获取一个值并在组件中修改它。然而，如果多个组件共享读取这些值并需要读取更新，getters 是一个很好的选择。

## 突变

这就是如何在状态中提交值。你不能直接调用它，因为突变是一个反应事件，我们希望我们的应用程序知道什么时候发生了变化。

```
**export const** mutations = {
  set(state, cars) {
    state.list = cars
  },
  add(state, value) {
    *merge*(state.list, value)
  },
  remove(state, {car}) {
    state.list.splice(state.list.indexOf(car), 1)
  }, 
  setCar(state, car) { state.car = car }
}
```

有了这个我就可以调用`$store.commit(‘cars/set’, [{id: 1, model: "Tacoma", brand: "Toyota"}])`等。当您希望用默认值预先填充存储，或者在计算属性中将其用作`set`处理程序时，这在具有`fetch`、`asyncData`和`nuxtServerInit`函数的组件中非常有用。

在组件中，您可以使用 mapMutations 助手快速获取映射到组件的 mutator 函数调用:

```
methods: {
  ...mapMutations({
    setCars: 'cars/set'
  })
}
```

这意味着在您的组件中，您可以通过`this.setCars([{id: 1, model: "Tacoma", brand: "Toyota"}])`访问它。

我在上面提到了在`state.things.thing`到`state.thing`模块之间移动。在这种情况下，您可能希望设置一个变异函数，可以轻松地将对象键映射到状态。要做到这一点，您需要:

```
setCar(state, form) {
  **let** keys = Object.keys(form);
  keys.forEach((key) => state[key] = form[key])
},
```

当然，您可以在这里验证您的状态，并防止意外的键污染状态。这完全取决于应用程序要求。

我想指出的是，改变嵌套很深的值也容易出错，很可能导致 Vuex 拒绝这种改变。试图变异`$store.state.cars.car.tire.brand`会告诉你这是不允许的。在这种情况下，最好创建一个处理单个汽车的`car.js`模块，因此它的状态包含单个汽车实体的属性。试着让你的状态对象尽可能的平坦。使用深度嵌套变异时创建新的存储。

## 行动

这些对于应用程序需要的任何异步活动都很有用。特别适用于调用后端服务器，或者需要非阻塞突变时。让我为这两种情况举一个例子。

```
**export const** actions = {
  **async** get({commit}) {
    **await this**.$axios.get('cars')
      .then((res) => {
        **if** (res.status === 200) {
          commit('set', res.data)
        }
      })
  },
  **async** show({commit}, params) {
    **await this**.$axios.get(`cars/${params.car_id}`)
      .then((res) => {
        **if** (res.status === 200) {
          commit('setCar', res.data)
        }
      })
  },
  **async** set({commit}, car) {
    **await** commit('set', car)
  }
}
```

这里我们定义了三个动作，`get`、`show`和`set`。Get 将获取我们的整个汽车集合，并在成功时用服务器返回的列表设置商店。显示是一样的，但是我们可以传入查询参数。这对于设置搜索或任何其他需要参数的场景非常有用。Last 是对我们之前的突变的添加，一个从上面调用突变“set”的动作。也许我们有一个 15，000 辆汽车的列表，这导致提交时 UI 延迟？这样，我们可以为使用承诺或异步/等待调用建立一个成功的未来。

## 插件

强烈推荐阅读用于 Vuex 的[五大插件](https://medium.com/js-dojo/5-vuex-plugins-for-your-next-vuejs-project-df9902a70de2)。只需创建一个`store/index.js`，并按照各自配置的建议将其导入。

## 严格模式

默认情况下，Nuxt 将 Vuex 配置为在开发中使用严格模式。如果你愿意，可以在`store/index.js`中禁用它，但这样做是一个*坏主意*。关闭它允许突变调用来自任何地方，有效地打破了架构的规则。在生产中它是禁用的，但你已经编写了知道规则没有被破坏的应用程序，所以当 vuex 不再担心在生产中检查这个问题时，它就成为一个性能提升。

一个常见的错误是试图使用`v-model=”$state.cars.car.color”`改变状态，这是行不通的，因为这实际上是值的只读版本。相反，它应该使用计算属性，或者 mapState/mapMutations 帮助器的组合。

## 回顾

我花了一点时间从过去的 Javascript 代码中改掉一些坏习惯，才真正体会到 Vuex 给应用程序带来的价值。我觉得不是每个 app 都需要的东西；相信那是狭隘的。在 JS 世界中有许多很好的状态管理实现，但是 Nuxt 已经接受了 Vuex，目前它工作得很好。

在用户界面中拥有状态的真实来源可以节省大量开发时间。用对变化的反应、一致的 API 和强大的插件生态系统来支持它，给了我们一个快速解决问题的工具箱。

## 在 Nuxt 中

Nuxt 在 pages 中提供了一个`fetch`方法。这对于在页面呈现之前挂钩操作非常有用。在 pages 中经常会看到这样的代码块:

```
**async** fetch({store}) {
  **await** store.dispatch("cars/get")
}
```

这将调用 cars 模块中的“get”操作，调用后端并将 cars 状态设置为 API 响应。

## Vuex 助手

Vuex 有映射助手，可以轻松地消除商店的呼叫。不要到处使用`$store.state...`，使用`mapState`会更干净。动作、突变和 getters 也有助手。

```
computed: {
  ...mapState({
    cars: state => state.cars.list
  })
}
```

这创建了一个函数调用，让我们引用`this.cars`而不是`this.$store.state.cars.list`。很清楚哪个更容易阅读。

`mapGetters`使用与`mapState`完全相同的参数，明显的例外是它不允许突变。Getters 是只读的，通常用于格式化数据。

mapState 正在做的事情是这样的:

```
computed: {
  cars: {
    get() {
      return this.$store.state.cars.list
    },
    set(val) {
      this.$store.commit('cars/set', val)
    }
  }
}
```

这也是一个有用的模式，当您需要执行定制的变化或者任何其他需要的预提交逻辑时。记住，虽然这些函数应该非常短，所以尽量保持 get/set 中的逻辑最少。

您还可以使用`mapMutations`将变异函数拉入模板:

```
methods: {
    ...mapMutations('cars', ['set', 'anotherMutation'])
}
```

然后调用`this.set(val)`而不是`this.$store.commit('cars/set', val)`

## 结论

我希望这澄清了很多关于 Vuex 的细微差别和困惑。用 Vue 编码是如此重要的一部分，以至于知道如何最大限度地利用它可以提高生产率。

编辑:如果你想要更多的软件模式，还有一个[跟进](/eating-my-advice-efficiently-improving-on-understanding-and-using-nuxt-vuex-6d00769014a2)。

## 参考:

 [## Vuex |开始使用

### Vue.js 的集中状态管理

vuex.vuejs.org](https://vuex.vuejs.org/guide/) [](https://nuxtjs.org/guide/vuex-store) [## Vuex Store - Nuxt.js

### 使用存储来管理状态对于每个大型应用程序都很重要，这就是为什么 Nuxt.js 在其…

nuxtjs.org](https://nuxtjs.org/guide/vuex-store) [](https://vuejs.org/v2/guide/instance.html#Instance-Lifecycle-Hooks) [## Vue 实例- Vue.js

### 每个 Vue 应用程序都从创建一个带有 Vue 函数的新 Vue 实例开始:尽管没有严格的关联…

vuejs.org](https://vuejs.org/v2/guide/instance.html#Instance-Lifecycle-Hooks) 

注意:我写了一篇后续文章，讨论了一些有用的片段和开发应用的设计模式。