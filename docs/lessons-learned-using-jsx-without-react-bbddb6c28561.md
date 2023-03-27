# 在没有反应的情况下使用 JSX 的经验

> 原文：<https://itnext.io/lessons-learned-using-jsx-without-react-bbddb6c28561?source=collection_archive---------6----------------------->

> [点击这里在 LinkedIn 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Flessons-learned-using-jsx-without-react-bbddb6c28561%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

启动项目是容易的部分，因为每个节点项目`yarn init -y`都将创建准系统，对于 JS 爱好者来说几乎是每周一次的任务，所以我们不要深究这个命令；现在，依赖性非常小，就个人而言，我喜欢使用 webpack，但是，为了简单起见，我们只使用 babel:

`yarn add -D @babel/cli @babel/core @babel/plugin-syntax-jsx @babel-plugin-transform-react`

下一步也很简单，在根目录下定义一个`.babelrc`文件并使用以下内容:

```
{
  “plugins”: [
    “@babel/plugin-syntax-jsx”,
    [“@babel/plugin-transform-react-jsx”, { “pragma”: “dom” }]
  ]
}
```

在这一点上有必要做一点解释；有了这些步骤，我们可以通过两件事将`<h1>Hi</h1>`转变成`dom(“h1”, null, “Hi”)`，

*   上定义编译指示。babelrc 将命名 fn，否则默认情况下将执行`React.createElement`
*   运行一个`@babel/cli`命令，但是让我们使用来自`package.json`的 npm 脚本来保持它的有用性:

```
"scripts": {
  "build": "babel example.js --out-dir lib"
}
```

如果你正在阅读这篇文章，你可能已经知道 JSX 是什么了。这样一来，当我被要求再次实现一个以前用 React 编写但用普通 JS 编写的组件时，我就有些纠结了。原因是该组件太简单了，以至于让组织中的每个团队都使用它是一种过度的尝试，因为他们必须包含许多与 React 几乎没有任何关系的项目的依赖关系，这并不是说这不可能，而是一个已经包含 angular 1.x 或 backbone 的项目只是举几个例子，如果没有一个很好的理由，几乎不想添加 React 16。

## 开始旅程(研究)

那么 JSX 是如何运作的呢？长话短说将 js 中的 HTML 语法转换成一个带有多个参数的函数，这些参数替换标签、属性和内容，比如:`<h1 className=”headline”>Hi</h1>`转换成`dom(“h1”, { className: “headline” }, “Hi”)`，所以我当时的想法是，如果我能重新实现 dom()应该做的事情，就可以了。

## 基本步骤

dom()必须读取 3 个参数，第一个参数会给我它应该被创建的元素的名称，然后
`const element = document.createElement(arg1)`，看起来是可行的，除非 arg1 不是一个字符串，在这种情况下将是一个来自组件的函数，其中`<CustomComponent />`将是`dom(CustomComponent, null, null)`，我将执行该函数而不是创建一个新元素。
另一方面，第二个参数是一个对象，因此至少启动一个基本的合并应该就可以了。

```
function dom(tag, attrs, ...children) {
  // Custom Components will be functions
  if (typeof tag === 'function') { return tag() } // regular html tags will be strings to create the elements
  if (typeof tag === 'string') {

    // fragments to append multiple children to the initial node
    const fragments = document.createDocumentFragment()
    const element = document.createElement(tag) children.forEach(child => {
       if (child instanceof HTMLElement) { 
         fragments.appendChild(child)
       } else if (typeof child === 'string'){
         const textnode = document.createTextNode(child)
         fragments.appendChild(textnode)
       } else {
         // later other things could not be HTMLElement not strings
         console.log('not appendable', child);
       }
    }) element.appendChild(fragments) // Merge element with attributes
    Object.assign(element, attrs) return element
  }
}
```

对于第二个论证来说仅仅是`Object.assign(element, arg2)`并且在很大程度上，它做到了；最后，对于第三个参数，如果是一个字符串，应该添加一个节点文本，这是定义部分，因为如果第三个参数是一个函数，就让循环重新开始，所以我们在这里:

*   我们可以创建新的 HTML 元素
*   我们也可以使用定制组件
*   我们可以添加类和其他简单的字符串参数
*   我们可以添加文本或其他元素作为子元素
*   我们可以呈现列表(兄弟姐妹)

对于基础知识来说还不错，这里有一个在那个阶段兼容的例子，以及[提交](https://github.com/alecsgone/jsx-files/commit/e20f3ae964b3af5e20d6c7e1a8b4d94934436868):

```
function Headline() {
  return (
    <h1 className="headline">Inital Line
      <br />
      new line
    </h1>
  )
}function Main() {
  return (
    <div>
      <Headline />
      <p>Lorem ipsum</p>
      <ul>
        <li><a href="">anchor</a></li>
        <li>2</li>
        <li><a href="">anchor2</a> More</li>
      </ul>
    </div>
  )
}const app = document.querySelector('.app')
app.appendChild(Main())
```

让我们列举一下我们遗漏了什么，至少对于 alpha 版本来说是“必须实现”的:

*   `Array.map( item => <tag>{item}</tag>)`
*   Refs 实现事件监听器或任何需要访问`HTMLNodeElement`的东西
*   片段(当前的 react 组件有它们，我不想重写到 div 中)

## 体面的实现(alpha)

从 map 开始，因为它是最常用的，检查 arg3 是否是一个数组，我们应该能够将这些函数的结果附加到文档片段上，然后附加到根上，就像，例如，在多个 LI 的情况下，根将是 UL/OL，这很清楚，现在，兄弟是 dom 函数的额外参数，而不仅仅是作为最后一个参数的数组， 我很高兴我们通过第一手资料知道了前两个参数是什么，也感谢巴别塔，我们可以`fn(arg1, arg2, …arg3){}`和 arg3 将永远是一个数组，即使只有一个项目，所以我们已经有了一个函数，让我们只是重复使用它。 [提交](https://github.com/alecsgone/jsx-files/commit/1ec5ad125cbcd4fc4c3ed30d7d9302a45048235f)

## 还有两个功能

引用和片段是唯一缺少的东西，所以引用就像询问第二个参数中的一个属性是否名为 ref 一样简单，并向同一个节点传递一个回调。在这一点上，a 对 React 如何工作有了一个粗略的想法，这是一个很好的练习，只是缺少片段，许多版本的 React 都缺少的功能，最初我创建了一个函数来导出并能够使用它，当时我不知道函数应该是什么样的，我只是返回了单词 Fragment 用于日志记录，有趣的是， 在我尝试记录那个单词后，我意识到如果函数返回[片段](https://github.com/alecsgone/jsx-files/commit/5c011ba221760f468baa1b609eef2a8d9773dfb5)，我应该只返回孩子😋(仍然对根 app 不起作用但是谁把片段导出来作为主要功能？ 我希望没有很多人😉有了这个基本的、不到 50 行的例子，你就能完成所有的事情。

```
import dom, { Fragment } from 'jsx-render'function Headline() {
  return (
    <Fragment>
      <h1 className="headline">Hello this in an h1
        <br />
        new line
      </h1>
      <h2>Second Headline</h2>
    </Fragment>
  )
}function Main() {
  return (
    <div>
      <Headline />
      <p>Lorem ipsum</p>
      <ul>
        <li><a href="">anchor</a></li>
        <li>More</li>
      </ul>
      <ol> {items.map(item => <li>{item}</li>)} </ol>
      <button ref={node => { 
        node.addEventListener('click', console.log) 
      }}>
        Click Me!
      </button>
    </div>
  )
}const app = document.querySelector('.app')
app.appendChild(Main())
```

## 结论/提示/问题/问答

*   JSX 很好地给了你实现自己的 pragma fn 的可能性，而且并不复杂。
*   从 React 组件的移植只是一个复制粘贴，因为组件是无状态的，只有一个点击函数来打开一个模态，但是打开/关闭类的切换不会伤害任何人，到目前为止一切都很好。
*   "如果你实现了这样的事件，僵尸怎么办？"—就公司目前的结构而言，这些组件将用于静态和非常简单的页面，没有关于 SPA 的故事，正因为如此，我并不太担心。
*   当交互非常简单，只需点击切换时，您不需要反应/还原，有时 1 小时的重新实现可以为您每次加载节省 30kb。
*   如何实现片段让我大开眼界。

## 其他方法

经过额外的研究，我还发现了一个库，它可以转换到 JSX，但不是为每个组件创建一个 fn()来在运行时执行，而是将每个组件直接转换为 document.createElement，有几个小的缺失功能不用担心，除非您需要 SVG 支持，我们将在后面讨论/实现。

## 笔记

示例 repo 使用了 lerna，以防您想尝试一下。[jsx-文件/蓝图](https://github.com/alecsgone/jsx-files/tree/master/packages/blueprint)