# 在 React 中创建 Dope 可扩展树视图

> 原文：<https://itnext.io/creating-a-dope-expandable-tree-view-in-react-5b32a36082d4?source=collection_archive---------0----------------------->

## 当单击父节点时，从服务器异步获取子节点

![](img/26650c16c8f9d0d3ebe21c4757011944.png)

照片由[阿达什·库姆穆尔](https://unsplash.com/@akummur?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

## 动机

我编写这个功能是作为显示组织层次结构的工作需求的一部分。结果比我最初预料的要棘手。问题陈述是结构化我们从后端 API 获取的数据，并将其显示在 Material UI 的 [TreeView 组件中。我们不得不获取大量的数据，所以不能一次加载整个树。当用户单击扩展按钮时，我们需要从后端逐个获取子节点。](https://material-ui.com/api/tree-view/)

## 解决办法

我使用来自 [TreeView API](https://material-ui.com/api/tree-view/) 的`selected`和`expanded`道具的组合来解决这个问题。在本例中，我们异步加载新的子节点，并在加载后展开父节点。为了能够在内存中保存这些数据，我们还需要使用一个[图形数据结构](https://medium.com/swlh/data-structures-graphs-50a8a032db03)(这是数据结构在工程面试之外唯一对我有用的时候)。在我的例子中，我调用了数据结构`TreeNode`，它有各种方法，例如*遍历*、*添加节点*和*搜索名称*:

TreeNode

这是怎么回事？首先，我们从声明一个类开始，这只是用传统的方式在 JavaScript 中声明函数的语法糖。该类允许我们定义类成员和构造函数来初始化该类的实例。`TreeNode`类的每个实例都有一个`id`、`root`、`name`和`children`。根表示图中的第一个节点。子节点是指向图中下一个节点的指针。id 是唯一的标识符，名称是我们将在屏幕上显示的标签。遍历函数允许我们访问图中的每个节点，还提供了一个回调函数，这意味着我们可以对图中的每个节点进行一些操作。例如，当在某个父节点下添加一个新的子节点时，我们遍历整个图，直到找到父节点的`id`，然后将新的子节点(作为`TreeNode`本身)添加到父节点的子数组中。

数据结构准备就绪后，我们现在可以实现 UI 表示部分了。假设你已经用 create-react-app 引导了你的项目，剩下的就很简单了。

我们从材质 UI 中的`TreeView`组件作为外部 JSX 开始。作为道具我们需要通过`defaultCollapseIcon`、`defaultExpandIcon`、`selected`、`onNodeSelect`、`expanded`到`TreeView`。前两个是简单的 SVG 图标，`selected`跟踪当前选择的节点，`expanded`是一个由`id`组成的数组，当前它们的子节点在屏幕上展开，`onNodeSelect`是选择新节点时要调用的函数。在`TreeView`内部，我们需要创建`TreeItem` s。我们可以递归地调用`createItemsFromTree`来添加我们需要的所有`TreeItem`。`handleChange`功能是一个事件处理程序，当点击时扩展/收缩树中的节点。它对后端 API 进行一个`async`同步调用来获取父节点的子节点，然后在 API 调用完成后显示它们。为了跟踪组件中的状态，我们使用 react 中的`useState`钩子来创建 3 个状态:`expanded`、`selected`和`tree`。将`TreeViewDemo`组件插入 ReactDOM.render 调用中，您可以看到如下所示的树:

![](img/b1df440265ceab63a5517138b91a19b1.png)

演示

## 结论

这个演示解决方案可以帮助您完成 80%的工作。其他需要考虑的是实现实际的 API 调用，在树无限大的情况下虚拟化`TreeItem` DOM 元素，设计组件样式，以及为底层 HTML 元素添加可访问性属性。如果你喜欢我的写作风格，请跟我来。我一个月发布 1 次，我喜欢网络，谢谢