# 如何创建活动的 IfFeatureFlag 指令

> 原文：<https://itnext.io/how-to-create-a-live-iffeatureflag-directive-871c36b6b943?source=collection_archive---------6----------------------->

一个负责特性标志检查的指令，而且是实时的！

![](img/cdce960caa7777ce79a40cf62c92e6ed.png)

旗帜

*照片由* [*利亚姆 Desic*](https://unsplash.com/@liamdesic?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) *上*[*Unsplash*](https://unsplash.com/s/photos/flags?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*。*

TL；速度三角形定位法(dead reckoning)

> GitHub 的指令代码。
> 
> 在 [StackBlitz](https://stackblitz.com/edit/if-feature-flag-directive?file=src%2Fapp%2Fapp.component.html) 的实例。

## **详情**

在使用特性标志的 Angular 项目中，您可能会有一些 HTML 和 TS 代码，如下所示:

出现特征标志时切换的视图示例。

*假设* `*featureFlags.checkIsOn*` *返回一个* `*Observable<boolean>.*`

这就是**精**。这段代码可以正常工作。

## **陈述性和简洁**

我的建议应该是这样的:

使用“ifFeature”指令的示例。

## 感兴趣吗？请继续阅读。

为什么不制定一个指令，就像`*ngIf`一样，它会处理繁重的工作，让你:

-少写样板文件

-编写更少的测试

-专注于重要的事情

## **幼稚的实现**

这是一个简短的版本:

“ifFeatureFlag”指令的简单实现。

它将输入中的特性标志名称和标志集合作为构造函数依赖项(一个服务),并且仅在特性标志打开时显示视图。或者如果特征标志是向下的(不存在)，则隐藏它。

*`*templateRef*`*被认为是和标志(服务)一起注入到构造函数中的，* `*viewContainerRef*` *。**

*这是幼稚的版本。它仍然需要:*

*-支持缺失特征标志的情况*

*-能够根据应用生命周期中功能标志的变化进行更新*

## ***丢失特征标志案***

*我们需要添加一个`else`处理程序。它可能看起来像这样:*

*处理“ifFeature”指令的 else 情况。*

*在实例属性中存储了标志和“else”模板之后，`ifFeature`和`ifFeatureElse`都委托给了`updateView`方法。现在我们可以去做:*

*如何使用“ifFeature”指令的示例。*

***现场更新***

*为了能够支持实时更新，因此当特性标志更新时，视图也会更新，我们需要包含可观察的特性链的概念:*

*[https://gist.github.com/dd9a02f07201f1d554fde6ac43be76a8](https://gist.github.com/dd9a02f07201f1d554fde6ac43be76a8)*

*假设`this.flags.checkIsOn(this.flag)`返回一个当特征标志改变时更新的可观察值。*

*现在，这允许我们的功能标志是活的！*

## ***一些小问题***

*当然，还有一些细节需要处理。其中之一是`this.flags.checkIsOn(this.flag).subscribe()`的“悬空”订阅。另一个是检查订阅是否已经存在，包括两个`@Input`调用`updateView`并可能创建第二个订阅的情况。在完成的代码中，这些都被处理了。*

*代码存在于 [GitHub](https://github.com/gparlakov/if-feature-flag/blob/main/src/app/flag-module/if-feature-flag.directive.ts) 中。*

*在 [StackBlitz](https://stackblitz.com/edit/if-feature-flag-directive?file=src%2Fapp%2Fapp.component.html) 中还有一个活生生的例子:*

*使用指令的 StackBlitz Angular 应用程序。*