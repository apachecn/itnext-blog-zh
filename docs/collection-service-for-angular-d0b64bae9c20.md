# Angular 的集合状态管理服务

> 原文：<https://itnext.io/collection-service-for-angular-d0b64bae9c20?source=collection_archive---------1----------------------->

## *在这篇文章中，我将向你展示如何使用*[*ngx-collection*](https://github.com/e-oz/ngx-collection)*库，以及为什么你可能想要这样做。*

![](img/43ffb75971f687fbeaa5854da84ecc16.png)

《朗姆酒街》，弗雷德里克·贾德·沃，1922 年

# 目的

每个 web 应用程序都有一些列表，或者在内部使用一些实体/项目列表。因此，对于不同的 web 框架，有多个库可以提供帮助。与 ngx-collection 相比，它们既有区别又有相似之处，请选择您认为更适合和更舒适的产品:)

集合服务将帮助您操作集合:创建一个项目集合，并安全地更新一个或多个项目(不改变它们)。

此外，您将获得一个视图模型，可用于监控收集状态:

*   **正在读取**(当前正在运行某个请求以替换集合中的所有项目)
*   **正在创建**(正在运行向集合添加新项目的请求)
*   **正在更新**
*   **正在删除**
*   **正在更改**(某些项目当前正在更新或删除)
*   **正在保存**(当前正在更新或创建一些项目)
*   **正在处理**(至少有一个正在运行的请求)

此外，这个视图还有一个项目列表、它们的状态以及一些其他有用的信息。

使用这个视图模型和**异步**管道，您可以轻松地修改您的视图，以反映与集合相关的所有过程——微调器、按钮的“禁用”状态、卡片的“展开”/“折叠”状态、项目的“聚焦”/“选定”状态。

# 利益

✅的每一次突变都是异步的:

*   使用 OnPush 策略处理反应式应用程序；
*   在具有默认更改检测策略的组件中工作；
*   可以仅与“异步”管道和 subscribe()一起使用，也可以与某些存储一起使用。

✅服务公司并不固执己见:

*   它不会规定您应该如何与您的 API 或其他数据源进行通信；
*   您决定如何运行和取消您的请求:每个变异方法都返回一个可观察值，因此您可以决定如何编排它们:switchMap、mergeMap、concatMap、forkJoin 或其他；
*   数据源可以是同步的(用“rxjs/of”就行)。

✅安全保证:

*   100%不可变；
*   防止重复；
*   错误和异常将被正确处理；
*   严格类型化—高级类型检查和代码完成；

# 收藏品

假设你是一个艺术收藏家，你需要一个 app 给你喜欢的画作打分。

我们可以像这样创造一个新系列:

```
 const paintingsCollection = inject(CollectionService<Painting>({
   comparatorFields: ['url']
});
```

在这里，您可以注意到收集服务最重要的事情:它需要知道可用于比较对象并声明它们相等的字段。

因为在每一次变异中，我们都在重新创建列表(为了有一个不可变的集合)，通常的' === '(通过引用比较对象)是不够的。

内置的比较器将允许您使用一个或多个字段、复合字段或嵌套字段。或者，您可以定义您的函数或类，并将其用作比较器。

# 数据源

集合服务只有一个职责:操作集合而不修改项目本身。

为了提供这些项目，您需要一些服务——可以是使用 Angular HttpClient 的小型服务，或者是一些具有缓存、失效、交叉更新和其他功能的复杂 API 客户端——收集服务将使用它们提供的可观察对象。Collection Service 中没有用于与数据提供者通信的内置工具，它为您提供了实现的绝对自由和组合不同工具的充分灵活性。

在我们的示例应用程序中，我们将使用一个模拟 API 客户端行为的简单服务:

```
export class PaintingsService {

  getPaintings(): Observable<Painting[]> {
    return timer(1000).pipe(
      map(() =>
        [
          {
            url: '...',
            title: 'A Walk at Twilight',
            artist: 'Vincent van Gogh',
            date: '1889-1890',
          },
          {
            url: '...',
            title: 'Siesta',
            artist: 'Joaquín Sorolla',
            date: '1911',
          },
  // ...
        ]  // shuffle
          .map((value) => ({ value, sort: Math.random() }))
          .sort((a, b) => a.sort - b.sort)
          .map(({ value }) => value)
      )
    );
  }

  ratePainting(painting: Painting, rate: number): Observable<Painting> {
    return timer(2000).pipe(
      map(() => ({
        ...painting,
        rate,
      }))
    );
  }
}
```

其中`Painting`模型具有这样的结构:

```
export type Painting = {
  readonly url: string;
  readonly title: string;
  readonly artist: string;
  readonly date: string;
  readonly rate?: number;
};
```

所有的字段都是`readonly`以确保我们不会意外地改变我们的条目。

# 成分

现在，我们将使用我们的集合:组件，负责呈现和更新项目。

在这个实现中，我使用 NgRx 组件存储作为所有组件逻辑的位置。这不是必需的——您可以使用常规订阅来代替，但这是我推荐的做法。让我们不要转移注意力去讨论这背后的推理；)

```
export interface PaintingsListState {}

@Injectable()
export class PaintingsListStore extends ComponentStore<PaintingsListState> {
  readonly collection = inject(PaintingsCollection);

  constructor(private readonly api: PaintingsService) {
    super({});
    this.load();
  }

  private readonly load = this.effect((_) => _.pipe(
      switchMap(() => this.collection.read({
          request: this.api.getPaintings(),
        })
      )
    )
  );

  readonly rate = this.effect<{ painting: Painting; rate: number }>((_) =>
    _.pipe(
      switchMap(({ painting, rate }) =>
        this.collection.update({
          request: this.api.ratePainting(painting, rate),
          item: painting,
        })
      )
    )
  );

  readonly remove = this.effect<Painting>((_) =>_.pipe(
      mergeMap((painting) =>
        this.collection.delete({
          request: timer(1000),
          item: painting,
        })
      )
    )
  );
}
```

让我们看看我们的事件处理程序。
首先是`load`:

```
load = this.effect((_) => _.pipe(
      switchMap(() => this.collection.read({
          request: this.api.getPaintings(),
      })
)));
```

当`load`事件发生时，我们将调用`collection.read()`并使用`api.getPaintings()`请求。

然后是`rate`:

```
rate = this.effect<{ painting: Painting; rate: number }>((_) =>_.pipe(
      switchMap(({ painting, rate }) =>
        this.collection.update({
          request: this.api.ratePainting(painting, rate),
          item: painting,
      })
)));
```

这里我们使用输入参数，`painting`和`rate`。当一个事件发生时，我们调用`collection.update()`，请求是`api.ratePainting()`，这里我们需要另一个参数:`item`。
它应该是我们想要更新的项目(或者只是具有相同主键的任何对象)。

通过使用`switchMap`，我们将取消当前正在运行的`rate`请求(如果现在有正在运行的请求)，因此只有最后一个`rate`将被应用。收集服务将正确处理它，因为每个动作都是异步的。

这里最后一个是`remove`:

```
remove = this.effect<Painting>((_) =>_.pipe(
      mergeMap((painting) =>
        this.collection.delete({
          request: timer(1000),
          item: painting,
      })
)));
```

和`rate`只有两点不同:
1。我们使用`mergeMap`是因为我们允许并行删除项目。
2。我们使用`timer`作为`request`，因为我们的 API 客户端没有删除项目的方法(毕竟这只是一个模仿)。

我们的组件本身将会很小:

```
@Component({
  selector: 'paintings-list',
  templateUrl: './paintings-list.component.html',
  styleUrls: ['./paintings-list.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true,
  imports: [
    CommonModule,
    // ...
  ],
  providers: [PaintingsListStore],
})
export class PaintingsListComponent {
  protected readonly store = inject(PaintingsListStore);

  protected readonly vm$ = this.store.collection.getViewModel();
}
```

就是这样！
只有两个字段:商店和视图模型。

在模板中，我们将呈现一个列表、一个旋转器、一幅画、星级和一个“删除”按钮。为了简洁起见，去掉了一些细节。

```
<ng-container *ngIf="vm$ | async as $">

  <mat-card class="spinner" *ngIf="$.isReading">
    <mat-spinner></mat-spinner>
  </mat-card>

  <mat-accordion>
    <mat-expansion-panel
      *ngFor="
        let item of $.items;
        trackBy: store.collection.getTrackByFieldFn('url')
      "
    >
      <mat-expansion-panel-header>
        <mat-panel-title>{{ item.title }}</mat-panel-title>
      </mat-expansion-panel-header>

      <ng-template matExpansionPanelContent>
        <mat-card>
          <mat-card-header>
            <mat-card-title>{{ item.artist }}</mat-card-title>
            <mat-card-subtitle>{{ item.date }}</mat-card-subtitle>
          </mat-card-header>

          <img mat-card-image 
               [src]="item.url" 
               class="frame" 
               [ngClass]="{'mutating': $.isMutating}"/>
        </mat-card>

      </ng-template>

      <mat-action-row class="actions">
        <div class="stars" 
             [ngClass]="{'spin': $.isUpdating}"
         >        
          <mat-icon (click)="store.rate({ painting: item, rate: 1 })">
            {{
             item.rate ? 'star' : 'star_border'
            }}
          </mat-icon>
        </div>

        <button 
          mat-mini-fab 
          color="warn" 
          [disabled]="$.isDeleting"
          (click)="store.remove(item)"
        >
            <mat-icon>delete</mat-icon>
        </button>
      </mat-action-row>
    </mat-expansion-panel>
  </mat-accordion>
</ng-container>
```

添加一些样式，我们的应用程序就准备好了: [StackBlitz](https://stackblitz.com/edit/ngx-collection-example?file=src%2Fapp%2Fcollections%2Fpaintings.collection.ts) 🖼️

你可以注意到突出动作的视觉效果。
“删除”按钮可以调用一次，因此在项目被删除时，该按钮被显示为“禁用”。
可以无限制地点击星星，因此星星不会被禁用——只会保存最后一个评级，而之前的请求将被取消。

# 共享收藏

这个库的设计原则之一是:使用集合服务应该很容易创建简单的应用程序和组件，但它也应该可以用于任何复杂程度的应用程序和组件。

使用 [ngx-collection](https://github.com/e-oz/ngx-collection) ，您可以创建共享集合——如果您的一个组件将更新一个项目，所有其他组件将立即在页面上反映它。为此，您只需要修改“可注入的”装饰参数:

```
@Injectable({ providedIn: 'root' })
export class PaintingsCollection extends CollectionService<Painting> {
  constructor() {
    super({
      comparatorFields: ['url'],
    });
  }
}
```

# 文档和 API

收集服务有多种方法，但要开始使用它，您只需要知道其中的 4 种方法:

*   创造
*   阅读
*   更新
*   删除

就是这样！

其他方法在这里只是作为附加的实用程序和助手。
完整的 API 文档可以在资源库(GitHub) 中找到[——它应该回答你所有关于细节的问题，本文中没有解释。](https://github.com/e-oz/ngx-collection#readme)

# 工具

作为额外的奖励(不要求你使用或学习)，你可以找到管道和助手，如`getTrackByFieldFn`或`|itemStatus`管道。

> 如果这个库对你有用，我会很高兴的！