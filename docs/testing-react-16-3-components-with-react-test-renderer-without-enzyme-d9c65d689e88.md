# 用 react-test-renderer 测试 React 16.3+组分(无酶)

> 原文：<https://itnext.io/testing-react-16-3-components-with-react-test-renderer-without-enzyme-d9c65d689e88?source=collection_archive---------1----------------------->

![](img/0e25a1e3629868d21b77c02eb07ffd63.png)

无耻地从[Algolia 2015 年同一主题的文章](https://blog.algolia.com/how-we-unit-test-react-components-using-expect-jsx/)中窃取:)

我最近开始了一个新的 React 前端项目，并一直在充分利用新的 React 16.3 上下文 API，使数据的深度共享更加干净。然而，在使用我的 go-to-react 测试库 [Enzyme](https://github.com/airbnb/enzyme) 编写测试时，我遇到了[这个问题](https://github.com/airbnb/enzyme/issues/1598)，它对我来说是一个阻碍！

遗憾的是，已经有 6 个月没有任何新的 Enzyme 版本了，他们[也无法告诉我下一个版本将会是什么时候](https://github.com/airbnb/enzyme/issues/1705)，所以我决定试用标准的 [React 测试渲染器](https://reactjs.org/docs/test-renderer.html)，它是库自带的(所以应该总是最新的！)

React 测试渲染器的文档没有多少有用的例子，所以我想在这里记录我的发现！

# 安装组件进行测试

与 Enzyme 类似，您使用一个`TestRenderer.create()`方法来创建一个组件树以备测试，如下例所示:

您会注意到我们在测试渲染器中使用了两种主要类型的对象:

`renderer`顾名思义，指的是你的组件周围的“包装”。对于这个对象，我发现只有几个有用的方法:

1.  `renderer.toJSON()` —这将返回一个 JavaScript 对象，代表组件的 HTML(或 react native)输出。
2.  `renderer.toTree()` —这将返回一个表示组件结构和 HTML 输出的 JavaScript 对象

在 NodeJS 中运行的测试中，您将希望 JSON.stringify()能够看到完整的深度！

`instance`是对 react 组件树的根组件**的引用。您将使用这个对象来进行断言，例如`instance.findAll()`。请参见下面的示例:)**

# 在树中定位组件

与 Enzyme 不同，react-test-renderer 中没有 CSS 选择器。基本上有三种方法可以找到组件

1.  使用`instance.find()`或`instance.findAll()`方法，采用一个“谓词”函数(例如`(el) => el.type == 'span'`)并对树中的每个组件运行它。
2.  使用`instance.findByType`或`instance.findAllByType()`方法，这些方法接受**组件构造器**并返回匹配的组件。
3.  使用`instance.findByProps`或`instance.findAllByProps()`，根据您指定的属性搜索组件。

# 示例:搜索带有特定文本的

下面的断言将找到一个`<button>Save Changes</button>`标签，并检查它是否被禁用。

```
const saveButton = instance.find(
    (el) => el.type == 'button'
        && el.children
        && el.children[0] == 'Save Changes'
);
expect(button.props.disabled).toEqual(true);
```

值得注意的是`find()`将**精确匹配一个组件**。如果它不匹配任何一个，或者匹配多个，那么它**抛出一个异常**。

# 按属性查找标签

下面的断言找到了一个`<input type="radio" />`标签

```
const radio = instance.find(
    (el) => el.type == 'input'
        && el.props.type == 'radio'
);
```

# 测试组件的缺失

因为`find()`函数**必须**匹配一个组件，所以我们不能用它来测试组件的缺失，但是我们可以用`findAll()`来测试:)

```
const checkedBoxes = instance.findAll(
    (el) => el.type == 'input'
        && el.props.type == 'checkbox'
        && el.props.checked == true
);
expect(checkedBoxes).toHaveLength(0);
```

# 在其他组件中搜索组件

像酶一样，您可以搜索组件的子树。在下面的例子中，我们寻找特定`<select>`中的所有`<option>`标签

```
const select = instance.find(
    (el) => el.type == 'select'
        && el.props.name == 'regions');const options = select.findAll(
    (el) => el.type == 'option');expect(options.length).toEqual(regionData.length);
```

# 通过构造函数搜索组件并读取属性

如果您想基于 React 组件属性而不是它们的输出来编写您的断言，您可以通过使用**组件构造函数进行搜索来实现。**

下面的断言将匹配一个`<CardHeader />` react 组件，并检查它的`title`属性。

```
import CardHeader from '@material-ui/core/CardHeader';const cardHeader = instance.findByType(CardHeader);expect(cardHeader.props.title).toEqual('My awesome app');
```

# 测试事件处理程序

幸运的是，使用 React 测试呈现器测试事件处理程序及其结果非常简单。它的工作原理和酶差不多。

你所需要做的就是获得一个对组件的引用(例如使用`find()`，然后通过`props`直接触发事件处理程序，例如:

```
const select = instance.find(
    (el) => el.type == 'select');// trigger the onChange event for the select box
select.props.onChange({
    target: { value: 'blue' }
});expect(colourChangerMock).toHaveBeenCalledWith('blue');
```

注意—根据事件处理程序的需要，您负责创建合适的事件对象。上面的 onChange 示例应该适用于大多数输入字段。

同样值得注意的是，呈现的组件树似乎会像在浏览器中一样得到更新，所以如果您的事件处理程序触发了组件更新，那么您应该能够立即检查它们。我不需要像过去用 Enzyme 那样用 React Test Render 调用任何`updade()`方法。

# 结论

幸运的是，react 测试渲染器似乎支持测试 React 组件所需的所有基本功能，其优势在于它与主 React 包位于同一个存储库中，因此它更有可能与最新版本一起工作！*(我看到他们正在增加异步渲染支持:)*

希望这篇文章能帮助像我一样苦于缺乏全面的测试渲染器文档的人:)