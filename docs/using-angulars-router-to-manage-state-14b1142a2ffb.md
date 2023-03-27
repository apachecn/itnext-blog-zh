# 使用 Angular 的路由器管理状态

> 原文：<https://itnext.io/using-angulars-router-to-manage-state-14b1142a2ffb?source=collection_archive---------1----------------------->

![](img/c6b883c75c02a1abb7190c830069d85c.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fusing-angulars-router-to-manage-state-14b1142a2ffb)

啊，url，一个在现代应用中经常被忽略的保存状态的地方，但是它是如此强大。你有没有去过一个网站，搜索，设置你所有的过滤器，刷新，然后一切都消失了？这是为什么设计良好的路由器/url 状态很重要的主要例子。虽然这篇文章是专门关于 Angular 的，但是在 URL 中存储状态的主要思想适用于所有的框架/应用程序。

让我们来看一个搜索页面的 URL 示例。对我们的 url 的要求是，它需要说明需要导航到哪个页面，保存用户搜索的值，并保存几个搜索过滤器。

```
/products?search=playstation?department=electronics&priceLimit=100
```

上面的 url 告诉我们的应用程序关于它的状态的一些事情，所以让我们分解它。

```
products - The page we are viewingsearch=playstation - The search valuedepartment=electronics - the department filterpriceLimit=100 - the price limit filter
```

因为 Angular 的路由器是反应式的，所以使用这些值作为我们搜索页面的来源变得非常简单。

```
import { Component } from '@anguar/core';
import { ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs/Observable';
import { switchMap } from 'rxjs/operators';import { SearchService } from './search.service';export interface SearchParams {
  department: string;
  priceLimit: string;
  search: string;
}@Component({
  template: `
    {{searchParams | async | json}}     {{results | async | json}}
  `
})
class MyComponent {
  searchParams: Observable<SearchParams> = this.route.queryParams results = this.searchParams.pipe(
    switchMap(params => this.ss.findProducts(params))
  ); constructor(
    private ss: SearchService, 
    private route: ActivatedRoute
  ) {} updateSearch(params: Partial<SearchParams>) {
    this.router.navigate(['.'], {
      queryParameters: params, 
      queryParamsHandling: 'merge'
    })
  }
}
```

现在，如果我们看一下上面的组件定义，我们会注意到 3 件事。

1.  查询参数是可观察的，这意味着每当查询参数改变时，我们的订户(在本例中是通过“异步”管道的模板)都会得到通知
2.  我们可以使用 RxJs“switch map”操作符来触发新的搜索。
3.  要更新我们的观点，我们所要做的就是改变其中的一个参数。(请参见组件中的“updateSearch”方法。可以想象当有人选中一个复选框时它被调用)

遵循这种模式，不需要手动管理本地状态或使用全局应用程序商店，URL 是视图的状态，会告诉我们何时需要改变。

现在你的路线并不是存放所有东西的理想地方，所以让我们来看一些例子，看看它有什么好处，有什么坏处。

好:

*   过滤器:想想亚马逊页面或 JIRA。
*   任何需要用户容易共享的东西:过滤器也是在多页表中维护正确“页面”的一个好例子
*   刷新时需要维护的任何状态:筛选器、页面、当前选项卡
*   简单值:字符串、数字、布尔值或这些值的数组，如下所示:

```
products?list=first?list=second&list=third
```

不好:

*   复杂值:url 非常适合存储简单的图元或图元数组(图元数组通过添加一个同名的查询参数来表示)，但不适合存储更复杂的结构，如…
*   对象:有时你可能想把 JSON.stringify 作为一个对象，并把它放在一个查询参数中，请不要…请？

最后，把你自己想象成用户，如果你有机会不得不回到正确的标签页或者重新选择过滤器或者搜索参数，你的用户也会有很大的变化。

如果您有任何其他有趣的用例或使用反应式查询或路由参数的实现，请在评论中告诉我！