# 🔥利用 NgRx 状态共享和 NX cli 构建角度微前端

> 原文：<https://itnext.io/building-angular-micro-frontend-with-ngrx-state-sharing-and-nx-cli-7e9af10ebd03?source=collection_archive---------0----------------------->

如何在几乎不编码的情况下构建健壮的微前端架构；)

![](img/a55534ddbb9621143ac55aa803a39b78.png)[](https://medium.com/subscribe/@easy-web) [## 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。

### 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。通过注册，您将创建一个中型帐户，如果您还没有…

medium.com](https://medium.com/subscribe/@easy-web) 

阅读完本文后，您将能够构建一个具有共享状态管理的高度可伸缩的微前端应用程序。请注意，这是一个技术分步指南，不包括概念概述。如果你想了解更多关于微前端架构的知识，[先查看这篇文章](https://medium.com/@easy-web/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad)。

# 内容

*   [**Github 项目** **环节；**](#dd36)
*   [**简介；**](#01e9)
*   [**术语；**](#1ea3)
*   [**概念证明；**](#4594)
*   [**生成 app 骨架；**](#0ad5)
*   [**连接微前端；**](#da2e)
*   [**分享状态；**](#b7d6)
*   [**建立猫态；**](#30e9)
*   [**将猫状态连接到 UI；**](#694f)
*   [**结论；**](#5f2d)
*   [**了解更多**](#e865)

# **Github 项目链接**

[](https://github.com/Vitashev/mf-app) [## GitHub - Vitashev/mf-app:带 NgRx 状态共享的角度微前端

### 这个项目是用 Nx 生成的。🔎智能，可扩展的构建框架 Nx 支持许多插件，增加了…

github.com](https://github.com/Vitashev/mf-app) 

# 介绍

**微前端** —是客户端架构设计，当单独的组件或页面托管在单独的域中时。如果考虑到**路由、状态管理、**和**组件共享**，这听起来可能很难实现。今天，我们将构建一个完整的应用程序，它拥有可扩展的**微前端**所需的一切。我们将使用 **NX** 命令行生成大部分架构，然后添加最少的编码。

# 术语

*   **Shell** —主容器 app，可能有一个或多个嵌入式微 app；
*   **微 app** —拥有一个业务子域，集成到**外壳**，单独托管**的小 app；**

# 概念证明

这个简单的 **Angular** app 将由 2 个**微前端**域组成:

*   以及**图库**页面——一个将展示猫咪图库的**微应用**。用户可以切换到*选择*或*取消选择*猫，以在**主页**上显示所选择的猫。
*   **主页**页面——是一个**外壳**应用，显示了一个*选中的猫*的列表。
*   **NgRx** 将负责*状态管理*。

![](img/06e23f1f38ec7cf9b586361cc561b3d1.png)

流程如下:

*   **Shell** app 会调度动作调用*cats*API*API*并保存状态下的响应。
*   当在**图库**页面切换猫时，我们从*选中猫状态*中*添加*或*移除*猫。
*   *选择的猫状态*将被共享到一个**主页**上，该主页被托管在一个单独的域中。

整个架构将由 **NX** 命令行协调。它将生成应用程序的框架，我们只需要一些基本的编码将它粘在一起。

我们走吧！

# 正在生成应用程序框架

先从生成微前端 app 结构开始，创建**Nx**workspace；

```
**npx create-nx-workspace mf-app**
```

它提示:

*在新的工作空间里创建什么*——选择 ***清空*** 回车；

*用 Nx 云？*——**——*没有*——**——**回车；**

**NX** 生成了一个 *monorepo* 项目结构，由于我们将要使用 **Angular** ，我们需要安装所有必需的 **Angular** 库，事实上，它只有一个命令:

```
cd mf-app/ 
**npm install --save-dev @nrwl/angular**
```

很简单，接下来让我们生成一个 **shell** app，这将是我们的 **Home** 页面来显示*选中的猫*。

```
**npx nx g @nrwl/angular:app shell --mfe --mfeType=host --port=3000 --routing=true**
```

出现提示时:“*您希望使用哪种样式表格式？*—***CSS***和**回车；**

然后尝试运行它:

```
**nx run shell:serve-mfe**
```

很好，接下来我们需要生成单独托管的**图库** app，稍后我们会把它和 **shell** 连接起来。

```
**npx nx g @nrwl/angular:app gallery --mfe --mfeType=remote --port=3001 --host=shell --routing=true**
```

并运行它:

```
**nx run gallery:serve**
```

# 连接微前端

我们已经完成了构建托管在单独端口中的 2 个应用程序，现在是时候将它们连接在一起了。首先，我们需要链接到**外壳**路由的**主页**和**画廊**页面。

运行这个命令来生成**主页**组件:

```
cd apps/shell/src/app
**nx g @nrwl/angular:component Home --module=app.module.ts**
```

将路线添加到:

*   **apps/shell/src/app/app . module . ts**

```
RouterModule.forRoot([
    **{
        path: '',
        component: HomeComponent
    },** {
        path: 'gallery',
        loadChildren: () => import('gallery/Module').then((m) => m.RemoteEntryModule),
    },
],
{ initialNavigation: 'enabledBlocking' }),
```

转到 **shell** app 组件，添加页面导航 HTML:

*   **apps/shell/src/app/app . component . html**

```
**<ul>
    <li routerLink="/">Home</li>
    <li routerLink="/gallery">Gallery</li>
</ul>
<router-outlet></router-outlet>**
```

重建后，应用程序将如下所示:

![](img/f45cc752defa5384cee62f30d0c03eaa.png)

# 共享状态

我们已经完成了一半，最后一步是生成 **NgRx** 存储并与*选择的猫*共享状态。

我们将从生成一个共享的*数据存储库*开始，它将包含我们的共享状态。

```
**nx g @nrwl/angular:lib shared/data-store**
```

> *💡*注意:所有放置在**共享**文件夹中的库将作为一个单独的注入

然后，我们需要将库同时暴露给 **shell** 和**微 app** 。

> *💡*注意:如果您使用为您配置库的新版本 nx，请跳过这一步

添加别名:

```
sharedMappings.register(path.join(__dirname, '../../tsconfig.base.json'), [
   ** '@mf-app/shared/data-store'**
]);
```

在两个地方:

*   **apps/gallery/web pack . config . js**
*   **apps/shell/web pack . config . js**

在我们生成 **NgRx** 商店之前，让我们创建一个**图库**模块，我们的商店将链接到该模块:

```
**nx g @nrwl/angular:module gallery-store --project=shared-data-store**
```

最后，生成 **NgRx** 带有门面服务的商店的另一个命令:

```
**nx g @nrwl/angular:ngrx gallery --module=libs/shared/data-store/src/lib/gallery-store/gallery-store.module.ts --directory state --no-interactive --facade**
```

从*共享*库中导出**门面**服务和**画廊** **商店**模块:

*   **libs/shared/data-store/src/index . ts**

```
export * from './lib/shared-data-store.module';
**export * from './lib/gallery-store/gallery-store.module';
export * from './lib/gallery-store/state/gallery.facade';**
```

# 建立猫之州

在我们完成了主要架构的原型之后，是时候进行一些编码了。我们将构建 *API* 服务，创建*选择猫状态*，最后，将它与**主页**和**图库**粘合在一起。让我们从服务开始:

```
cd libs/shared/data-store/src/lib/gallery-store
**nx g @nrwl/angular:service api/GalleryApi**
```

添加一些代码:

*   **libs/shared/data-store/src/lib/gallery-store/API/gallery-API . service . ts**

```
export class GalleryApiService {
    constructor(**private http: HttpClient**) {} ** getCatsList() {
        const limit = 20;
        const url = `https://www.reddit.com/r/catswithjobs/.json?limit=${limit}`;** **return this.http.get(url).pipe(
            map((response: any) => {
                const cats = [] as any;
                response.data.children.forEach((res: any) => {
                    const title = res.data.title;
                    const id = res.data.id;
                    const url = res.data.preview?.images[0]?.resolutions[2]?.url;
                    if (url) {
                        cats.push({
                            id,
                            title,
                            url:url.replaceAll('&amp;', '&')
                        });
                    }
                });**
              **  return cats;
           })**
    **  );
    }**
}
```

更新依赖关系:

*   **libs/shared/data-store/src/lib/gallery-store/gallery-store . module . ts**

```
imports: [
    CommonModule,
    **HttpClientModule**,
    ....
],
providers: [GalleryFacade, **GalleryApiService**],
```

接下来，我们需要更新副作用:

*   **libs/shared/data-store/src/lib/gallery-store/state/gallery . effects . ts**

```
....
fetch({
    run: (action) => {
        **return this.galleryApiService.getCatsList().pipe(
            map((res) => GalleryActions.loadGallerySuccess({
                    gallery: res,
                })
            )
        );**
    },
....
constructor(
    private readonly actions$: Actions,
    **private galleryApiService: GalleryApiService**
) {}
....
```

然后让我们添加*切换选择猫*动作:

*   **libs/shared/data-store/src/lib/gallery-store/state/gallery . actions . ts**

```
**export const toggleSelectCat = createAction(
   '[Gallery] Toggle Select Cats',
   props<{ cat: any }>()
);**
```

实施*减速器*:

*   **libs/shared/data-store/src/lib/gallery-store/state/gallery . reducer . ts**

```
...
export interface State extends EntityState<GalleryEntity>{
    selectedId?: string | number;
    **selectedCats: Map<string, any>;**
...
export const initialState: State = galleryAdapter.getInitialState({
    **selectedCats: new Map(),**
    loaded: false,
});
....
const galleryReducer = createReducer(
    initialState,
    **on(GalleryActions.toggleSelectCat, (state, { cat }) => {
        const newState = { ...state };
        if (newState.selectedCats.has(cat.id)) {
            newState.selectedCats.delete(cat.id);
        } else {
            newState.selectedCats.set(cat.id, cat);
        }
        return newState;
    }),**
...
```

添加*选择猫状态*选择器:

*   **libs/shared/data-store/src/lib/gallery-store/state/gallery . selectors . ts**

```
**export const getSelectedCats = createSelector(
    getGalleryState,
    (state: State) => state.selectedCats
);**
```

最后一步是更新**门面**服务:

*   **libs/shared/data-store/src/lib/gallery-store/state/gallery . facade . ts**

```
**...
selectedCats$ = this.store.pipe(select(GallerySelectors.getSelectedCats));**constructor(private readonly store: Store) {}**isCatSelected(catId: any) {
    return this.selectedCats$.pipe(
        map((selectedCats) => selectedCats.has(catId))
    );
}****toggleSelectCat(cat: any) {
    this.store.dispatch(GalleryActions.toggleSelectCat({ cat }));
}**
...
```

我们已经完成了*状态*逻辑的实现，这只是一点编码，但是主要部分已经准备好了。最后一点是——将状态与我们的 UI 连接起来。

# 将 cats 状态连接到 UI

我们需要做的第一件事是将我们的**图库** **商店**模块连接到 **shell** app，它将对**商店**和**门面**服务进行依赖注入。

*   **apps/shell/src/app/app . module . ts**

```
imports: [
    BrowserModule,
    **StoreModule.forRoot({}),
    EffectsModule.forRoot([]),
    GalleryStoreModule,**
...
```

> *💡*注意:如果您在控制台中看到这样的警告:
> 
> 警告:未指定所需的版本，无法自动确定版本。在描述文件(/Users/macbook/Desktop/projects/micro-frontend/MF-app/node _ modules/@ angular/common/package . JSON)中找不到“[”所需的版本。它需要在依赖项、开发依赖项或对等依赖项中。](http://twitter.com/angular/common)
> 
> 这意味着 webpack 找不到共享包的版本，您需要显式地定义它。为了修复它，在 **shell** 和**micro**app web pack . config . js 中显式设置它

```
shared: {
   '@angular/core': {
      singleton: true,
      strictVersion: true,
      **requiredVersion: '12.2.9',**
   },
   '@angular/common': {
      singleton: true,
      strictVersion: true,
      **requiredVersion: '12.2.9',**
   },
...
```

好了，我们还有几个步骤，让我们将 UI 添加到**图库**页面。

*   **apps/gallery/src/app/remote-entry/entry . component . ts**

```
**import { Component } from '@angular/core';
import { GalleryFacade } from '@mf-app/shared/data-store';****@Component({
   selector: 'ng-mfe-gallery-entry',
   template: `<div class="container">
      <ng-container *ngFor="let cat of cats | async">
         <div class="child" (click)="toggleSelectCat(cat)" [ngClass]="{ selected: isSelected(cat.id) | async }">
             <h3>{{ cat.title }}</h3>
            <div>
              <img [src]="cat.url" alt="" />
            </div>
          </div>
       </ng-container>
   </div>`,
   styles: [`
      .container {
          display: grid;
          width: 100%;
          grid-template-columns: repeat(4, 1fr);
      }
      .selected {
          border: 3px solid purple;
      }
      img {
          width: 20vw;
      }`,
    ],
})
export class RemoteEntryComponent {
    cats = this.galleryFacade.allGallery$ as any;
    selectedCats = this.galleryFacade.selectedCats$;
    constructor(private galleryFacade: GalleryFacade) {}** **toggleSelectCat(cat: any) {
        this.galleryFacade.toggleSelectCat(cat);
    }** **isSelected(catId: any) {
        return this.galleryFacade.isCatSelected(catId);
    }
}**
```

此外，更新 shell 应用程序中的**主页**页面组件

*   **apps/shell/src/app/home/home . component . ts**

```
**import { GalleryFacade } from '@mf-app/shared/data-store';
import { Component, OnInit } from '@angular/core';
import { map } from 'rxjs/operators';****@Component({
    selector: 'mf-app-gallery-entry',
    template: `<div class="container">
       <ng-container *ngFor="let cat of cats | async">
           <div class="child">
               <h3>{{ cat.title }}</h3>
               <div>
                   <img [src]="cat.url" alt="" />
               </div>
           </div>
       </ng-container>
    </div>`,
    styles: [`
        .container {
            display: grid;
            width: 100%;
            grid-template-columns: repeat(4, 1fr);
        }** **img {
            width: 20vw;
        }`,
    ],
})****export class HomeComponent {
    cats = this.galleryFacade.selectedCats$.pipe(
        map((selectedCats: any) => Array.from(selectedCats.values()))) as any;** **constructor(private galleryFacade: GalleryFacade) {}
}**
```

最后，从 shell 中的 app 组件调用 *API* :

*   **apps/shell/src/app/app . component . ts**

```
...
**export class AppComponent implements OnInit {
    constructor(private galleryFacade: GalleryFacade) {}
    ngOnInit(): void {
        this.galleryFacade.init();
    }
}**
```

不要忘记清理以下所有样式:

*   **apps/gallery/src/app/app . component . CSS**
*   **apps/shell/src/app/app . component . CSS**

享受最终结果:

![](img/06e23f1f38ec7cf9b586361cc561b3d1.png)

# 结论

该应用程序可以使用相同的命令轻松扩展为另一个**微型**应用程序。如果你学到了新的东西，请分享并留下评论。

[](https://medium.com/subscribe/@easy-web) [## 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。

### 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。通过注册，您将创建一个中型帐户，如果您还没有…

medium.com](https://medium.com/subscribe/@easy-web) [](https://medium.com/@easy-web/membership) [## 通过我的推荐链接加入 Medium 维塔利·舍甫琴科

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

medium.com](https://medium.com/@easy-web/membership) 

# **了解更多信息**

[](https://medium.com/@easy-web/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad) [## 微前端如何改变 Angular 的未来？

### 让我们来看看为什么 Angular 是最好的微前端

medium.com](https://medium.com/@easy-web/how-micro-frontend-changes-the-future-of-angular-bb4deb2cfdad)