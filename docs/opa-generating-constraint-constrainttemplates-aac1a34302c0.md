# OPA 生成约束/约束模板

> 原文：<https://itnext.io/opa-generating-constraint-constrainttemplates-aac1a34302c0?source=collection_archive---------3----------------------->

![](img/45b5ff5711efedcf785b355d5170ea22.png)

konstraint 标志

最近，我开始使用开放策略代理(OPA)网关守护设备编写要在 Kubernetes 集群上实施的策略。该过程包括使用减压阀语言创建策略/断言，然后将逻辑打包到 OPA CRD 的:

*   **ConstraintTemplate** —具有 rego 逻辑的资源，并定义策略是否具有输入参数(由约束提供)。
*   **约束** —定义什么资源类型将触发 ConstraintTemplate 中定义的策略的评估。此外，这是您可以向策略参数传递值的地方。

我面临的挑战之一是将我的减压阀逻辑嵌入到 ConstraintTemplate 中。我经历了几次反复，想分享一下最终的选择。

方法 1——我最初从一个带有 sed 命令的原始 bash 脚本开始。这是一个糟糕的方法，因为它没有验证 YAML。它充其量是 hacky。它主要读取. spec.targets.rego 中的值，该值被设置为文件名。然后，我会读取该文件并将其加载到 YAML 中。

方法 2——在与我的 Google 客户工程师交谈时，她向我介绍了一个用于 Forsetti 策略的 python 脚本，它也是基于 OPA 的。我不得不根据自己的需要做一些小的更新。虽然这有效，但我必须继续维护这个脚本。来源:[https://github . com/Google cloud platform/policy-library/blob/master/scripts/inline _ rego . py](https://github.com/GoogleCloudPlatform/policy-library/blob/master/scripts/inline_rego.py)

方法 3——灵光一现，一位同行在 [github](https://github.com/plexsystems/konstraint) 上发现了这块宝石。konstraint 是一个可以执行以下操作的实用程序:

*   生成约束模板
*   生成约束(仅当模板中没有参数时)
*   生成文档—使用类似于 Javadoc 的注释和文档格式，自动生成文档

基于 konstraint 提供的价值，我决定使用它来减少我个人管理的代码量

# 生成约束模板/约束

要生成 YAML 内容，请运行 konstraint create。

```
konstraint create opa --output yaml
```

它遵循一些惯例:

*   opa/ folder 下的文件夹名称将驱动为 ConstraintTemplate 生成的 CRD 的名称。
*   避免在文件夹名称中使用下划线，以符合 kubernetes 对所有资源强制执行的 DNS 命名标准。
*   生成的文件将如下所示

```
constraint_<folder_name>.yaml
template_<folder_name>.yaml
```

如上所述，如果您有带参数的策略，将不会生成约束。你需要自己管理它们。您将看到一条警告:

```
WARN[0000] skipping constraint generation due to use of parameters name=<policy name> src=opa/<folder>/<file>.rego
```

要生成 ConstraintTemplate / Constraint，请在包定义之前创建一个文档块。它看起来会像这样:

```
# @title this is a basic title for my policy
#
# This is the longer explanation of my policy details 
# and the opinions it enforces
# @enforcement deny  # This is the default, but could be dryrun# The line below is used to generate the constraint by linking the resource types of interest to the policy
# @kinds core/pod bigquerydatasets.crnm.cloud.google.com/BigQueryDataset# The line below defines parameters for the ConstraintTemplate. As such, you need to define the Constraints yourself.
# @parameter <name> array string
# @parameter resource_types array string
```

更多信息，请参考 konstraint [文档](https://github.com/plexsystems/konstraint/blob/main/docs/constraint_creation.md)。

# 生成文档

要生成文档，请运行 konstraint doc 命令。它将从当前目录中扫描，以查找带有 violation[]条目(匹配 OPA 框架/预期)的 rego 文件。

因此，它将使用从 rego 代码中提取的信息创建一个 policies.md。这些细节将是未来博客文章的基础。

```
konstraint doc --no-rego
```

# 结论

konstraint 超出了我的预期，帮助我实现了尽可能自动化的目标。如果没有它或我的临时方法，我将不得不构建 rego 文件并手动将其复制到 ConstraintTemplates 中。绝对不是一个可持续的方法。

系统生成的文档是最棒的……是下一篇博客的焦点。