# 使用 VueJS 和 Laravel 进行实时唯一输入验证

> 原文：<https://itnext.io/vuejs-and-laravel-realtime-unique-input-validation-a6d9220be1c5?source=collection_archive---------2----------------------->

![](img/05fda0b8f2ea04385325c14a26d3d447.png)

> 我最近遇到了一个问题，我需要实时验证文本输入的唯一性。当您需要创建用户名时，这在注册表单中很常见。对于用户来说，不断地提交表单来查看他们的输入是否被接受是非常烦人的。这就是服务器端验证发挥作用的地方。

在我的项目中，我已经使用了 Vuetify 和 VeeValidate，所以我将围绕这两个工具来设计我的解决方案。这里讨论的概念可以用于其他包，但是可能需要一些修改。

我将使用我正在制作的关于多租户 web 应用程序系列中的演示项目。您可以在下面找到本系列第 1 部分和 github repo 的链接。

[](https://github.com/sadnub/laravel-tenancy-passport-demo) [## sad nub/laravel-租赁-护照-演示

### 带护照和租约的 Laravel 演示。为 sad nub/laravel-tenancy-passport-demo 开发做出贡献，创建一个…

github.com](https://github.com/sadnub/laravel-tenancy-passport-demo) [](https://medium.com/@sadnub/hyn-tenancy-5-2-and-laravel-passport-a0d11c5a08eb) [## Laravel 护照和 Hyn \租赁-第 1 部分

### 这是你们期待已久的帖子。我已经花了三个月的时间来构建我的 SaaS 应用程序…

medium.com](https://medium.com/@sadnub/hyn-tenancy-5-2-and-laravel-passport-a0d11c5a08eb) 

现在来点好的！

因此，在这个特定的应用程序中，每个租户都注册了一个子域，这将使他们进入自己的控制面板。这个子域是在注册期间选择的，并且需要是唯一的。因此，我们的目标是让他们能够实时看到他们输入的值是否有效。这将使用一个 API 端点来完成，该端点简单地检查数据库中的值。如果值存在，则返回 false，反之亦然。

如前所述，我将使用 VeeValidate 包进行验证。查看文档！

 [## VeeValidate

### 基于模板的 Vue.js 验证框架

baianat.github.io](https://baianat.github.io/vee-validate/) 

Vuetify 对于 UI 组件来说是非常棒的，并且可以减少开发中的大量工作。看看这些文件！

[](https://vuetifyjs.com/en/getting-started/quick-start) [## 快速入门- Vuetify.js

### 立即开始使用 Vue 和 Vue 化。支持 vue-cli-3、现有项目等。

vuetifyjs.com](https://vuetifyjs.com/en/getting-started/quick-start) 

# vee 验证配置

使用这个包非常容易。你只需要把它作为一个插件来添加，并配置一些选项。这是我的`app.js`文件。

需要`inject: false`配置，因为我们将在 Vue 之外添加验证错误。稍后我会分享我的 axios 配置来解释这一点。我们只需要记住在需要验证字段的组件中注入根`$validator`实例。查看文档了解更多信息。

# API 端点

对于这一部分，我创建了一个 API post route，它调用注册控制器中的一个方法。很标准的东西。

# 注册控制器

`checkDomain`方法将把租户 URL 附加到请求中，然后通过验证器检查其唯一性。如果验证失败，422 响应将与验证消息一起发回。我有一个 axios 拦截器，它捕获这 422 条消息，并将它们添加到 VeeValidate 错误包中，因此我们不必担心处理这些消息。

当验证成功时，将以 VeeValidate 期望的格式发回一个 json 响应。

这是我正在使用的 axios 拦截器。它需要 Vue 根实例来添加验证错误，所以请确保执行`export const vm = new Vue()`以便您可以导入它。

# 授权 API 文件

我在我们的 api.js 文件中添加了一个函数来检查域。这是我们可以导入到组件中的文件。

# 注册页面

这是所有验证逻辑的位置。如果你需要在应用程序的其他部分使用相同的逻辑，你可以把它移到你的`app.js`文件中。

有一个名为 isUnique 的函数，每次触发验证时都会调用它。然后我们使用 Validator.extend()方法创建一个新的验证规则。然后，我们可以在字段上指定这个验证规则。我在`mounted()`钩子中指定了所有这些。

看一下 fqdn 字段。添加了 unique 规则，并且有一些逻辑根据验证状态切换图标。我使用 Vuetify v-text-field append 槽将图标放在输入中。然后我们可以使用 v-if 语句来呈现正确的图标。最后，我使用 VeeValidate `data-vv-delay`属性将验证次数限制为每 500 毫秒 1 次。这将减轻我们的 API 的一些负担。

# 包扎

现在，您有了一个全功能的解决方案来实时验证数据的唯一性！你的用户会非常感激的。

希望这对你有所帮助！如果你有任何问题、意见和建议，请在评论中告诉我。

*看看我下面的其他文章吧！*

[第 0 部分](https://medium.com/@sadnub/laravel-multi-tenant-app-setup-part-0-ee4c730f4c2a) — Laravel 多租户应用设置

第一部分 — Laravel 护照和 Hyn \租赁

[第二部分](https://medium.com/@sadnub/vuejs-and-laravel-api-part-2-711c4986281c) — VueJS 和 Laravel API

[第 3 部分](https://medium.com/@sadnub/laravel-multi-tenant-testing-part-3-a37901054ec6) — Laravel 多租户测试