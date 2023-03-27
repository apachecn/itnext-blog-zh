# Vue —深入构建 Vue 应用程序体验

> 原文：<https://itnext.io/vue-deep-dive-to-build-a-vue-app-experience-39eeea3a18c7?source=collection_archive---------2----------------------->

![](img/f31ef2084d29f6b9c9fd4b25ede16bfb.png)

在过去的一年多时间里，vue 已经从 1 升级到 2，vue-cli 变得越来越强大，Vue 宣布将 vue-resource 更改为 vue-axios。以下是使用 Vue 构建应用程序时的一些有用信息。我们可以讨论更多更好的解决方案。:)
(这篇文章适合已经在用 vue 的人。)

# 1.关于通过 vue-cli 设置项目:

使用:

vuex，vue-router，vuex，karma 或者 jest(取决于你的 vue-cli 版本)，nightwatch(我更喜欢 [cypress](https://www.cypress.io/) )。

结构(在 src 文件夹中):

*   组件(这里的组件甚至可以在这个项目之外独立工作)
*   通用(全局项目组件，如页眉、页脚等。)
*   页面(您可以在更大的页面中添加更多的子页面文件夹和子组件。)
*   存储(使用模块文件夹对您的状态进行分类。)
*   api(关于一些异步请求设置，这里我把 api 和动作分开。)
*   路由器(vue-路由器和相关设置。)
*   js(全局项目 js 文件，如 util.js、constants . js…等。)
*   手写笔/ sass/scss / less..等等。(全局项目样式设置。)
*   故事(故事书)
*   区域设置(vue-i18n)
*   混合蛋白
*   指示的
*   …

## 关于 webpack 别名:

```
// in webpack.bas.conf.js
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'), // vue-cli default setting
      'styl': resolve('src/stylus'), // for stylus using
      '~assets': resolve('src/assets'), // for js using
      'assets': resolve('src/assets') // for stylus using
    }
  },
```

我们可以在 js 中导入像“@/components/IModal”这样文件。图像 src 怎么样？这里我用的是“~assets/example.png”。为什么？因为当我们在 css 预处理器作用域中编写样式时，别名设置必须加上“~”才能使用。

```
<style lang="stylus" scoped>
    @import '~styl/variables' // Here we have add more "~" to use the alias setting in stylus scope.
    div {
        background-image: url('~assets/select-arrow-double.svg') // Here we have add more "~" to use the alias setting in stylus scope.
    }
</style>
```

所以`'~assets': resolve('src/assets')`是为了 js。然后我们可以用 js/css 写同样的设置。
我把它从所有的' @ '分离到 js，css，img(资产)。
如果项目结构发生变化，我们只需要改变别名设置。

## 怎么了?单元测试坏了！

无论是在 karma(旧的 vue-cli)还是 jest(新的 vue-cli)中，你的 webpack 设置都非常复杂。你可能会遇到一些问题，比如 es6，导入使用 es6 的库，忽略 css 预处理器，img…等等。

```
// in test/unit/index.js (old vue-cli karama setting ) be careful of the test scope and file.
// ignore api, stylus folder..etc.
const srcContext = require.context('../../src', true, /^\.\/(?!main(\.js)?$|(?:api|stylus|stories)(?![^\/]))/)
```

## 添加故事书

和现在一样，在 vue 项目中使用 storybook 非常容易。一些错误已经解决了。

```
npm i -g @storybook/cli
cd my-vue-project
getstorybook
npm run storybook
```

更多详情可以查看我的 github:
[https://github.com/sky790312/new-website](https://github.com/sky790312/new-website)

# 2.使用 vue 开发:

## 讨论数据初始化和更改:

你可以使用 vuex 作为初始数据，并传递给组件，你想改变它，但你不能直接改变状态。所以你必须声明组件的数据。类似于编辑用户信息，但在项目中共享用户状态的场景。有一些解决办法:

*   使用 vuex 初始化`beforeMount`/`mounted`/`beforeRouterEnder`…等中的值。根据您的情况，更改 vuex /更改组件的数据。您可以拥有子组件，并在其他组件中更改用户状态。如果你使用的是实时数据库，比如“firebase ”,用户状态不仅在编辑页面编辑时会改变，而且随时随地都会改变。所以你可能需要更多的“观察”来改变你的组件的数据和处理你的 vuex。
*   使用`watch` with immediately，那么你就不用在“beforeMount”和“watch”中写同样的代码了。(在 beforeMount 和“watch”中添加类似“handleInit()”的方法句柄。)
*   使用`computed`和`data`。小心使用“get”来获取 vuex 状态，使用“set”来调用操作并更改 vuex 状态。

## 关于异步数据:

*   必须小心 rendor 的生命周期！最简单的方法是使用“xxxReady”v-if 来防止子组件 rendor 或如果您的数据没有准备好而触发错误。你必须明确你的应用程序的流程。什么时候应该需要准备什么东西来防止模板渲染错误。

## 无法在组件数据中调用 vue 方法:

*   您可以将一些相同的 vue 数据配置到 v-for。看一看:

```
// html template
    <i-tab :isCentered="false">
      <i-tab-pane
        v-for="iTab in iTabs
        :key="iTab.name"
        :name="iTab.name"
        :to="iTab.to"
        :method="iTab.method">
      </i-tab-pane>
    </i-tab>
    // js
    // this iTabs config can't use directly in vue data, you can use compueted.
    iTabs () {
      const { name = '' } = this.$route.query
      return [{
        name: 'tabA',
        to: `/buildings/${name}`,
        method: () => {
            this.handleTabA()
        }
      }, {
        name: 'tabB',
        to: `/buildings/${name}`,
        method: () => {
          this.handleTabB()
        }
      }]
```

事件喜欢按钮。我们可能希望用 v-for 和 js 中的 config 编写更少的代码。可以用`computed`。

## 当您想要更改客户端站点中的语言时，请小心 vue-i18n:

如果我们可以改变客户网站的语言，我们可能会面临一些问题，当我们显示/编辑和改变语言。`watch`你的 lang，并设置编辑组件数据。类似`select option`的一些案例。或者某些数据只是组件`mount`中的初始数据，而不是模板中的数据。

## 如果你有一些发电机组件，更复杂的应该是处理:

一些生成器组件，如“IForm”，“可修改的”，你可以在你的 vue 数据/计算中配置它。我们所有人都应该小心！

## Mixins 很好，但是你可能在 component 中找不到这个方法。

Mixins 无法导入，如:

```
import { 
    toUpperCase,
    sort,
    ...
} from '@/js/utils'
// import and use it to vue method.
```

所以可能很难调试方法在哪里。但是使用 mixins 数据/方法还是一件好事。只是需要多加小心！

## 使用 vuex 处理异步请求。

我喜欢用 vuex 来控制所有的异步请求。所有异步请求都在操作中处理，但不是所有操作都需要处理异步请求。

# 3.关于 vue 中的 HTML:

## 小心 vue 中的 html:

Vue html 模板连续 rendor。在组件或 vuex 的 html 中编写类似于" [user.plan.id](http://user.plan.id/) "的东西可能会遇到一些麻烦。

```
// You may fetch some async data. You must make sure when your html template render this line, your object is already define! (user.id may from "undefined" to "some value", but when user.plan is"undefined", user.plan.id will trigger error!)You can define your data structure fully. Or you can use v-if to make sure your flow and the component will render correct without problem.
```

你可以从 js 文件中导入一些方法，比如 util.js。如果你想在 vue html 模板中直接使用它，它可能会触发错误，因为当 vue 渲染时，vue 不知道方法是什么。

```
<template><select>
    <option 
      v-for="country in countriesList" // use it directly may trigger error!
      :key="country.id">
    </option>
    ... </select>
</template>// You can import it and attach it to vue method like:
    import {
      countriesList
    } from '@/util'

    methods: {
        countriesList,
        ...
```

如果经常使用，可以直接添加到 vue 原型中。

## “v-if”非常好用，再把两个逻辑移到 js 控制:

```
<div
    v-if="itemType === 'list' && isPayUser()"
    class="container">
    ...
</div>
// replace this to..
<div
    v-if="shouldShowList()"
    class="container">
    ...
</div><div
    :type="itemType === 'list' ? 'list' : 'item'"
    class="container">
    ...
</div>
<div
    :type="getItemType()"
    class="container">
    ...
</div>
// replace this to..
```

## 小心使用 v-if v-else 与 v-for，v-for execute 在 v-if 之前。不要使用更多的 html 标签，使用“模板”

```
// don't use html tag, you can use more template tag
<template v-if="isBrowserSupport">
</template>
...
<template v-else>
</template>
```

v-if in v-for:[https://github.com/vuejs/vue/issues/3106](https://github.com/vuejs/vue/issues/3106)

# 4.关于 vue 中的 CSS:

## 项目样式和组件样式

```
// No matter the component create by yourself, or by library/framework. Component should have itself style even in other projects. This kind of component like modal, dropdown, table, alert, datepicker..etc. If you create your own components. Be careful of mixing the project style into your component. (same to js config)
```

## 全局项目样式设置

在您的项目中使用通用样式是解决方案之一:

```
// in global stylus setting, you may have variable.stylus, extend,stylus..etc.
// in App.vue, use it without "scoped"
<style lang="stylus">
@import '~styl/index'
</style>// use "scoped" vue way to have css scoped for all components.
<style lang="stylus" scoped>
@import '~styl/variables'
#app {
```

您可能会面临在组件之外更改组件样式，但这已经是趋势了。
你可以通过> > >来改变它:

```
// in App.vue
<style lang="stylus" scoped>
>>> .i-modal {
    ...
}
```

## 班级设置

将类名直接设置为普通的 html 标签，并使用 js 来配置 vue 组件。

```
// assign it's class name directly to normal html tag.
<button
  class="btn btn-round btn-primary"
  :disabled="isUserExpired()"
  @click="handleEditItem()">
  {{ $t('button.update') }}
</button>// assign it's class name setting to vue component(If the component create by yourself).
// some plugin maybe control style by js directly
<i-modal
  v-if="exampleModal.options.shouldShow"
  :className="exampleModal.className"
  :options="exampleModal.options">
</i-modal>
```

## 样式应该由 js 还是 css 控制？

复杂方式使用 js，简单方式使用 css。尽管如此，一些动画必须直接控制它。

```
// control in js
<i-icon
  :name="socialIcon.name"
  :color="socialIcon.color"
  :width="socialIcon.width"
  :height="socialIcon.height">
</i-icon>
// replace this to..
<i-icon
  :name="socialIcon.name"
  :className="socialIcon.className">
</i-icon>
// you may have default icon style, and setting your project style by class.
```

## 交易

所有的原则都是一样的，如果动画是复杂的，或者可以重用，甚至结合复杂的规则。你应该使用 vue transation。如果你只想在两秒钟后淡入。直接用 css 就好了。尝试使用 js 来配置和控制 css 样式。

```
// html
<transition :name="animationType">
    <keep-alive>
        <router-view id="view"></router-view>
    </keep-alive>
</transition>// js
this.animationType = 
    animationType === 1 ?  'slide-left' :
    animationType === 2 ?  'slide-right' :
    animationType === 3 ?  'slide-top' :
    animationType === 4 ?  'slide-bottom' : ''
```

在 vue 官方网站上有一些交易实例，你可以自己扩展或者使用一些库。

## 结论

用 vue 开发应用是一种很好的体验。Vue 现在变得更强大了！祝 vue 在任何职场都能得到更多的关注。欢迎讨论开发 vue 项目的最佳方式。感谢你阅读这篇文章。:)

# 谢谢