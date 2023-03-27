# Formik 中的自定义组件

> 原文：<https://itnext.io/custom-components-in-formik-6b9e5f39626c?source=collection_archive---------7----------------------->

![](img/b12bd8f27106b1c0197b411f20c0e0fb.png)

形式和[反应](https://reactjs.org/)。他们在一起不相配吗？如果你在做任何严肃的*反应*开发，你迟早会构建一个复杂的表单。推出自己的自制框架的诱惑可能会出现，但你必须与之斗争。有很多好的现有解决方案可供选择。

你可以选择[来完成任务。在这种情况下，我想向您展示如何为它构建一个定制的输入组件。](https://jaredpalmer.com/formik/)

# 等等，福米克？

*福米克*是 block 上新的[酷小子。引用官方文件:](https://jaredpalmer.com/formik/)

> *在无眼泪的情况下做出反应。*

我当然分享眼泪的部分。我曾经用 [redux](https://redux.js.org) 构建我的表单，使用 [react-redux-form](https://github.com/davidkpiano/react-redux-form) 。这是一个很好的库，不要误会我的意思，但最终在连接东西时会有很多摩擦。更不用说和政府打交道了。为什么，不管怎样？回过头来看，在 Redux*中拥有我的表单状态似乎并没有给我多大帮助。*

福米克采取了一种非常不同的方法。它是声明性的，依靠纯粹的*做出反应*。它使用[渲染道具](https://reactjs.org/docs/render-props.html)，这种模式在 *React* 生态系统中迅速传播。说到这里，这篇文章最终帮助我更好地理解了他们。

不管怎样，到目前为止，我的印象是 *Formik* 不碍事，让你专注于构建你的表单。

# 用户化

常规的组件，加上少量的造型，会让你走得很远。它们涵盖了大多数典型的用例。但是如果你想增加情趣呢？

例如，我有一个小应用程序，我想给一个星级，从一到五。我是先用常规的数字输入实现的，感觉没意思。我想点击星星，就像这样:

![](img/e72cc6e7604da5ed459c7d738fb3b5e7.png)

建造这样的东西时， *Formik* 进展如何？休息之后继续。

# 代表性成分

在我们进入实际的表单内容之前，让我们构建`Stars`作为显示组件。我们正在建造一排五星。我们有一个 prop ( `count`)来设置已满的星星的数量，还有一个 handler ( `handleClick`)来知道特定的星星何时被点击。

`Star`组件是一个薄薄的包装，包裹着一个[字体很棒的](https://fontawesome.com/)可点击图标。

# 自定义输入组件

我们终于到了实质性的部分。我们如何让`Stars`组件 *Formik* 知道呢？

我们将在[字段](https://jaredpalmer.com/formik/docs/api/field)中呈现我们的`Stars`表示组件。它还使用了渲染道具，我们将使用这些道具将它连接到我们的`Stars`。

有一个包含`value`的`field`散列，也就是集合星的数量。这将是`count`的输入。为了在点击一个星号后更新这个值，我们在`form`散列中使用了`setFieldValue`函数。这将在内部更新 *Formik* 上的值。

# 将它整合到一个表单中

现在我们已经准备好了自定义输入组件，我们可以在一个常规的 *Formik* 表单中呈现它，我们都准备好了:

# Codesandbox

您可以在下面的沙箱中使用这个工作实现。查看并扩展它以满足您的需求。

 [## 自定义表单组件代码沙盒

### CodeSandbox 是一个为 web 应用程序量身定制的在线编辑器。

codesandbox.io](https://codesandbox.io/embed/custom-formik-component-2t80u?fontsize=14) 

*原载于 2019 年 8 月 15 日*[*https://hceris.com*](https://hceris.com/custom-components-in-formik/)*。*