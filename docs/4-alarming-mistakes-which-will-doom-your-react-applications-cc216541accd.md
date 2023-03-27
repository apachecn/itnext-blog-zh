# 4 个惊人的错误，它们将毁灭你的 React 应用程序

> 原文：<https://itnext.io/4-alarming-mistakes-which-will-doom-your-react-applications-cc216541accd?source=collection_archive---------9----------------------->

如果你想让你的 react 应用程序陷入一个糟糕的管理，错误滋生，耗费时间的代码黑洞，那么做下面列出的四件事…告别任何新功能和利润，因为开发人员的时间都花在修复错误上了。

![](img/891c5853a17c9608417270490add509a.png)

照片由[凯勒·伍兹](https://unsplash.com/@caleb_woods?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/shame?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

## 在组件中嵌套组件

不要将组件嵌套在使用变量或函数来渲染 JSX 的组件中，这是有条件的或需要插值的。与组成成分反应。当某个东西足够复杂，以至于内联 JSX 三元组不够用，额外的处理和复杂的条件要求它被提取到自己的函数中时，只需创建一个新的组件。

> 不要将 JSX 放在变量或函数中进行条件渲染。使用具有描述性名称的组件。

使用`<Components />`代替渲染函数(如`renderComponent`)。不要将 JSX 放在变量或函数中进行条件渲染。使用具有描述性名称的组件。例如:

`renderBottomFooterConditions() { ...` ❌

`<MyComponentBottomFooterConditions` ✅

将每个组件分割成更小的单元允许将条件和插值分割到文件中，从而减少组件的大小和复杂性。

## 玩嵌套对象

当不使用类型系统时，停止通过组件属性传递嵌套对象。当您使用类型系统时，这可能更容易被忽略，但是这样做仍然会将特定的对象结构耦合到组件。

> 展平您的组件道具

在某些情况下，您可能希望将一个对象作为组件属性传递，例如*“您有一组公共的数据处理逻辑，需要在应用程序的多个地方发生*，但这种情况很少见，通常与列表有关，请参见[我的文章](https://medium.com/@lukebrandonfarrell/component-composition-for-dummies-cd94515a360e)。

如果您有一个组件，它呈现来自服务器的图像并计算适合容器的纵横比，您需要传递两个属性；`authToken`、`imageUrl`，你要拉平属性:

```
<AuthImage
  authToken={...} ✅
  imageUrl={...} ✅
```

不应将整个对象传递给组件，而是将组件绑定到该对象结构:

```
<AuthImage
  imageStuff={{ authToken: ..., imageUrl: ... }} ❌
```

使用对象比使用单值变量更复杂，也更容易出错；布尔值，数字，字符串。

## 不使用描述性命名

这不是 Twitter，你没有字符数限制…停止给你的变量和函数取简短的通用名称，这会让其他开发人员在阅读你的代码时毫无头绪。文件、变量和函数应该有表达性和描述性的名字。你的代码应该被其他人理解。例如

`let messageForTheWarningBanner = messages?.message_for_banner` ✅

`let message = messages?.message_for_banner` ❌

第一个变量立即告诉我们更多关于它包含的信息。

在 JavaScript 中解构对象和数组时，应该重新分配变量名以匹配当前上下文。例如

```
const { name: companyName } = useSelector(
     state => getOrganisation(state)
); 
const { email: currentEmail } = useSelector(
     state => getUser(state)
);
```

> 代码是给人类的，不是给计算机的

## 不解释“为什么”

> 请…评论你的代码。

注释极大地增加了项目的健壮性，不注释代码就像卖一本没有说明的“自己构建”书架。如果你不知道如何注释代码，这里有一个简单的秘密… 🥁🥁🥁

> 解释“为什么”,而不是“如何”

注释用于解释代码背后的“为什么”。“如何做”可以从代码中推断出来。谈谈你选择编写这部分代码的原因。下面是一个真实应用程序中单个变量的注释:

```
/** 
* Used so the rebrand alert does not appear multiple times, 
* as if the user exits the app, this will trigger new user data 
* to be fetched, which will trigger new organisation data to be 
* fetched, which will lead to multiple FETCH_ORGANISATION_RESPONSE 
* and multiple instances of this generator function waiting at 
* call(waitUntilStack.... 
*/ 
let isRebrandAlertShowed = false;
```

“如何”可以从代码中推断出来…“为什么”永远消失了

将这些原则应用到您的应用程序中，以赢回代码库的稳定性，并利用干净代码的力量来构建功能并影响更多的人。