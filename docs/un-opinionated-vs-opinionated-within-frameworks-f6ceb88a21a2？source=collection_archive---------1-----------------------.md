# 框架内不固执己见与固执己见

> 原文：<https://itnext.io/un-opinionated-vs-opinionated-within-frameworks-f6ceb88a21a2?source=collection_archive---------1----------------------->

![](img/9bfaff2c2a47db23960e6ba6d1001936.png)

Angular 或 React 之类的每个框架的第一个优点和缺点是它“固执己见”或“不固执己见”，这是使用该框架的一个重要原因，也是不使用另一个框架的一个重要原因。在很长一段时间里，项目文件结构在我看来似乎是框架固执己见的主要原因，但它还有更多。

实用程序固执己见的本质也由它为开发人员提供的“瑞士军刀(SAF)”的丰富程度来定义。每个框架都将提供一些 SAF，供我们开发人员以特定的方式编写代码。

例子:
1。反应——功能组件、挂钩、JSX/TSX
2。Angular — HttpClient，RouterModule，I18N，双向绑定

我喜欢用这两个框架作为主要的例子，因为它们之间有着截然不同的差异。

> 一个公用事业越不固执己见，它就变得越多样化，但结果是它变得不那么结构化。

可以提供的一个最简单的例子是从路由中获取查询参数。

# 差异

**Angular**

```
import {ActivatedRoute} from '@angular/core'
constructor(private **route**: ActivatedRoute) { const queryParams = **route**.snapshot.queryParams; **route**.queryParams.subscribe(console.log)}
```

我们有两种不同的可选方式来呈现组件，每种方式都提供了一种略有不同的获取查询的方式。是的，这些模式之间有很多相似之处，也许在幕后他们使用相同的解决方案，但这种方式给了我们灵活性和高度的多样性，可以在我们的应用程序中做得更好。
在给出的解决方案中，我们使用普通的 JS 或开源库

```
import {useSearchParams} from 'react-router-dom' export const Component = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const id = searchParams.get("id") const search = window.location.search;
   const params = new URLSearchParams(search);
   const id2= params.get("id");}export class ComponentClass {
   constructor() { const search = this.props.location.search;      URLSearchParams(search).get("id"); 
}}
```

获取查询参数的例子很傻，也很小，但它向我们展示了一个简单的例子，说明了给出的瑞士刀子如何给我们带来好处或伤害。我不是说一个比另一个好，但我要说的是:**注意你的瑞士军刀里的工具。**

*知道自己有什么或者没有什么，可以提高自己的编码速度和工作质量。*

拥有一个固执己见或不固执己见的框架是一种从框架的方向出发的理想主义。例如，Angular 是一个面向企业的框架，这意味着它将共享知识库和**预测代码理想化。** 这个决策有它的收益，例如，捆绑销售规模和业绩，但是对于一个企业来说，这些收益也许是值得的。

# **预测码**

预测代码是一种正确地或高精度地假设某事是如何编写和实现的能力。这是验证一个团队/小组和代码库有多统一的一种方式。
预测代码可以分为几个主要区域:
1 .命名转换
2。社区范围的最佳实践
3。团队/小组同意的最佳实践
4。常用工具
5。林挺和静态代码分析
6。类型化(或者一些有助于 IDE 的解决方案，比如 JsDocs)

所有 6 个标准都可以在任何框架中完成。尽管框架的自以为是程度将决定上面列出的项目中有多少是通过框架处理的，有多少是团队/小组必须自己处理的。

预测码的结果是:
1。新的团队/小组成员将很快适应现有团队的编码风格。
2。代码审查更简单，因为一致同意的标准使得接受或拒绝 PRs 更容易。
3。调试容易多了，因为我们可以预测错误的位置和易受攻击的点。
4。创建 schematics(生成代码的代码)可以提高开发速度，并将一致同意的最佳实践“强加”给团队。

# 摘要(TLDR)

固执己见和不固执己见的框架可以改成“*多*固执己见和*少*固执己见”。框架意见的水平可以通过它提供并强加给用户的工具数量来衡量和感知。
实用程序数量的结果决定了我们代码的预测性。
由于处理代码库的框架和/或团队，任何代码库都可以并且应该变得结构化和“固执己见”。

祝你的项目好运:)