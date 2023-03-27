# OPA —验证策略

> 原文：<https://itnext.io/opa-validating-policies-d6d05992e13a?source=collection_archive---------4----------------------->

![](img/6938ef300fd79eee170742a3f2d17b2c.png)

LGTM 竖起大拇指！

制定 OPA 策略需要结合使用减压阀语言和 YAML 配置。要验证策略是否有效，有几种方法:

*   **单元测试** —用单元测试验证一组独立的断言。像大多数语言一样，减压阀也有一个单元测试框架。基于 rego 语言对于新手的复杂性，我再怎么强调单元测试的使用也不为过。单元测试可能是另一篇博客的主题。
*   **部署到集群** —创建您的 rego，将其打包到 OPA YAML 中，并使用 kubectl 或您选择的方法提交到集群
*   **使用验证工具** —验证可以检查语法错误、语言和语义。

在这篇博客中，我将重点介绍如何使用验证工具向策略工程师提供快速反馈。这个工具非常有价值，因为我在很长一段时间内都没有可用的集群。它引发了许多即使部署到集群也不会发现的错误。

# 策略验证工具

Google 在其 Forsetti 合规引擎中广泛使用 OPA，也在 Anthos 策略控制器中使用它。因此，他们开发了一个[工具](https://github.com/GoogleCloudPlatform/policy-library#linting-policies)来支持与您的策略相关的 rego 和 YAML 的本地验证。

在下面的例子中，我将在 docker 容器中运行这个工具，这样我就不用担心安装它了(我相信它是基于 Java 的)。下面的命令将当前工作目录中的一些文件夹挂载到容器中，以便它可以读取 YAML 文件和库(如果有的话)。

```
docker run -it --rm \
		-v "$(CURDIR):/workspace" \
		-w=/workspace \
		gcr.io/config-validator/pollicy-tool:latest \
		lint \
		--policies /workspace/yaml \
		--libs /workspace/lib
```

该工具的结果可能有些冗长。一个问题一个问题地通读，并在过程中解决它们。请记住，该工具在遇到第一个错误时就会退出。

我已经将该工具集成到这个示例库[的 makefile 中](https://github.com/patpicos/rego_doc_gen)

# 发现的示例问题

## OPA 期望

我最初开发的许多策略都是以 rego 世界中相当常见的格式完成的。然而，OPA 希望最终的答案采用不同的格式

```
deny[msg] {
  1 == 1
  msg := "This is raised violation"
}
```

我必须将我的违规转换成如下所示，以便 OPA Gatekeeper 揭露您的违规

```
violation[{"msg": msg}] {
  1 == 1
  msg := "This is raised violation"
}
```

## 约束模板验证

在我开始使用 Konstraint 从 rego 代码生成 ConstraintTemplates 之前，我是手工构建它的。

验证策略工具做的一件事是确保 CRD 名字是骆驼大小写，并且名字是小写版本。

```
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  creationTimestamp: null
  name: policyone   # Must be lowercase version of the spec.crd.spec.names.kind
spec:
  crd:
    spec:
      names:
        kind: PolicyOne
```

## 无效的减压阀函数

在我的一个策略中，我使用了 regex.match。验证器抱怨说这不是一个有效的函数，尽管它在文档中。我不得不改用 re_match()函数。

这是一个非常模糊的问题，除了这篇文章之外，几乎没有什么细节

# 结论

如您所见，策略工具在 rego 逻辑和 YAML 语义方面都做了非常深入的验证。它为开发人员提供了快速的反馈，而无需访问非常强大的集群。