# 角度 10 评级表示例

> 原文：<https://itnext.io/angular-10-rating-form-example-4c47b629dd66?source=collection_archive---------6----------------------->

在本教程中，我们将通过示例了解如何使用 Bootstrap 4、HTML Select 和 Angular 10 表单创建评级组件。我们将使用来自`ng-bootstrap`的`ngb-rating`组件。我们还将看到如何在反应式表单中使用带有`ngFor`指令的 HTML 选择控件。如何分别使用`[ngValue]`和`value`属性将 select 元素绑定到 TypeScript 对象或字符串文字，以及如何分配默认值以从元素数组中进行选择。

> [*NgBoostrap*](https://ng-bootstrap.github.io/#/home)*是 Bootstrap UI 组件的角度适配版。使用 ng-bootstrap，我们可以很容易地将 bootstrap 库集成到我们的 Angular 项目中，并很容易地使用它令人敬畏的 UI 组件。
> Bootstrap 经过反复测试，可完全响应多种平台和屏幕尺寸。此外，它现在是几乎所有地方都采用的工业标准。*

HTML 表单在大多数 web 应用程序中是必需的。当您有多个选项并且希望用户在提交表单之前选择其中一个选项时，可以使用表单中的选择。在 Angular 中，可以将对象用于选项值，而不仅仅是字符串。

这是一个带有选择控件的组件模板示例:

在我们的角度组件中，我们需要一个包含一些城市的`cities`数组。

我们使用`value`属性将城市绑定到`select`，但是您也可以使用`ngValue`来代替。属性仅用于字符串，而属性可以用于对象。接下来我们将看到另一个用对象代替字符串使用`select`的例子。

如果我们有一个下拉菜单，需要显示 TypeScript 数组中对象的名称，这就很方便了。但是当从下拉列表中选择元素时，你需要选择数组元素的`id`来查询数据库。在这种情况下，我们需要使用`ngValue`,因为它处理 TypeScript 对象，而不仅仅是字符串。

在上一个教程中，您已经看到了如何从 npm 安装 Angular 10 CLI，所以让我们从创建一个新项目开始。

# 创建新的 Angular 10 项目

让我们从使用以下命令创建一个新的 Angular 10 项目开始:

```
$ ng new AngularRatingExample ? Would you like to add Angular routing? Yes ? Which stylesheet format would you like to use? CSS
```

# 添加 ng 引导

我们将看到如何在我们的项目中设置 Bootstrap 来使用`ng-bootstrap`设计我们的 UI。为此，我们首先需要使用以下命令将 npm 中的`bootstrap`和`ng-bootstrap`添加到我们的项目中:

```
$ cd AngularRatingExample $ ng add @ng-bootstrap/ng-bootstrap
```

这将为您的`angular.json`文件中指定的默认应用程序安装`ng-bootstrap`。

由于`ng-bootstrap`依赖于 i18n，我们还需要使用以下命令将包添加到我们的项目中:

```
$ ng add @angular/localize
```

接下来，打开`src/app/app.module.ts`文件，在`AppModule`的`imports`数组中添加`NgbModule`和`ReactiveFormsModule`，如下所示:

```
// [...] import { ReactiveFormsModule } from '@angular/forms'; 
import { NgbModule } from '@ng-bootstrap/ng-bootstrap';

@NgModule({ imports: [ // [...] ReactiveFormsModule, NgbModule ], })
```

接下来，打开`src/app/app.component.html`文件并更新如下:

我们简单地用一些用 Bootstrap 4 样式的 HTML 标记包装路由器出口。

# 创建 10 度角评级组件

接下来，让我们创建一个角度为 10°的组件来封装我们的表单。回到您的终端，运行以下命令:

```
$ ng generate component rating
```

# 使用带有反应式表单的`ngFor`指令的 HTML 选择控件

接下来，打开`src/app/rating/rating.component.html`文件并添加以下表单:

```
<form [formGroup]="form" (ngSubmit)="submit()"> <div class="form-group"> 
  <label>Book</label> 
  <select formControlName="book"> 
    <option *ngFor="let book of books" [ngValue]="book">{{book.name}}</option> 
   </select> 
</div> <div class="form-group"> 
   <ngb-rating [max]="5" formControlName="rating"></ngb-rating> </div> <button [disabled]="form.invalid || form.disabled" class="btn btn-primary">
Rate the book!</button> </form> 
```

我们在反应式表单中使用了带有`ngValue`的 HTML select 元素和`ngFor`指令。

是角度重复指令。它只是为列表中的每个元素重复宿主元素。

此示例中的语法如下:

*   `<option>`是主体元素。
*   `books`保存来自`RatingComponent`类的图书列表。
*   `book`保存列表中每次迭代的当前 book 对象。

接下来，更新`src/app/rating/rating.component.ts`文件，如下所示:

```
// [...] 
export class RatingComponent implements OnInit 
{ books = [ { name: 'Book 1' }, { name: 'Book 2' }, { name: 'Book 3' }, { name: 'Book 4' }, { name: 'Book 5' } ]; form = new FormGroup({ book: new FormControl(this.books[0], Validators.required), rating: new FormControl('', Validators.required), }); submit() { console.log(JSON.stringify(this.form.value)); 
this.form.reset(); } }
```

我们还使用`book: new FormControl(this.books[0], Validators.required)`从数组手册中为 select 提供了一个默认值。第一个参数是默认值。

接下来，我们需要将我们的评级组件添加到路由器配置中。打开`src/app/app-routing.module.ts`文件，更新如下:

```
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router'; 
import { RatingComponent } from '../rating/rating.component.ts'; const routes: Routes = [ {path: '', component: RatingComponent} ]; @NgModule({ imports: [RouterModule.forRoot(routes)], exports: [RouterModule] }) 
export class AppRoutingModule { }
```

最后，您可以使用以下命令启动开发服务器:

在这个 Angular 10 教程中，我们已经介绍了如何基于来自`ng-bootstrap`的`ngb-rating`组件构建一个带有评级组件的基本表单。我们已经看到了如何在反应式表单中使用带有 ngFor 指令的 HTML select 控件。如何将 select 绑定到 TypeScript 对象而不是 string，以及如何为 select 分配默认值。

*最初发布于*[*https://shabang . dev*](https://shabang.dev/angular-10-rating-form-example-ng-bootstrap-select-ngvalue/)*。*