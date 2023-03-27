# 角度搜索和分页

> 原文：<https://itnext.io/angular-search-pagination-41572ef0078f?source=collection_archive---------1----------------------->

## 带有用法示例

![](img/08ff836f74bdd05dc0c0c50d06093f08.png)

在现实世界的应用程序中，处理大量数据并为用户提供搜索和分页的可能性是很常见的。因此，在本文中，我将展示一种方法，如何创建一个处理带有*去抖*的搜索输入的**搜索组件**，以及一个处理不同数据页面请求的**分页组件**；两个人一起工作。

# **搜索组件**

让我们从我们的*搜索组件*开始。我们这里的目标是为用户提供一个输入，这样他就可以输入一个搜索字符串来过滤大集合的结果。然而，我们必须注意到，通过使用直接属性绑定，我们不会得到用户输入的每个字符之间的时间间隔(*去抖时间)。*

大多数时候这是一个问题，因为我们可能会根据提供的输入向服务器发出请求，而您希望等待用户完成输入；**这既是为了不做许多不必要的请求，也是为了提供更好的可用性。**

我们可以通过 *rxjs* 很简单的解决这个问题。首先，我们需要在组件中创建一个类型为 *string 的*主题*。*这是我们要用来**订阅输入值**的变化，并处理我们的*去抖*。

> “每一个主体都是可观察的和观察者。你可以**给**订阅一个**主题**，你可以调用**下一个**来馈入**值**以及错误并完成。”

```
private _searchSubject: Subject<string> = new Subject();
```

此外，我们将向我们的组件添加一个**输出**，它将在去抖动 之后发出一个带有输入值 ***的事件。这样，在用户输入完***之后，就可以将我们需要的任何请求(或者处理过滤器逻辑，如果分页是在前端完成的话)绑定到提供的搜索字符串**上。**

```
@Output() setValue: EventEmitter<string> = new EventEmitter();
```

然后，我们需要创建订阅本身。我们将使用 *rxjs 中的 ***管道()*** 函数来完成。*

> “您可以使用**管道**将操作员连接在一起。管道允许您将多个函数组合成一个函数。`pipe()`函数将您想要组合的函数作为其参数，并返回一个新函数，该函数在执行时会按顺序运行组合的函数。

```
constructor() {
  this._setSearchSubscription();
}private _setSearchSubscription() {
  this._searchSubject.pipe(
    debounceTime(500)
  ).subscribe((searchValue: string) => {
    // Filter Function
  });
}
```

我们还使用了*，一个由 *rxjs* 库提供的操作符；其接收在触发订阅之前应该等待多少毫秒的输入。这里我们使用的是*500 毫秒*，我相信这是一个相当不错的等待“按键间隔”的时间。*

**Angular Docs 有一个关于 rxjs 库的简单页面，你可以查看下面的链接:*[*https://angular.io/guide/rx-library*](https://angular.io/guide/rx-library) *深入解释 RxJS 不是本文的目的，但是你可以在他们的文档中找到更多:*[*https://rxjs-dev.firebaseapp.com/api*](https://rxjs-dev.firebaseapp.com/api)*

*然后，我们需要创建**方法**我们将**将**绑定到 HTML **输入**，当用户在搜索栏(HTML 输入元素)上键入时，它将**触发我们的*主题*** 。*

```
*public updateSearch(searchTextValue: string) {
  this._searchSubject.next( searchTextValue );
}*
```

*还有别忘了在 ***onDestroy*** 上**退订它**以免内存泄露*；*正如我之前在[理解角度生命周期挂钩](/understanding-angular-life-cycle-hooks-91616f8946e3)中所写的。*

```
*ngOnDestroy() {
  this._searchSubject.unsubscribe();
}*
```

*最后，我们需要我们的模板标记，在这个例子中，这将是非常简单的；但是当然，在实际应用中，人们会相应地定制和设计它。为了将**与 ***updateSearch*** 方法绑定，我们将使用 ***keyup*** 方法。***

```
*<input
  type="text"
  (keyup)="updateSearch($event.target.value)"
/>*
```

## *最终代码*

*因此，把我们所有的部分放在一起，这将是实现了*去抖*策略的搜索输入组件的最终代码。*使用时，该组件提供一个输入，该输入将触发一个事件，表明用户已经完成键入，因此我们可以向其添加任何我们需要的逻辑。**

```
*@Component({
  selector: 'app-search-input',
  template: `
    <input
      type="text"
      [placeholder]="placeholder"
      (keyup)="updateSearch($event.target.value)"
    />`
})
export class SearchInputComponent implements OnDestroy { // Optionally, I have added a placeholder input for customization 
  @Input() readonly placeholder: string = '';
  @Output() setValue: EventEmitter<string> = new EventEmitter(); private _searchSubject: Subject<string> = new Subject(); constructor() {
    this._setSearchSubscription();
  } public updateSearch(searchTextValue: string) {
    this._searchSubject.next( searchTextValue );
  } private _setSearchSubscription() {
    this._searchSubject.pipe(
      debounceTime(500)
    ).subscribe((searchValue: string) => {
      this.setValue.emit( searchValue );
    });
  } ngOnDestroy() {
    this._searchSubject.unsubscribe();
  }}*
```

# *分页组件*

*对于我们的分页组件，我们必须做两件主要的事情:**呈现**所有可能的**页码**供用户选择，以及**检测**何时**页面已经改变**以便从选择的页面中检索数据。*

*首先，我们将向组件添加一个*输入*属性，以接收创建分页所需的信息:页面大小的**和项目总数**的**。***

```
*export interface MyPagination {
  itemsCount: number;
  pageSize: number;
}export class PaginationComponent { public pagesArray: Array<number> = [];
  public currentPage: number = 1; @Input() set setPagination(pagination: MyPagination) {
    if (pagination) {
      const pagesAmount = Math.ceil(
        pagination.itemsCount / pagination.pageSize
      ); this.pagesArray = new Array(pagesAmount).fill(1);
    }
  }
}*
```

*你会注意到，我使用了一个 setter 来拦截接收到的输入属性，而不是直接输入属性。这是因为两件事:用**对*pagesAmount****进行舍入(如果由于任何原因它还没有舍入的话)，以及用 *pagesAmount* 填充一个数字数组。**

**好吧，那我们为什么需要一组数字呢？以便为用户呈现所有可能的页面。在 Angular 中，我们不能直接获取一个数字，并要求一个**NGF 循环*特定的次数，所以这是我通常用来克服这个问题的一个策略。**

**我们所做的是:使用一个数字数组，我们可以循环遍历它，并使用*索引*来检索我们想要的数字。因为我们想要的只是一个常规的有序数字列表，所以很容易实现，如下面的标记所示。**

```
**<span
  *ngFor="let page of pagesArray; let index = index"
  [ngClass]="{ 'active': currentPage === index + 1 } 
  (click)="setPage(index + 1)"
>
  {{ index + 1 }}
</span>**
```

**在这个标记中，我们**呈现所有可能的页面**供用户选择。我们添加了一个 *ngClass* 来设置当前选中页面的样式，以便让用户知道他当前在哪个页面。此外，我们还插入了一个 ***点击*动作**，它将**发出一个事件**，让父组件知道所选页面已经更改。**

```
**@Output() goToPage = new EventEmitter<number>();public setPage(pageNumber: number): void { // Prevent changes if the same page was selected
  if (pageNumber === this.currentPage)
    return; this.currentPage = pageNumber;
  this.goToPage.emit(pageNumber);
}**
```

**现在，让我们也添加**两个箭头**，让我们的用户生活更轻松；一到**后退**一页，一到**前进**一页。但是，当我们当前位于**第一页**时，我们将**隐藏*****左箭头*** ，当我们当前位于**最后一页**时，**隐藏*****右箭头*** 。**

```
**<span
 *ngIf="currentPage !== 1"
 (click)="setPage(currentPage - 1)"
>
  &lt; <!-- This is the simbol for '<' icon -->
</span><span
  *ngFor="let page of pagesArray; let index = index"
  [ngClass]="{ 'active': currentPage === index + 1 } 
  (click)="setPage(index + 1)"
>
  {{ index + 1 }}
</span><span 
  *ngIf="currentPage !== pagesArray.length"
  (click)="setPage(currentPage + 1)"
>
  &gt; <!-- This is the simbol for '>' icon -->
</span>**
```

**但是我们这里还有一个**问题**！如果我们的 *itemsAmount* 是几百，而我们的 *pageSize* 很小怎么办？或者甚至成千上万个项目？我们将会一次**呈现所有的页面**，并且我们将会有一个非常糟糕的可用性，因为所有的数字都挂在那里。**

**对于这个问题有一些可能的设计解决方案，比如隐藏中间的页面，或者在某个数字后隐藏最后的页面。我要展示的这个很容易实现，我相信在某些情况下会很有趣；也就是**将数字**从打印改为使用一个 ***选择 html 元素*，每页**作为一个选项。**

**因此，回到我们的标记，我们将在呈现页码的部分添加以下更改:**

```
**<!-- Here I decided the max pages amount before changing the rendering strategy to be 10, but you could change it however you want it and even create an environment variable if necessary -->
<ng-container *ngIf="pagesArray.length <= 10" >
  <span
    *ngFor="let page of pagesArray; let index = index"
    [ngClass]="{ 'active': currentPage === index + 1 }"
    (click)="setPage(index + 1)"
  >
    {{ index + 1 }}
  </span>
</ng-container><ng-container *ngIf="pagesArray.length > 10" >
  <select
    [ngModel]="currentPage"
    (ngModelChange)="setPage($event.target.value)"
  >
    <option
      *ngFor="let p of pagesArray; let index = index"
      [value]="(index + 1)" >
      {{ index + 1 }}
    </option>
  </select>
</ng-container>**
```

## **最终代码**

**因此，把我们所有的部分放在一起，这将是一个简单而有效的分页组件实现的最终代码。*它接收页面大小和项目总数的输入，并允许用户选择他想要查看的页面，触发指示所选页面的事件，以便处理所需的分页逻辑/请求。***

```
**export interface MyPagination {
  itemsCount: number;
  pageSize: number;
}@Component({
  selector: 'app-pagination',
  template: `
    <div class="pagination" >
      <span
        *ngIf="currentPage !== 1"
        (click)="setPage(currentPage - 1)"
      >
        &lt;
      </span> <ng-container *ngIf="pagesArray.length <= 10" >
        <span
          *ngFor="let page of pagesArray; let index = index"
          [ngClass]="{ 'active': currentPage === index + 1 }"
          (click)="setPage(index + 1)"
        >
          {{ index + 1 }}
        </span>
      </ng-container> <ng-container *ngIf="pagesArray.length > 10" >
        <select
          [ngModel]="currentPage"
          (ngModelChange)="setPage($event.target.value)"
        >
          <option
            *ngFor="let p of pagesArray; let index = index"
            [value]="(index + 1)" >
            {{ index + 1 }}
          </option>
        </select>
      </ng-container> <span 
        *ngIf="currentPage !== pagesArray.length"
        (click)="setPage(currentPage + 1)"
      >
        &gt;
      </span>`,
  styleUrls: ['./pagination.component.scss']
})
export class PaginationComponent { public pagesArray: Array<number> = [];
  public currentPage: number = 1; @Input() set setPagination(pagination: MyPagination) {
    if (pagination) {
      const pagesAmount = Math.ceil(
        pagination.itemsCount / pagination.pageSize
      ); this.pagesArray = new Array(pagesAmount).fill(1);
    }
  } public setPage(pageNumber: number): void {
    if (pageNumber === this.currentPage)
      return; this.currentPage = pageNumber;
    this.goToPage.emit(pageNumber);
  }
}**
```

# **用法示例**

**为了提供一个实际的例子，我将创建一个示例组件，其中我们有一个人员的分页列表，并希望允许用户搜索一个人的姓名，选择一个页面并过滤列表结果。**

## **搜索和分页模块**

**首先，我们将为这些组件创建一个**模块**，我们可以将它导入到我们需要的模块中。**

```
**@NgModule({
  declarations: [
    SearchInputComponent,
    PaginationComponent
  ],
  imports: [
    BrowserModule
  ],
  exports: [
    SearchInputComponent,
    PaginationComponent
  ]
})
export class SearchAndPaginationModule { }**
```

**然后，导入**模块。****

```
**...@NgModule({
  declarations: [
    ...
    ListComponent
  ],
  imports: [
    ...
    SearchAndPaginationModule
  ],
  providers: [
    ...
    MyService
  ],
  ...
})
export class ExampleModule { }**
```

**现在，让我们假设我们有一个**服务**与服务器通信来检索用户的信息。假设该方法接收一个**搜索字符串**和**当前页面**作为参数来过滤列表结果。**

**这里我们将保持我们的标记非常简单，只是为了展示我们创建的组件可以如何使用。下面是一个组件的最终代码，它同时使用了我们的**搜索组件** **&** **分页组件。****

```
**@Component({
  selector: 'app-list',
  template: `
    <app-search-input
      placeholder="Search by name"
      (setValue)="filterList($event)"
    ></app-search-input> <ul>
      <li *ngFor="let user of users" >
        {{ user.name }}
      </li>
    </ul> <app-pagination
      [setPagination]="{
        'itemsCount': totalUsersAmount,
        'pageSize': 10
      }"
      (goToPage)="goToPage($event)"
    ></app-pagination>`
})
export class ListComponent implements OnInit { public users: Array<User>;
  public totalUsersAmount: number = 0; private _currentPage: number = 1;
  private _currentSearchValue: string = ''; constructor(
    private _myService: MyService
  ) { } ngOnInit() {
    this._loadUsers(
      this._currentPage,
      this._currentSearchValue
    );
  } public filterList(searchParam: string): void {
    this._currentSearchValue = searchParam; this._loadUsers(
      this._currentPage,
      this._currentSearchValue
    );
  } public goToPage(page: number): void {
    this._currentPage = page; this._loadUsers(
      this._currentPage,
      this._currentSearchValue
    );
  } private _loadUsers(
    page: number = 1, searchParam: string = '' 
  ) {
    this._myService.getUsers(
      page, searchParam
    ).subscribe((response) => {
      this.users = response.data.users;
      this.totalUsersAmount = response.data.totalAmount; }, (error) => console.error(error));
  }}**
```

**希望有帮助😉**

## ****参考文献****

**[*https://angular.io/guide/rx-library*](https://angular.io/guide/rx-library)[https://it next . io/understanding-angular-life-cycle-hooks-91616 f 8946 E3](/understanding-angular-life-cycle-hooks-91616f8946e3)
[https://angular . io/guide/component-interaction # intercept-input-property-changes-with-a-setter](https://angular.io/guide/component-interaction#intercept-input-property-changes-with-a-setter)**