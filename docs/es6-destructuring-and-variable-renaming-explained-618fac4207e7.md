# ES6 析构和变量重命名解释！

> 原文：<https://itnext.io/es6-destructuring-and-variable-renaming-explained-618fac4207e7?source=collection_archive---------4----------------------->

![](img/ac9dbdd8b495174b6069d837538d270a.png)

析构对象和数组可能是 ES6 中最常用的特性，这是有道理的。这种简单的技术对于编写更简洁、可读性更强的 JavaScript 代码来说是惊人的。ES6 附带的另一个很酷的特性是变量重命名。唯一的问题是，这可能会给仍在努力掌握现代 JavaScript 所能提供的一切的编码人员带来一些困惑。让我们在实践中看看这两个概念，并向您展示陷阱通常出现在哪里。

# 实践中的基本解构

假设我们有一个`person`对象，它的键`name`和`surname`声明如下:

```
const person = {
  name: 'Barry',
  surname: 'Doyle',
};
```

如果你想以传统方式将`person`的`name`和`surname`记录到控制台，你可以这样做:

```
console.log(person.name, person.surname); // Barry Doyle
```

但是因为我们都是 DRY 的忠实粉丝(不要重复自己的话)，我们可以使用 ES6 来破坏像这样的人的`name`和`surname`:

```
const { name, surname } = person;console.log(name, surname); // Barry Doyle
```

这显然需要写更多的行，所以现在可能不太理想。但是当你开始处理更大的对象时，它会有很大的帮助。作为一般的经验法则，如果析构阻止你重复你自己，那么就使用它！否则就别烦了，不值得。例如，如果我们只想将`name`记录到控制台，那么我会强烈反对只对一个变量使用析构。

# 更深的嵌套析构

上面我给了你一个简单的例子。让我们来看看更复杂的东西。如果我们有一个客体的客体呢？想象一下，我扩展了我的`person`对象来包含一个名为`skills`的键，它是一个代表我的一些技能的键的对象。出于这个例子的目的，我将给我的`skills`对象两个键:`JavaScript`和`React`。这些键中的每一个都有一个里面有一个键的对象的值。我的每个对象的键都是`years`,它代表我在对象键中的经验年限。下面是我们新的扩展`person`对象的声明:

```
const person {
  name: 'Barry',
  surname: 'Doyle',
  skills: {
    JavaScript: {
      years: 7,
    },
    React: {
      years: 4,
    },
  },
};
```

同样，分别为`JavaScript`和`React`注销我的`name`、`surname`和多年经验的老式方法是做以下事情:

```
console.log(
  person.name,
  person.surname,
  person.skills.JavaScript.years,
  person.skills.React.years
); // Barry Doyle 7 4
```

现在，使用我们出色的 ES6 析构技术，我们可以简化控制台日志，如下所示:

```
const { name, surname, skills: { JavaScript, React } } = person;console.log(name, surname, JavaScript.years, React.years)
// Barry Doyle 7 4
```

这是一个奇怪的不寻常的例子，但我希望你明白这一点。我们可以更进一步，从`JavaScript`或`React`对象中析构`years`。然而，我们不能同时从两个变量中得到它，因为那样我们会有一个变量名指向两个不同的值，这是行不通的。一个解决方法是在我们析构变量时重命名`year`变量。

# 析构时重命名变量

让我们以上面的例子为例，将名称`javaScriptYears`和`reactYears`分别分配给分配给`JavaScript`和`React`对象的`years`键。为了保持代码的整洁，我将对析构语句进行多行处理。

```
const {
  name,
  surname,
  skills: {
    JavaScript: { years: javaScriptYears },
    React: { years: reactYears },
  },
} = person;console.log(name, surname, javaScriptYears, reactYears);
// Barry Doyle 7 4
```

使用这种技术，我们避免了通过从我们的`person`对象中析构两个不同的`years`值而产生的重复变量名问题。以便在析构变量时重命名变量。您需要在您正在析构的变量后添加一个`:`,后跟您想要将析构变量重命名为的变量名。

注意我们怎么说`years: javaScriptYears`而不是`years: { javaScriptYears }`。做前者会将`years`重命名为`javaScriptYears`，而做后者意味着`years`是一个带有键`javaScriptYears`的对象，而你正试图从`years`中析构它。

# 析构可以在方法内部完成

关于析构最好的部分是，它可以很容易地在运行中完成。假设我们有一个名为`people`的对象数组，实现如下:

```
const people = [
  { name: 'Barry' },
  { name: 'Michael },
  { name: 'Doyle' },
];
```

现在让我们通过`people`绘制地图，并以传统方式将每个`name`登录到控制台:

```
people.map(person => console.log(person.name));
// Barry
// Michael
// Doyle
```

这就是你如何用 ES6 的奇特方式做到的:

```
people.map(({ name }) => console.log(name));
// Barry
// Michael
// Doyle
```

现在看起来可能没那么神奇。但是相信我，在处理更复杂的数据结构时，这些技术确实节省了大量时间，并使所有内容更容易阅读。

# 一个常见的陷阱和这篇文章的灵感

作为一名全职 React 开发人员，我经常看到同事们因为对变量的析构和重命名有点混淆而放弃在事件处理程序中析构。

看看这个基本的 React 组件:

```
import React, { Component } from 'react';
// ^^^ Look! we're even destructuring in our imports :Dexport default class SimpleInput extends Component {
  state = {
    inputText: '',
  } constructor(props) {
    super(props);
    this.handleInputChange = this.handleInputChange.bind(this);
  } handleInputChange(event) {
    this.setState({ inputText: event.target.value });
  }
  // ^^^ Our focus is going to be here render() {
    return (
      <input
        onChange={this.handleInputChange}
        value={this.state.inputText}
      />
    );
    // ^^^ Notice how we don't bother destructuring inputText
    // ^^^ from this.state because it doesn't prevent "DRY"
  }
}
```

这是一个简单的 React 组件，实际上并没有做太多事情。但是嘿！它应该正常工作。我希望我们把注意力放在`handleInputChange`事件处理程序上。

人们在试图析构变量时犯的一个常见错误是他们这样做:

```
handleInputChange({ target: value }) {
  this.setState({ inputText: value });
}
```

当我们试图这样做的时候，你看，我们得到了疯狂的错误。然后我们开始不可避免地放弃，回到传统的方式去做事情，因为它从未让我们失望。

这个实现有什么问题？

当我们想从`target`对象中取出`value`时，我们实际上是将`target`对象重命名为`value`。记住，我们需要做这个`({ target: { value } })`而不是`({ target: value })`。

我们可以通过将被析构的`value`变量重命名为`inputText`来进一步发展这种析构技术，并使用另一个方便的 ES6 特性，通过这样做使我们的代码可读性更好:

```
handleInputChange({ target: { value: inputText } }) {
  this.setState({ inputText });
}
```

我还没有深入研究这个特性，但是这里有一个 TL:dr；
`({ inputText })`与`({ inputText: inputText })`相同。

# 结论

我希望这篇文章教会了你一些新的东西，或者至少巩固了你对这些流行的 ES6 技术的理解。

这篇文章的灵感来自于[完整的软件开发人员职业指南:如何快速学习编程语言，如何在编程面试中胜出，如何获得你梦想中的软件开发人员工作](https://amzn.to/2DrCUiR)。这本书在提供的链接上非常便宜，而且价值远远超过你为它支付的费用。毫无疑问，它是迄今为止对我作为一名专业软件开发人员的成长影响最大的因素！

如果你喜欢这篇文章，请留下一些赞或掌声。另外，请务必在[脸书](https://www.facebook.com/barrymichaeldoyle)、[推特](https://twitter.com/barrymdoyle)、[媒体](https://medium.com/@barrymdoyle)、 [YouTube](https://www.youtube.com/barrymichaeldoyle?sub_confirmation=1) 和 [LinkedIn](https://www.linkedin.com/in/barry-michael-doyle-11369683/) 上关注我，随时更新我制作的新内容。

现在出去写史诗代码吧！

*原载于 2018 年 9 月 24 日*[*【www.barrymichaeldoyle.com】*](https://www.barrymichaeldoyle.com/destructuring/)*。*