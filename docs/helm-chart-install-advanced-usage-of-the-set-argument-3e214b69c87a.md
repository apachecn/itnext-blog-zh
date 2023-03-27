# 舵图安装:“Set”参数的高级用法

> 原文：<https://itnext.io/helm-chart-install-advanced-usage-of-the-set-argument-3e214b69c87a?source=collection_archive---------0----------------------->

![](img/bc0f8845d8dd4e13b276ea256482ef60.png)

当使用`helm install`命令安装舵图表时，`--set`参数允许您定义图表模板中使用的值。

`--set`参数是在`values.yaml`文件中定义值的替代方法。

`--set`的一个简单用法如下:

```
$ helm install codecentric/keycloak --set keycloak.replicas=2
```

相反，要用一个`values.yaml`文件设置相同的值，您可以将该值添加到文件中，然后使用`-f`参数指定文件的路径。

```
# values.yaml
keycloak:
  replicas: 2
```

# 使用`--set`的挑战

在一些特殊情况下，很难在命令行上指定 YAML 属性/值。

这些特殊情况涉及包含与 YAML 语法冲突的字符的属性和值或未记录的行为。

# 为什么不用 values.yaml？

处理这些特殊值的常见解决方法是恢复使用一个`values.yaml`文件。但是，在某些情况下使用`values.yaml`是不可行的。

使用`values.yaml`文件不可行的一种常见情况是当值或属性被期望在运行时作为动态值提供时。

# 为舵安装提供动态值

不能在`values.yaml`中注入或使用动态值。外壳脚本可以提供动态值功能。

在本例中，shell 脚本提示用户输入，然后通过`--set`参数将用户的输入注入到舵图中。

```
#!/bin/bash# read dynamic value
read -p "Keycloak replicas: " replicas# set dynamic value
helm install codecentric/keycloak --set keycloak.replicas=$replicas
```

使用`values.yaml`文件无法实现前面的示例。

# 数组

数组在`values.yaml`文件中是这样定义的:

```
# values.yaml
keycloak:
  ingress:
    hosts:
    - "auth1"
    - "auth2"
```

Helm 提供了两种在命令行定义数组值的方法。

第一种方法是用`{}`包装值列表。数组中的元素用逗号分隔。

```
$ helm install codecentric/keycloak \
  --set 'keycloak.ingress.hosts={auth1,auth2}'
```

在命令行上定义数组值的第二种方法是通过索引指定单个数组元素。

```
$ helm install codecentric/keycloak \
  --set 'keycloak.ingress.hosts[0]=auth1' \
  --set 'keycloak.ingress.hosts[1]=auth2'
```

# 带有特殊字符的属性名

当属性名包含特殊字符时，它们在`values.yaml`文件中不需要任何特殊语法。

```
# values.yaml
keycloak:
  podLabels:
    app.kubernetes.io/part-of: security
```

但是，在命令行上，必须使用`\`对属性名中的特殊字符进行转义。

```
$ helm install codecentric/keycloak \
  --set 'keycloak.podLabels.app\.kubernetes\.io/part-of=security'
```

需要转义的特殊字符在[Helm 源代码](https://github.com/helm/helm/blob/v3.4.2/pkg/strvals/parser.go#L159)中定义。他们是`.`、`[`、`,`和`=`。

# 多行字符串

在一个`values.yaml`文件中，使用标准的 YAML 语法定义多行字符串:

```
# values.yaml
keycloak:
  extraEnv: |
      - name: KEYCLOAK_HTTP_PORT
        value: "80"
      - name: KEYCLOAK_HOSTNAME
        value: auth.k8salliance.com
```

*注意* `*extraEnv*` *值实际上是一个字符串，尽管它包含 YAML 语法。Keycloak 舵图将值作为字符串读取，然后将该字符串作为舵模板进行处理。*

不幸的是，在带有转义符的命令行上，值字符串中不可能包含换行符(例如`\n`),就像您所期望的那样。

您可以在命令行上通过用引号将整个属性键/值括起来来设置相同的值。

```
$ helm install codecentric/keycloak \
--set "keycloak.extraEnv=- name: KEYCLOAK_HTTP_PORT
  value: \"80\"
- name: KEYCLOAK_HOSTNAME
  value: auth.k8salliance.com"
```

前面的命令行语法将产生与`values.yaml`相同的值。