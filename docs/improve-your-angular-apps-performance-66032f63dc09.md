# 如何提高 Angular 应用程序的性能

> 原文：<https://itnext.io/improve-your-angular-apps-performance-66032f63dc09?source=collection_archive---------1----------------------->

充分利用 Angular 的性能。

![](img/4c1d4615b95d5e23e5c86f565711fac2.png)

# 减少您的包裹大小

如果不需要支持老浏览器，就掉 ES5 支持。所有现代浏览器都支持 ES6。

1.  转到您的`tsconfig.json`，将“目标”从 es5 更改为 es6
2.  当构建你的应用程序时，使用`--es5-browser-support=false`和`ng build`从你的包中排除 es5 浏览器聚合填充，或者从`angular.json`中将其永久设置为 false。
3.  从应用中移除未使用的依赖项。
4.  让你的角服务树摇动。不要将它们导入并添加到你的 ngModule 中的`providers`数组中。相反，在用`@Injectable()`装饰您的服务时，使用`providedIn`属性，只有当它被注入或用于任何其他组件/服务时，才让它包含在包中，如下所示

```
@Injectable({
  providedIn: 'root'
})
export class MyService {}
```

5.使用差动加载。如果您使用的是 Angular version 8+版本，您应该能够使用差异加载来避免向现代浏览器提供不必要的聚合填充。只需将包含以下内容的 browserslist 文件添加到您的项目中，并注释掉除 zone.js 之外的所有从`src/pollyfills.ts`导入的内容，并确保您的 tsconfig.json 中有“`target": “es2015"`。

```
> 0.5%
last 2 versions
Firefox ESR
Chrome 41 # Support for Googlebot
not dead
not IE 9-11 # For IE 9-11 support, remove 'not'.
```

6.启用常春藤[https://next.angular.io/guide/ivy](https://next.angular.io/guide/ivy)

# 延迟加载和代码分割

有了 Angular，就可以非常容易地基于路由对模块进行延迟加载

代替

```
// app-routing.module.ts
const routes: Routes = [{ path: home', component: HomeComponent }, { path: 'about', component: AboutComponent
 }, { path: '', redirectTo: 'home', pathMatch: 'full'
}]
```

使用

```
// app-routing.module.ts
const routes: Routes = [{ path: home', component: HomeComponent}, { path: 'about', loadChildren: './about/about.module#AboutModule'
}, { path: '', redirectTo: 'home', pathMatch: 'full'
}]
```

注意，我们没有延迟加载 home 组件，因为它是默认路径，这是初始加载所需要的。延迟加载将导致 Angular 触发另一个 http 请求，这实际上会损害我们应用程序的性能。

# **使用服务人员缓存资产**

服务工作者是一项令人敬畏的技术，它允许您开发渐进式 Web 应用程序(PWA ),即使您不想构建 PWA，也可以使用它们来缓存资产和 HTTP 请求，几乎可以实现即时加载。Angular 使其非常容易通过 angular-cli 选择加入。

你要做的就是

```
ng add @angular/pwa
```

注意:只有当你的网站支持 HTTPS 的时候，你才可以安装服务人员。

# 避免内存泄漏

取消订阅您的事件监听器、observables 和`ngOnDestroy`中的`setIntervals`，否则它们会留在内存中，即使您的组件被破坏，它们也会继续运行。

# 去抖&节流你的事件监听器

如果您有一个频繁触发的事件，如输入更改事件或滚动事件，最好根据用例去抖或抑制事件监听器，以避免每帧多次运行逻辑，影响应用的 FPS 和性能。

# 记忆和缓存昂贵的功能

这实际上不是角度特定的，但是如果你有被多次调用的昂贵的函数。您总是可以缓存结果并从缓存中返回结果，而不是再次计算结果。

# 最小化 DOM 元素

不用创建你不需要的包装 div，或者避免不能在同一个 div 上应用两个指令的问题，比如`*ngIf`和`*ngFor`，你可以使用`<ng-container>`，它不会给你的树增加额外的 DOM 元素。

# 优化变化检测

这些变化需要对 Angular 以及变化检测的工作原理有更深入的了解。

1.  使用 *OnPush* changeDetection。在组件中设置`changeDetection:ChangeDetectionStrategy.OnPush`实际上是告诉 Angular 避免深入检查输入数组/对象，并且只在输入引用不相等的情况下运行组件子树的变化检测。这可以节省框架进行深度相等检查的时间，以及该组件和该组件的所有其他子组件的变更检测时间，这些子组件将因此被标记为脏的。
2.  通过使用*轨迹。如果您使用可能随时间变化的`*ngFor`来呈现一个项目数组，您可以告诉 Angular 将每个 DOM 元素与该数组中某个项目的值绑定在一起，比如项目的 id。下次数组改变时，Angular 将只呈现/更新改变的项目，而不是重新呈现整个 DOM 元素数组。*
3.  使用*ng zone . runoutsidangeral .*如果您在组件中使用类似`setTimeout`、`setInterval`和`Promise`的异步 API，这些 API 不会在回调中改变组件的状态，也不需要触发更改检测，您可以告诉 Angular 在执行回调时不要触发更改检测。

```
ngZone.runOutsideAngular(() => {
  setTimeout(() => {
    doSomethingThatDoesntUpdateState() }, 5000)})
```

或者从您的应用程序中完全禁用 ngZone，并使用`applicationRef.tick()` 手动触发异步 javascript APIs 的变更检测，以获得更高级的用例

```
platformBrowserDynamic().bootstrapModule(AppModule, {
      ngZone: 'noop';
})
```

# 结论

您还可以做其他事情来优化您的应用程序的性能，其中大多数已经默认启用，如 Pure Pipes、enableProdMode、AOT 和带有`ng build --prod`的构建优化器。

这篇文章的灵感来自 [Minko Gechev](https://medium.com/u/a7db01a91fcb?source=post_page-----66032f63dc09--------------------------------) 写的[角度性能清单](https://github.com/mgechev/angular-performance-checklist)，如果你对你可以做的其他性能优化和更详细的解释感兴趣，你应该看看。

如果你喜欢我的内容，愿意支持我，你可以[请我喝啤酒](https://www.buymeacoffee.com/khaledosman)