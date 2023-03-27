# 更新处于反应状态的对象的属性

> 原文：<https://itnext.io/updating-properties-of-an-object-in-react-state-af6260d8e9f5?source=collection_archive---------1----------------------->

![](img/8933e4fa3854e49902a096a7f2866351.png)

[*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fupdating-properties-of-an-object-in-react-state-af6260d8e9f5) 上分享这篇文章

尽管如此，正如大多数人所知，`setState`是异步的，React 足够聪明，可以在一个动作中处理多个`setState`:

```
clickHandler() {
  this.setState({
    a: 1
  });
  this.setState({
    b: 2
  });
}render() {
  console.log('render');
  return <button onClick={this.clickHandler}/>;
}
```

你会注意到，不仅`a`和`b`通过点击按钮得到更新，在控制台中，只有一个`"render"`被打印出来。

如果您有一个处于状态的对象，并且想要更新该对象的不同属性，它仍然以同样的方式工作吗？让我们看看如何在`setState`中更新一个对象。假设我们有一个对象，我们使用一个表单来编辑它。

```
this.state = {
  todoList: {
    day: '' // Monday, Tuesday, etc...
    items: []
  }
}
```

在表单中，有一个下拉列表，用于选择从周一到周日的某一天。根据您选择的日期，将有另一个多选项目列表。这里的用例是，如果你选择了`Monday`，并且选择了一堆项目，那么你想选择另一天，比如说`Tuesday`。您刚刚为`Monday`选择的项目列表会发生什么变化？我们不能保留这份清单，因为它已经失效了，因为时间已经不多了。

当然，我们需要做一些清理工作。最直接的方法是清空`items`列表。让我们看一下代码(这里将使用 spread 运算符和一些 ES6，拜托，你必须知道)…

```
onDaySelect(day) {
  this.setState({
    todoList: {
      ...this.state.todoList,
      day,
      items: []
    }
  })
}
```

所以，我们将`day`和`items`合二为一`setState`。这里绝对没有问题。

如果，todoList 比这个例子要复杂得多。例如，该表单中有许多复选框和输入项。我希望有一个通用的方法来更新对象。

这里我用`[RamdaJS](http://ramdajs.com/)`作为例子:

```
setObjectByPath(fieldPath, value) {
  this.setState({
    todoList: R.set(R.lensPath(fieldPath), value, this.state.todoList)
  })
}
```

这样，无论对象有多复杂，您都可以轻松地为属性设置一个值，即使它嵌套在对象或数组中。(此处使用`lensPath`，以便您可以设置类似`todoList.someObject.someNestedField`的内容。

下面是我们在上面的例子中如何使用它:

```
onDaySelect(day) {
  this.setObjectByPath(['day'], day);
}onItemsSelect(items) {
  this.setObjectByPath(['items'], items);
}onSomeNestedFieldChange(value) {
  this.setObjectByPath(['objectName', 'fieldName'], value);
}
```

优雅！

所以，我们来套用一下`onDaySelect`中的上例。

```
onDaySelect(day) {
  this.setObjectByPath(['day'], day); // 1
  this.setObjectByPath(['items'], []); // 2
}
```

你很快就会注意到，它不知何故不能正常工作。

起初，我认为 React 有问题，因为它没有为我更新这两个属性。事实上，第一条线从来不起作用。为什么？

我就跳过无聊的解释了。在`setObjectByPath`函数中，总是以`this.state.todoList`为原点。由于`setState`是异步的，当两个`setObjectByPath`被调用时，状态根本没有改变。想象一下`todoList`对象在更新前是这样的:

```
{day: 'Monday', items: [1,2,3]}
```

下面是我们调用`onDaySelect`函数时实际发生的情况:

```
{day: 'Monday', items: [1,2,3]} -> {day: '***Tuesday***', items: [1,2,3]}
{day: 'Monday', items: [1,2,3]} -> {day: 'Monday', items: ***[]***}
```

问题不在 React 上，是在我写的函数上！也许…

这意味着，如果您需要在`setState`中一次更改一个以上的属性，您就不能使用那个漂亮的通用`setObjectByPath`方法。有多逊？！同样的事情发生在`Redux`身上。

与`Redux`不同，React 很聪明，不需要总是将新状态的最终结果放在`setState`函数中。当你只把`a`放入`setState`时，你可以期望属性`b`仍然存在。而在`Redux`中，如果你只是在新的状态下返回`a`，就不要指望能再找到`b`了。这意味着，您需要处理`Redux`中的整个状态对象。这也意味着，您需要将所有特定的逻辑放在一个地方，并且不能委托给多个处理程序。

在`setObjectPath`中，它以这样的方式结束:

```
setObjectPath(fieldPath, value) {
  if (fieldPath[0] === 'day') {
    this.setState({
      R.compose(R.set(), R.set()) // sets ***day*** and ***items*** together
    })
  } else if // more other cases...
}
```

尽管如此，这种方法仍然比在每个`inputHandler`中到处都有非泛型`set`函数要好。

那么，问题的解决方案是什么呢？

…

抱歉，我没有给你的。

我唯一能想到的是，把物体压平，摊到州上去，就像你给吐司涂花生酱一样:

```
this.state = {
  day: '',
  items: [],
  someField: '',
  someNestedField: ''
  // ...
}
// no more todoList object
```

我不能说这种做法是不优雅，还是丑陋，但我真的不喜欢。原因之一是，你需要在最后把所有的字段重新打包成一个对象。它不直观，增加了额外的复杂性。

开放讨论~