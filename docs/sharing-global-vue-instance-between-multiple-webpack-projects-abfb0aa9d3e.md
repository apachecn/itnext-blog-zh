# 在多个 webpack 项目之间共享全局 Vue 实例

> 原文：<https://itnext.io/sharing-global-vue-instance-between-multiple-webpack-projects-abfb0aa9d3e?source=collection_archive---------5----------------------->

![](img/44930a853a045cac16f47e120fe774cc.png)

贷方:[期限](https://tenor.com/view/homer-simpson-guanchidoguan-clones-clon-simpson-gif-12210267)

*这是我的故事的后续:* [*将 Vue 组件添加到老式的 web 应用*](/adding-vue-components-to-old-school-web-apps-1f6c2339d599) *。*

如果你试图将 Vue 与基于插件的 CMS 系统集成，比如 WordPress 或 Elgg，你很快就会遇到在多个插件之间共享 Vue 实例的问题。例如，您可能有一个集成了基于 Vue 的设计系统的插件，比如 Vuetify，并对其进行了重新设计以匹配您的项目的外观。在稍后阶段，您决定添加一个新插件，该插件将重用您的设计系统的组件库中的组件。每当插件想要在其上构建组件时，您都可以导入设计系统并设计其样式，但是如果我们可以在多个插件之间共享全局 Vue 实例呢？

`webpack`配置支持`[externals](https://webpack.js.org/configuration/externals/#externals)`参数，该参数允许我们定义不会被 webpack 导入和捆绑的外部依赖项，但将从全局解析。

首先，我们需要将 Vue 导入到我们的项目中，并确保它被定义为`window.Vue`。这可以通过使用带有 CDN src 的`<script>`标签来实现，或者，如果您使用 RequireJS，可以通过手动分配或垫片来实现。

```
require(['path/to/vue'], function(Vue) {
    window.Vue = Vue; require([
      'vuetify-plugin/index', 
      'stripe-plugin/index', 
      'payment-form-plugin/index'
    ], function() {
      // Mount your Vue app
    });
});
```

现在我们可以更新所有子项目中的 webpack 配置，将 Vue 定义为外部的。

```
**// If you are using laravel-mix
mix**.webpackConfig({
   **externals**: {
      **vue**: **'Vue'**,
   },
});// Otherwise just update your webpack config, wherever it is defined
```

一旦将 Vue 定义为外部的，就可以继续使用`import`语法，但是 webpack 实际上不会导入或捆绑任何东西，而是从全局 Vue 实例中解析它。

通过这种方法，每个插件都可以有自己独立的构建过程，并使用插件、组件、指令和 mixins 扩展全局 Vue 实例，这样当另一个插件想要重用它们时，它们已经被定义和配置好了。

这将允许您隔离插件中的功能，而不必担心重叠和重复，例如，如果您有 10 个不同的插件来构建支付表单，您不必为每个插件导入和配置条纹密钥。