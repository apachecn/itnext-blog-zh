# 角度✔️中的清洁代码清单

> 原文：<https://itnext.io/clean-code-checklist-in-angular-%EF%B8%8F-10d4db877f74?source=collection_archive---------0----------------------->

![](img/7594137f293b4daad1fda083a05a7208.png)

格伦·卡斯滕斯-彼得斯在 [Unsplash](https://unsplash.com/search/photos/structure?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

# **简介**

Angular 已经迅速发展成为构建前端、跨平台 web 应用程序的最流行的框架之一。它提供了许多开箱即用的特性，比如路由系统、依赖注入框架、表单处理等。Angular 还强制您同时使用 TypeScript 和 RxJS，因为它已经是 Angular 生态系统的一部分。这种广泛的功能使 Angular 成为大型企业解决方案的良好候选。

虽然 Angular 是一个非常强大的框架，具有广泛的工具包，但它很难掌握，尤其是如果它是您的第一个 JavaScript 框架。为了降低复杂性，我决定整理一份干净的代码清单，其中包括我个人对编写干净的、可投入生产的 Angular 代码的建议。

# **背景**

随着我们成为更好的开发人员，构建和组织代码变得越来越重要。我们希望编写的代码能够提高可读性、可伸缩性、可维护性和性能，并最大限度地减少调试时间。

正如马丁·戈尔丁所说:

> 编写代码的时候，要把最终维护你的代码的人想象成一个知道你住哪儿的暴力精神病患者。

听起来令人不安，但这并没有减少信息的真实性。程序员才是真正的作者，其他开发者才是他们的目标受众。试图理解别人的代码所花费的时间占据了我们日常工作的很大一部分。因此，我们应该有意识地努力提高代码的质量。这对于创建一个成功的、可维护的产品是必不可少的。

# **风格指南**

在 Angular 中寻找最佳实践的逻辑起点是 [Angular 风格指南](https://angular.io/guide/styleguide)。这是一个关于语法、惯例和应用程序结构的固执己见的指南。《风格指南》介绍了首选的约定，并举例说明了为什么应该使用它们。

# 角度 CLI

[Angular CLI](https://cli.angular.io/) 是创建和使用 Angular 应用程序的绝佳工具。它消除了手动创建每个文件的繁琐工作，从而提高了工作效率。只需几个命令，您就可以:

*   从头开始创建项目
*   脚手架组件、指令和服务
*   Lint 您的代码
*   为应用服务
*   运行单元测试和端到端测试

你可以在这里找到更多关于 Angular CLI [的信息。](https://www.intertech.com/Blog/angular-tutorial-getting-started-with-the-angular-cli/)

# **文件夹结构**

随着应用程序规模的增长，建立一个便于管理和维护代码库的结构非常重要。无论你决定使用哪种结构，保持一致并选择整个团队都满意的结构是很重要的。

文章“[如何为你的 Angular 项目](/choosing-a-highly-scalable-folder-structure-in-angular-d987de65ec7)定义一个高度可伸缩的文件夹结构”讨论了这些主题，并且可以在决定你自己的结构时作为一个参考点。

# **编写可读代码**

如果你希望重构尽可能高效，拥有可读的代码是很重要的。可读的代码更容易理解，这使得调试、维护和扩展变得容易。

## **文件名**

添加新文件时，请注意您决定使用的文件名。文件名应该一致，并用点分隔来描述特征。

```
|-- my-feature.component.ts
or
|-- my-service.service.ts
```

## **变量和函数名**

您需要命名变量和函数，以便下一个开发人员理解它们。描述性强，使用有意义的名称——清晰胜于简洁*。*

这将帮助我们避免编写这样的函数:

```
function div(x, y)) {
 const val = x / y;
 return val;
}
```

希望更像这样:

```
function divide(divident, divisor) {
  const quotient = divident / divisor;
  return quotient;
}
```

## **写小的纯函数**

当我们编写函数来执行一些业务逻辑时，我们应该保持它们小而干净。小功能更容易测试和维护。当你开始注意到你的函数变得冗长和混乱时，把一些逻辑抽象成一个新的逻辑可能是个好主意。

这避免了像这样的功能:

```
addOrUpdateData(data: Data, status: boolean) {
  if (status) {
    return this.http.post<Data>(url, data)
      .pipe(this.catchHttpErrors());
  }
  return this.http.put<Data>(`${url}/${data.id}`, data)
    .pipe(this.catchHttpErrors());
  }
}
```

希望更像这样:

```
addData(data: Data) {
  return this.http.post<Data>(url, data)
    .pipe(this.catchHttpErrors());
}updateData(data: Data) {
  return this.http.put<Data>(`${url}/${data.id}`, data)
    .pipe(this.catchHttpErrors());
}
```

## **删除未使用的代码**

知道一段代码是否被使用是非常有价值的。如果您让未使用的代码留下来，那么在将来，很难或者几乎不可能确定它是否被真正使用。因此，您应该优先删除未使用的代码。

## **避免代码注释**

虽然有评论的情况，但你真的应该尽量避免。你不希望你的注释弥补你在代码中表达信息的失败。注释应该随着代码的更新而更新，但是更好的利用时间的方法是编写代码来解释它自己。不准确的评论比根本没有评论更糟糕，正如匿名的所说:

> 代码从不说谎，注释会。

这样可以避免编写这样的代码:

```
// check if meal is healthy or not
if (meal.calories < 1000 &&
    meal.hasVegetables) {
  ...
}
```

希望更像这样:

```
if (meal.isHealthy()) {
 ...
}
```

# **关注点分离**

Angular 是围绕关注点的分离而构建的。这是一种设计模式，它使我们的代码更容易维护和扩展，更具可重用性和可测试性。它帮助我们封装和限制组件的逻辑，以满足模板的实际需要，仅此而已。关注点分离是用 Angular 编写干净代码的核心，并使用以下规则集:

*   将应用程序分成多个模块。项目变得更有组织性、可维护性、可读性和可重用性，并且我们能够延迟加载特性。

```
|-- modules
|    |-- home
|    |     |-- home.spec|module|component|scss||routing.module|.ts
|    |-- about
|    |     |-- about.spec|module|component|scss|routin.module|.ts
```

*   当我们发现自己需要在应用程序其他部分重用业务逻辑时，我们应该考虑创建一个服务。服务是 Angular 的核心部分，也是放置可重用业务逻辑的好地方。服务最常见的用例是 HTTP 相关的事件。通过使用一个服务来管理我们的 HTTP 调用，我们确切地知道哪里需要进行更改，并且这些是唯一受影响的行。

*快提示！您还可以创建一个 API 服务，为您处理大量与 HTTP 相关的逻辑。我推荐看看* [*这个*](https://github.com/gothinkster/angular-realworld-example-app/blob/master/src/app/core/services/api.service.ts) *GitHub-example。*

*   您应该为您的应用程序创建一个类似“公共框架”的东西，在其中包含公共组件。当您不想让相同的代码污染您的组件时，这很方便。因为 Angular 不允许我们在不同的模块之间直接导入组件，所以我们需要将这些组件放在共享模块中。

```
// src/app/shared/components/reusable/resuable.component...export class ReusableComponent implements OnInit {
  @Input() title: string;
  @Input() body: string;

  @Output() buttonClick = new EventEmitter(); constructor() { } ngOnInit() {} onButtonClick(){
    this.buttonClick.emit('Button was clicked');
  }
}--------------------------------------------------------------------// You can then use this component directly inside the components of
// your choice// src/app/some/some.component@Component({
  selector: 'app-some',
  template: `<app-reusable [title]="'Awesome title!'"
               [body]="'Lorem ipsum'"
               (buttonClick)="buttonClick($event)>
             </app-reusable>`,
})
export class SomeComponent implements OnInit { constructor() {} ngOnInit() {} public buttonClick(e){
    console.log(e);
  }
}
```

*快速提示！我们可以通过使用* `*ng-content*` *标签来控制“父”组件的 HTML 内容。阅读更多关于内容投影用* `*ng-content*` [*这里用*](https://codecraft.tv/courses/angular/components/content-projection/) *。*

*   当我们发现自己处于多个 HTML 元素共享相同行为的情况时(例如，点击时某段文本被高亮显示)，我们应该考虑使用属性指令。属性指令用于改变 HTML 元素的行为或外观。

# **利用打字稿**

TypeScript 是 JavaScript 的超集，主要提供静态类型、类和接口。Angular 是基于 TypeScript 构建的，因此，将 TypeScript 与 Angular 结合使用是一种愉快的体验。它为我们提供了先进的自动完成，快速导航和重构。拥有这样的工具不应该被认为是理所当然的。

要充分利用 TypeScript:

*   我们应该总是使用接口来强制类实现声明的函数和属性。

```
// .../burger.model.ts
export interface Burger {
  name: string;
  calories: number;
}// .../show-burger.component.ts
this.burger: Burger;
```

*   我们应该利用类型联合和交集。这在处理来自 API 的数据时非常方便。

```
export interface Burger {
  ...,
  bestBefore: string | Date;
}
```

*快速提示！我们应该总是用不同于“any”的类型定义来声明变量和常量。严格类型的代码不容易出错。*

# **使用 TSLint**

极大地改善开发体验的一种方法是使用 TSLint。这是一个静态代码分析工具，我们在软件开发中使用它来检查 TypeScript 代码是否符合编码规则。有了这些规则，当你做错事的时候会给你一个错误。

## [带 TSLint 更漂亮](https://www.npmjs.com/package/tslint-config-prettier)

可以把 TSLint 和 beauty 结合起来。Prettier 是一个神奇的工具，它通过解析代码并重新打印代码来加强一致的风格，并且有自己的规则。使用 TSLint 进行更漂亮的设置为您的应用程序打下了坚实的基础，因为您不再需要手动维护您的代码样式。Prettier 负责代码格式化，TSLint 处理剩下的部分。

## [皮棉带粗](https://www.npmjs.com/package/husky)

即使有了这些规则，也很难维持下去。在将代码投入生产之前，您很容易忘记运行这些命令，这将导致错误的结果。解决这个问题的一种方法是使用 husky。Husky 允许我们在提交暂存文件之前运行自定义脚本，从而保持我们的生产代码整洁有序。

# RxJS 以角度表示

RxJS 是一个反应式编程库，允许我们处理异步数据流。Angular 与 RxJS 打包在一起，因此充分利用它是我们的巨大优势。

## 可管道化运算符

RxJS 5.5 引入了可管道操作符。这种基于管道的可观察组合方法允许我们在每个操作符的基础上进行导入，而不是导入所有内容。可管道化操作符也有树摇动的优势，允许我们构建自己的定制操作符，而不需要扩展`Observable.prototype`。

这将帮助我们避免编写这样的代码:

```
...const name = this.loadEmployees()
  .map(employee => employee.name)
  .catch(error => Rx.Observable.of(null));
```

希望更像这样:

```
const name = this.loadEmployees()
    .pipe(
      map(employee => employee.name),
      catchError(error => of(null))
    );
```

## 在模板中订阅

想象一下我们需要订阅多个流的情况。手动将每个属性映射到相应的值，并在组件被破坏时手动取消订阅，这将是一件令人头疼的事情。

使用异步管道，我们可以直接在模板中订阅流，而不必将结果存储在中间属性中。当组件被销毁时，订阅将终止，这使得订阅处理更容易，并有助于代码更干净。

这将帮助我们避免编写这样的代码:

```
@Component({
  ...
  template: `<items [items]="item">`
})
class AppComponent {
  items: Item[]; constructor(private itemService: ItemService) {} ngOnInit() {
    this.loadItems()
      .pipe(
        map(items => this.items = items;
      ).subscribe();
  } loadItems(): Observable<Item[]> {
    return this.itemService.findItems();
  }
}
```

希望更像这样:

```
@Component({
    ...
    template: `<items [items]="items$ | async"></items>`
})
class AppComponent {
  items$: Observable<Item[]>;

  constructor(private itemService: ItemService) {} ngOnInit() {
    this.items = this.loadItems();
  } loadItems(): Observable<Item[]> {
    return this.itemService.findItems();
  }
}
```

## 避免内存泄漏

虽然 Angular 在使用异步管道时负责取消订阅，但当我们自己必须这样做时，它很快就变得一团糟。未能取消订阅将导致内存泄漏，因为可观察的流是开放的。

解决方案是用`takeUntil`操作符组合我们的订阅，并使用一个在组件被破坏时发出值的 subject。这将完成可观察链，导致流取消订阅。

这有助于我们避免编写这样代码:

```
this.itemService.findItems()
  .pipe(
    map((items: Item[]) => items),
  ).subscribe()
```

希望更像这样:

```
 private unsubscribe$: Subject<void> = new Subject<void>(); ... this.itemService.findItems()
    .pipe(
       map(value => value.item)
       takeUntil(this._destroyed$)
     )
    .subscribe(); ... public ngOnDestroy(): void {
    this.unsubscribe$.next();
    this.unsubscribe$.complete();
    this.unsubscribe$.unsubscribe();
  }
```

*快速提示！现在，我们有了另一种方法来实现这一点，这就是所谓的子链接。是沃德贝尔* *为方便退订开发的* [*库。*](https://www.npmjs.com/package/subsink)

## 不要使用嵌套订阅

有些情况下，您可能需要使用来自多个可观察流的数据。在这种情况下，通常应该尽量避免所谓的嵌套订阅。嵌套订阅变得难以理解，并可能带来意想不到的副作用。我们应该使用像`switchMap`、`forkJoin`和`combineLatest`这样的可链接方法来压缩我们的代码。

这将帮助我们避免编写这样的代码:

```
this.returnsObservable1(...)
  .subscribe(
    success => {
      this.returnsObservable2(...)
        .subscribe(
          success => {
            this.returnsObservable3(...)
              .subscribe(
                success => {
                   ...
                },
```

希望更像这样:

```
this.returnsObservable1(...)
  .pipe(
    flatMap(success => this.returnObservable2(...),
    flatMap(success => this.returnObservable3(...)
   )
   .subscribe(success => {...});
```

*+快速提示！在处理多个流时，对于何时使用合适的操作符可能会产生混淆。我推荐看看这篇关于这个话题的文章。*

## RxJS 中的主题

一个主体既是被观察者又是观察者。因为主题允许我们将数据发送到可观察的流中，所以人们倾向于滥用它们。这在不熟悉 RxJS 的开发人员中尤其流行。为了确定什么时候是使用主语的好时机，我们首先要看看什么是主语，什么不是主语。

是什么科目**:**

*   主题是多播的，这意味着您可以在一个观察者上创建多个订阅。您可以用`next()`方法通知列表中的所有观察者。这是 RxJS 中主语的主要用法。
*   由于主体同时实现了可观察性和观察者，它们适用于输入和输出事件。

哪些科目**不是**:

*   RxJS 科目不能重复使用。不允许对已完成的主题调用`next()`方法。

*快速提示！除了普通科目之外，还有一些特殊类型的科目，如异步科目、行为科目和重放科目。你可以在这里* *阅读更多关于那些类型的科目* [*。*](https://alligator.io/rxjs/subjects/)

# **清理带有路径别名的导入**

考虑这样一种情况，您需要在应用程序层次结构的底层导入一些东西。这将导致一个如下所示的导入语句:

```
import 'reusableComponent' from '../../../shared/components/reusable/reusable.component.ts';
```

然后假设您需要更改这个文件的位置。然后，您必须更新使用该功能的每个文件的路径。这听起来不是很有效率吧？

我们可以通过使用别名引用我们的文件来清理这些导入，如下所示:

```
import 'reusableComponent' from '@app/shared/components/reusable.component.ts';
```

为了能够做到这一点，我们需要在我们的`tsconfig.json`文件中添加一个`baseUrl`和期望的`paths`:

```
{
  "compilerOptions": {
    ...
    "baseUrl": "src",
    "paths": {
      "@app:": ["@app/*"]
    }
  }
}
```

最常见的导入用例是来自核心和共享模块的组件和服务。为了避免写出这些特性的完整路径，我们将创建多个`index.ts`-文件，导出这些特性:

```
// src/app/shared/components/index.tsexport * from './reusable/reusable.component.ts';// src/app/sharedexport * from '/components';
```

我们现在可以像这样引用文件导入:

```
import 'ReusableComponent' from '@app/shared/';
```

*快速提示！在共享模块或核心模块中使用路径别名导入服务或组件时要小心。这可能会导致“循环依赖”。在那些情况下，你需要写完整的路径。*

# 用动画为你的应用增添趣味

动画被定义为从初始状态到最终状态的过渡。它是任何现代 web 应用程序不可或缺的一部分。动画不仅帮助我们创建了一个伟大的用户界面，而且还使应用程序变得有趣和好玩。一个结构良好的动画可以让用户参与到应用程序中，并增强用户体验。

# **懒惰加载你的模块**

如果您使用多模块架构，延迟加载您的模块可能是个好主意。只有用于向用户显示初始内容的特征模块应该被同步加载。使用延迟加载的优点是，在用户实际访问路由之前，模块不会被加载。这有助于通过只加载我们当前展示的模块来减少总的启动时间。

# 减少最终束尺寸

包的大小对应用程序的性能有很大的影响。这不仅仅是下载速度的问题，我们发送到浏览器的所有 JavaScript 都需要经过解析和编译才能执行。检查我们的包可能很困难，但是当我们看到膨胀来自哪里时就容易多了。`webpack-bundle-analyzer`插件允许我们可视化生产构建的内容。

# **状态管理**

当构建大型、复杂的应用程序时，其中有大量来自和去往数据库的信息，并且状态在多个组件之间共享，您可能会开始考虑添加状态管理库。通过使用状态管理库，您能够将应用程序状态保存在一个地方，这减少了组件之间的通信，并使您的应用程序更可预测、更易于理解。

# **结论**

Angular 是一个非常强大的框架，有很多功能。但是如果你是这个游戏的新手，它所有不同的选项和诸如此类的东西会让你不知所措。希望通过遵循这里概述的指南，一些概念对您来说变得更加清晰。虽然没有关于如何编写干净代码的蓝图，但是这里有一些关键的要点:

**编写可读代码**。专注于编写易于理解的代码。

**关注点分离**。封装和限制组件、服务和指令中的逻辑。每个文件应该只负责一个功能。

**利用打字稿**。TypeScript 提供了高级自动完成、导航和重构功能。拥有这样的工具不应该被认为是理所当然的。

**使用 TSLint** 。TSLint 检查 TypeScript 代码是否符合现有的编码规则。再加上漂亮和沙哑，这是一个很好的工作流程。

**RxJS 角度**。RxJS 附带 Angular，充分利用它对我们来说是一大优势。

**清理带有路径别名的导入**。我们可以通过使用路径别名来引用我们的文件，从而大大清理我们的导入。

**懒惰加载你的模块**。惰性加载通过只加载我们当前展示的模块来帮助我们减少应用程序的启动时间。

**状态管理(如有必要)**。状态管理是一个很好的工具，但是应该只用于大型复杂的应用程序，其中多个组件共享相同的状态。

关于此主题的其他有用资源:

*   https://medium . freecodecamp . org/best-practices-for-a-clean-and-performant-angular-application-288 e7b 39 eb6f
*   [https://code-maze.com/angular-best-practices/](https://code-maze.com/angular-best-practices/)
*   [https://github.com/ryanmcdermott/clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)

*我就这样了。如果您能就这些指南给我一些反馈，我将不胜感激。你有其他的推荐吗？你会做些不同的事情吗？*