# 参考舵图表值文件中的其他值

> 原文：<https://itnext.io/reference-other-values-in-helm-chart-values-file-19d44d9276c7?source=collection_archive---------0----------------------->

![](img/351e51b48167e4ed50c19b6d038f591c.png)

由 [Vardan Papikyan](https://unsplash.com/@varpap?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

# 背景

Helm 提供了一种简单的模板语言，允许我们轻松地引用在“值文件”中定义的配置值。

```
**apiVersion**: v1
**kind**: ConfigMap
**metadata**:
  **name**: my-configmap
**data**:
  **myvalue**: "Hello {{ .Values.name }}"
```

例如，在上面的舵图模板中，我们引用了在“值文件”中定义的配置值“名称”。假设配置字段“name”的值是一个字符串“world”，模板最终将呈现如下:

```
**apiVersion**: v1
**kind**: ConfigMap
**metadata**:
  **name**: my-configmap
**data**:
  **myvalue**: "Hello world"
```

在舵图模板中引用一个“值”看起来简洁明了。但是我们如何引用舵图“值文件”中的其他值呢？

# 为什么？

下面是一个示例“值文件”,它提供了一个 API 端点 URL 列表:

```
**apiOneUrl**: http://example.com/apiOne/v0
**apiTwoUrl**: http://example.com/apiTwo/v3
**apiThreeUrl**: http://example.com/apiThree/v7
```

您可能会发现所有配置字段都包含字符串“http://example.com/”。如果我们能提供如下“值文件”就好了:

```
**baseUrl**: http://example.com
**apiOneUrl**: "{{ .Values.baseUrl }}/apiOne/v0"
**apiTwoUrl**: "{{ .Values.baseUrl }}/apiTwo/v3"
**apiThreeUrl**: "{{ .Values.baseUrl }}/apiThree/v7"
```

这不仅会减少冗余，节省大量的打字时间，而且会使将来更新配置值变得更加容易。然而，由于 Helm 的模板引擎不会处理“值文件”,如果您试图在您的模板中引用`.Values.apiOneUrl`,您将只能获得原始字符串`{{ .Values.baseUrl }}/apiOne/v0`作为结果。

# `tpl`功能营救

Helm chart `tpl`函数允许开发人员将字符串作为模板中的模板进行评估，这只是我们这里的保护程序。该函数需要两个参数:

*   第一个参数是要处理的模板字符串。
*   最后一个参数是在模板字符串处理过程中使用的上下文数据。我们通常可以传递当前上下文数据`.`,以使所有配置值在模板处理期间可用。

如果您在模板中尝试以下操作:

```
**MyApiOneUrl**: {{ tpl .Values.apiOneUrl . }}
```

你会发现`.Values.apiOneUrl`的值`"{{ .Values.baseUrl }}/apiOne/v0"`会被转换成`"http://example.com//apiOne/v0”`，上面的模板会被渲染成:

```
**MyApiOneUrl**: http://example.com/apiOne/v0
```

# 非字符串类型的问题

该解决方案适用于简单的字符串类型配置值。但是，如果配置值是非字符串类型的值(例如“map”或“list”)该怎么办呢？

考虑以下“值文件”:

```
**baseUrl:** http://example.com **apiUrls:**
  **apiOneUrl**: "{{ .Values.baseUrl }}/apiOne/v0"
  **apiTwoUrl**: "{{ .Values.baseUrl }}/apiTwo/v3"
  **apiThreeUrl**: "{{ .Values.baseUrl }}/apiThree/v7"
```

如果您在模板中尝试以下操作:

```
**MyApiUrls: {{** tpl .Values.apiUrls . **}}**
```

您将得到以下模板错误:

```
wrong type for value; expected string; got map[string]interface {}
```

由于`tpl`函数期望一个模板字符串作为第一个参数，这个错误消息是有意义的。但是，我们如何解决这个问题并将我们的解决方案扩展到这个常见的用例呢？

一种解决方案是在传递给`tpl`函数之前，使用`toYaml`函数将非字符串类型值(本例中为“map”)转换为“YAML”字符串:

```
**MyApiUrls:
{{** tpl (.Values.apiUrls | toYaml) . **|** indent 2 **}}**
```

如果您尝试上面的模板，您会发现它会正确地呈现为:

```
 **MyApiUrls:
  apiOneUrl**: http://example.com/apiOne/v0
  **apiTwoUrl**: http://example.com/apiTwo/v3
  **apiThreeUrl**: http://example.com/apiThree/v7
```

# 完整的解决方案

为了使解决方案具有可移植性，我们可以定义一个[“命名模板”](https://helm.sh/docs/chart_template_guide/named_templates/)来检测配置值类型&并相应地应用不同的逻辑:

```
{{ define "render-value" }}
  {{- if kindIs "string" .value }}
    {{- tpl .value .context }}
  {{- else }}
    {{- tpl (.value | toYaml) .context }}     
  {{- end }}
{{- end }}
```

要在模板中使用，可以使用`include`函数调用“命名模板”:

```
**MyApiOneUrl**: {{ include "render-value" ( dict "value" .Values.apiOneUrl "context" .) }}
**MyApiUrls:** {{ include "render-value" ( dict "value" .Values.apiUrls "context" .) | indent 2}}
```

上面的模板将呈现如下:

```
**MyApiOneUrl**: http://example.com/apiOne/v0
**MyApiUrls:
  apiOneUrl**: http://example.com/apiOne/v0
  **apiTwoUrl**: http://example.com/apiTwo/v3
  **apiThreeUrl**: http://example.com/apiThree/v7
```

如果你想省去自己定义“命名模板”的功夫，也可以使用 [Bitnami 的“通用”库图](https://github.com/bitnami/charts/tree/master/bitnami/common)。要使用它，只需将以下依赖项添加到您的`Chart.yaml`中:

```
dependencies:
  - name: common
    version: 1.11.1
    repository: https://charts.bitnami.com/bitnami
```

然后，您将能够调用包含的“命名模板”,其逻辑类似于以下内容:

```
**MyApiOneUrl**: {{ include "common.tplvalues.render" ( dict "value" .Values.apiOneUrl "context" .) }}
```

# 摘要

Helm 提供了一个简单而强大的模板引擎，允许我们将配置值合并到模板中。然而，有时我们可能希望将这种能力扩展到配置文件:也就是掌舵人的“值文件”。本文介绍了一个基于`tpl`函数的解决方案及其更通用的“命名模板”实现。

如果你不熟悉 Helm 的“命名模板”功能，你可能也想看看[我的相关博客](/use-named-templates-like-functions-in-helm-charts-641fbcec38da)来了解更多关于“命名模板”可以实现的功能。