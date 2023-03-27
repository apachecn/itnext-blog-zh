# Helm 3 伞状图表和子图表中的全球价值

> 原文：<https://itnext.io/helm-3-umbrella-charts-global-values-in-sub-charts-666437d4ed28?source=collection_archive---------3----------------------->

![](img/4e310c64bd5909030f3892bd77e706e0.png)

在 Kubernetes 世界中，Helm 伞状图表允许通过父子机制提供子图表。放置在 Helm 的 charts 文件夹中的子图表可以通过在父图表的 *Charts.yaml* 文件中声明子图表来部署。[掌舵](https://helm.sh/)的一个简洁的特性是能够通过使用如下所示的全局变量将值从父(伞状)图表下推到子图表。

```
**global:
  foo: bar**
```

通过指定全局变量，子图表现在可以引用父图表的 *values.yaml* 文件中的全局值，如下所示。

```
**{{- if .Values.global.foo }}
  key1: {{ .Values.global.foo }}
{{- else }}
  key1: {{ .Values.foo }}
{{- end }}**
```

# 解决“在接口{ }中找不到”错误

这达到了我们的目的。如果 foo 全局变量在父图表中声明，子图表将使用它的值，有效地覆盖它自己的 *values.yaml* 文件中的值。问题解决了？不完全是！事实证明，尝试只部署子图表会导致问题，因为模板引擎会尝试解析子图表 *values.yaml* 文件中的所有变量。由于全局变量不存在，您会得到如下结果:

```
**[bash]: helm lint .
[ERROR] templates/: render error in "myproject/templates/deployment.yaml": template: 
myproject/templates/deployment.yaml:8:40: executing "myproject/templates/deployment.yaml" at
 <.Values.global.foo>: can't evaluate field namespace in type interface {}**
```

似乎，没有办法解决这个问题，网上的常见答案指出，在运行 helm 时，必须传入父代的值文件，从而将全局变量传入子图表并允许它运行。然而，这不是一个最优的解决方案，至少对我们的项目来说不是，也许你的是不同的。一切都没有丢失，这可以通过简单地省略变量名本身来解决。

```
**{{- if .Values.global }}
  key1: {{ .Values.global.foo }}
{{- else }}
  key1: {{ .Values.foo }}
{{- end }}**
```

模板引擎将首先评估 IF 语句。因为没有找到全局变量，所以没有计算内部范围，林挺成功完成。在上面的例子中，将使用本地的 *values.yaml* 文件。如果没有找到变量的两个后续部分，模板引擎似乎会抛出一个错误，或者可能是因为全局变量类型是一个保留关键字。不管怎样，我很高兴这对我们的项目有用！

# 结论

Helm 是一个新的工具堆栈，有很多奇怪的地方。例如，它在 range 和 if 语句中限定变量范围的方式。我将在以后的帖子中解决这个问题。

事后看来，这是一个简单的解决方案，但有时最简单的解决方案是最难找到的。

个人网站: [christopherparker.nl](https://christopherparker.nl)