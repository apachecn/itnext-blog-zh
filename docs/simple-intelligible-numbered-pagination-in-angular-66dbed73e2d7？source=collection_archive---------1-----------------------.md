# 简单易懂的角度编号分页

> 原文：<https://itnext.io/simple-intelligible-numbered-pagination-in-angular-66dbed73e2d7?source=collection_archive---------1----------------------->

![](img/09c5ce46d1d436a376d61e959b997f21.png)

[工作室媒体](https://unsplash.com/@studiomediainc?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/book?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

如果你在你的应用程序中使用 [Angular Material](https://material.angular.io/) 组件，你可能已经注意到它们的[分页器](https://material.angular.io/components/paginator/overview)有一个非常小但可扩展的 API。然而，有时候避免导入一个完整的角度材质模块和*修改*API 来满足你的需求是有意义的。

在本文中，我们将使用 Angular 和[函数式编程](https://en.wikipedia.org/wiki/Functional_programming)，用最少的代码构建一个非常简单的编号分页。我们将从定义我们的需求开始，然后打破这个过程的每一步来创建简单的内联分页，您可以用它来进行前端和后端分页。

# 要求

当你构建一个模块或组件时，你需要知道需求是什么。在设计方面，它可以改变一切。对于这种编号分页，我们需要:

1.  创建最少的信息*关闭*分页:开始索引，每页的总项数和结果数。
2.  分页组件上显示的最少信息:页面上显示的数字数(标尺)。
3.  显示第一个、最后一个、上一个和下一个按钮。当然，还有基于标尺的编号分页。
4.  光标(活动页面)在标尺上移动时的自然行为。
5.  仅当页面实际发生变化时发出页面变化事件**。**
6.  符合[坚实](https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)的原则。
7.  尽可能小，但要容易理解。

# 第一步

首先，我们需要创建一个[角度模块](https://angular.io/guide/ngmodules)来保存编号分页的所有逻辑。我们会尽可能地缩小规模:

```
*import* { NgModule } *from* '@angular/core';
*import* { CommonModule } *from* '@angular/common';@NgModule({
  imports: [CommonModule],
  declarations: [],
  exports: [],
})*export* class NumberedPaginationModule {}
```

根据你的栈，这个模块要么放在你的`shared/`文件夹中，要么作为你的 UI 库的一部分。

一旦我们有了这个模块，我们就可以创建我们的组件。

```
*import* { Component, ChangeDetectionStrategy, Input, Output } *from* '@angular/core';@Component({
  selector: 'numbered-pagination',
  templateUrl: './numbered-pagination.component.html',
  styleUrls: ['./numbered-pagination.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})*export* class NumberedPaginationComponent { maxPages: number; @Input() index: number;
  @Input() totalCount: number;
  @Input() pageSize: number;
  @Input() rulerLength: number;}
```

基本上，我们拥有创建分页组件所需的所有变量，其中:

> `*maxPages*` *代表我们分页的最后一页* `*index*` *是活动页面* `*totalCount*` *是项目总数* `*pageSize*` *是每页的结果数* `*rulerLength*` *是标尺上显示的页数*

如您所见，`maxPages`不是`@Input()`，因为定义最大页数是`NumberedPaginationComponent`的职责。

例如，如果我们的分页不仅仅是内联的 T30，而是由路由定义的，那么将`index`作为`@Input()`将启用动态分页。

我们现在将在模块中注册我们的组件。

```
*import* { NgModule } *from* '@angular/core';
*import* { CommonModule } *from* '@angular/common';
*import* { NumberedPaginationComponent } *from* './components';@NgModule({
  imports: [CommonModule],
  declarations: [NumberedPaginationComponent]
  exports: [NumberedPaginationComponent],
})*export* class NumberedPaginationModule {}
```

模块现在完全准备好了🎉

# 第二步

现在我们的`NumberedPaginationModule`已经可以导入了。我们需要在我们的部分上下功夫。我们已经看到我们的组件有几个`@Input()`，但我们现在将设置一些默认值，以便符合 TypeScript 严格模式，因为我们希望为我们的分页提供一些预设。

我们还希望在页面更改时发出一个事件，所以我们将添加一个简单的`@Output()`来通知父组件要加载的页面。

```
*import* { Component, ChangeDetectionStrategy, EventEmitter, Input, Output } *from* '@angular/core';@Component({
  selector: 'numbered-pagination',
  templateUrl: './numbered-pagination.component.html',
  styleUrls: ['./numbered-pagination.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})*export* class NumberedPaginationComponent { maxPages: number = 0; @Input() index: number = 1;
  @Input() totalCount: number = 100;
  @Input() pageSize: number = 5;
  @Input() rulerLength: number = 5; @Output() page: EventEmitter<number> = new EventEmitter<number>(); constructor() {
this.maxPages = Math.ceil(this.totalCount / this.pageSize);
  }}
```

对于这个例子，我将在构造函数中设置`maxPages`。

# 第三步

到目前为止，我们已经满足了构建编号分页的前两个需求。让我们继续第三个，模板。

我不打算详细说明，因为阅读和理解正在发生的事情是非常容易理解和简单的。我将只分享 HTML 和界面结果。

```
<ol *class*="pagination-container">
<li *(click)*="navigateToPage(1)">First page</li>
<li *(click)*="navigateToPage(index - 1)">Previous page</li>
<li
  **ngFor*="let page of pagination.pages; trackBy: trackByFn"
  *class*="pagination-number"
  *[class.active]*="page === pagination.index"
  *(click)*="navigateToPage(page)">
    {{ page }}
</li>
<li *(click)*="navigateToPage(index + 1)">Next page</li>
<li *(click)*="navigateToPage(maxPages)">Last page</li>
</ol>
```

> `*navigateToPage(n)*` *会处理导航并发出事件* `*trackByFn*` *会帮助渲染引擎画得更快*~超俗化 init？

通过一些小的设计，我们会得到这样的东西:

![](img/c1dde5179aae5600faf8fd995afad715.png)

标尺是标签之间的部分

# 第四步

是时候回到我们的组件了。基于我们在本文开头定义的需求，我们已经讨论了数字 1 到 3。这一步将完成剩余的需求。

在我们的模板中，有一个`*ngFor`循环遍历一个`pages`数组。这个数组属于一个叫做`pagination`的物体。首先，我们想为这个对象创建一个接口。

```
*export* interface NumberedPagination {
  index: number;
  maxPages: number;
  pages: number[];
}
```

这是一个非常简单的接口，但它将包含我们分页工作所需的一切。

现在让我们使用一个`getter`在我们的组件中创建`pagination`对象，因为我们希望这个对象对变化做出反应。

```
*import* { Component, ChangeDetectionStrategy, Input, Output } *from* '@angular/core';
*import* { NumberedPagination } *from* './../../interfaces';@Component({
  selector: 'numbered-pagination',
  templateUrl: './numbered-pagination.component.html',
  styleUrls: ['./numbered-pagination.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})*export* class NumberedPaginationComponent { maxPages: number = 0; @Input() index: number = 1;
  @Input() totalCount: number = 100;
  @Input() pageSize: number = 5;
  @Input() rulerLength: number = 5; @Output() page: EventEmitter<number> = new EventEmitter<number>(); constructor() {
this.maxPages = Math.ceil(this.totalCount / this.pageSize);
  } get pagination(): NumberedPagination {
    const { index, maxPages, rulerLength } = *this*;
    const pages = ruler(index, maxPages, rulerLength); *return* { index, maxPages, pages } *as* NumberedPagination;
  }}
```

我们的分页对象由活动页面`index`、分页的最大页数`maxPages`和我们希望在`ruler`上显示的页数`pages`组成。

`pages`变量由`ruler()`方法的输出组成。让我们来看看这个函数:

```
const ruler = (currentIndex: number, maxPages: number, rulerLength: number): number[] => {
  const array = new Array(rulerLength).fill(null);
  const min = Math.floor(rulerLength / 2); *return* array.map((_, index) => rulerFactory(currentIndex, index, min, maxPages, rulerLength));
};
```

在这个方法中，我们创建一个空数组，其大小与由预置或`@Input()`定义的`rulerLength`一样大。在我们的例子中，`rulerLength`就是`5`。我们将使用来自`min`的值为标尺创建一个逻辑行为，因为有 3 种不同的方式来移动标尺，但我们将在下面看到。在我们的例子中`min`将会是`2`。

> 注意:如果我们想要一把对称的尺子，我们可以将尺子的长度设为偶数。我们可以很容易地改变它的值:
> `rulerLength = rulerLength % 2 === 0 ? rulerLength + 1 : rulerLength`

这个`ruler`方法实际上是返回一个数字数组。例如，当用我们的预置创建组件时，它将返回`[1, 2, 3, 4]`。

在方法的回归中，我们可以看到一个叫做`rulerFactory`的新方法。`rulerFactory`旨在避免`if-else`语句。相反，我们将使用[工厂设计模式](https://en.wikipedia.org/wiki/Factory_method_pattern)从`ruler`函数中抽象出一堆 if。

在我们剖析`rulerFactory`方法之前，我们需要引入一个枚举。由于标尺有 3 种不同的行为，我们创建一个枚举来保存它们:

```
*export* enum RulerFactoryOption {
  Start = 'START',
  End = 'END',
  Default = 'DEFAULT',
}
```

> `*Start*` *将在活动页面到达标尺中间之前使用。* `*End*` *是最后一个行为。当我们接近最后一页的时候。* `*Default*` *是默认行为。*

现在让我们来看看`rulerFactory`:

```
const rulerFactory = (currentIndex: number, index: number, min: number, maxPages: number, rL: number): number => {
  const factory = {
    [RulerFactoryOption.Start]: () => index + 1
    [RulerFactoryOption.End]: () => maxPages - rL + index + 1,
    [RulerFactoryOption.Default]: () => currentIndex + index - min,
  }; *return* factory[rulerOption(currentIndex, min, maxPages)]();};
```

注意:为了更好的可读性，我不得不把 `*rulerLength*` *重新命名为 rL。*

这可能看起来有点晦涩难懂，但工厂所做的实际上非常简单。目标是在标尺上移动时有一个自然的流程。ruler 工厂使用`rulerOption`定义了 ruler 必须返回的行为。换句话说，除了开头和结尾，活动页面将始终位于屏幕中央。由于最后两种情况，标尺没有改变，只有突出显示的页面:

![](img/54955fdeb78e1161b1e0ff131d0734e6.png)

开始、默认和结束行为

为了定义我们是在标尺的起点、中间还是终点，我们使用了`rulerOption`函数:

```
const rulerOption = (currentIndex: number, min: number, maxPages: number): RulerFactoryOption => {
  *return* currentIndex <= min
    ? RulerFactoryOption.Start
    : currentIndex >= maxPages - min
    ? RulerFactoryOption.End
    : RulerFactoryOption.Default;
};
```

在本文结束之前，我们需要用上面模板中介绍的两种方法更新我们的组件:

```
navigateToPage(pageNumber: number): void {
  *if* (allowNavigation(pageNumber, *this*.index, *this*.maxPages)) {
    *this*.index = pageNumber;
 *this*.page.emit(*this*.index);
  }
}trackByFn(index: number): number {
 *return* index;
}
```

我们的`navigateToPage`方法将触发索引更新，并且只有在页面不同于当前页面并且页面存在的情况下才会发出`page`事件。

`trackByFn`帮助 Angular 跟踪`*ngFor`循环上的变化，并使绘画更快。严格来说是性能方面。

根据`allowNavigation`，如果没有改变，我们不想触发页面事件或更新索引，因此我们进行如下测试:

```
const allowNavigation = (pageNumber: number, index: number, maxPages: number): boolean => {
  *return* pageNumber !== index && pageNumber > 0 && pageNumber <= maxPages;
};
```

演示:[https://stack blitz . com/edit/simple-numbered-pagination-angular](https://stackblitz.com/edit/simple-numbered-pagination-angular)

# 结论

该组件只有 70 行代码，所以它是一个非常小的零依赖模块，可以很好地完成工作。

如果您更喜欢使用`if-else`语句而不是工厂语句，您也可以删除`rulerFactory`方法，直接在`.map()`中运行条件语句。代码更少，你也不需要枚举。但是依我看，遵循严格的原则总是更好。

我希望你喜欢这篇文章，并祝大家玩得开心。都是爱。