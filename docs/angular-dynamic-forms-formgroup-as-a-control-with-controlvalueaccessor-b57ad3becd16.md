# Angular-form group 中的动态表单作为具有 controlValueAccessor 的控件

> 原文：<https://itnext.io/angular-dynamic-forms-formgroup-as-a-control-with-controlvalueaccessor-b57ad3becd16?source=collection_archive---------6----------------------->

![](img/cc652904c58590fa771a5d6588f500cd.png)

几个月前，我写了一篇关于 Angular 的"[动态表单模式](/dynamic-forms-in-angular-the-pattern-beyond-the-buzz-da12f560b0d4)的文章，在那篇文章中，我回顾了**反应式表单 API** ，并试图解释通过迭代预定义的描述符来动态使用它的概念。

那篇文章有相当多的观点，随着时间的推移，我不得不更新我用于嵌套表单组的一个例子，因为我最近认为它是一个反模式…

[](/dynamic-forms-in-angular-the-pattern-beyond-the-buzz-da12f560b0d4) [## 有角度的动态形式——嗡嗡声之外的模式

### 一个概念，而不是 API

itnext.io](/dynamic-forms-in-angular-the-pattern-beyond-the-buzz-da12f560b0d4) 

如果你不熟悉[**control value accessor**](https://angular.io/api/forms/ControlValueAccessor)接口，去看看吧。

通过使用它，您可以将您的组件称为表单控件，并使用“formControlName”指令将它连接到您的表单组，并在它发生任何更改时获得更新(反之亦然)。

欢迎您使用下面的实例，并查看我的[以前的帖子](/dynamic-forms-in-angular-the-pattern-beyond-the-buzz-da12f560b0d4) t，现在更新了。

干杯，

利伦。