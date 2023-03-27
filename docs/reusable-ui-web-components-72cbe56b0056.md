# 可重用的用户界面— Web 组件

> 原文：<https://itnext.io/reusable-ui-web-components-72cbe56b0056?source=collection_archive---------4----------------------->

代码可重用性是代码速度最重要的事情之一。当构建一个应用程序时，你想把它分成具有不同职责的服务或模块，并在应用程序的几个地方使用它们。例如，当访问数据库或向远程服务器发送请求时，您不希望在代码中的任何地方实现请求过程和格式化。您想要做的是抽象出如何访问数据库以及如何从服务器请求数据的实现细节，并公开一个高级 API 供消费者使用。我们知道如何用后端服务和业务逻辑代码很好地做到这一点，但是，在前端世界中，等式中缺少了一大块，即表示层——UI。

![](img/bd35d7ceb062916687421ade09510f3b.png)

哈尔·盖特伍德在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

UI 与其他层有些不同。首先，意识到 UI 需要被分成单元需要时间。回到普通或 JQuery 时代，html 在页面上是平面的，我们使用 javascript 应用行为，这在当时是最好的事情。它具有某种可重用性，但是该单元没有公开任何高级 API。该装置的任何重大改变都需要得到消费者的支持。

如今，我们使用框架或库，如 Angular、React、Vue、Elm 或任何现有的东西。我们使用高级 API 创建组件，这些组件可以从项目中提取或在模块中创建，因此任何 web 应用程序都可以重用。实际上不是任何网络应用。只有那些使用相同框架作为组件的人。如果您想将组件暴露给所有框架，您需要将所有内容提取到本机核心层，然后再用特定的框架实现进行包装。核心中的每一个变化都需要反映在所有的实现中。

![](img/f4f20e89f9e95f61cceed7200e86bb63.png)

照片由[内森·杜姆劳](https://unsplash.com/@nate_dumlao?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

拯救世界的 Web 组件。“web 组件是一套不同的技术，允许您创建可重复使用的自定义元素，将它们的功能封装在代码的其余部分之外，并在您的 Web 应用程序中使用它们”——MDN。有了 web 组件，我们可以用高级 API 创建可重用的 UI，而不用考虑使用它的框架。框架与 web 组件一起工作，就像它们与常规的本地 html 元素(如输入和按钮)一起工作一样。

Web 组件标准由 4 部分组成:HTML 模板，允许重用较小的 HTML 部分；HTML 导入，允许将 HTML 部分分成不同的单元；影子 DOM，允许隔离组件样式和行为；以及自定义元素，将所有内容打包在一起，并公开高级 API。

所有未知元素在 DOM 树中显示为 HTMLUnknownElement 类型，没有脚本、样式和结构。要创建一个 web 组件，我们需要注册一个带有标记名的自定义元素，并实现一个扩展 HTMLElement 的类，该类将添加脚本、样式和结构。

为了注册一个新的定制元素，我们使用 customElements API:

```
customElements.define(‘word-count’, WordCount);
```

当使用 word-count 时，WordCount 的一个新实例将被实例化并附加到元素上。WordCount 将计算给定的文本属性中的单词数，并显示自定义消息。要显示自定义消息，我们需要创建一个影子 DOM 来保存消息，然后构建消息本身并将其添加到影子 DOM 中。

要在自定义元素类 contractor 中创建影子 DOM，请执行以下操作:

```
const shadow = this.attachShadow({ mode: ‘open’ });```
```

为了创建初始的 HTML 结构，我们创建了一个单独的文件，该文件将使用 HTML import 加载。在该文件中，我们声明了一个要在组件中实例化的 HTML 模板。DOM 查询无法访问 HTML 模板，并且模板中的脚本将不会运行，直到我们加载模板以保留我们努力实现的封装。在模板中创建了一个 style 标签，它只影响 shadow DOM 中的元素。Slot 元素充当 light DOM 的占位符。

在实现中，我们有样式定义、带有两个槽的标题部分(一个命名，另一个不命名)和定制元素。slot 标签中的值是默认值—如果使用者不指定任何值，将会显示这些值。

为了使用 HTML 导入加载 HTML 文件，我们将添加一个新的链接标签，如下所示:

```
<link rel=”import” href=”word-count/word-count.html” id=”word-count-link”>
```

rel="import "告诉浏览器我们将使用 HTML import 加载文件。浏览器将以非阻塞的方式下载文件，并确保它不会加载超过一次。要加载文件并使用其内容，我们可以在文件底部添加一段代码，将它添加到文档中，或者我们可以手动加载它并在我们的代码中使用它。我们将手动加载它，因为我们希望保留封装，并且不希望将数据泄漏到全局范围内。

要加载文件并实例化模板:

```
const shadow = this.attachShadow({ mode: ‘open’ });const wordCountLink = document.getElementById(‘word-count-link’) as HTMLLinkElement;const wordCountFile = wordCountLink.import as any;const templateWordCount = wordCountFile.getElementById(‘template-word-count’);shadow.appendChild(document.importNode(templateWordCount.content, true));
```

在这里，我们创建一个影子 DOM，获取 link 元素，导入文件，查询模板的导入内容，并使用 document.importNode 实例化模板，这将创建模板的深层副本。

为了计算给定文本中的字数，我们需要读取文本属性并相应地更改 DOM。我们还需要使 DOM 与文本属性的每次变化保持一致。我们可以使用 attributeChangedCallback 回调，它将在每次属性改变时触发。attributeChangedCallback 是我们可以用于 web 组件四个生命周期事件之一。其他的有 connectedCallback、disconnectedCallback 和 adoptedCallback。attributeChangedCallback 将仅针对我们将在 observedAttributes 静态属性中声明的属性触发。要访问影子 DOM，我们可以使用 this.shadowRoot。

下面是 WordCounter 的完整实现:

为了把所有的东西组合在一起，我们可以使用任何我们想要的框架。所有的框架和库都能够使用自定义元素和它的 API，就像它们处理本地元素一样，比如输入和按钮。例如，这里的 text 属性是我们的高级 API，我们将更改它以影响组件。

使用 React 类和 MobX:

变化的基本流程非常简单。当输入字段发生变化时，赋予自定义组件的文本属性也会发生变化。在我们的自定义组件中，因为 observedAttributes 返回“text”，所以 attributeChangedCallback 触发并调用 updateWordCount 来获取。词和。对容器进行字数统计，并用正确的值更新它们的 innerText。

Web 组件是一个好主意，但并不是一切都是光明的。前端世界最大的问题之一也发生在这里——浏览器支持。并非所有主流浏览器都支持 web 组件。我们仍然需要加载 polyfills 来使用它们。此外，polyfills 不能填充所有内容。例如，影子 DOM 封装无法修复，所以我们仍然需要小心自定义样式、脚本和 id。未来还在后头。

编辑:要了解今天如何使用聚合物构建 web 组件，请阅读我的下一篇文章。

![](img/7df73ca65c8f419e36dbfb5b02a38a4b.png)

由 [Ryan Wong](https://unsplash.com/@provenwong?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片