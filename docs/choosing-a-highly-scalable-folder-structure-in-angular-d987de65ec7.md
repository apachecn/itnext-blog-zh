# 如何为 Angular 项目定义高度可扩展的文件夹结构

> 原文：<https://itnext.io/choosing-a-highly-scalable-folder-structure-in-angular-d987de65ec7?source=collection_archive---------0----------------------->

![](img/8aba515fedad18e0a2c6803d89c82e3a.png)

马丁·范·登·霍维尔在 [Unsplash](https://unsplash.com/search/photos/structure?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

*更新！本文的代码现在可以从 GitHub 获得。您可以在这里查看项目*[](https://github.com/mathisGarberg/angular-folder-structure)**该项目的结构与本文中讨论的略有不同，但基于相同的概念和思想。**

# *前言*

*为我的 Angular 应用程序找到一个合适的文件夹结构是我努力了很长时间的事情。在我的第一个真实世界的 Angular 项目中，这成了一个真正的问题——尤其是当应用程序变大的时候。我最终不断地添加新功能，却没有真正的结构。这使得定位文件变得困难，并且对文件夹结构进行额外的改变变得耗时。但在构建 Angular 应用程序时，我确实很好地理解了该做什么和不该做什么。*

*本文的目标是帮助处于类似情况的其他开发人员。*

**我想强调的是，本文中使用的实践非常适用于我的单一用例，绝不应该严格遵循。项目的文件夹结构会因多种因素而改变。但是，如果您对侧重于多模块架构的结构感兴趣，并且非常关注可伸缩性，请继续阅读。**

**注意！[+]表示文件夹中有其他文件。**

```
*|-- app
     |-- modules
       |-- home
           |-- [+] components
           |-- [+] pages
           |-- home-routing.module.ts
           |-- home.module.ts
     |-- core
       |-- [+] authentication
       |-- [+] footer
       |-- [+] guards
       |-- [+] http
       |-- [+] interceptors
       |-- [+] mocks
       |-- [+] services
       |-- [+] header
       |-- core.module.ts
       |-- ensureModuleLoadedOnceGuard.ts
       |-- logger.service.ts
     |
     |-- shared
          |-- [+] components
          |-- [+] directives
          |-- [+] pipes
          |-- [+] models
     |
     |-- [+] configs
|-- assets
     |-- scss
          |-- [+] partials
          |-- _base.scss
          |-- styles.scss*
```

## *棱角分明的风格指南*

*在 Angular 中寻找最佳实践的逻辑起点是 [Angular 风格指南](https://angular.io/guide/styleguide)。该指南对语法、约定和应用程序结构提出了自己的观点。*

*对我来说真正突出的关键指导方针是“*构建应用程序，以便您可以快速定位代码*”和“*对实现有一个近期的看法和一个长期的愿景。从小处着手，但要记住应用程序的发展方向*。这意味着你不应该把自己局限在一个结构上，因为它会随着项目的不同而发生很大的变化。*

## *模块*

```
*|-- modules
       |-- home
           |-- components
           |-- pages
           |    |-- home
           |         |-- home.component.ts|html|scss|spec
           |
           |-- home-routing.module.ts
           |-- home.module.ts*
```

*[Tom Crowley](https://medium.com/@motcowley) 有一篇关于这个主题的非常有趣的文章([在这里找到](https://medium.com/@motcowley/angular-folder-structure-d1809be95542))，他从一个简单的棱角分明的项目到一个非常坚固的文件夹结构。我非常喜欢为页面和组件指定文件夹的模块的想法。它非常适合缩放，并且组件和页面逻辑是分离的。所以如果你对这个结构背后的思考过程感兴趣，就去那里吧。*

# *核心模块*

*`CoreModule`承担根`AppModule`的角色，但不是 Angular 在运行时引导的模块。`CoreModule`应该包含单体服务(这是通常的情况)，通用组件和其他功能，每个应用程序只有一个实例。为了防止在其他地方重新导入核心模块，您还应该在核心模块的构造函数中为它添加一个保护。*

```
*|-- core
       |-- [+] authentication
       |-- [+] footer
       |-- [+] guards
       |-- [+] http
       |-- [+] interceptors
       |-- [+] mocks
       |-- [+] services
       |-- [+] header
       |-- core.module.ts
       |-- ensureModuleLoadedOnceGuard.ts
       |-- logger.service.ts*
```

*`authentication`文件夹只是处理用户的认证周期(从登录到注销)。*

```
*|-- authentication
     |-- authentication.service.ts|spec.ts*
```

*`footer`和`header`文件夹包含全局组件文件，在整个应用程序中静态使用。这些文件将出现在应用程序的每个页面上。*

```
*|-- header
     |-- header.component.ts|html|scss|spec.ts
|-- footer
     |-- footer.component.ts|html|scss|spec.ts*
```

*`http`文件夹处理来自应用程序的 http 调用之类的东西。我还添加了一个`api.service.ts`文件，将所有通过我们的应用程序运行的 http 调用保存在一个地方。该文件夹包含用于与不同 API 路由交互的文件夹。*

```
*|-- http
     |-- user
          |-- user.service.ts|spec.ts
     |-- api.service.ts|spec.ts*
```

*Angular 4.x 引入了一个期待已久的处理 http 请求的特性——`HttpInterceptor`接口。这允许我们捕捉和修改来自 API 调用的请求和响应。文件夹是我发现特别有用的拦截器的集合。*

```
*|-- interceptors
       |-- api-prefix.interceptor.ts
       |-- error-handler.interceptor.ts
       |-- http.token.interceptor.ts*
```

*`guards`文件夹包含了我在应用程序中用来保护不同路径的所有防护措施。*

```
*|-- guards
     |-- auth.guard.ts
     |-- no-auth-guard.ts
     |-- admin-guard.ts* 
```

*模拟对于测试特别有用，但是您也可以使用它们来检索虚构的数据，直到后端建立起来。`mocks`文件夹包含我们应用程序的所有模拟文件。*

```
*|-- mocks
       |-- user.mock.ts*
```

*所有附加的**单例**服务都放在`services`文件夹中。*

```
*|-- services
     |-- srv1.service.ts|spec.ts
     |-- srv2.service.ts|spec.ts*
```

# *共享模块*

*`SharedModule`是任何共享组件、管道/过滤器和服务应该去的地方。当这些项目将被重用时，可以在任何其他模块中导入`SharedModule`。共享模块不应该依赖于应用程序的其他部分，因此也不应该依赖于任何其他模块。*

```
*|-- shared
     |-- [+] components
     |-- [+] directives
     |-- [+] pipes*
```

*components 文件夹包含所有“共享”组件。这是像`loaders`和`buttons`这样的组件，多个组件将从中受益。*

```
*|-- components
     |-- loader
          |-- loader.component.ts|html|scss|spec.ts
     |-- buttons
          |-- favorite-button
               |-- favorite-button.component.ts|html|scss|spec.ts
          |-- collapse-button
               |-- collapse-button.component.ts|html|scss|spec.ts*
```

*`directives`、`pipes`和`models`文件夹包含应用程序中使用的指令、管道和模型。*

```
*|-- directives
      |-- auth.directive.ts|spec.ts|-- pipes
     |-- capitalize.pipe.ts
     |-- safe.pipe.ts|-- models
     |-- user.model.ts
     |-- server-response.ts*
```

# *配置*

*config 文件夹包含应用程序设置和其他预定义的值。*

```
*|-- configs
     |-- app-settings.config.ts
     |-- dt-norwegian.config.ts*
```

# *式样*

*项目的全局样式放在`assets`下的`scss`文件夹中。*

```
*|-- scss
     |-- partials
          |-- _layout.vars.scss
          |-- _responsive.partial.scss
     |-- _base.scss
|-- styles.scss*
```

*`scss`文件夹只包含一个文件夹——partials。部分文件可以由其他 scss 文件导入。在我的例子中，`styles.scss`导入所有的分音来应用它们的样式。*

# *惰性装载*

*该应用程序使用延迟加载，这意味着在用户实际访问路线之前不会加载该模块。通过使用“模块”一节中描述的结构，您可以很容易地在主应用程序路由文件中引用每个模块。*

# *摘要*

*总之，在 Angular 中创建文件夹结构没有蓝图。根据你正在做的项目，结构会有很大的变化。本文介绍了一个使用多个模块的明确用例，这些模块又被分成页面和一组共享的组件。但这里有一些 github 回购协议的链接，结构更扁平:*

*   *[https://github.com/ngx-rocket/starter-kit/tree/master/src](https://github.com/ngx-rocket/starter-kit/tree/master/src)*
*   *[https://github . com/gothinkster/angular-real world-example-app](https://github.com/gothinkster/angular-realworld-example-app)*

*在 https://mathisgarberg.no 上查看我的网站*