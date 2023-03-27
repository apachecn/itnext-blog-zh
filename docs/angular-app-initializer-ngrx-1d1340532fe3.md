# Angular APP_INITIALIZER + NgRx

> 原文：<https://itnext.io/angular-app-initializer-ngrx-1d1340532fe3?source=collection_archive---------2----------------------->

![](img/7fcf862a568f48ba998b3a269970fca5.png)

**有什么问题？Angular 5 的 APP_INITIALIZER 期望你的代码会为它的解析返回一个承诺。**

**为什么这是个问题？**Angular 的大部分架构已经升级为使用 observables，但这一关键部分却没有。

**背景故事:**我们的应用程序允许深度链接到带有参数的页面，这些参数可以导致组件触发动作(比如搜索)，进而触发 HTTP 调用。问题是某些 HTTP 调用需要 auth 令牌、XSRF 令牌等，它们需要在执行之前存在。如果我们没有所需的令牌，我们的系统将产生失败的 HTTP 调用，并显示 401 和 500 错误。

我们可以解决这个问题的一个简单、乏味且可能容易出错的方法是确保所有的[效果](https://github.com/ngrx/platform/blob/master/docs/effects/README.md)手动依赖于我们现有的令牌。这将需要每个开发人员添加到他们的效果中，这将产生一个 HTTP 调用来复制和粘贴代码。这有很多重复的代码，这也会让修复 bug 变得很痛苦。不好了。

进入 Angular 的 app 生命周期钩子: [App 初始化](https://angular.io/api/core/APP_INITIALIZER)。阅读 Angular 团队的文档表明，应用程序初始化器是:

> 当应用程序被初始化时将被执行函数。

好吧，很简单。深入研究代码，你会发现你必须为它提供一个工厂/服务，这个工厂/服务有一个名为 load 的方法，这个方法返回一个承诺。

这一切都很好，除了我们正在使用 Angular + NgRx (Observables)构建我们的应用程序，并且我们需要能够在知道我们拥有所需信息后进行调度和操作。

我尝试了几种不同的方法，从使用 RxJs' [of](https://github.com/ReactiveX/rxjs/blob/master/compat/observable/of.ts) 操作符包装订阅，然后使用它的。toPromise()方法无济于事。

最后，我决定采用这种方法，即创建一个新的承诺来包装我们的订阅，然后如果我们过滤掉操作，只查看保存或错误，如果我们收到“保存”，我们就解决我们的承诺，如果我们收到“错误”，我们就拒绝承诺。

```
@Injectable()
class MyInitService { constructor(private subject: ActionsSubject) {} load(): Promise<any> {
    const promise = new Promise((resolve, reject) =>
      this.subject.pipe(
        filter(action => action instanceof InfoSaved ||
                         action instanceof InfoError)
        ).subscribe(action => {
          if (action.type === ActionType.INFO_SAVED) {
            resolve();
          } else {
            reject();
          }
        })
      })
      this.subject.next(new LoadInfo());
      return promise;
   }
}function MyInitServiceFactory(service: MyInitService): Function {
  return () => service.load();
}
```

Angular 团队提交了一个[bug](https://github.com/angular/angular/issues/15088)，所以这个解决方案暂时可以用。我希望这能对你有所帮助，或者如果你找到了不同的解决方案，我很乐意听听你的意见！

干杯！