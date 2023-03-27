# 反应/减少 API 加载和错误

> 原文：<https://itnext.io/react-redux-api-loading-errors-e783972c5424?source=collection_archive---------2----------------------->

## 管理 API 请求状态并妥善处理错误

![](img/433b0c6d8336e81991071539de049346.png)

威尔·斯图尔特在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

我将向您展示我如何在全局范围内处理 API 请求和 API 错误的加载指示器，如果需要的话，特别是针对一个视图。我将使用 **React/Redux** ，但是你可以在其他应用程序设置中使用这些想法。我们将涵盖:

> **加载指示器**(概述)
> **错误管理**(概述)
> 
> **API 请求动作的约定**
> 
> **加载指示器**(深入)
> **错误管理**(深入)

这篇文章是一篇更大的文章[我的牛逼 React/Redux 结构](https://medium.com/@robertsavian/my-awesome-react-redux-structure-6044e5007e22)的一部分。在那篇文章中，我有一个[示例应用程序](https://codebelt.github.io/react-redux-architecture/)，它使用了下面描述的这些技术。查看源代码示例:

*   [React/Redux(类型脚本—类)](https://github.com/codeBelt/react-redux-architecture/tree/TypeScript)
*   [React/Redux(JavaScript-Classes)](https://github.com/codeBelt/react-redux-architecture/tree/JavaScript)
*   [React Hooks/Redux(TypeScript-Functional)](https://github.com/codeBelt/react-redux-architecture/tree/ts/function)
*   [React Hooks/Redux(JavaScript-Functional)](https://github.com/codeBelt/react-redux-architecture/tree/js/function)

# 装载指示器(概述)

让我们直接向您展示这个约定/模式在您的视图中是如何工作的。看下面`mapStateToProps`内的`selectRequesting`。我正在使用[重新选择](https://github.com/reduxjs/reselect)来创建一个选择器，以从 [Redux](https://github.com/reduxjs/redux) 存储中计算出`ShowsAction.**REQUEST_SHOW**`动作类型是否已经被分派*(开始)*或者伴随的`ShowsAction.**REQUEST_SHOW_FINISHED**`动作类型是否已经被分派*(结束)*。我们以`isRequesting`成为`true`或`false`而告终。

请求选择器示例

注意`selectRequesting`是如何接受一个数组的，这样你就可以指定一个或多个动作类型，并且只在所有动作完成后显示/移除加载指示器。如果你想更细化，你可以有一个`isRequestingShow`、`isRequestingActors`等等。

# **API 请求动作的约定**

下面的约定/模式是我如何管理 API 请求的加载和错误逻辑。

任何以`REQUEST_`开头的动作类型都是开始动作:

```
**SomeAction.*REQUEST_SOMETHING***
```

然后，如果任何以`REQUEST_`开始并以`_FINISHED`结束的动作类型是已完成的动作:

```
**SomeAction.*REQUEST_SOMETHING_FINISHED***
```

这样，您可以创建逻辑来确定动作的加载状态是`true`还是`false`。我使用的是[通量标准动作](https://github.com/acdlite/flux-standard-action)模式，该模式有一个`error`属性，可以设置为`true`或`false`。这允许我们在操作完成或重新启动时显示或删除错误消息。下面是一个动作的[通量标准动作](https://github.com/acdlite/flux-standard-action)模式的例子:

通量标准动作示例

使用这种模式，一个动作不能包含除了`type`、`payload`、`error`和`meta`之外的属性，因此您的所有动作都是一致的和可预测的。

在我向您展示实现这一切的代码之前，让我们来看看如何在同一个视图中显示错误。

# 错误管理(概述)

类似于上面的`selectRequesting` [重新选择](https://github.com/reduxjs/reselect)选择器，我有几个选择器来处理错误。下面你可以看到我在`mapStateToProps`中添加了一个`requestErrorText`，但是我也有一个`hasErrors`和`selectRawErrors`选择器，我可以在不同的情况下使用它们。现在我们可以检查并显示错误消息(如果存在的话)。“所有错误”选择接受一组操作类型，因此如果需要，您可以检查多个操作类型。需要指出的一点是，对于错误，你使用以`_FINISHED`结尾的动作类型，而对于加载状态，**不是**必须以`_FINISHED`结尾。

错误选择器示例

# **加载状态**(深入)

## 行动

如上所述，这一切都归结到如何命名的行动类型。

下面的`ShowsAction`正在使用 [Redux-Thunk](https://github.com/reduxjs/redux-thunk) ，它允许我们调度多个动作。通过下面的序列，我们可以知道一个 API 何时开始，何时结束:

*   **第 8 行:**发出启动动作`**REQUEST_**SHOW`
*   **第 10 行:**触发 API 请求`ShowsEffect.**requestShow(74)**`
*   **第 13 行:**发出一个完成动作`**REQUEST_**SHOW**_FINISHED**`

带有 Thunk 示例的操作

## 请求减速器

减速器做了大部分工作。看看下面的文件，我会一行一行地描述正在发生的事情。

请求减速器示例

*   **第 6 行:**检查动作上的`type`属性，查看字符串值是否包含`REQUEST_`字符串。
*   **第 8 行:**如果`isRequestType`为真，我们可以假设`type`是开始`REQUEST_*`或完成`REQUEST_*_FINISHED`请求动作。如果不是，我们返回`state`不要用减速器`state`改变任何东西。
*   **第 13 行:**从动作类型中删除字符串`_FINISHED`。这意味着`requestName`将永远是`REQUEST_*`。我们保留它，这样我们就可以将它用作第 24 行的**对象键。**
*   **第 20 行:**检查动作的`type`属性，查看字符串值是否包含`_FINISHED`字符串。
*   **第 24 行:**添加`requestName`作为减速器`state`的键，如果`type`包含`_FINISHED`字符串，则设置**真**值，否则设置**假**。下面是减速器`state`的样件:

```
{
  'ShowsAction.REQUEST_SHOW': true,
  'ShowsAction.REQUEST_EPISODES': false,
}
```

## 请求选择器

现在我们知道了`RequestingReducer`是如何工作的，我们可以创建一个[重选](https://github.com/reduxjs/reselect)选择器来检查任何启动请求动作类型的`RequestingReducer`状态，如下所示:

```
***selectRequesting*(**state, [ShowsAction.***REQUEST_SHOW***]**)**
```

查看下面的第 5 行,我们可以看到选择器循环遍历我们传入的动作类型数组，并使用 [some](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some) 方法来确定数组中的任何项目是否有值为 **true** 。

请求选择器示例

就这样，现在我们可以通过请求动作来管理我们的加载状态。

# **错误管理**(深入)

我建议从示例[源代码](https://github.com/codeBelt/react-redux-architecture)中寻找，但是我所有的操作都与处理 API 请求的“效果”文件(例如 ShowsEffect)一起工作。效果要么返回预期的 API 数据，要么返回一个`HttpErrorResponseModel`。如果你看下面的例子，你会看到`model instanceof HttpErrorResponseModel`，它允许我们确定我们的系统是否从 API 请求返回一个错误。

带有 Thunk 示例的操作

使用[通量标准动作](https://github.com/acdlite/flux-standard-action)模式，我们可以将错误属性设置为真，并将有效载荷属性分配给`HttpErrorResponseModel`。

## 误差缩减器

在我讨论`ErrorReducer`之前，我想指出所有其他的 reducers 都不处理有错误的动作。只有`ErrorReducer`处理来自 API 请求的错误。下面你可以看到我做了一个提前返回的减速器`state`如果这个动作是错误的。

```
export default class ShowsReducer {
  static *initialState* = {
   ...
  };

  static ***reducer***(state = ShowsReducer.*initialState*, action) {
 **if (action.error) {
      return state;
    }**

    switch (action.type) {
      ...
    }
  }
}
```

下面是`ErrorReducer`，你可以阅读评论来了解发生了什么:

ErrorReducer 示例

## 误差选择器

当使用一个`ErrorSelector`方法时，你总是使用完成的动作类型`(e.g. REQUEST_*_FINISHED)`，并且你可以使用一个或多个动作类型，因为选择器接受一个数组。下面你可以看到一个例子:

```
**selectErrorText(**state, [ShowsAction.***REQUEST_SHOW_FINISHED***]**)**
```

在`ErrorSelector`中有三个选择器，它们是:

*   **selectrawrors:**从有错误的已完成动作类型中返回一个`HttpErrorResponseModel`数组。
*   **selectErrorText:** 从有错误的已完成操作类型中返回所有错误消息的字符串。
*   **hasErrors:** 如果数组中某个已完成的动作类型有错误，则返回 **true** 。否则将返回一个假值。

你可以阅读评论来了解发生了什么:

错误选择器示例

# 感谢阅读！

本质上，我们让归约器和选择器做所有的工作，把复杂的逻辑从视图中去掉。

请查看我的相关文章:

[](https://medium.com/@robertsavian/my-awesome-react-redux-structure-6044e5007e22) [## 我的超赞反应/还原结构

### 在 Redux 中，是跟踪应用程序当前状态的单个对象。您使用减速器来管理…

medium.com](https://medium.com/@robertsavian/my-awesome-react-redux-structure-6044e5007e22) 

如果你喜欢这篇文章，请分享它，关注我，阅读我的其他[文章](https://medium.com/@robertsavian)和/或用我下面的推荐链接注册 Medium。谢谢！

[](https://medium.com/@robertsavian/membership) [## 通过我的推荐链接加入 Medium—Robert S(代码带)

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

medium.com](https://medium.com/@robertsavian/membership)