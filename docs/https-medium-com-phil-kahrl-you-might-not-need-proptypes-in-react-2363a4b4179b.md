# 在 React:arc core 过滤器简介中，您可能不需要 PropTypes

> 原文：<https://itnext.io/https-medium-com-phil-kahrl-you-might-not-need-proptypes-in-react-2363a4b4179b?source=collection_archive---------0----------------------->

![](img/f1c3b5512b6feeaf19cbee9f50544722.png)

滤镜让我们在嘈杂的世界里看清事物。

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fhttps-medium-com-phil-kahrl-you-might-not-need-proptypes-in-react-2363a4b4179b)

[React 库](https://reactjs.org/)在网络开发者中变得非常受欢迎。React 经常使用的一个有用的附件是 [PropTypes](https://reactjs.org/docs/typechecking-with-proptypes.html) ，这是一种在运行时断言 React 组件需要什么类型的数据才能正确呈现的方法。运行时断言(如 PropTypes)独立于构建时类型检查，对于测试和调试非常有用，并将大大降低复杂应用程序的维护成本。在本文中，我将首先讨论 React PropTypes，然后讨论如何使用 [ARCcore 过滤器库](https://encapsule.io/docs/ARCcore/filter)做出类似但更好的断言。到本文结束时，您将了解如何使用 ARCcore。Filter 为任何 JavaScript 函数提供声明性类型检查，以使您的应用程序更加健壮和可维护。

首先，让我们通过一个简单的例子来讨论 React 属性类型。我们的示例由一个 React 组件组成，该组件接受一组 props 数据。数组中的每个对象都有一个“名称”和一个“url”成员，该组件呈现一个列表，其中列表中的每个项目都包含一个指向 url 的链接，该链接带有名称的链接文本。参见代码沙箱中的[示例代码，如下所示:](https://codesandbox.io/s/8zrjojjl2)

在该示例中,“ImageList”组件通过使用 PropTypes 声明来验证它正在接收数组；

```
static propTypes = { renderData: PropTypes.objectOf( PropTypes.arrayOf( PropTypes.shape({ url: PropTypes.string.isRequired, name: PropTypes.string.isRequired }) ))};
```

使用 PropTypes 声明，如果向组件传递的参数不是“renderData”属性的数组，或者如果数组的元素不包含字符串“url”和“name ”,控制台将记录一个错误，警告我违反了组件的数据协定。

您可以通过将第 6 行的 prop 类型更改为:

```
renderData: PropTypes.objectOf(PropTypes.object)
```

重新加载页面，查看 JavaScript 控制台，您会看到如下错误:

```
Warning: Failed prop type: Invalid prop `renderData.imageList` of type `array` supplied to `ImageList`, expected `object`. in ImageList (created by App) in App
```

因为我想对 PropTypes 进行深度类型检查，所以我需要使用 PropTypes 的“shape”函数。

```
PropTypes.shape({ url: PropTypes.string.isRequired, name: PropTypes.string.isRequired})
```

[ARCcore 过滤器](https://encapsule.io/docs/ARCcore/filter)库提供了一种更好的方法，通过它的“过滤器规范”机制来指定对象形状。描述 ImageList 组件所需数据的筛选器规范示例如下所示:

```
inputFilterSpec: { ____label: "ImageList filter spec", ____description: "Filter spec for props for the image list view", ____types: "jsObject", //The entity below is an object imageList: { ____types: "jsArray",  //The object has an imageList array element: { ____types: "jsObject",  //Each array element must be an object url: { ____accept: "jsString" //The object has a string 'url' }, name: { ____accept: "jsString" //The object has a string 'name' } } }}
```

[过滤器规范](https://encapsule.io/docs/ARCcore/filter/specs)只是一个 JavaScript 对象。对象中以 *quanderscore* "____ "开头的字段有特殊的含义。前两个字段' ___label '和' ____description '是描述过滤器规格的可选字段。下一个字段:

```
____types: “jsObject”
```

表示下面要描述的东西是一个物体。该对象被进一步指定，因为它将包含一个名为“imageList”的成员，并且该列表的每个元素将包含“url”和“name”的键，这两个键都是字符串。过滤器规范的格式将让我们指定任意复杂的对象，通过查看过滤器规范，我们可以很容易地看到该对象的形状。

使用对象而不是嵌套函数来指定输入有几个优点。一个是对象只是数据，而不是代码，这使得规范是声明性的。不需要为对象编写测试，解析对象(ARCcore 过滤器)的代码有自己的测试，以确保根据过滤器规格正确验证输入。对象易于阅读，可以根据需要导入、序列化、通过网络发送和共享。

过滤器规格只是对象，本身不做任何事情，但作为功能的一部分来实现一个 [ARCcore 过滤器](https://encapsule.io/docs/ARCcore/filter)。ARCcore Filter 只是一个函数包装器；它有两种滤波器规格，一种用于输入，一种用于输出。它还具有接受输入、产生和输出的身体功能。由于我们只对这个例子中的类型验证感兴趣，我们可以创建一个带有空体函数的过滤器，使用上面的过滤器规范来进行类型验证。

使用上面的过滤器规范，我可以编写一个简单的 ARCcore 过滤器，在一个自定义验证器中调用它，我可以将它用于我的组件 PropType。参见[这里的代码沙箱示例](https://codesandbox.io/s/zx48po911m)，也如下所示。

带有使用 ARCcore 过滤器实现的自定义验证器的 PropType 示例。

在进一步讨论之前，我们先来详细讨论一下什么是 ARCcore 滤波器。

1.  ARCcore 过滤器是一种复合函数(函数链)。每次调用过滤器时都会调用三个函数。一个函数根据 inputFilterSpec 验证输入，另一个函数是过滤器的编写器提供的 bodyFunction，第三个函数根据 outputFilterSpec 验证 body 函数的输出。
2.  要创建新的过滤器，您需要调用工厂函数“arccore.filter.create”(上面代码中的第 3 行)。工厂函数被传递一个对象，该对象包括输入和输出过滤器规范、主体函数和一些其他元数据，包括唯一标识符(第 4 行)以及名称和描述(第 4 和第 5 行)。与过滤器类似，过滤器的工厂函数返回一个带有“错误”和“结果”成员的对象。
3.  过滤器总是返回带有“结果”和“错误”成员的对象。如果错误成员为非空，则可以使用结果。
4.  调用过滤器的模式是通过请求方法将数据传递给过滤器，检查是否有错误，如果错误为空，则使用结果。
5.  过滤器的有用工作是通过实现 body 函数来完成的。在这个例子中，bodyFunction 很简单，因为我们只对输入的类型验证感兴趣。

使用 ARCcore 过滤器为 PropTypes 创建一个自定义验证器可以省去为类型检查编写实际函数的工作。Filter Spec 本身只是一个对象，所以现在我的类型检查变成了声明性的，我可以指定任何复杂程度的类型检查。

要了解类型验证是如何工作的，请参见上面的沙箱。在“App.js”中，更改数组中的一个对象，如第 6 行所示:

```
{name-1: "cat",url: "https://static.pexels.com/photos/20787/pexels-photo.jpg"}
```

现在，其中一个对象的成员是“名称-1”，而不是“名称”。重新加载渲染页面将在控制台中产生以下错误:

```
Warning: Failed prop type: Filter [3RR5VT_5TBWuzX_VhMG4dQ::ImageDisplayPropsFilter] failed while normalizing request input. Error at path '~.imageList[0].name': Value of type 'jsUndefined' not in allowed type set [jsString].     in ImageList (created by App)     in App
```

该消息告诉您数据中违反了什么约束，以及违反来自哪个过滤器。消息中 operationId 的存在使得即使在大型代码库中也能够容易地找到过滤器。

所以现在我们有了深度类型验证，可以通过修改过滤器规范来轻松更改。Filter 的另一个特性是，它不通过任何不在 Filter 规范中的内容，换句话说，它过滤掉不需要的成员，同时验证需要的成员。

这可能是宣布胜利和停止写作的好时机，但只有一个问题。PropTypes 和 ARCcore Filter 之间有很多重叠。如果您为 React 组件声明了一个 PropType，那么当 React 开始呈现该组件时，If 首先调用一个函数来验证 props，然后它调用该组件作为一个函数来实际呈现。以类似的方式，如果您有一个过滤器，首先调用一个函数来验证输入，然后对该输入运行一个函数，返回一个结果。具有 PropTypes 的 React 组件和 ARCcore 过滤器都在做同样的事情，因为它们将函数链接在一起；一个函数用于验证输入，另一个函数用于实际操作并返回结果。除了在应用程序中复制功能，我们还可以做得更好，那么为什么不干脆用 Filter 做所有的事情，完全不用 PropTypes 呢？我们可以简单地创建一个过滤器，使用上面的过滤器规范，它有一个 body 函数，使用导入到过滤器中的原始组件返回呈现的内容。现在，任何想要呈现 ImageList 的组件都可以简单地调用我们创建的过滤器(呈现过滤器)，而不是直接调用 ImageList。通过调用渲染过滤器，我们可以保证传递正确类型的数据，并处理任何错误。我们使用渲染过滤器的应用程序可以在下面的代码沙箱中看到:

使用 ARCcore 过滤器进行渲染消除了对属性类型的需求

在上述沙箱的“App.js”中，注意调用过滤器并使用三元运算符呈现结果的模式。

```
const renderingResponse = ImageListRenderingFilter.request({ imageList: images});const renderResult = renderingResponse.error ? ( <div>{renderingResponse.error}</div>) : ( renderingResponse.result);return <div>{renderingResponse.result}</div>;
```

我们通过调用“请求”函数将数据传递给过滤器。该请求返回一个带有“结果”和“错误”成员的对象。如果“错误”不为空，我们将得到一个可以使用的“结果”。这种模式积极地检测错误，并以一致的方式报告它们。这类似于我们在为组件使用 PropTypes 时的模式，只是我们对此更加明确。

到目前为止，我们已经讨论了 PropTypes 和 ARCcore Filter，下面是两者之间的比较。

## 在调用函数之前使用函数链来验证输入？

两者都这样做。

## 声明有效输入的方式？

PropTypes:可用简单类型、嵌套函数或编写自定义函数。

ARCCore 过滤器:使用对象进行声明

## 可以用在哪里？

PropTypes:测试环境中 React 组件上的 props。

ARCcore 过滤器:JavaScript 中的任何地方。

## 错误处理？

PropTypes:仅在开发环境中将错误记录到控制台。

ARCcore Filter:强制程序员显式处理错误。

简而言之，ARCcore Filter 类似于 PropTypes，只是它是一种更通用的解决方案，并且以声明方式执行数据验证。

# 过滤器规格有趣的事实

您可以为单个成员指定多个允许的类型，例如:

```
____accept: ['jsString', 'jsNull', 'jsUndefined']
```

您可以为树的任何部分指定默认值:

```
____types: 'jsObject',
____defaultValue: {foo: {bar: 'baz'}, {aNumber: 1}},
```

以上对于生成符合规范的数据集非常方便。

您可以指定一组允许的值:

```
____inValueSet: ['ready', 'waiting', 'error']
```

或范围:

```
____inRangeInclusive: { begin: 0, end: 100 }
```

更多细节见 F [过滤器规格文件](https://encapsule.io/docs/ARCcore/filter/specs)。

## 运行时断言的优势

上面的例子是一个基本的例子，通过编写一个“渲染过滤器”可以用 ARCcore 过滤器做什么。ARCcore Filter 提供了一个内置运行时断言的函数包装器。向代码中添加运行时断言最初会增加更多的工作量，但从长期来看会降低代码库的总拥有成本。请考虑以下几点:

1.  您的大部分精力不是花在编写代码的前期成本上，而是花在维护代码上。如果事情尽可能明确，包括明确的错误处理和类型声明，代码的维护和调试就会容易得多。
2.  web 应用程序的祸根是不一致性。许多 web 应用程序似乎使用无效数据，并且只会以难以调试的微妙方式显示错误。调试行为不一致的应用程序是极其困难和耗时的。
3.  许多数据依赖是通过异步请求来满足的，在异步请求中，静态类型不提供任何安全性。
4.  轻松做出断言的能力使得测试更加容易，只有测试才能保证代码中没有错误。
5.  生产应用程序中的运行时断言提供了一种优雅地处理错误的方法。

以下是 ARCcore 过滤器可用于的一些其他用途:

1.  在 web 应用程序中的呈现组件和异步 API 之间建立显式数据协定。
2.  构建防止退化的共享组件库。
3.  组成复杂的对象层次结构。
4.  用基于消息的路由构建插件系统。

到目前为止，您应该对 ARCcore Filter 的工作原理有所了解，它可以用来改进 JavaScript 中的任何代码库。本文中提供的示例只是 ARCcore Filter 的皮毛。更多关于 ARCcore Filter 以及其他有用库的信息可以在 [encapsule.io](https://encapsule.io) 找到。