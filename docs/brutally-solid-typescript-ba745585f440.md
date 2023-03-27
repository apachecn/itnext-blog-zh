# 极其坚固的打字稿

> 原文：<https://itnext.io/brutally-solid-typescript-ba745585f440?source=collection_archive---------1----------------------->

我将在一个类型脚本环境中教授和演示 stripped down SOLID。
坚实的[【2】](https://en.wikipedia.org/wiki/SOLID)有 5 条设计原则，当正确理解时，它们会改变你思考代码和编写代码的方式。

![](img/3a4a8eaf7dcafd13229a215aab6e9ad7.png)

野兽派建筑，盖泽尔图书馆由威廉佩雷拉。1970 年，加州圣地亚哥。 [*瑞安*](https://www.shutterstock.com/image-photo/geisel-library-university-california-san-diego-1200639814) *威梭托*

# 单一责任

类、函数或模块只有一个改变的理由

*   函数、类、模块、文件和文件夹将由一个单独的域或该域的一部分组成，这些域被分割成单元，因此是“改变的唯一原因”。
*   将不相关的代码组合在一起会导致代码中出现错误等意想不到的副作用。
*   小单位有整体工作流程的好处，可以单独测试。
*   公开 API 的小单元在它们的领域中的角色是什么，导致更容易的组合。
*   项目结构应该反映这些单独的领域
*   我个人认为这是最复杂的原则。

这个代码示例违反了**所有坚实的**原则

我们将从解决单一责任开始。这个用户类负责很多事情。发送包含 url 的电子邮件，并构造电子邮件消息。

现在这个用户只对自己负责。动机邮件很容易测试，因为它返回一个字符串，不依赖于任何东西。

电子邮件客户端仅负责与电子邮件相关的任务，并且可重复使用。我们添加了另一个函数只是为了证明后面的另一个观点

这更接近真正的固体。这个功能负责的太多了。但是要记住一个好的概念，一些函数负责组成更大的任务组。就像文件夹的职责是保存其他文件一样，这些文件由覆盖一个领域(如用户、产品、队列)的公开 API 和代码组成。

# (多态)打开-关闭[ [9](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle#Polymorphic_open/closed_principle)

“对扩展开放，对修改关闭”

*   使用和导出接口的方式要让你依赖它们而不是具体化。
*   这暴露了一个不允许修改并且可以扩展的 api。
*   使用接口将 api 的交互定义为抽象结构会导致低耦合。

很简单，现在如果我们要实现另一个电子邮件发送器，我们可以。看起来不是什么太重要的事情，但是过一会儿就有了。

# 利斯科夫替代

"对于 a 的 b 亚型，a 可以被 b 型取代. "

*   实现接口可以很容易地从一个类/对象替换到另一个类/对象。
*   可以通过继承实现，但更喜欢接口/组合。7
*   带来出色的互操作性。
*   也有助于创建模拟类或对象。

假设需求发生了变化，我们需要使用另一个电子邮件客户端，我们创建了另一个电子邮件客户端，与之前的客户端实现不同。

现在，在这种情况下，代码可以更容易地用任何一种实现来切换。

# 界面分离

“制作小型客户端专用界面”

*   使用这些经过磨砺的接口的函数和类导致了它可以访问的方法的范围很窄。
*   如果将来底层的具体类型或实现发生变化或被重构，使用其接口的客户端不会受到影响。
*   当测试依赖于接口的函数或类时，不需要模仿整个对象。
*   轻量级和易于构建的模型。

`sendMotivationalEmail`函数的一个部分问题是，如果另一个开发人员想要使用该函数，他们可以完全访问 email 类的所有方法。这是向开发人员传达意图的重要一点。

我们现在已经缩小了函数可以访问的范围。我们的意图通过代码来传达。

# 依赖性倒置

“高级模块和低级模块之间的交互应该被认为是它们之间的抽象交互”

*   将依赖项以构造函数或函数参数的形式从较低级别的模块传递到较高级别的模块。
*   让更高级别的模块接受接口类型中的这种依赖性。
*   使测试变得容易，依赖性的界限变得清晰。
*   导致低耦合，允许不同的实现或业务需求变化，但模块之间的契约保持不变。

这是固体聚集的地方。`main`函数负责引导。`sendMotivationalEmail`负责撰写电子邮件，它依赖于发送电子邮件的抽象。如果我们想测试整个功能，这就特别有用。

下面是利用 solid 提供的所有优势的示例测试。

现在有了一个注入依赖关系的明确方法，我们不需要实现整个 email 类。代码不再到达真正的电子邮件服务器，并且不需要使用库，只需要简单的编程。依赖关系是从范围之外输入的，因此不是在类中构造的。这提供了移动代码、添加新功能的强大能力，并且不需要引导整个应用程序来测试其中的一小部分

有些例子是人为的，有些功能不完全是一个人的责任，但我希望你能明白这些原则能实现什么。

# 结论

在表面上，固体的大部分原理都围绕着界面。这是因为接口本质上是一个抽象的构造，不依赖于任何具体化。代码交互现在是松散的契约，任何东西都可以遵守。这产生了非常松散的耦合。由于单个或几个方法接口使得你的代码库只依赖于它所需要的，重构和模仿变得简单了。与让代码单元负责一件事情的单一职责相结合，导致了每一层的抽象。
这种抽象以及用接口定义 api 边界允许代码的插入和取出以及重构，而对代码库的其余部分影响最小。

下面是一些精选的资源，它们帮助我对 SOLID 和其他一些通用编程原则有了更完整的理解。

最后但并非最不重要的一点是…当你拼写全名[ [10](https://www.youtube.com/watch?v=ewc1hixzYPY) 时，不要忘记所有的大写字母

[1](https://stackify.com/solid-design-principles/):【https://stackify.com/solid-design-principles/】T2 长而实的 java 好例子
2:[https://en.wikipedia.org/wiki/SOLID](https://en.wikipedia.org/wiki/SOLID)维基上实
[3](https://blog.cleancoder.com/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html):[https://blog . clean coder . com/uncle-bob/2014/05/12/theopenclosedprinciple . html](https://blog.cleancoder.com/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html)打开关闭(Bob 叔叔)
[4](https://www.youtube.com/watch?v=zzAdEt3xZ1M):[https://www.youtube.com/watch?v=zzAdEt3xZ1M](https://www.youtube.com/watch?v=zzAdEt3xZ1M) [](https://www.youtube.com/watch?v=zzAdEt3xZ1M) [7](https://en.wikipedia.org/wiki/Composition_over_inheritance):[https://en.wikipedia.org/wiki/Composition_over_inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)作文超过继承
[8](https://reactjs.org/docs/composition-vs-inheritance.html):[https://reactjs.org/docs/composition-vs-inheritance.html](https://reactjs.org/docs/composition-vs-inheritance.html)React 作文 vs 继承
[9](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle#Polymorphic_open/closed_principle):[https://en . Wikipedia . org/wiki/Open % E2 % 80% 93 closed _ principle #多态 _open/closed_principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle#Polymorphic_open/closed_principle) 多态打开关闭
[10](https://www.youtube.com/watch?v=ewc1hixzYPY):[https://www.youtube.com/watch?v=ewc1hixzYPY](https://www.youtube.com/watch?v=ewc1hixzYPY)