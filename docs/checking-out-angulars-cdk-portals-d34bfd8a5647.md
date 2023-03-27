# 查看 Angular 的 CDK 门户网站

> 原文：<https://itnext.io/checking-out-angulars-cdk-portals-d34bfd8a5647?source=collection_archive---------1----------------------->

![](img/829935138be9bc1d343dd47d9e3180ee.png)

作为前端开发人员，我们有一大套工具来用不同的方法创建 UI 组件(目的是在考虑性能、可读性等的基础上选择最好的一个)..)

当使用框架开发时(至于 2019 年，我们仍然使用框架，目前我更喜欢 Angular..)我们有一套抽象概念，我们应该以某种方式使用这些抽象概念，以便在不滥用框架的情况下最大限度地利用框架——换句话说，遵循它的“最佳实践”。

在 Angular 中，当你想在某个地方动态地渲染一个组件或模板时，你有几种方法可以做到:

*   以编程方式使用 ComponentFactoryResolver 服务:

```
**const factory** = **this**.resolver.resolveComponentFactory(NewAwesomComponent);
**this**.**componentRef** = **this**.viewContainerRef.createComponent(factory);
```

*   通过使用*** ngcomponentfoutlet**或*** ngtemplatefoutlet**结构指令
*   **通过使用 **< ng-content >** 将内容投影到另一个主机(不完全是动态的，但它将在另一个主机中呈现，而不是在它注册的地方，所以请注意，它不会咬你)。

每当我使用* **ngComponentOutlet，**我不得不将我想要动态呈现的组件导入托管组件的事实感觉有点不对，托管组件意识到它必须呈现的动态组件，因此它们变成了耦合。

# 救援门户:

如果我只希望我的主机组件有一个占位符/槽，我可以在其中呈现另一个组件，而不需要从主机模板显式导入和引用它，该怎么办？

> “利用平台”(lol)

Angular 的组件开发工具包——目前，我看不出有任何理由不在我的项目中使用它，Angular 的人在维护它方面做得很好，它很容易使用，很快提供了一组内置于我欣赏的[编码标准](https://github.com/angular/components/blob/master/CODING_STANDARDS.md)中的 UI 组件。

当传送门出现的时候，我在哪里？闭上眼睛，因为使用 ComponentFactoryResolver 是如此有趣…

门户的官方定义是:

> 一个`Portal`是一个 UI，它可以被动态地呈现到页面上的一个空位上。

(类似于 WC [slot](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot) 元素的概念，只是可移植视图不是新的 DOM 树)。

它非常简单，让我们看看它的使用情况:

# 示例说明:

我创建了一个简单的标签栏菜单，在菜单下放置了一个<foo>组件。当用户选择一个选项卡时，其相关的视图部分组件将出现在 foo 组件内**:**</foo>

应用程序

```
<app-nav-menu *ngFor=**"let section of sections"**>
   <button (*click*)=**"setSelected(section)"**> {{section}}</button>
</app-nav-menu>
<hr>
<app-foo></app-foo>
```

foo.html

```
<div **class**=**"app-foo"**>
   <p> Im foo and I like *it* </p>
   <div id=**"nav-view-slot"**></div>
</div>
```

门户 API 公开了一个 [CdkPortal](https://material.angular.io/cdk/portal/api#CdkPortal) 指令和一个 [DomPortalOutlet](https://material.angular.io/cdk/portal/api#DomPortalOutlet) 类

> [DomPortalOutlet](https://material.angular.io/cdk/portal/api#DomPortalOutlet) :一个 PortalOutlet，用于将门户附加到 Angular 应用程序上下文之外的任意 DOM 元素。
> 
> [CDK portal](https://material.angular.io/cdk/portal/api#CdkPortal):a`TemplatePortal`的指令版本。因为指令*是*一个 TemplatePortal，指令实例本身可以附加到一个主机上，支持门户的声明性使用。

基于 Nir Kaufman 在 AngularNYC 的一次演讲，我创建了下面的组件，他们的用法如下:(复制的比实际创建的多…)。

```
<ng-container *cdkPortal>
   <ng-content></ng-content>
</ng-container>
```

这实际上是一个伟大的演讲！真的很有趣，我真的建议观看它，以更好地掌握门户网站的使用。