# 使用 Web 组件处理数据

> 原文：<https://itnext.io/handling-data-with-web-components-9e7e4a452e6e?source=collection_archive---------0----------------------->

![](img/65949d7799265cbbae71c054ea537ade.png)

照片由 [Denys Nevozhai](https://unsplash.com/@dnevozhai?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

如果您在过去几年中没有生活在岩石下，您可能听说过术语 Web 组件，这是一个 API 集合，用于轻松扩展现有的 HTML，并且用于构建可重用的组件，而不需要像 React 这样的框架。

尽管如此，大多数开发人员倾向于支持上述框架，因为它们给了他们更多的工具和灵活性。像数据绑定和状态管理这样的东西，对于构建数据驱动的交互式 ui 来说无疑是很重要的。

在 React 的例子中，还有另一个很棒的概念:[道具](https://reactjs.org/docs/components-and-props.html)。它的 API 提供了一种将数据从父组件传递到其子组件的简单而直接的方式。这个数据实际上可以是任何东西；字符串、数字、对象甚至函数。

```
<MyComponent
  variant="primary"
  onUpdate={functionCallback)}
  items={[1, 2, 3]}
/>
```

[Web 组件](https://developer.mozilla.org/en-US/docs/Web/Web_Components)，或者更确切地说[定制元素](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)，有一个类似的概念叫做属性和特性。然而，不幸的是，它们使用起来略有不同，不如 React 道具舒适。

让我们来看看如何通过 4 种不同的方式实现同样的事情——将复杂的数据传递到我们的自定义元素中，好吗？

# #1 —使用属性

属性是将数据传递到自定义元素的最简单方法:

```
<custom-list filter="some filter"></custom-list>
```

定制元素的属性与我们已经知道并喜欢的属性没有什么不同:`class`、`value`、`label`以及更多我们每天在 HTML 中使用的属性。甚至还有一个所有可用标准属性的列表[作为参考。让我们看一些代码:](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes)

```
<img src="image-1.jpg" alt="Some image"><script>
  var img = document.getElementsByTagName("img")[0]; img.getAttribute("src"); // -> "image-1.jpg"
  img.getAttribute("alt"); // -> "Some image"
</script>
```

在上面的例子中，我们创建了一个新的图像元素，并传递了一些属性，比如`src`和`alt`。使用`getAttribute()`函数，可以在 JavaScript 中访问所有这些属性。

如果您使用过 DOM，您可能知道有一种方法不仅可以获取属性，还可以设置属性:

```
img.setAttribute("src", "new-image.jpg");
```

在这个例子中，我们用一个不同的图像源替换了旧的图像源，如果您的网站上有这个图像，它将会继续从新的路径下载。

同样的概念也适用于定制元素，但是您可以自由选择如何命名每个属性。因为自定义元素必须总是扩展`HTMLElement`，所以我们可以免费获得这个行为。

让我们从上面看一下`custom-list`元素的实现:

```
class CustomList extends HTMLElement {
  constructor() {
    super(); console.log(this.getAttribute("filter"));
  }
}customElements.define("custom-list", CustomList);
```

*   你首先会注意到的是构造函数，我们称之为`super()`。您不必总是使用构造函数，因此可以省去这段代码，但在某些情况下，构造函数非常有用，例如在初始化属性、状态、事件侦听器或其他与元素相关的内容时。请记住:一旦使用了构造函数，就需要调用`super()`来让定制元素正常工作。
*   下面，我们记录了`this.getAttribute("filter")`，它将打印出我们在 HTML 中赋予`filter`的任何值。
*   在最后一行，我们只需注册定制元素，使其在全球范围内可用。

为了查看结果，您现在可以使用这个定制元素并为`filter`属性赋值:

```
<custom-list filter="Hello World"</custom-list>
```

在浏览器的控制台中，应该会显示“Hello World”。

但是我们可以更进一步，在 JavaScript 中以编程方式设置和获取任何属性:

```
var list = document.querySelector("custom-list");list.getAttribute("filter"); // -> "Hello World"
list.setAttribute("filter", "A new value");
list.getAttribute("filter"); // -> "A new value"list.setAttribute("whatever", "Suprise");
list.getAttribute("whatever"); // -> "Suprise"
```

您会看到展示的行为与标准 HTML 元素相同，但是您可以自由决定使用哪些属性以及如何命名它们。

## 访问元素内部的属性

到目前为止，我已经向您展示了如何在自定义元素的外部设置和获取属性。但是在许多情况下，您可能需要访问自定义元素中的这些属性，否则，它们将毫无用处，对吗？

为了与定制元素中的属性进行交互，我们使用相同的方法`getAttribute`和`setAttribute`，如下所示:

```
class CustomList extends HTMLElement {
  render() { this.innerHTML = `
      <p>The filter is: ${this.getAttribute("filter")}
    `;
  }
}
```

这次的不同之处在于，您需要使用`this`关键字，它指的是自定义元素本身。JavaScript 中的`this`可能有点混乱，但谢天谢地[中有很多关于救援的解释](https://medium.com/quick-code/understanding-the-this-keyword-in-javascript-cb76d4c7c5e8)。

## 观察属性

很好，我们已经了解了如何设置和获取自定义元素的属性。但是如果一个属性在页面生命周期中发生了变化，比如用户在输入中输入了一些东西，那该怎么办呢？

```
<input type="text"><script>
  var list = document.querySelector("custom-list");
  var input = document.querySelector("input"); input.onchange = (evt) => {
    list.setAttribute("filter", evt.target.value);
  }
</script>
```

Web 组件规范得到了您的支持:

```
class CustomList extends HTMLElement {
  static get observedAttributes() {
    return ["filter"];
  } attributeChangedCallback(name, oldValue, newValue) {
    // do something when an attribute has changed
  }
}
```

`observedAttributes`是一个静态 getter 方法，它总是需要返回我们想要观察的所有属性的数组。在这个例子中，它只观察到了`filter`属性(当然也可能有更多)。

这只是第一步。下面，你会看到一个叫做`attributeChangedCallback`的方法，它是 Web 组件[生命周期](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#Using_the_lifecycle_callbacks)的一部分。每当我们观察到的任何属性发生变化时，这个方法就会运行，并执行您在内部定义的任何逻辑。它接收三个参数:已更改属性的名称、旧值和新值。

为了避免不必要的重新渲染，最好的做法是检查值是否真的改变了。

```
attributeChangedCallback(name, oldValue, newValue) {
  if (oldValue === newValue) {
   return;
 }

  console.log(`The attribute ${name} has changed`);
}
```

首先，我们检查新值是否与旧值不同。如果不是，我们停止执行，其他什么也不会发生。如果确实不同，我们就可以运行任何代码，比如计算、重新渲染或者你需要的任何东西。

## 但是像对象这样的复杂数据呢？

不过，有一点需要注意:传递属性只适用于字符串。甚至像`items="2"`这样传递的数字也会被转换成一个字符串。传递对象、数组或函数就更不可能了。

传递对象的一种(相当难看的)方法是使用`JSON.stringify`将它转换成字符串:

```
<custom-list items="[{id:1},{id:2}]"></custom-list><script>
  var list = document.querySelector("custom-list"); list.setAttribute("items", JSON.stringify(...));
</script>
```

但是，您的自定义元素需要将该字符串解析回一个对象:

```
render() {
  try {
    JSON.parse(this.getAttribute("items"));
  } catch (ex) {...}
}
```

使用这种方法将复杂数据传递到 Web 组件中是不可取的，如果可能的话，应该避免使用这种方法。让我们看看更好的方法。

# #2 —使用属性

我不知道你，属性和性质的区别一开始让我很困惑。我花了一些时间才明白过来，所以让我试着解释一下。

看看这个简单的`img`元素。属性是我们在 HTML 中设置的，比如:`<img src="image-1.jpg">`。这个元素现在有一个名为`src`的属性，也可以设置许多其他属性，比如`alt`或`width`。

属性赋予元素以意义，在语义上很重要。它们主要被屏幕阅读器、搜索引擎和其他人用来理解你的 HTML 结构。

一旦浏览器解析了您的 HTML，它会将`img`(连同所有其他元素)转换成一个 [DOM 元素对象](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)，在本例中是一个`HTMLImageElement`。对于每个 HTML 标签，都有一个 DOM 对象来定义它的行为、外观等等。好奇看看有多少？[看这里](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement)。这些物体有一大堆属性，我们当时只使用了其中的几个:

```
var HTMLImageElement = {
  src: "image-1.jpg",
  alt: null,
  accessKey: null,
  ...
};
```

属性就像绑定到对象的变量。

当浏览器在渲染期间创建所有这些 DOM 元素对象时，其属性将反映 HTML 属性。这就是为什么 image-1.jpg 的`src`是*，但是其他属性仍然被设置为`null`(我们在 HTML 中没有用到它们)。这意味着`src`属性反映了`src`属性。浏览器在幕后为我们创建并同步这两个。*

*实际上，这比我刚才解释的要复杂得多。如果你有兴趣了解细节，在 StackOverflow 的这个帖子中有一个很好的[解释。](https://stackoverflow.com/questions/6003819/what-is-the-difference-between-properties-and-attributes-in-html)*

## *Getters 和 setters*

*在我们可以在自定义元素中使用属性之前，我们需要理解[getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)和[setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set)的概念。*

*在类或函数中使用 Getters 和 setters 来设置和获取这些属性。看看这个例子:*

```
*class User {
  constructor() {
    this._name = "John Doe";
    this._age = 25;
  } get name() {
    return this._name;
  } set name(name) {
    this._name = name;
  } get age() {
    return `This user is ${this._age} years old`;
  } set age(age) {
    if (age < 18) {
      throw new RangeError("User must at least be 18 years old!");
    } this._age = age;
  }
}*
```

*   *我们有一个名为`User`的类，它拥有两个属性:`_name`和`_age`。请注意这两个属性是如何以下划线开头的；这不是强制性的，但被认为是实现类似[私有变量](https://stackoverflow.com/questions/44734399/what-is-the-underscore-in-javascript)的最佳实践。*
*   *在第 7 行，您将看到第一个 getter `name`，它只返回分配给`_name`的值。这是最简单的吸气剂形式。*
*   *在下面的第 11 行，您将遇到第一个 setter `name`，它用于为属性`_name`赋值。因此，它需要接收一个参数。*
*   *下一个 getter 基本上做了同样的事情，但是在这里我把一些文本放在了用户的年龄上。这应该向您展示，您可以从 getter 返回任何内容，而不仅仅是属性值。但不要对它太着迷。*
*   *最后，`_age`的最后一个设置器包括一些基本的验证——我们不接受 18 岁以下的用户，所以如果有人试图设置更小的年龄，我们会抛出一个异常。*

*如果你想和这个类互动，你应该这样做:*

```
*var user = new User();console.log(user.name); // -> "John Doe"
console.log(user.age); // -> "This user is 25 years old"user.name = "Jane Doe";console.log(user.name); // -> "Jane Doe"user.age = 17; // Ka-bumm!*
```

*尽管您可以这样做，但直接访问这些属性并不是一个好的做法:*

```
*var user = new User();console.log(user._name) // -> "John Doe"
// (°_°)*
```

## *自定义元素中的属性*

*你还在吗？太好了，你已经通过了干燥理论。让我们看看这些知识如何帮助我们实现定制元素。*

```
*class CustomList extends HTMLElement {
  constructor() {
    super();

    this._items = [];
  } set items(value) {
    this._items = value;
  }

  get items() {
    return this._items;
  }
}*
```

*同样，我们定义了 getter 和 setter 方法，我们希望传递给自定义元素的每个属性都需要这些方法。*

*既然我们已经将`items`声明为元素的可配置属性，我们可以使用 JavaScript 传递它:*

```
*var list = document.querySelector("custom-list");list.items = [1, 2, 3];console.log(list.items); // -> [1, 2, 3]*
```

*注意到我们没有使用 HTML 传递属性了吗？没错，只有属性可以用 HTML 传递，其他的都需要用 JavaScript 来完成。*

*同样，您不一定需要 getter 和 setter，可以直接访问属性，但我不鼓励这样做。*

```
*list._items = [1, 2, 3];console.log(list._items); // -> [1, 2, 3]
// (°_°)*
```

*幸运的是，我们并不局限于使用数组。每种数据类型都可以传递，比如这个函数:*

```
*list.callback = () => console.log("Hello World");list.callback(); // -> "Hello World"*
```

*这就是我们在自定义元素中处理复杂数据的方式。还不算太糟，是吧？但是等等，还有更多。*

# *#3 —使用事件*

*使用属性和特性是很好的，但是如果您想要基于某些事件做出反应呢？几乎所有的 HTML 元素都有我们可以监听的事件，比如`input`:*

```
*var input = document.querySelector("input");input.addEventListener("keydown", handleKeyDown);*
```

*`input`元素内置了逻辑，每当用户按下一个键时就会发出`keydown`事件。我们不能为自定义元素使用类似的东西吗？是的，我们可以！*

*然而，我们不能依赖像`keydown`或`click`这样的事件，它们是特定标准元素的 JavaScript 规范的一部分。相反，我们必须构建自己的事件，同时利用[自定义事件 API](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Creating_and_triggering_events) 。*

```
*class ClickCounter extends HTMLElement {
  constructor() {
    super();

    this._timesClicked = 0;

    var button = document.createElement("button");
    button.textContent = "Click me";
    button.onclick = (evt) => {
      this._timesClicked++;

      this.dispatchEvent(new CustomEvent("clicked", {
        detail: this._timesClicked
      }));
    };

    this.append(button);
  }
};customElements.define("click-counter", ClickCounter);var counter = document.querySelector("click-counter");counter.addEventListener("clicked", (evt) => {
  console.log(evt.detail);
});*
```

*在上面的代码示例中有很多事情要做。深呼吸，伸展你的手臂，让我们开始吧。*

*   *在第 1 行，我们创建了一个名为`ClickCounter`的新定制元素类。*
*   *然后我们定义构造函数并调用`super()`来继承`HTMLElement`。*
*   *现在，在第 5 行，我们定义了属性`_timesClicked`，它作为元素的本地状态。我们希望用户每点击一次按钮就增加 1。*
*   *在第 7 行和第 8 行，我们创建了一个新按钮并设置了一些文本内容。*
*   *在第 9 行，我们为`onclick`定义了一个函数，每次用户点击按钮时都会调用这个函数。在这个事件回调中，我们首先将计数器加 1，然后调度一个自定义事件。*
*   *这个定制事件被称为`clicked`，但是它可以是你喜欢的任何名字。作为第二个参数，我们传递一个选项来配置这个自定义事件。`detail`属性用于传递我们自己的数据和事件。这可以是一个对象，数组，字符串，甚至是一个函数。稍后，我们将使用`detail`属性来获取用户点击按钮的次数。*
*   *下面，我们将按钮添加到自定义元素的 DOM 中。*
*   *在第 23 行，我们使用`document.querySelector`获取元素的引用，并将其保存到一个变量中。最后，我们添加一个事件监听器，它监听我们前面定义的完全相同的事件，并添加一个回调函数。这个函数接收`evt`对象作为唯一的参数，这个对象的一部分是属性`detail`，在其中我们可以访问某人点击按钮的次数。*

*这个相当简单的例子应该让您对如何将事件侦听器附加到自定义元素有一个基本的了解。事件的触发时间和方式以及随事件一起提交的数据都由您决定。*

*但是有一个问题:这种技术在有许多不同的 Web 组件需要相互通信的应用程序中并不真正起作用(也不能扩展)。您不希望为每个组件监听许多不同的事件，这会使您的代码不可读且复杂。让我们来看看如何解决这种情况。*

# *#4 —使用事件总线*

*最后一种技术也利用了自定义事件 API，但是我们没有监听组件上的本地事件，而是定义了一个应用程序范围的全局总线，该总线传输我们的事件并使它们随处可访问。*

*根据您的需要，事件总线可能会变得相当复杂。在下面的例子中，我想保持简单，这样你就可以完全理解这个概念背后的想法。*

```
*class EventBus {
  constructor() {
    this._bus = document.createElement('div');
  }

  register(event, callback) {
    this._bus.addEventListener(event, callback);
  } 

  remove(event, callback) {
    this._bus.removeEventListener(event, callback);
  } fire(event, detail = {}) {
    this._bus.dispatchEvent(new CustomEvent(event, { detail }));
  }
}var bus = new EventBus();export default bus;*
```

*上面的代码示例是受这篇文章启发的[，这篇文章更加详细。](https://pineco.de/creating-a-javascript-event-bus/)*

*基本上，我们创建了一个新的 JavaScript 类。一旦这个类被初始化，我们就创建一个新的 HTML 元素(它不会被追加，也不会在 DOM 中使用)。这个 HTML 元素是必需的，因为自定义事件需要绑定到实际的 HTML 元素。*

*然后我们定义 3 种方法:*

*   *允许我们向总线添加新的事件监听器。事件名称以及回调由您决定。*
*   *`remove`从总线上删除一个现有的事件监听器。*
*   *最后，`fire`分派一个事件(同样，您可以选择名称)，我们可以使用`detail`参数附加额外的数据。*

*最后 3 行确保我们初始化总线并导出它的引用。*

*在整个应用程序中，您最有可能使用许多不同的文件，因此需要注册或触发事件的每个文件现在都可以导入它:*

```
*import EventBus from "./event-bus.js";EventBus.register("someevent", (evt) => {...});EventBus.fire("someevent");*
```

*因为我们只导出和导入实例(`var bus = new EventBus()`)，所以我们确保所有的事件都由同一个总线处理。*

*或者，如果您不想导出/导入事件总线，您可以将其附加到全局`window`对象(我不建议这样做):*

```
*class EventBus {...}window.EventBus = new EventBus;*
```

*就是这样！我们可以使用这种技术来允许我们的定制元素之间的通信。*

# *结论*

*在本文中，我有幸向您介绍了将数据传递到定制元素的 4 种不同方式。让我们回顾一下:*

## *属性*

*   *可以通过 HTML 和 JavaScript 设置，使用`getAttribute`和`setAttribute`。*
*   *只处理字符串。*
*   *可以观察到。*
*   *非常容易和直接使用。*

## *性能*

*   *只能在 JavaScript 中使用。*
*   *可以处理所有数据类型。*
*   *非常适合在自定义元素中获取复杂数据。*
*   *定制元素应该有 getters 和 setters 来正确处理它们。*

## *事件*

*   *非常适合从定制元素中获取数据。*
*   *使用自定义事件 API。*
*   *当在许多事件中使用许多不同的元素时会变得混乱。*

## *事件总线*

*   *非常适合组件之间的通信。*
*   *不同的通信通道可以有多条总线。*
*   *建造这辆公共汽车需要一些努力。*

*你选择用什么方法工作应该取决于你需要达到什么目的。在一个应用程序中结合一些或所有这些技术并不少见。*

*如果你有任何问题，请在评论中告诉我。编码快乐！*