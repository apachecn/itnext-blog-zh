# HTML6 真正需要的(生活水平)

> 原文：<https://itnext.io/what-html6-really-needs-living-standard-be0d962a7997?source=collection_archive---------4----------------------->

自 2014 年 HTML5 发布以来，即使有了所有新的 API、功能和改进，许多开发人员已经在考虑下一版本的 HTML。

![](img/28d297856573337163de7fd69e704e48.png)

## 只是一点背景

有些人仍然不知道 HTML 标准是如何工作的，有一些组织提出了他们自己的模式和协议，浏览器应该接受并使用它们，但唯一对此有真正发言权的组织是 W3C，原因很清楚。所以 W3C 分析了 web 需要什么，他们编写并规划了语言指南，这将成为浏览器的标准。

在 Internet explorer 的早期，W3C 已经存在，但实际上微软并不太关心标准，他们做他们想做的。直到 Google Chrome 出现并改变了游戏规则，因为是的，Google Chrome 有时也不关心标准，但至少它提供了更好的东西，W3C 会在他们的 HTML 文档中标准化这些东西。谷歌很强大。

# HTML6 真正需要的特性

好吧，我知道实际上根本不会有 HTML6 版本，但如果你想更惊讶，今天我们不使用 HTML5，因为该标准已经更新，这被称为“Livin 标准”。现在我们正在使用 HTML5.2，但是 web 开发将只关心未来引用的 HTML。所以，是的，总有一天我们会有 HTML6，但会被命名为 HTML，并且会逐渐到来，而不是一夜之间。

因此，在这里，我提出了一些功能，如果可能的话，我今天就想拥有这些功能。

## 更多本土元素

如果我们想要使用用户设备的摄像头，正确的方法是捕获视频，然后将其显示到视频元素中，然后为了捕获快照，我们需要获取帧并将其绘制到画布中。听起来我们想要更多的步骤。

在下一个 HTML 最终版本中，我们必须能够处理更简单、更本地的组件，不仅仅是为了制作更好的网络应用，也是为了 PWA(渐进式网络应用),最终实现史蒂夫·乔布斯的梦想。

HTML 本地摄像头实现示例

这些元素将有自己的 API，甚至其中一些不需要 HTML 元素，如 NFC、Contacts、SMS 或 VR 访问。

## 新元素

网络的发展向我们展示了网络技术不仅可以构建网站，还可以构建复杂的网络应用程序和界面，因此为了改善我们的 DOM 管理，更多的专用标签将会非常有用。

新可能元素的示例

不使用带有 id 属性的

，我们甚至可以在 CSS 样式表中处理得更好。一些开发人员建议，如果我们可以通过名称标签直接调用 id，那会很好，但我真的认为这不是一个好主意。

```
<router></router><script>
 *var* element = document.getElementById(“router”);
</script>
```

除此之外，这是没有用的，因为我们已经有了 getElementByName()，这驱使我们用糟糕的解决方案来区别我们的标签和 HTML 本地标签

```
var tag = document.getElementsByName(“tag”);
```

如果你在这篇文章之前做了一点研究，也许你已经看到了下一种新的可能的标记来解决专用原生 html 标签的使用，但是，这不仅是无用的和冗长的，这将导致更大的网站获得相同的性能。

可怕的 HTML6 建议

我的解决方案是像往常一样使用标签，但是用新的标签来帮助反应式编程、网页设计者和开发者创建更好的网页布局。

> 记住接吻，保持简单愚蠢。

## 预处理器

我不是这方面的大粉丝，但许多开发人员都是，所以这根本不是一个坏特性。这将改进 web 编码，即使这会增加 CPU 的使用量，开发者也会喜欢它。

毫无疑问，这将极大地缩短开发时间。

## 不仅仅是 JavaScript

我知道这至少在今天是不可能的，我也不想这样，但是如果其他开发人员可以在脚本标签中设置另一种语言而不仅仅是 JavaScript，那就太好了。

```
<script type=”text/python”>
    el = dom.elementId(“Element”)
    el.html(“This line will be printed in the element”)
</script>
```

这将是一个真正的游戏改变者，因为网络开发者会在一夜之间增加。一些 Java 开发人员可以在这里使用冗长的语法。

```
<script type=”text/java”>
    import html;

    class Main {
        public static void main(String args[]){
            element div = html.doc.getElementById(“element”);
            div.write(“Element display text”);
        }
    }
</script>
```

# 结论

这是一个快速的回顾，在我看来，HTML 需要什么来改进 web 开发，我希望我能很快看到至少一个这样的特性，所以 web 技术一直是软件应该如何开发的一个例子。