# 使用角度路由策略缓存组件

> 原文：<https://itnext.io/cache-components-with-angular-routereusestrategy-3e4c8b174d5f?source=collection_archive---------0----------------------->

![](img/4377c4a4ba58dd2ab82f0328f00eec2f.png)

[routereustrategy](https://angular.io/api/router/RouteReuseStrategy)供应商允许控制角度路线和组件生命周期的行为。

每次我们在组件之间导航时，Angular 都会破坏之前的组件并创建新的组件。在许多情况下，我们可能不希望这样，因为每次我们加载一个组件，我们可能会再次运行昂贵的操作，如 http 请求。

关于这个服务的文档并不多，所以我决定写下一些文档，并附上一个例子。

# 它是如何工作的？

**routerusestrategy**接口定义了 5 种方法:

*   *shouldReuseRoute* 每当我们在路线之间导航**时，这个方法就会被调用。*未来*是我们要离开的路线(不确定为什么叫未来)*当前*是我们要着陆的路线。如果返回 TRUE，则路由不会发生(这意味着路由没有改变)。如果它返回 FALSE，那么路由发生，其余的方法被调用。**

```
shouldReuseRoute(future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot): boolean {
  // default action 
  return future.routeConfig === curr.routeConfig;
}
```

*   *shouldAttach* :当我们在这条路线的组件上着陆时，刚刚打开的*路线* **调用这个方法。一旦组件被加载，这个方法被调用。如果该方法返回 TRUE，那么将调用*检索*方法，否则将从头开始创建组件:**

```
shouldAttach(route: ActivatedRouteSnapshot): boolean;
```

*   *retrieve* :如果 *shouldAttach* 返回 TRUE，则调用该方法，提供当前*路线*(我们刚刚着陆)作为参数，并返回一个存储的 *RouteHandle* 。如果返回 null，则没有任何效果。我们可以使用这种方法手动获取任何存储的 *RouteHandle* 。这不是由框架自动管理的。我们有责任实施它:

```
retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle | null;
```

*   *shouldDetach* :当我们**离开当前** *路径*时调用。如果返回 TRUE，那么将调用*存储*方法:

```
shouldDetach(route: ActivatedRouteSnapshot): boolean;
```

*   *store* :只有当 *shouldDetach* 返回 true 时，才会调用该方法。我们可以在这里管理如何存储*例程句柄。*我们在这里存储的内容将在*检索*方法中使用。它提供了我们要离开的路线和*路线手柄*:

```
store(route: ActivatedRouteSnapshot, detachedTree: DetachedRouteHandle): void;
```

# 使用示例

让我们看一个典型的例子:

1.  **搜索表单组件**加载*/搜索表单*路径。我们进行搜索
2.  导航到*/搜索结果*路径，该路径加载**搜索结果组件**
3.  点击一个结果并导航到 */detail/1* ，加载**细节组件**
4.  点击浏览器后退按钮返回导航到*/搜索结果*路径，再次加载**搜索结果组件**

步骤 **4** 将导致**搜索结果组件**再次运行相同的搜索，向 API 发出无用的 GET 请求。

我们希望缓存并避免在步骤 **4** 完全重新加载搜索结果组件(并保留滚动),并且仅在从步骤 **1** 导航到步骤 **2 时重新加载。**

**cache-route-reuse . strategy . ts:**

```
import { RouteReuseStrategy } from '@angular/router/';
import { ActivatedRouteSnapshot, DetachedRouteHandle } from '@angular/router';
export class CacheRouteReuseStrategy implements RouteReuseStrategy {
storedRouteHandles = new Map<string, DetachedRouteHandle>();allowRetriveCache = {
  'search-results': true
};shouldReuseRoute(before: ActivatedRouteSnapshot, curr:  ActivatedRouteSnapshot): boolean {if (this.getPath(before) === 'detail' && this.getPath(curr) === 'search-result') {    
    this.allowRetriveCache['search-results'] = true;
  } else {
    this.allowRetriveCache['search-results'] = false;
  } return before.routeConfig === curr.routeConfig;
}retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle | null {
  return this.storedRouteHandles.get(this.getPath(route)) as DetachedRouteHandle;
}shouldAttach(route: ActivatedRouteSnapshot): boolean {
  const path = this.getPath(route);
  if (this.allowRetriveCache[path]) {
    return this.storedRouteHandles.has(this.getPath(route));
  }

  return false;
}shouldDetach(route: ActivatedRouteSnapshot): boolean {
  const path = this.getPath(route);
  if (this.allowRetriveCache.hasOwnProperty(path)) {
    return true;
  } return false;
}store(route: ActivatedRouteSnapshot, detachedTree: DetachedRouteHandle): void {
  this.storedRouteHandles.set(this.getPath(route), detachedTree);
}private getPath(route: ActivatedRouteSnapshot): string {
  if (route.routeConfig !== null && route.routeConfig.path !== null) {
    return route.routeConfig.path;
  } return '';
}
}
```

**app.module.ts** 在 *providers* 配置中插入复用策略的提供者:

```
...
providers: [{
  provide: RouteReuseStrategy,
  useClass: CacheRouteReuseStrategy
}],
...
```

我使用 *shouldReuseRoute* 方法只是为了理解导航流程:用户是从细节组件导航回搜索结果组件吗？如果是，则加载搜索结果的最后一个缓存组件(*should attach*=>true， *retrieve* 被调用)，如果否，则重新创建组件并执行新的搜索。

由于 Angular 网站上的文档缺乏这方面的内容，所以我尝试对 RouteReuseStrategy 进行了详细的描述。 *shouldReuseRoute* 我认为有一个错误的参数名称，因为*未来*参数似乎是我们刚刚离开的路线，所以过去。

希望对其他开发者有帮助。

[https://github.com/angular/angular/issues/16713](https://github.com/angular/angular/issues/16713)