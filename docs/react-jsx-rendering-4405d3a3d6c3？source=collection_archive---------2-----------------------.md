# 反应:JSX 和渲染

> 原文：<https://itnext.io/react-jsx-rendering-4405d3a3d6c3?source=collection_archive---------2----------------------->

![](img/8f4481631a127e1318ab2fa325ed2415.png)

如果您是 React 新手，您可能听说过 JSX，或 *JavaScript XML —* 它是一种类似 XML 的元素和组件代码。在本文中，我们将了解什么是 JSX&为什么我们应该在 React 应用程序中使用它。我们还将了解什么是元素，以及如何将它们呈现到 DOM 中。

🤓*想与 web dev 保持同步吗？*
🚀*想要将最新消息直接发送到您的收件箱？
🎉加入一个不断壮大的设计师&开发者社区！*

**在这里订阅我的简讯→**[**https://ease out . EO . page**](https://easeout.eo.page/)

# 什么是 JSX？

如前所述，JSX 是一种类似 XML 的代码，我们可以在用 React 编码时使用。它是由脸书团队开发的，旨在提供更简洁的语法，帮助简化开发人员的体验。让我们看看我们的第一个例子:

```
const greeting = <h1>Hello, World!</h1>;
```

简单吧？

这里我们有的既不是字符串也不是 HTML。是 JSX！在构建 UI 元素时，我们可以使用它来充分利用 JavaScript 的能力。虽然它不是强制性的，但它确实是一个非常有用的工具——当我们在 JavaScript 代码中使用 UI 时，它很好地阐明了这一点。

# 使用 JSX

让我们扩展上面的例子，加入一个嵌入的表达式。

```
const user = 'Bob Burger';
const greeting = <h1>Hello, {user}</h1>;

ReactDOM.render(
  greeting,
  document.getElementById('root')
);
```

我们使用花括号`{}`将变量嵌入到 JSX 表达式中。在这些花括号中，我们可以嵌入任何有效的 JavaScript 表达式。例如`user.firstName`或`printName(user)`。

注意:我们将在文章的后面详细讨论渲染，现在还不要去关心上面的渲染方法！

让我们嵌入一个调用的 JavaScript 函数的结果:

```
function printName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Bob',
  lastName: 'Burger'
};

const greeting = (
  <h1>
    Hello, {printName(user)}!
  </h1>
);

ReactDOM.render(
  greeting,
  document.getElementById('root')
);
```

# 引擎盖下的 JSX

当我们渲染组件时，JSX 到底在做什么？

```
function Greeting() {
  return <h1>Hello, World!</h1>
}
```

由`Greeting`组件呈现的每个元素都被分解成`React.createElement`调用。上述示例将文件传输到:

```
function Greeting() {
  return React.createElement("h1", {}, "Hello, World!")
}
```

## React.createElement()

让我们看另一个例子:

```
const greeting = (
  <h1 className="speak">
    Hello, world!
  </h1>
);
```

编译后，该代码如下所示:

```
const greeting = React.createElement(
  'h1',
  {className: 'speak'},
  'Hello, world!'
);
```

这两个代码块是相同的。本质上，一个对象是这样创建的:

```
const greeting= {
  type: 'h1',
  props: {
    className: 'speak',
    children: 'Hello, world!'
  }
};
```

这个对象被称为 React 元素，它的功能非常类似于对您在屏幕上看到的内容的描述。React 使用这些对象构建 DOM 并保持其最新。

从本质上说，JSX 只是让`React.createElement(component, props, …children)`功能看起来更顺眼。另一个例子:

```
<Navbar backgroundColor = "purple" opacity = {0.8}>
  Menu bar
</Navbar>
```

将传送到:

```
React.createElement(Navbar, {
  backgroundColor: "purple",
  opacity: 0.8
}, "Menu bar");
```

现在让我们继续看更多的概念…

# JSX 的道具

在我的下一篇文章中，我们将深入探讨**道具**！现在最好记住，在构建组件时——它们通常会渲染子组件，这需要数据来正确渲染。我们传入的参数就是我们所说的道具。在 JSX，有几种方法可以做到这一点，例如:

```
// Defaults to "true" if no value is passed
<MyComponent connected />// String literals passed as props
<MyComponent user= "Bob Burger" />// Expressions (below example will evaluate to 10)
<MyComponent total = {1 + 2 + 3 + 4} />// Spread attributes: passes the whole props object
<MyComponent selected = {...this.state} />
```

注意:`if`语句和`for`循环不是 JavaScript 中的表达式，所以不能直接在 JSX 中使用！相反，您可以这样编码:

```
function NumberType(props) {
  let answer;
  if (props.number % 2 == 0) {
    answer = <strong>even</strong>;
  } else {
    answer = <i>odd</i>;
  }
  return <div>{props.number} is an {answer} number</div>;
}
```

我们可以看到我们的道具被传递到条件中，被评估，然后被返回——所有这些都是通过 JSX 完成的。

# JSX 的儿童

随着您的应用程序变得越来越大，您会发现一些组件需要渲染子组件。然后这些子组件还需要渲染更多的子组件，以此类推！有了 JSX，我们可以很好地管理这些元素的树状结构。经验法则是——组件返回的任何元素都会成为其子元素。

让我们快速看一下用 JSX 渲染子元素的方法:

## 字符串文字

```
<MyComponent>I'm a child!</MyComponent>
```

在这个非常基本的例子中，字符串`I’m a child`是`MyComponent`的子元素。通过`MyComponent`的`props.children`可以到达。

## 小时候的 JSX 元素

假设我们要返回一个 HTML 子节点`<header>`，它有两个自己的子节点:`<Nav />`和`<SearchBox />`。我们可以这样做:

```
function Header(props) {
  return (
    <header>
      <Nav />
      <SearchBox />
    </header>
  )
}
```

## 公式

我们还可以将表达式作为子对象传递，以呈现到我们的 UI 中。这对于待办事项应用程序非常有用，例如:

```
function TodoItem(props) {
  return <li>{props.content}</li>;
}

function TodoList() {
  const todos = ['paint house', 'buy more chips', 'conquer world'];
  return (
    <ul>
      {todos.map((content) => <TodoItem key={content} content={content} />)}
    </ul>
  );
}
```

## 功能

函数在处理重复时很有用，比如呈现重复的 UI 元素。我们可以创建 React 将自动为我们呈现的结构。

让我们看一个例子，我们使用一个`.map()`函数在网站上创建新页面:

```
// Array of current pagesconst pages = [
  {
    id: 1,
    text: "Home",
    link: "/"
  },
  {
    id: 2,
    text: "About",
    link: "/about"
  },
  {
    id: 3,
    text: "Contact",
    link: "/contact"
  }
];// Renders a <ul> which generates the <li> childrenfunction Nav() {
  return (
    <ul>
      {pages.map(page => {
        return (
          <li key={page.id}>
            <a href={page.link}>{page.text}</a>
          </li>
        );
      })}
    </ul>
  );
}
```

要向我们的网站添加一个新页面，我们只需要向数组`pages`添加一个新对象&让 React 处理剩下的事情！

# 渲染元素

我相信你在这篇文章中已经看到，当使用 JSX 的时候，我们是在使用元素来渲染我们的页面。一个*元素*描述了您在屏幕上看到的内容:

```
const element = <h1>Hello!</h1>;
```

诸如此类的多种元素组合在一起将形成组件。在我的下一篇文章中，我们将详细讨论组件！

## 将我们的元素呈现给 DOM

通常，在我们的 HTML 中会有一个像这样的`<div>`:

```
<div id="root"></div> 
```

这就是我们的 DOM 节点。它里面的一切都是由 React DOM 处理的。

为了将 React 元素呈现到我们的根节点中，我们将两者都传递给`ReactDOM.render(),`,如下所示:

```
const element = <h1>Hello!</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

`Hello!`将渲染到我们的页面。

## 更新渲染元素

注意，React 元素是不可变的！一旦创建了元素，就不能更改其子元素或其属性。如果想要更新 UI，您需要创建一个新元素并将其传递给`ReactDOM.render()`。

***你准备好让你的 JavaScript 技能更上一层楼了吗？*** *今天就开始用我的新电子书吧！无论你是想学习你的第一行代码，还是想扩展你的知识面并真正学习基础知识..*[*JavaScript 掌握完全指南*](https://gum.co/mastering-javascript) *带你从零到英雄！*

![](img/dde515044536421c6c999650977f80c4.png)

*现已上市！👉*[https://gum.co/mastering-javascript](https://gum.co/mastering-javascript)

# 结论

我们走吧！我们已经介绍了 JSX 和渲染的基础知识。我希望你开始看到这些概念对我们作为开发人员构建 React 应用程序是多么有用。

通过利用 JSX 的能力在 JavaScript 中传递元素，我们正在构建非常可行的代码。JSX 的结构与 React 配合得非常好，尽管 JSX 不是强制性的——但它提供了一种奇妙的开发体验。

我希望这篇文章对你有用！你可以[跟着我](https://medium.com/@timothyrobards)上媒。我也在[推特](https://twitter.com/easeoutco)上。欢迎在下面的评论中留下任何问题。我很乐意帮忙！

# 关于我的一点点..

嘿，我是提姆！👋我是一名开发人员、技术作家和作家。如果你想看我所有的教程，可以在[我的个人博客](http://www.easeout.co)上找到。

我目前正在撰写我的[自由职业完整指南](http://www.easeout.co/freelance)。坏消息是它还不可用！但是如果这是你可能感兴趣的东西，你可以[注册，当它可用时会通知你👍](https://easeout.eo.page/news)

感谢阅读🎉