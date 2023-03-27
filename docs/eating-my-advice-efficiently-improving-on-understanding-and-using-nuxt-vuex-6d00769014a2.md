# 采纳我的建议:有效地理解和使用 Nuxt + Vuex

> 原文：<https://itnext.io/eating-my-advice-efficiently-improving-on-understanding-and-using-nuxt-vuex-6d00769014a2?source=collection_archive---------1----------------------->

![](img/954275becb3d206cfa00257f5d5db276.png)

在一年前写的这篇文章的第一部分[中，我开始谈论修改设计。我想承认这个堆栈有很多了不起的地方，并邀请人们讨论他们的方法。](/efficiently-understanding-and-using-nuxt-vuex-7905eb8858d6)

很多都来自于栈中的日常工作，关注 Github 和其他地方关于它的喋喋不休，感觉是时候把事情安排好了。没有特定的流程，只是一些概念或注释的集合。

## UI 真的很慢

查看[https://nuxt-community . github . io/nuxt-i18n/SEO . html #改善性能](https://nuxt-community.github.io/nuxt-i18n/seo.html#improving-performance)。我花了几天时间调试它，直到我发现 nuxt-i18n 是它的来源。

## 样式指南(棉绒外面)

我把代码放在我的组件文件夹中。所有带有. vue 文件扩展名的东西都需要有一个`name`键。页面名称基本上是`PathPage`遵循一点[原子设计](http://bradfrost.com/blog/post/atomic-web-design/)模式。

# 干燥 Nuxt 应用程序

我非常诚实地使用单文件组件的混合，并且通常围绕它们的用例和/或它们管理的实体来组织我的项目。

```
orders (molecules)/
  mixins/
    validations.js
    operations.js
    getters.js
  OrdersSelectMenu.vue
  OrderForm.vue
  etc...core (atoms)/
 buttons/
 alerts/
 progress/
 etc..mixins (particles)/
  formatters/
    dates/
      format.js
      fromNow.js
    strings/
    etc... apis/external/
    operations.jspages/
  orders/
    index.vue
  orders.vue
```

## 用 Vue 的合成 api！

我想在一个不同的话题中讨论这个问题，因为它非常大。不过[https://vue-composition-API-RFC . net lify . com/# logic-reuse-code-organization](https://vue-composition-api-rfc.netlify.com/#logic-reuse-code-organization)出于好奇。这是一个我需要了解的话题，因为据我所知，很多事情都在进行中。在内部，我认为组合 api 可以用来构建模块化的[关注点](https://effectivesoftwaredesign.com/2012/02/05/separation-of-concerns/)，然后呈现层可以是非常快速的 [mvvvm 模式](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)。

## 使用 mixins！

比如我有一个`components/orders/DataTable.vue`组件。在这个模式中，我设置了一个期望，这里有一个对`orders`的 Vuex 模块依赖。这个文件夹可以有自己的一组 mixins，我将其视为私有。我也有一个全局 mixins 文件夹，我试着用它来存放一些实用程序之类的东西。大多数时候，某个东西在一个地方开始是 mixin，然后被向上或向下重构为 public/private 状态。Mixins 真的很容易推理和维护。

下面是一个 mixin，我用它对来自查询参数的数据表进行过滤、排序和分页:

```
export default {
  watchQuery: true,
  watch: {
    // Allows for this mixin to be extended using object notation
    sortable(newval) {
      this.$router.push({
        path: this.$route.path,
        query: { ...this.$route.query, ...newval }
      })
    }
  }
}
```

然后在我的`pages/orders/index.vue`里我只是导入这个:

```
mixins: [watchQuery],
data() { return { sortable: {} } },
```

现在，您的页面可以通过使用 sortable 对象中的值，从查询参数中设置组件的起始值。常见的键有 page，rows，sortBy，desc，你懂的。这还会更新查询参数并触发路由器事件。

有了这个，应用程序可以切换到使用`b-table-simple`甚至`-lite`，因为 UI 不再需要全功能的`b-table`组件来处理给定项目的排序/过滤/分页。在大多数情况下，数据的完全服务器端呈现是有利的。

我也真的厌倦了在这么多地方看到`import { mapActions } from 'vuex'`。

所以又是 mixins 来救援了！

```
// operations.js
import { ***mapActions*** } from 'vuex'

export default {
  methods: {
    ...***mapActions***({
      create: 'items/create',
      update: 'item/update',
      destroy: 'item/destroy'
    })
  }
}
```

最终，它成长为自己的类，用于我的商店和组件中的所有操作。它给了我一个跨组件/页面推理 API 的简单方法，只需要导入它。因此，现在组件文件夹可能看起来像这样:

```
components/
  cars/
    DataTable.vue
    CarForm.vue
    DetailsCard.vue
    operations.js
    computed.js
    getters.js
```

其中`operations`为汽车提供调用 vuex 存储动作的方法，`computed`为我的汽车列表提供获取/设置状态的映射，`getters`为我的 UI 用于格式化信息的任何格式化或其他操作提供映射。

## Webstorm 的 Vue 和 Typescript 支持非常出色

Webstorm 的 Vue 插件的一个优秀特性是突出显示一大块模板代码。然后重构->提取-> Vue 组件。它让您直接命名组件，然后在同一个目录中创建一个新组件。Webstorm 还跟踪导入语句，并在组件移动时更改路径。

我相信 VSCode 和其他人也可以做到这一点，但是我对特别有用的工具感兴趣。另外，我没有因为推荐 Jetbrain 的工具而得到任何东西。

# 我喜欢 Vuex，但是…

假设您的任务是构建一些个性化的仪表板。所以 API 需要在很多地方被调用，对于所有这些实体，都有一个地方来填充响应数据。好极了，一切都很好，然后应用程序开始感觉不那么快了。它使用 48gb 内存，因为 vuex 在数组中存储了 1000 个项目...

如果你的应用程序必须管理许多实体，而你想要一个类似 AREL 的语法来管理所有这些，那么就去看看 [vuex-orm](https://vuex-orm.github.io/vuex-orm/) 。它非常容易设置，并且有助于简化实体之间的复杂关系。它还有一个很棒的插件系统。也很好地记录和一个积极友好的松弛渠道。

如果你在看 vuex-orm，并认为“太复杂了”，那么看看 https://github.com/davestewart/vuex-pathify 的[。我没有用过它，但我经常看到它获得好评。](https://github.com/davestewart/vuex-pathify)

否则，在编写组件时，尽量使它们无状态。理想情况下，组件接收道具并发出事件。这些事件可以触发父组件的突变，然后更新道具的值。使用 destroy 钩子清除状态来节省内存，使用 Nuxt 的页面中的`fetch`来恢复状态。vuex 缓存有很棒的插件，但是依赖后端 API 缓存头要好得多。

# 重置状态

在我所有的 vuex 模块中，我写了一个将状态设置回默认状态的动作。然后在`store/index.js`中，我有一个重置函数，它调用我想要清除的模块中的每个重置动作。

## 默认状态

```
const defaultState = {
  list: [],
  page: 1,
  total: 0,
  per: null
}

export const state = () => ***JSON***.parse(***JSON***.stringify(defaultState))
```

这样你就能确定这个状态是从一个没有原型污染的新鲜物体开始的。也许太激进了。

## 关于服务对象

服务对象就是这样；它是一个对象(通常是一个类),带有特定行为的通用操作，例如验证输入。

我见过很多这样的代码；甚至说这是最佳实践:

```
services/
  orders.jsclass Orders
...
getBySomething() { this.$axios.get('/orders/something')...

getByPaychecks() { this.$axios.get('/orders/weird')...

getByGetinBy() { this.$axios.get('/orders/flex')...
```

通过采用这种封装服务调用的模式，它将 Vuex 动作排除在外，而 Vuex 动作通常是模块中最大的部分。很好，对吗？较小的模块更容易推理和测试。

我沿着这条路走了一段时间，在 Nuxt/Vuex 中实现了一个反模式。原因如下:

1.  不使用操作会中断反应流
2.  Javascript 绝对不会让开发人员创建动作模块，并像 mixins 一样在任何地方导入它们。
3.  Nuxt 有一个服务对象必须遵守的 SSR 请求生命周期，否则会导致难以调试的奇怪加载错误。

服务对象的一个用途是将 axios 调用的配置从逻辑中分离出来，验证参数，并检查授权。例如:

```
./store/orders.js 
async getAllOrders({commit}, params) {
  await this.$axios.get(Orders.ordersPath, Orders.params(params))
}
...class Ordersconst ordersPath = '/orders'function params(params) { orderParams = validate(params)
  if(orderParams.valid) {
    return orderParams
  }
}
...
```

服务对象可以防止无效和过多的 API 调用，但我不认为它们可以替代 Vuex 动作。服务对象与动作并不相互排斥；它们可以一起使用。

总的来说，我发现很容易保持应用程序的设计依赖于 Nuxt 的`/page`级别来编排对服务器的调用，然后尽量避免在我的组件中直接导入`operations.js`。

```
/pages/orders.vue:
import operations from '@/components/Orders/operations.js'export default {
  name: 'OrdersPage',
  mixins: [operations]
}
```

# 捆绑大小

## 不要把所有的东西捆在一起。

你不必`yarn install`每一个前端依赖项，尤其是如果你想使用 moment.js 的话，我建议在`nuxt.config`或者布局的`head`键中使用 CDN urls。您可以通过将`nuxt.config.js`设置为`analyze: true`来生成您的包的 webpack 分析。参见关于分析的文档[。](https://nuxtjs.org/api/configuration-build)

# 有用的片段

说到 moment.js，如果你碰巧切换到了 [date-fns](https://date-fns.org/) 这里有一个有趣的小片段:

```
// plugins/datefns.js
import Vue from 'vue'
import { distanceInWordsToNow, format } from 'date-fns'
import FormatTime from '@/components/FormatTime'
import FromNow from '@/components/FromNow'

Vue.filter('fromNow', function(timestamp = new ***Date***()) {
  return distanceInWordsToNow(timestamp) + ' ago'
})

Vue.filter('formatTime', function(
  date = new ***Date***(),
  how = 'h:mmA MMM Do YYYY'
) {
  return format(date, how)
})

Vue.component('format-time', FormatTime)
Vue.component('from-now', FromNow)
```

和 components/FormatTime.vue

```
<template>
  <span>{{ time | formatTime }}</span>
</template>

<script>
export default {
  name: 'FormatTime',
  props: {
    time: {
      type: ***String***,
      default: () => new ***Date***().toDateString()
    }
  }
}
</script>
```

和组件/FromNow.vue

```
<template>
  <span>{{ time | fromNow }}</span>
</template>

<script>
export default {
  name: 'FromNow',
  props: {
    time: {
      type: ***String***,
      default: () => new ***Date***()
    }
  }
}
</script>
```

如果你仍然需要更多，https://www.npmjs.com/package/vue-date-fns 已经做得很好了。

## 处理器械组

众所周知，Bootstrap 的难点在于成排的卡组。我写了一个 mixin 来把我的数组分成任意大小的数组。

```
export default {
  computed: {
    chunkSize() {
      if (process.client) {
        return ***window***.innerWidth > 768 ? 3 : 2
      }
    }
  },
  methods: {
    foldm(r, j) {
      return r.reduce(
        (a, b, i, g) => (!(i % j) ? a.concat([g.slice(i, i + j)]) : a),
        []
      )
    }
  }
}
```

要使用它，只需调用 foldm:

```
<b-card-group v-for="(***chunk***, ***i***) in foldm(items, chunkSize)" :key="***i***" deck>
  <item-summary-card
    v-for="***item*** in ***chunk***"
```

## 结论

我希望你觉得这些比特有用！我希望这些信息会随着我将来的添加和编辑而扩展。请随意在这里发表你的意见！