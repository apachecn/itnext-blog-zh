# 探索 CSS 画图 API

> 原文：<https://itnext.io/https-medium-com-bobrov1989-exploring-the-css-paint-api-e3ab26d59218?source=collection_archive---------3----------------------->

![](img/215c016961946bf88d6baeca6b26a87f.png)

CSS Paint API 是 Houdini 项目的第一部分，可在浏览器的稳定版本中使用。It 谷歌 Chrome 团队在 3 月 6 日将其添加到 Chrome 65 中。这就是为什么这是一个很好的时间去尝试和开始实验。我想让你开始做自己的实验。

# 胡迪尼是什么？

在我们开始探索 CSS Paint API 之前，让我简单介绍一下 Houdini 项目。我不会说太多的细节，但是如果你想了解更多，我会给你提供资源链接。

因此，Houdini 是一组 API，允许您与 CSS 引擎内部进行交互。不幸的是，在此之前，我们仅限于通过 JavaScript 使用 CSSOM (CSS 对象模型)的某个部分。所以我们可以用 JS 尝试 polyfill CSS，但是要在浏览器渲染 stage 之后。并且在改变之后，浏览器需要再次执行屏幕渲染。但是有了 Houdini，我们可以像对待 JS 一样扩展样式。

Houdini APIs 在这里与 CSS 解析器、CSSOM、级联、布局、绘画和复合渲染阶段一起工作。API 有两大类:CSS 属性&值和工作小程序。worklets 涵盖了渲染状态访问:布局、绘制和合成。当属性和值关注于解析器扩展时，使用 CSSOM 和 cascade。你可以在这里查看每个 API [的浏览器实现状态，在这里](https://ishoudinireadyyet.com/)查看更多相关信息[。](https://developers.google.com/web/updates/2016/05/houdini)

# CSS 画图小工具

CSS 画图 API 是一种 worklets，你怎么能理解它的名字与画图渲染过程。它是做什么的？它允许你用 JavaScript 创建自定义的 CSS 函数来绘制图像作为背景。然后对任何需要图像的 CSS 属性使用这个函数。比如你可以用它来做`background-image`、`border-image`或者`list-style-image`。但更令人兴奋的是，它还可以用于自定义 CSS 属性，我们稍后将回到它们。

对于用 JavaScript 画图，您允许使用受限版本的 Canvas API。为什么有限？出于安全原因，您不能从图像中读取像素或呈现文本。但是你可以画圆弧，矩形，路径等等。

# 为什么我们需要 CSS 画图 API？

目前我脑海中有几个用例:

1.  CSS polyfill——当然，我们可以用 JavaScript 为 CSS 编写 poly fill，但就可用性和性能而言，这不是一个好主意。你可以在这里读到一些关于那个[的想法。但是 CSS Paint 是一个很好的选择，例如，看看`conic-gradient`](https://philipwalton.com/articles/the-dark-side-of-polyfilling-css/) [polyfill 示例](https://lab.iamvdo.me/houdini/conic-gradient)。
2.  减少 DOM 节点数量——有时我们需要添加虚拟的 DOM 节点，比如仅仅为了视觉效果的`span`。此外，一些动画可能需要额外的元素。找实现材质设计的画师【涟漪】[动画](https://lab.iamvdo.me/houdini/ripple)。在原始材质设计库中，它为该动画创建了两个额外的`span`元素，而使用 worklet 则不需要这样做。现在假设页面上有 10 个具有“涟漪”效果的按钮，CSS paint 为您节省了 20 个 DOM 节点。
3.  花哨的背景——你可以用不同寻常的图案和背景为最终用户创造某种新的体验。好在它们不会影响性能，可以作为渐进增强的一部分。

# 如何在样式中使用 CSS 画图？

要在样式表中使用自定义画图，您需要使用`paint`函数，并传递您的画图名称，以及任何接下来需要的参数。下面是它可能的样子:

在上面的例子中，我们使用了名为`my-custom-paint`的自定义画图，接下来让我们想象它允许我们传递额外的参数，比如颜色:

看起来和我们的一些 CSS 内置函数也很相似，比如`linear-gradient`:

由于`paint`只是 CSS 声明的一个值，所以对于带有纯色或图像的旧浏览器来说很容易将其退回:

或者我们可以检查 CSS 中的浏览器支持:

在这种情况下，不支持 CSS Paint API 的浏览器将忽略最后一个`background-image`声明，而使用一些静态图像。

但是，如果我们将创建一些广泛使用的画师，这将是伟大的自动化回退插入，因为人类可能会忘记它。我有一个好消息要告诉你，PostCSS 可以用一个插件为我们做这件事。要编写这样一个插件，我们不需要很多行代码。PostCSS 为我们提供了一堆方便的助手工具来遍历 CSS AST(抽象语法树)并操作它。下面是这样一个插件的例子，它用一个作为`fallbackValue`选项传递的静态回退值来替换自定义绘画:

这个插件将遍历所有 CSS 规则，然后遍历其中的所有声明。它查找`my-css-paint`调用，并在替换值之前插入克隆的语句以进行回退。不是那个🚀科学，不是吗？

# 如何创建自定义 CSS 画图？

那么如何创建自定义 CSS 画图呢？只需三个步骤:

1.  声明一个自定义绘制类
2.  注册油漆
3.  加载工作流

所以，首先我们要声明 CSS 画图类。它应该是一个带有`paint`方法的 JavaScript 类。我们稍后将探讨这种方法及其参数，现在只看基本实现:

之后我们需要注册新定义的画师:

我们使用了`registerPaint`函数，将 paint name 作为第一个参数，将我们的类引用作为第二个参数。在这里，我想注意到我们的 paint 模块文件与类和注册调用有一个单独的上下文。这意味着我们不能访问全局浏览器范围内的任何函数或变量，甚至不能加载任何依赖脚本。

接下来，最后一步是加载 worklet，这样之后，您就可以在样式表中使用它了:

首先，我们检查`paintWorklet`在浏览器中是否可用，然后注册我们的自定义画图，调用`CSS.paintWorklet`上唯一可用的方法`addModule`。它接受一个参数——我们的 worklet JavaScript 文件的路径。在这里，您还可以通过一个额外的`else`语句选择 CSS Paint 的基于 JavaScript 的回退。

这里应该如何看待最终结果:

# 实践

画图 API 介绍完后，最好的想法就是尝试一下。让我们从第一个例子开始——创建一个绘制几个圆圈作为背景的画师。首先，让我们为 paint 定义一个类并注册它:

我们用名为`paint`的方法创建了`CirclesPainter`类。这个方法接受两个参数:`ctx`和`geom`对象，前者是我们的画布上下文，后者由两个属性组成。`geom`包含我们画布表面的`width`和`height`。然后使用我们的上下文，我们在循环中画四个圆，并用蓝色填充它们。最后，我们在页面上加载我们的工作流:

为了使用它，我们用类名`circles`创建了简单的`div`，并在样式表中添加了后续规则:

所以我们把它做成方形，并添加黑色作为老浏览器的备用颜色。就是这样！你可以在 GitHub 上查看[结果](https://vitaliy-bobrov.github.io/css-paint-demos/hello-world/)和[代码](https://github.com/vitaliy-bobrov/css-paint-demos/tree/master/src/hello-world)。这是演示:

我现在要提到的一件事是，我们没有添加任何 resize 事件监听器，但是浏览器会在任何布局改变时自动调用`paint`方法。当前的 Chrome 实现使用主 UI 线程进行绘画渲染，但在未来，它将使用一个单独的线程。可以想象一些沉重的动画或者背景，对主线程的影响为零。这将是一个巨大的性能提升！

您的背景可能是响应性的，这种响应性取决于元素本身的大小，没有任何关于`resize`事件的监听器。直到`element queries`还在提议你可以根据元素大小生成不同的图片。用[源代码](https://github.com/vitaliy-bobrov/css-paint-demos/tree/master/src/responsive)试用[这个例子](https://vitaliy-bobrov.github.io/css-paint-demos/responsive/)。当元素改变尺寸时，我们用另一种颜色填充我们的圆。

这一切都很好，但接下来我想让我们的绘画可配置。所以让我介绍几个 CSS 变量:

*   `--circles-offset`–控制圆圈之间的距离
*   `--circles-count`–表示要渲染的圆的数量
*   `--circles-opacity`–改变圆圈的不透明度

有了这些变量，我们就能创造出一些可以随时间变化的模式。为了访问 painter 类中的 CSS 变量，我们需要定义名为`inputProperties`的静态属性。它应该是我们希望在`paint`方法中定位的 CSS 属性和变量的`Array`。对数组中的每个属性进行更改，浏览器将为我们调用 render，而无需任何额外的代码行。下面是更新后的`CirclesPainter`:

所以我们为`inputProperties`添加了 getter，它返回我们想要使用的 CSS 变量列表。然后，我们可以访问我们用`props`参数订阅的所有属性。它包含带有值的属性的 CSS 映射。我们用变量名调用`get`方法来获取它的值作为`string`。此外，我们定义了一些缺省值，即使它们没有在样式中指定。我们用这个值来动态渲染圆形。

我们应该用变量来更新样式:

在此检查[代码](https://github.com/vitaliy-bobrov/css-paint-demos/tree/master/src/circles-with-params)和[结果](https://vitaliy-bobrov.github.io/css-paint-demos/circles-with-params/)。

对于 GitHub 演示，我添加了一个简单的脚本来连接 CSS 变量和输入控件，你可以在这里找到它。

所以现在我们可以在运行时修改渲染参数，只需用 CSS 或 JavaScript 更新变量。但是如果我们看看像`linear-gradient`这样的内置 CSS 函数，它们接受传递额外的参数给函数本身:

我们是否也可以为自定义绘画实现相同的行为？答案是——是的！为此，我们需要使用另一个 Houdini API，它仍然是 Chrome 中的一个实验。它被称为 CSS 类型化 OM，它允许你使用内置的 CSS 引擎类型，如颜色，图像，长度等。然后我们可以将它们作为参数传递给`paint`函数。

由于功能现在仍处于实验阶段，我们需要在 Chrome 中启用“实验网络平台功能”标志。转到`chrome://flags/#enable-experimental-web-platform-features`并启用它。

之后，我们应该向我们的 paint 类添加一个新的静态属性— `inputArguments`。像`inputProperties`一样，它订阅对所列参数的任何更改，但应该包含一组 CSS 类型。让我们用参数替换 CSS 变量:

所以我们的`CirclesPainter`现在包括了包含三个参数的`inputArguments`列表:`<number>`、`<number>`和`<percentage>`。您可能会提到 CSS 类型的语法-`<type name>`。此外，您可以在 TypeScript–`<number | percentage>`中以尽可能相似的方式接受类型联合。你可以在“CSS 值和单位”规范草案[中找到所有可用的类型。](https://drafts.csswg.org/css-values-4)

然后在`paint`方法中，我们得到了类似于 JavaScript 函数`arguments`对象的`args`参数，但是只有`Array`包含了`CSSUnitValue`对象。每个单元对象由两个属性`value`和`unit`组成。所以在我们的例子中，我们访问了每个参数的所有值。我们应该修改我们的 CSS 来使用它:

在此查看[代码](https://github.com/vitaliy-bobrov/css-paint-demos/tree/master/src/circles-with-args)和[结果](https://vitaliy-bobrov.github.io/css-paint-demos/circles-with-args/)。

# 动画片

这很酷，但是我们如何为一个画家创造动画呢？正如我之前所说的，我们的 worklets 在一个单独的上下文中执行，没有`requestAnimationFrame`甚至没有`setTimeout`函数。如何实现动画？第一种解决方案是使用 CSS 变量。让我们试着用 CSS 来制作动画:

而且…这个解决方案不会像我们预期的那样工作。它只是在 50%点将不透明度从 1 切换到 0。但是为什么呢？答案很简单，所有 CSS 变量都只是字符串，它们类似于 SASS 或更低版本中的变量——变量被简单的字符串插值替换为值。为了制作动画，一些 CSS 属性浏览器需要应用插值函数，但它不知道如何将一个字符串插值到另一个字符串。它有唯一的内置功能来动画颜色，长度，数字，但不是字符串。这就是为什么它只在 50%切换值。在这种情况下，我们可以使用`requestAnimationFrame`用 JavaScript 动画化变量。这样的脚本可能如下所示:

不太好，我们能做得更好吗？是的，使用自定义属性 API。这个 API 现在也在 Chrome 的旗帜下，所以不要忘记启用它。它允许我们用类似于变量的语法注册自定义 CSS 属性，但这次是用分配给它的 CSS 类型。因此，浏览器将有一个关于如何动画的想法，我们可以使用 CSS 动画和过渡！

所以要注册自定义属性，我们需要调用`CSS.registerProperty`并传递 options 对象:

正如你所看到的，我们需要用`name`选项给属性命名。然后我们用`syntax`属性指定它的类型，另外我们说子节点和初始值不会 100%继承它。

之后，我们可以在样式表中使用新创建的自定义属性:

现在我们可以使用`transition`平滑地改变圆形的不透明度。在这里查看[代码](https://github.com/vitaliy-bobrov/css-paint-demos/tree/master/src/circles-animation-with-custom-property)和[结果](https://vitaliy-bobrov.github.io/css-paint-demos/circles-animation-with-custom-property/)。

# 结论

今天开始学习 CSS 画图 API，探索如何创建自己的画图，如何使用输入属性和参数，CSS 变量和自定义属性，以及如何制作动画。在下一篇文章中，我将使用从那篇文章中获得的知识实现更多的生产就绪示例。如果你正在使用最新的 Chrome 阅读这篇文章，你可能会提到我正在使用定制油漆来制作材料设计背景，你可以在这里查看[和查看](https://vitaliy-bobrov.github.io/css-paint-demos/md-bg/)[代码](https://github.com/vitaliy-bobrov/css-paint-demos/tree/master/src/md-bg)。自己尝试 CSS 画图 API 吧！

# 资源

*   [开发者谷歌](https://developers.google.com/web/updates/2018/01/paintapi)
*   [胡迪尼选秀](https://drafts.css-houdini.org/css-paint-api/​)
*   [W3C CSS 画稿](https://www.w3.org/TR/css-paint-api-1/​)
*   [演示](https://lab.iamvdo.me/houdini/)
*   [我的演示](https://vitaliy-bobrov.github.io/css-paint-demos/​)

*原载于*[*vitaliy-bobrov . github . io*](https://vitaliy-bobrov.github.io/blog/exploring-the-css-paint-api/)*。*