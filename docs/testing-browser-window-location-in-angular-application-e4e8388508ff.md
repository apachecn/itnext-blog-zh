# 测试浏览器窗口。角度应用程序中的位置

> 原文：<https://itnext.io/testing-browser-window-location-in-angular-application-e4e8388508ff?source=collection_archive---------0----------------------->

![](img/0bd95812b239e45f9f13e77aafbe7127.png)

在我们走之前，我需要澄清一些存在于角度世界与位置物体的混淆。有来自`@angular/common`的位置和默认可用的本地 DOM 位置。尽管 Angular 的版本提供了`.go()`功能，但实际上它只与路由器交互，并不像 DOM object 那样重新加载页面。所以，对于真正的浏览器交互，你必须使用 DOM 版本，这给你带来了一个问题如何测试它？

不幸的是，不可能窥探它，因为它不是一个可写的对象。

```
const spy = spyOn(location, ‘assign’).and.stub();
...Error: <spyOn> : assign is not declared writable or has no setter
```

如果你尝试这样做，这是典型的 Jasmine 错误。

## 救助依赖注射

我们需要用依赖注入机制将它注入我们的组件，而不是直接操纵`location`。注射后，我们可以用双倍测试来代替它。

DI 工作原理简介。如果您熟悉它，只需跳到代码示例。这是某个模块中的典型提供商部分

```
providers: [
  MyService,  // short way
  { provide: MyService, useClass: MyService } // full way
]
```

模块中的每个提供者声明实际上等同于完整的提供者描述，其中`MyService`不是作为类名，而是作为一个*标记*来标识您的服务。这个标记 Angular 取自关于类的 TypeScript 元数据。详见[依赖注入指南](https://angular.io/guide/dependency-injection)。

希望 Angular 为我们提供了`InjectionToken`类，我们可以用它来生成任何自定义令牌。

```
import { [InjectionToken](https://angular.io/api/core/InjectionToken) } from '@angular/core';
...
export const LOCATION_TOKEN = new InjectionToken<Location>('Window location object');
```

为了能够使用这个令牌，我们必须使用 Angular 中也提供的特殊属性装饰器`@Inject`。

```
@Inject(LOCATION_TOKEN) private location: Location
```

现在，让我们把所有的部分结合在一起

```
export const LOCATION_TOKEN = new InjectionToken<Location>('Window location object');@Component({
  providers: [
    { provide: LOCATION_TOKEN, useValue: window.location }
  ]
})
export class SomeComponent {
  constructor(@Inject(LOCATION_TOKEN) private location: Location) {} useIt() {
    this.location.assign('xxx');
  }
}
```

这种技术不仅可以用于注入位置对象，还可以用于注入没有特定类的任何其他实体，如其他 WebAPI 对象、应用程序配置等。