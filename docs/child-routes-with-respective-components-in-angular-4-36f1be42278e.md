# 角度为 4 的子路由及其各个组件

> 原文：<https://itnext.io/child-routes-with-respective-components-in-angular-4-36f1be42278e?source=collection_archive---------0----------------------->

![](img/c32a8b051da685a768c7980dd1628c7c.png)

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fchild-routes-with-respective-components-in-angular-4–36f1be42278e)

像我在这里做的一样，将你的应用程序分解成不同的模块后，你可能想要一个父路由，它的子路由加载各自的组件。我们试图避免的是我们的`app-routing.module.ts`变得像下面这样杂乱:

为了实现这一点，我们首先需要将我们的应用程序分成几个模块，然后使用一个简单的角度约定将子路径与子组件匹配起来。本文的目的是为如何实现这一点提供明确的指导。angular.io [指南](https://angular.io/guide/router#milestone-4-crisis-center-feature)也很不错，但我写这篇文章是为了帮助我学习这个主题，并希望在我学习的时候帮助其他人。我也会尽量简洁。

## 先决条件:

1.  本指南假设了一些角度路由的基本知识(`<base href>`、`<router-oultet>`等)。如果你不熟悉这些，你可以在这里阅读更多相关信息[。](https://angular.io/guide/router#the-basics)
2.  通过 Angular CLI 创建的 Angular 4+应用程序
3.  除 AppModule 之外的至少一个模块(默认情况下由 CLI 生成)。我将额外的模块称为**次级模块**，而 AppModule 是**主模块**。
4.  主模块目录中的两个或更多组件。其中之一将是**父组件**。
5.  辅助模块应该有自己的`module`和`routing`文件。你可以在我之前的[指南](https://medium.com/@astamataris/setting-up-routing-in-a-multi-module-angular-4-app-using-the-router-module-d8e610196443)中看到怎么做。

**步骤 1:调整父模块路由文件，声明父组件及其子组件。**

## 步骤 2:在父组件模板中添加路由器出口标记

差不多就是这样了！父组件将根据子组件选择的 URL 呈现自身。孩子的模板当然会在路由器出口动态渲染。

如果你不能正确遵循，也许可以试着先阅读我的另一个简单的[指南](https://medium.com/@astamataris/setting-up-routing-in-a-multi-module-angular-4-app-using-the-router-module-d8e610196443)关于如何模块化你的应用，然后尝试这个指南中提到的步骤。他们两个都很矮。

正如我上面提到的，写这篇文章是为了方便我的学习，也希望能帮助其他人。欢迎任何意见或建设性的批评！