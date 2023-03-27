# 一种学习反应的方法

> 原文：<https://itnext.io/a-way-to-learn-react-b95056eafebb?source=collection_archive---------2----------------------->

![](img/5d79e47342156a81db9b84f082006d49.png)

丹尼尔·里昂在 [Unsplash](https://unsplash.com/search/photos/mountain?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

# TLDR；

这篇文章的目的不是“教你在 5 分钟内做出反应”，而是给出一根鱼竿，而不是一条鱼。所以，准备好你的耐心和注意力，你会需要的。

# 开始

那么，如何开始呢？当我试图学习一个新的框架时，这是我最痛苦的问题之一。好吧，你可以阅读所有的文档，尝试所有的例子，润色你的“待办事项”应用程序，但是到最后，你可能对事情是如何工作的理解还不够。

我是否学到了足够的知识来开始使用真正的应用程序？

有什么是我遗漏了或者仅仅是弄错了的吗？

还有很多类似的问题，要回答它们，你需要花更多的时间或者找一个导师。

本文描述了一种学习 react 的方法，这种方法将帮助你在相对较短的时间内获得更多的经验，这将足以从真正的应用程序开始。而且，这将使你在学习像*反应路由器*或*冗余*这样的东西时更加顺畅。

# 心理框架

学习的最好方式是建立你的[心智框架](https://www.wikiwand.com/en/Schema_(psychology))，它将引导你通过复杂的案例，这些案例在文件中没有描述。我用来开发这个框架的一个方法是，像看书一样仔细阅读文档，并尝试大部分代码示例。这有助于创建一个结构图，您可以在进行过程中添加更多细节。如果有些东西看起来太复杂，不要担心，只是把它放在一边，试着读另一章，第二天再回到那个困难的事情上。

不要试图在第一次尝试时就破解一个困难的话题，给它一些时间，比如 20-30 分钟左右，如果它仍然很难获得，那么就暂时离开它。沏茶或散步，你的大脑会在后台处理事情。如果你认为文档不够清晰，可以使用 Google 或 Stack Overflow，或者查看其他资源，比如这本超棒的 [react 手册。](https://medium.freecodecamp.org/the-react-handbook-b71c27b0a795)

在花一些时间研究文档之后，你会发现你对事物是如何工作的有了一些基本的理解。从应用程序开始，这可能就足够了，但是在练习之前，测试一下你自己:试着给接下来的每个术语下一个清晰的定义:

1.  JSX
2.  虚拟 Dom
3.  做出反应。成分
4.  做出反应。纯组件
5.  功能成分
6.  类的公共/静态成员
7.  render(方法)
8.  ReactDOM.render
9.  状态
10.  构造器
11.  设置状态
12.  小道具
13.  属性类型
14.  默认道具
15.  组件生命周期
16.  事件
17.  成分组成
18.  高阶组件
19.  渲染道具
20.  语境
21.  入口
22.  裁判员

上面的一些内容来自 React 文档的“高级概念”一章。但是我在开发过程中经常遇到它们，它们正在成为现实应用的基本要素。

如果上面的一些术语仍然很难得到，再次，把它们放在一边，第二天再回来，使用谷歌，Stack Overflow 或任何你觉得有用的东西。但是，不要放弃，一次又一次地尝试，直到你至少在一些基本的水平上做到这一点。

# 实践

到这一步，你应该有所了解了。很好。甚至可能考虑实现自己的待办事项应用程序。不要那样做。我建议从一些基本的东西开始，你会在 react 应用开发中看到很多。

有一个[模板项目](https://github.com/dmitriykharchenko/react-nano-template)从 React 组件开始。它有简单的`babel`和`webpack`设置，对于尝试小的反应非常方便。

我浏览了一下我的项目，整理了一个简短的清单:

1.  一个简单的功能组件，呈现一周中的当天。
2.  使用功能组件组合来呈现`hello world`的类组件
3.  触发挂载上的`alert()`的类组件
4.  每秒接收新道具并使用`shouldComponentUpdate`避免道具不变时重新渲染的组件
5.  处理它所呈现的元素事件的组件(例如 onClick)
6.  呈现 500 个元素的列表的组件(列表数据可以在组件外部生成)
7.  提供鼠标坐标的高阶组件(HOC)。
8.  每秒重新渲染一次的组件，并从 5-10 个组件中随机选择一个组件进行渲染。
9.  使用`componentWillUnmount`清理混乱(在组件安装上创建混乱)
10.  一个定义默认背景颜色的上下文和一个在点击按钮时随机更新颜色的组件
11.  一个向文档头添加元标签的组件(直接改变 DOM)，在 props 更新时更新它们，在卸载时删除它们
12.  提供当前日期/时间的渲染道具组件
13.  使用门户呈现自定义元标签的组件
14.  一个不受控制的组件，在表单提交时用来自`<input/>`的内容触发警报

尝试解决尽可能多的问题，而不要搜索解决方案。即使答案不理想，你不喜欢，也没关系，你总是可以回去重写。

# 更多的练习

你的下一步将是解决技术挑战。试着谷歌一下，尽可能多地找到它们。向你的朋友询问他们在面试中遇到的挑战，寻找像[https://codesignal.com/](https://codesignal.com/)这样的网站

以下是从第一个谷歌页面中发现的一些挑战:【https://github.com/kkvesper/react-test-task[https://github.com/uber/coding-challenge-tools](https://github.com/uber/coding-challenge-tools)

[https://daveceddia.com/react-practice-projects/](https://daveceddia.com/react-practice-projects/)[https://developerjobsboard . co . uk/2018/07/28/an-example-senior-react-redux-developer-task/](https://developerjobsboard.co.uk/2018/07/28/an-example-senior-react-redux-developer-task/)[https://www.learnstorybook.com/react/en/simple-component/](https://www.learnstorybook.com/react/en/simple-component/)

问题:[https://github.com/sudheerj/reactjs-interview-questions](https://github.com/sudheerj/reactjs-interview-questions)

如果你第一次尝试就没能解决问题，那也没关系。暂时放下那个任务，然后在尝试另一个或第二天再回来。完成清单上的所有问题后，回头看看你已经解决的问题。我保证你会发现它们中的大多数都有很大的改进空间，抓住这个机会，重写/修复/重构它们。

## 下一步是什么？

这取决于你。您可以开始学习更高级的东西，如路由、状态管理和 API 服务器交互。此外，你可以为开发能够经受住时间和你的人类队友的考验的应用程序设置心态，这里有一篇介绍文章。最后，还有一篇关于[我们如何构建应用的文章。](/how-to-structure-your-react-app-2-2cf3b8040634)

如果你有任何问题，或者你觉得这篇文章遗漏了一些重要的细节，我希望听到你的反馈！干杯！