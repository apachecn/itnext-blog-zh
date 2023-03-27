# 使用命名模板，如舵图中的函数

> 原文：<https://itnext.io/use-named-templates-like-functions-in-helm-charts-641fbcec38da?source=collection_archive---------0----------------------->

![](img/980ad0a681047158f715138f88f6379a.png)

照片由[格伦·凯莉](https://unsplash.com/@glencarrie?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# 像函数一样调用舵模板

[Helm](https://helm.sh/) 提供了[命名模板](https://helm.sh/docs/chart_template_guide/named_templates/)特性，允许用户共享和重用模板片段和部署逻辑。但是仅仅封装&共享复杂的逻辑是不够的。有时，您可能想知道是否有可能像调用函数一样调用命名模板。即:

*   传递具有复杂数据结构的值
*   具有复杂数据结构的返回值

这篇博客将从命名模板的常见用法开始，探索如何用命名模板构造复杂的函数调用。

# ` template '指令与` include '函数

使用命名模板的默认方式是使用`[template](https://helm.sh/docs/chart_template_guide/named_templates/)` [指令](https://helm.sh/docs/chart_template_guide/named_templates/):

```
{{- template "mychart.labels" }}
```

您也可以选择提供一个输入值:

```
{{- template "mychart.labels" $var }}
```

`template`指令的一个缺点是你不能将模板执行/渲染结果捕获到一个变量中或者传递给另一个函数。幸运的是，Helm 提供了一个类似于`template`指令的 [include 函数](https://helm.sh/docs/howto/charts_tips_and_tricks/#using-the-include-function)，同时捕获指定模板的输出。例如

```
{{- $myVar := include "mychart.labels" $var }}
```

因此，`include`函数是调用已定义的命名模板的首选方式。

*请注意:与* `*template*` *指令不同，在调用* `*include*` *功能时，输入值是强制的。*

# 常见用法

通常，您会定义自己的命名模板

```
{{- define "mychart.labels" -}}
  **labels**:
    **myLabel**: {{ .Values.myLabel }}
{{- end -}}
```

然后，您可以调用您的模板并将当前范围`.`传递给它:

```
{{- include "mychart.labels" . }}
```

但是，如果您需要向模板传递多个参数，该怎么办呢？

# 使用多个参数调用模板

虽然`include`函数只接受一个输入值，但这并不妨碍您将所有参数组合到一个[字典](https://helm.sh/docs/chart_template_guide/function_list/#dictionaries-and-dict-functions)中，并将其作为单个输入值传递给`include`函数。

例如，假设我们想要定义一个计算两个参数`a` & `b`之和的模板:

```
{{- define "mychart.add" -}}
{{- $r := add .a .b }}
{{- $r }}
{{- end -}}
```

要使用模板，我们可以按如下方式调用它:

```
{{- $result := include "mychart.add" (dict "a" 3 "b" 4) }}
```

如果你使用[我之前的博客介绍的 var_dump 命名模板](https://medium.com/itnext/dump-variables-in-helm-charts-c4da458b0227)来验证结果，它将打印`"7"`。

```
{{- $result := include "mychart.add" (dict "a" 3 "b" 4) }}
{{- include "magda.var_dump" $result }}
```

# 返回复杂的数据结构

您可能会注意到，命名模板输出只能是一个字符串。如果试图输出非字符串值，它将被转换为字符串。

一个简单的解决方法是使用 [mustToJson](https://helm.sh/docs/chart_template_guide/function_list/#type-conversion-functions) 函数将返回值转换成 Json 字符串，让 template 输出 JSON 字符串，然后使用 [mustFromJson](http://masterminds.github.io/sprig/defaults.html) 函数恢复原来的值。

```
{{- define "mychart.getNumbers" }}
{{- $r := regexFindAll "[\\d\\.]+" . -1 }}
{{- mustToJson $r }}
{{- end -}}
```

上面的模板接受一个字符串作为输入值，并返回一个表示数字的子字符串列表。

例如，输入字符串“3.1 加 5.4 的结果是 8.5”:

```
{{- $result := include "mychart.getNumbers" "The result of 3.1 plus 5.4 is 8.5!" | mustFromJson }}
{{- include "magda.var_dump" $result }}
```

var_dump 的结果将是一个列表:`["3.1", "5.4", "8.5"]`。

# 递归函数-斐波那契数

也可以用 Helm 命名的模板构造递归函数。以[斐波那契数](https://en.wikipedia.org/wiki/Fibonacci_number)问题为例。在给定位置生成斐波那契数的函数的递归实现在 javascript 中如下所示:

```
function fibonacci(num) {
    if(num < 2) {
        return num;
    }
    else {
        return fibonacci(num-1) + fibonacci(num - 2);
    }
}
```

我们可以如下实现 Helm 命名模板:

```
{{- define "mychart.fibonacci" -}}
{{- /* For ease of reading, assign input value to variable $num */}}
{{- $num := . }}
{{- /* test if $num is lower than 2 */}}
{{- if lt $num 2 }}
  {{- $num | mustToJson }}
{{- else }}
  {{- add (include "mychart.fibonacci" (sub $num 1) | mustFromJson) (include "mychart.fibonacci" (sub $num 2) | mustFromJson) }}
{{- end }}
{{- end -}}
```

我们可以使用我们的实现来输出斐波那契数列的列表:

```
{{ $result := list }}
{{- range $num := until 10 }}
  {{- /* here we need to use `=` for assignment only.  */}}
  {{- /* Use `:=` will delcare a local variable and won't update $result.  */}}
  {{- $result = append $result (include "mychart.fibonacci" . | mustFromJson) }}
{{- end }}
{{- /* dump the result  */}}
{{- include "magda.var_dump" $result }}
```

上面的模板会将`[0,1,1,2,3,5,8,13,21,34]`输出到控制台。

# 尝试一下

要尝试一下，您可以:

*   用`helm create`创建一个虚拟舵图表
*   将任何示例复制并粘贴到模板文件中
*   使用`helm template`运行示例

你可能注意到了`"magda.var_dump"`的用法。它是一个名为 template 的助手，可以暂停模板渲染，并将给定的变量转储到控制台。更多细节，请看看[我之前的关于舵图](https://medium.com/itnext/dump-variables-in-helm-charts-c4da458b0227)中变量转储的博客。