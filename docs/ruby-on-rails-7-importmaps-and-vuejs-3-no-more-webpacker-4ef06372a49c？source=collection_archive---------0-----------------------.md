# Ruby on Rails 7，importmap 和 VueJS 3，没有更多的 webpacker

> 原文：<https://itnext.io/ruby-on-rails-7-importmaps-and-vuejs-3-no-more-webpacker-4ef06372a49c?source=collection_archive---------0----------------------->

Ruby on Rails 7 和 gem [importmap-rails](https://github.com/rails/importmap-rails) 让我们可以使用 javascript 模块和库，而不需要传输或捆绑。我记得当我修改 webpacker 配置以将 VueJS 3 添加到我的 rails 项目时失败了，并开始使用 VueJS 2 来代替它。现在我有一个新的机会尝试第三版。

那么，我们如何开始，只是添加宝石到你的宝石档案

```
gem 'importmap-rails'
```

然后安装

```
bundle installrails importmap:install
```

这将添加 **config/importmap.rb** 文件和默认 js 端点**app/JavaScript/application . js**。

还有别忘了删除一切和 webpacker，gem，配置文件，node_modules 有关的东西。

gem 的使用包含两个步骤。

**添加包**

作为第一步，我们需要添加 js 依赖项来导入映射配置。这可以通过两种方式实现。

通过运行特殊命令

```
./bin/importmap pin vue@nextor ./bin/importmap pin vue@next --download
```

或者通过手动将 pin 链接添加到 importmap.rb

```
pin "vue", to: "[https://ga.jspm.io/npm:vue@3.2.26/dist/vue.esm-browser.js](https://ga.jspm.io/npm:vue@3.2.26/dist/vue.esm-browser.js')"
```

我更喜欢第二种方式，因为您可以直接指定使用什么包。所以在使用 VueJS 3 之后，你会有这样的 importmap.rb 配置。

```
pin "vue", to: "[https://ga.jspm.io/npm:vue@3.2.26/dist/vue.esm-browser.js](https://ga.jspm.io/npm:vue@3.2.26/dist/vue.esm-browser.js')"pin_all_from "app/javascript/controllers", under: "controllers"pin "application", preload: true
```

**添加 VusJS 3 控制器**

第二步是添加 vuejs 控制器。所以在**app/JavaScript/application . js**中为你的控制器添加导入。

```
import "controllers/component_controller"
```

然后可以在 app/JavaScript/controllers/component _ controller . js 文件中添加控制器

```
import * as Vue from "vue"const point = "#point"const element = document.querySelector(point)
if (element !== null) {
  const app = Vue.createApp({
    data() {
      return { count: 1 }
    },
    created() {
      console.log("count is: "+ this.count) // => "count is: 1"
    }
  })
  const vm = app.mount(point)
}
```

仅此而已。你不需要做任何 webpacker 配置，添加加载器或任何其他东西。

**优点**

*   不需要配置 webpacker，所以至少 rails 开发人员的 js 痛苦要少得多
*   部署应用程序所浪费的时间更少，因此您可以节省几分钟，因为您不需要编译 js

**缺点**

*   不能**要求**文件，需要使用一些额外的库
*   可能需要找到另一种方式来使用 cli 库，比如 ttag-cli。