# 转储舵图中的变量

> 原文：<https://itnext.io/dump-variables-in-helm-charts-c4da458b0227?source=collection_archive---------2----------------------->

![](img/a7867a3cd2aeddbe47e3cb86a108bfa4.png)

照片由[通讯社跟随](https://unsplash.com/@olloweb?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# 调试你的舵图

Helm 允许你用 [Go 模板语言](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/)描述你的 Kubernetes 部署，并提供不同的配置值来定制你的部署。对于简单的部署，您可能不需要调试您的舵图。然而，当你试图通过调用高级的[模板函数](https://helm.sh/docs/chart_template_guide/function_list/)来增加更多的自动化时，你将会错过`console.log`或`println()`等价的全功能编程语言。有没有可能将一个变量/值转储为 JSON 字符串，这样我们就可以得到“预期的 map[string]interface {}”的线索；得到 map[string]string "类型错误？

# 舵图中的转储变量

不幸的是，Helm 并没有像其他全功能编程语言那样提供一个相当于`console.log`或`println()`的内置函数。但是，使用“ [fail](https://helm.sh/docs/chart_template_guide/function_list/#fail) 和[musttopretyjson](https://helm.sh/docs/chart_template_guide/function_list/#type-conversion-functions)函数，很容易创建一个可重用的“[命名模板](https://helm.sh/docs/chart_template_guide/named_templates/)”来实现类似的功能。

下面是一个实现示例:

上面几行简单地定义了一个命名模板“magda-var_dump ”,它接受您的输入值，将其转换为 JSON 字符串，并使用`fail`函数输出到控制台。

要使用它，您可以在模板中的任何地方调用命名模板，并传递您想要转储的任何值/变量。

例如，将以下内容添加到您的任何模板文件中

```
{{- $myVar := (list "a" 1 "x" (dict "x1" nil)) }}{{- template "magda.var_dump" $myVar }}
```

运行`helm install`或`helm upgrade`时，将立即中断模板渲染并打印以下内容:

```
The JSON output of the dumped var is:
[
  "a",
  1,
  "x",
  {
    "x1": null
  }
]
```

# 其他示例用法

您可能会发现命名模板的其他有趣用法。例如:

*   打印当前图表元数据:

```
{{- template "magda.var_dump" .Chart }}
```

*   打印当前作用域运行时值:

```
{{- template "magda.var_dump" .Values }}
```

*   甚至打印当前范围:

```
{{- template "magda.var_dump" . }}
```

*   或者打印根范围/上下文:

```
{{- template "magda.var_dump" $ }}
```

# 使用发布的库图表

要试用它，您不必添加上面的命名模板。相反，您可以将我发布的库图表` [magda-common](https://github.com/magda-io/magda/tree/master/deploy/helm/magda-common) '添加到 Chart.yaml 文件的[依赖列表](https://helm.sh/docs/chart_best_practices/dependencies/)中:

```
dependencies:
- name: magda-common
  repository: https://charts.magda.io
  version: 1.0.0-alpha.0
```

然后，你将能够在你的舵图的任何模板文件中调用命名模板`magda.var_dump`。