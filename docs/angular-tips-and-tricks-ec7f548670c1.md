# 棱角——技巧和窍门

> 原文：<https://itnext.io/angular-tips-and-tricks-ec7f548670c1?source=collection_archive---------0----------------------->

![](img/6f69727a298f7d36033522357bdb8119.png)

图片来源:[http://gatewayacademyindia . com/blog/gate/tips-and-tricks-to-crack-gate-exam/](http://gatewayacademyindia.com/blog/gate/tips-and-tricks-to-crack-gate-exam/)

> *这篇文章旨在展示一些可以在 Angular 中使用的技巧，使代码更加整洁*并提高应用程序的性能。

最近，我在 Angular 中完成了一个项目，在这个过程中，我学到了一些东西，可能会为其他开发人员提供一些有用的指导。所以，如果你是第一次潜入 **Angular** 或者只是在寻找一个更好的方法来管理你的 Angular 项目，这里有一些你可能想在这个过程中记住的提示和技巧。

1.  **取消订阅 RxJS observables** 每次组件或指令被销毁时，对 observables 的订阅仍然有效。所以重要的是取消订阅它来释放系统中的内存，否则，你会有内存泄漏。
    有许多方法，但我更喜欢使用 **takeUntil** 函数:

```
*import* { Component, OnDestroy, OnInit } *from* "@angular/core";
*import* { Observable, Subject, interval } *from* "rxjs";
*import* { takeUntil } *from* "rxjs/operators";

@Component({
    selector: "app-test",
    template: `<div>Test</div>`
})
*export class* TestComponent implements OnDestroy, OnInit {
    private destroyed$ = *new* Subject<*void*>();

    private ticks$ = interval(1000);

    public ngOnInit() {
        *this*.ticks$
            .pipe(takeUntil(*this*.destroyed$))
            .subscribe(data => console.log(data));
    }

    public ngOnDestroy() {
        *this*.destroyed$.next();
        *this*.destroyed$.complete();
    }
}
```

你可以在这里阅读更多关于 **takeUntil** 函数:[http://reactivex.io/documentation/operators/takeuntil.html](http://reactivex.io/documentation/operators/takeuntil.html)

如果你喜欢 decorators，你可以看看这个包，它提供了一个声明性的方法来取消订阅组件被破坏时的 observables:[https://github.com/NetanelBasal/ngx-take-until-destroy](https://github.com/NetanelBasal/ngx-take-until-destroy)

*PS:如果你正在使用****async pipe****就不用担心退订了，因为它* *会自动退订。此外，take(n)、takeWhile(谓词)、first()和 first(谓词)之外的方法会自行取消订阅。*

**2。使用类型脚本别名创建更性感的导入**

当代码库开始增长时，您将看到如下导入:

```
import { MyComponent } from '../../../../../../shared/components/mycomponent'
```

这是非常烦人的，因为你不知道你需要写多少点-点-斜线才能到达正确的位置。实际上，Typescript 编译器允许使用[路径映射](https://www.typescriptlang.org/docs/handbook/module-resolution.html)，所以我们可以像这样导入我们的文件:

```
import { MyComponent } from '@shared/components/mycomponent'
```

我们需要做的就是在`tsconfig.json`文件的`compilerOptions`部分定义`paths`和`baseUrl`属性。重要的是要知道所有的`paths`都是相对于`baseUrl.`来解析的

示例:

```
{
    "compilerOptions": {
    ...
    "baseUrl": "./src",
        "paths": {
            "@shared/*":["app/modules/shared/*"],
            "@core/*":["app/modules/core/*"]
        }
    ...
    }
}
```

**3。在*ngFor 中使用 track by**

当我们使用 ***ngFor** 指令时，DOM 的变化由对象标识来跟踪。这对于大多数情况来说没问题，但是随着不可变实践和 Redux 的引入，我们每次都有新的对象。这意味着每次 ***ngFor** 都会在 DOM 中渲染列表，但是我们知道 DOM 操作代价很大。

当我们将 **trackBy** 与* **ngFor** 一起使用时，它开始由给定标识而不是由对象标识跟踪的变更传播。

示例:

```
<div *ngFor="let post of posts;trackBy:trackByFn"></div>...identify(index,item){
    return item.id;
}
```

更多关于 **trackBy** 的信息可以在这里找到**:**[https://net basal . com/angular-2-improve-performance-with-track by-cc 147 b 5104 e 5](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5)

**4。使用接口代替类**

有时我们需要一些服务器数据的模型或定义。这些接口可用于此目的，而不会给最终输出带来额外开销。与类不同，接口在构建创建期间被完全移除，因此它们不会向我们的最终包添加任何不必要的代码。

**5。清除生产中的所有控制台日志**

在开发过程中，我们写了很多`console.logs`来调试我们的应用程序，有时我们会显示一些我们不想让用户看到的信息。我们可以使用几行用于生产构建的代码非常简单地删除它。

在`main.js`中添加此内容:

```
*import* { environment } *from* './environments/environment';

*if* (environment.production) {
    *// Remove console logs in production* window.console.log = () => {
    };
    enableProdMode();
}
```

6。使用和`[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)`检查您的包

分析 bundle 这是提升 app 性能的一个好的开始。可视化 webpack 输出有助于您理解包的组成，了解哪些模块占用了其中的空间，并识别不必要的依赖关系。

这就是观想的样子:

![](img/32c44fff422110094f27e11d34ee68b8.png)

(来自[github.com/webpack-contrib/webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)的屏幕记录)

关于这个工具的更多信息，你可以在这里阅读:[https://alligator.io/angular/bundle-size/](https://alligator.io/angular/bundle-size/)

**7。不要偷懒加载默认路由**

延迟加载模块这是一个很好的特性，可以帮助我们减少启动时间，按需加载代码。我们不需要在启动时加载所有东西，我们只需要在应用加载时加载用户期望看到的内容。一个不好的做法是延迟加载默认路由。

假设我们有以下配置:

```
const routes: Routes = [
  { path: '', redirectTo: '/mymodule', pathMatch: 'full' },
  { path: 'mymodule', loadChildren: './mymodule.module#MyModule' }
];
```

当用户打开应用程序时，他将被重定向到`/mymodule` 路由，这将触发`MyModule.`的延迟加载。这将触发一个额外的 HTTP 调用来下载`mymodule.module`，并将执行一些不必要的操作(解析和评估 JavaScript VM)。结果，这会降低初始页面呈现的速度。因此，将默认页面路由声明为非懒惰是一个很好的做法。

**8。使用自定义错误处理程序捕获所有错误**

**Angular** 提供了一个内置的**全局异常处理**服务，这样当一个错误发生时，这个服务将捕获它，并将错误细节打印在控制台中。我们可以扩展这个异常处理程序来添加一些额外的功能，比如出于分析或其他原因将错误发送到您的后端服务器。

*扩展 Angular 的错误处理程序*

扩展 Angular 的错误处理程序这是一个简单的任务。我们需要做的就是创建一个扩展 Angular 的`ErrorHandler`并覆盖`handleError`方法的类。

示例:

```
*import* {ErrorHandler} *from* '@angular/core';

*export class App*ErrorHandler *extends* ErrorHandler {
    constructor(){
        *super*(*false*);
    }

    public handleError(error: any): *void* {
        *// Add your logic here.
        super*.handleError(error);
    }
}
```

之后，我们应该在`app.module.ts:`中注册我们的自定义`AppErrorHandler`

```
[@NgModule](http://twitter.com/NgModule)({
    declarations: [ AppComponent ],
    imports: [ BrowserModule ],
    bootstrap: [ AppComponent ],
    providers: [
        {provide: ErrorHandler, useClass: AppErrorHandler}
    ]
})
```

更多信息可以在这里找到:[https://www . log Gly . com/blog/angular-exception-logging-made-simple/](https://www.loggly.com/blog/angular-exception-logging-made-simple/)

**奖金**:

**将您的 console.log 参数包装在一个对象文字中，以打印变量名及其值。**
`console.log(isLoggedIn)
console.log({ isLoggedIn })`

![](img/8ac9e1bbf47b3e17b6ffee99441b63a2.png)

来源:[https://Twitter . com/DaniStefanovic/status/1011923716085821440](https://twitter.com/DaniStefanovic/status/1011923716085821440)

感谢阅读这篇文章，我希望你喜欢它！如果你有一些其他的技巧和诀窍分享，写在评论里吧。