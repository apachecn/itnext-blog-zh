# 使用 react-beautiful-dnd 在拖动过程中动态更新位置

> 原文：<https://itnext.io/dynamically-update-positions-during-drag-using-react-beautiful-dnd-4a986d704c2e?source=collection_archive---------2----------------------->

# 什么是 react-beautiful-dnd？

react-beautiful-dnd 是一个用于 ReactJS 的拖放库。它是超级强大的，给你一个开箱即用的可用性吨。你可以在这里阅读更多信息[。本文假设您对这个库有一个基本的了解，并且希望能够添加一些漂亮的修饰功能。在本例中，它在拖动动作期间动态显示项目的位置，如下所示！我们开始吧！](https://github.com/atlassian/react-beautiful-dnd)

![](img/e83869de7fc3749aede422c283836f3f.png)

# react-beautiful-dnd 拖的时候到底在干嘛？

首先，为了简单起见，从现在开始我将把 react-beautiful-dnd 称为 dnd。

至于 dnd 如何处理传递给它的数据，标准情况下，我们在这里使用的是一个 DragDropContext 组件，它有一个由 Draggables 数组填充的 Droppable。例如，上述组件的代码如下:

```
<DragDropContext onDragEnd={this.onDragEnd}> <Droppable droppableId="droppable"> {provided => ( <div ref={provided.innerRef}> {this.state.content.map((c, index) => ( <Draggable key={c.id} draggableId={c.id} index={index}> {(provided, snapshot) => ( <ContentWrapper ref={provided.innerRef} {...provided.draggableProps} {...provided.dragHandleProps} > <ContentBlock content={c} displayPosition={index + 1} /> </ContentWrapper> )} </Draggable> ))} </ContentWrapper> )} </Droppable></DragDropContext>
```

** `ContentWrapper`和`ContentBlock`都是[样式化的组件](https://github.com/styled-components/styled-components)，为本文中的示例提供了美学外观。他们俩只是一个风格的`div`

`displayPosition`这里只是该项在数组+ 1 中的索引，所以对于数组`[A, B, C]`来说，各自的显示位置是`A: 1, B: 2, C: 3`

我们使用`this.state.content`作为我们的数据数组。当您拖动时，dnd 会更新数组的顺序以反映这些更改。为了保存，您需要做的就是在`<DragDropContext>`上挂接`onDragEnd`事件并更新状态。类似于:

```
onDragEnd = result => { if (!result.destination) { return } const content = reorder(this.state.content, result.source.index,   result.destination.index) this.setState({ content })}
```

太好了！这是一个函数，它根据一个元素的移动来改变一个数组。这是函数:

```
const reorder = (list, startIndex, endIndex) => { const result = Array.from(list) const [removed] = result.splice(startIndex, 1) result.splice(endIndex, 0, removed) return result}
```

好了，这是一大堆连续的代码，让我们绕回来。当我们拖动时，只有这些而没有更新，我们在这里会有什么？

![](img/aad2f28ad2d628b9ead283ad4cd93e4d.png)

厉害！拖动结束后，我们调用`onDragEnd`，它又调用`reorder`。Reorder 将根据拖动的结果改变我们的数组，瞧！我们完成拖动，数组被重新排序，我们把它提交给 state。决定`displayPosition`在重新渲染时更新的索引。

因此，要在拖动时让它工作，我们所要做的就是添加与`onDragEnd`完全相同的函数，并在`onDragUpdate`事件上调用它，对吗？让我们将`onDragEnd`重新命名为`onDrag`，这样更具包容性，并将其添加到:

```
<DragDropContext onDragUpdate={this.onDrag} onDragEnd={this.onDrag}>
```

太好了，让我们看看实际情况:

![](img/07c13b88f2bf9cc6dc454a2967acf1b4.png)

哇哦。到底发生了什么事？？基本上，你的数组在两个地方被修改(从 dnd 和从你的`onDrag`中的`setState`)，在状态改变时被重新渲染，尽管已经在拖动动作中，这些都产生了一些相当不可靠的副作用。最重要的是，当我们完成时，数字甚至不正确。那么我们该如何应对呢？

# 分离拖动处理程序

首先，让我们把我们的`onDrag`函数恢复成`onDragEnd`函数，我们就恢复了原来的功能。让我们也创建一个新的`onDragUpdate`函数，我们暂时将它留空。

# 将 displayPosition 添加为单独的属性

从 index 中导出`displayPosition`工作得很好，但是问题是我们在拖动过程中不断地重新计算。让我们把它从渲染中提取出来，放到每一项内容的一个新键中。

```
const resetDisplayPositions = list => { const resetList = list.map((c, index) => { const slide = c slide.displayPosition = index + 1 return slide }) return resetList}
```

这是在数组中的每一项上创建一个`displayPosition`键，对应的值为`index + 1`。如果我们在初始化时运行它，我们就可以一直拥有这个密钥:

```
getContent = () => resetDisplayPositions(this.props.content)constructor(props) { super(props) this.state = { content: this.getContent(), }}
```

好了，我们已经把它提取出来了，但是现在它只在组件初始化的时候被调用，而我们需要在每次重新排序的时候更新它们。所以我们只需要在`reorder()`中添加一个调用

```
const reorder = (list, startIndex, endIndex) => { const result = Array.from(list) const [removed] = result.splice(startIndex, 1) result.splice(endIndex, 0, removed) return resetDisplayPositions(result)}
```

好了，我们完全回到了原来的功能，我们已经提取了`displayPosition`。让我们转到`onDragUpdate`

# onDragUpdate

好的，我们知道我们想要更新`displayPosition`，但是在我们更新的时候不更新数组的底层顺序(一旦拖动完成，我们将调用`reorder()`)。每当被拖动的项目改变位置时，就会触发`onDragUpdate`,所以让我们想一想当这种情况发生时我们需要改变什么:

*   被拖动项目的`displayPosition`需要改变
*   被拖动项目传递的项目也需要其`displayPosition`改变

太好了！我们开始吧。让我们从基础开始，处理结果中是否缺少目的地:

```
onDragUpdate = result => { if (!result.destination) return const { content } = this.state const dragged = content[result.source.index] const previousDraggedIndex = dragged.displayPosition
```

酷，现在让我们将拖动项的`displayPosition`设置为新的索引。

```
dragged.displayPosition = result.destination.index + 1
```

半路上！现在让我们改变被跳过的项目。有一个地方有点棘手，那就是方向取决于拖动的方向，所以让我们取出索引差来得到 1 或-1。这总是绝对值为 1，因为每次更新时都会调用它。

```
const draggedIndexDifference = dragged.displayPosition - previousDraggedIndex
```

好了，现在剩下唯一要做的就是遍历数组，更改受影响的幻灯片并更新状态！

```
const updatedContent = content.map((c, index) => { const slide = c if (slide.displayPosition === result.destination.index + 1 && index !== result.source.index) { slide.displayPosition -= draggedIndexDifference } return slide})
```

这里只有一张幻灯片受到影响，因为我们在这里检查两件事:

*   如果幻灯片的`displayPosition`是被拖动的幻灯片所在的位置(这样我们就可以把范围缩小到被拖动和受影响的幻灯片)
*   如果一个幻灯片不是被拖动的幻灯片，我们只剩下受影响的幻灯片为被拖动的幻灯片让路

如果阻力向上，该幻灯片的新`displayPosition`增加 1，反之，如果阻力向下，则增加 1。我们剩下的就是重置状态(记住，数组的顺序没有改变，只是相关的 displayPositions 没有改变)。这是该函数的最终版本:

```
onDragUpdate = result => { if (!result.destination) { return } const { content } = this.state const dragged = content[result.source.index] const previousDraggedIndex = dragged.displayPosition dragged.displayPosition = result.destination.index + 1 const draggedIndexDifference = dragged.displayPosition - previousDraggedIndex const updatedContent = content.map((c, index) => { const slide = c if (slide.displayPosition === result.destination.index + 1 && index !== result.source.index) { slide.displayPosition -= draggedIndexDifference } return slide }) this.setState({ content: updatedContent, })}
```

这给了我们顶部的全功能风格！

![](img/e83869de7fc3749aede422c283836f3f.png)

# 概述

所以让我们用简单的英语快速回顾一下这里发生的事情的步骤(你有上面所有的代码！)

*   在初始渲染一个对象数组时，一个`displayPosition`的键被放在每个值为`index + 1`的对象上
*   那个`displayPosition`用来显示一个项目在数组中的位置
*   拖动时，`displayPosition`在`onDragUpdate`事件期间更新
*   当拖动完成时，整个数组在`onDragEnd`事件期间被重新排序
*   在`reorder`期间，显示位置被重置为索引+ 1

就是这样！希望你发现这是有益的和快乐的拖动！