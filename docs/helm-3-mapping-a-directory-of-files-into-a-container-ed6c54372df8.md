# Helm 3 —将文件目录映射到容器中

> 原文：<https://itnext.io/helm-3-mapping-a-directory-of-files-into-a-container-ed6c54372df8?source=collection_archive---------0----------------------->

![](img/1d7e831867b70c570084b8312d710c07.png)

在大多数情况下，当使用 Helm 将文件映射到容器时，标准方法是使用 ConfigMap、Volume 和 VolumeMount。对于单个文件，这种方法非常好，但是如果需要将一个文件目录映射到一个容器中呢？如果这些文件名不符合 YAML 关键字命名标准，该怎么办？

使用 Helm 的主要原因可能是基于一组不断变化的变量动态构建 Kubernetes 对象。每次运行 Helm 升级时动态生成 ConfigMap 的功能非常强大，但是当与用于文件映射的卷装载结合使用时，特别是在 YAML 不支持的每个部署或文件名都有许多文件发生变化的情况下，这将带来一些麻烦的挑战。这篇文章将尝试解决其中的一些问题，并提出一个优雅的动态映射文件目录的解决方案。

# 简单的例子—一个文件

假设我们希望将一个文件映射到一个特定路径的容器中。实现这一点的简单方法如下:

## 第一步:

创建一个配置映射来映射名为 *config.yaml l* ike 的文件，如下所示:

```
**apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-single-file
binaryData:
{{ range $path, $bytes := .Files.Glob (printf "/path/to/configfile/config.yaml")}}
{{ $name := base $path }}
{{- printf "%s" $name}}{{ print ": "}}{{ $.Files.Get $path | b64enc }}
{{ end }}**
```

## 第二步:

在作业、StatefulSet、DaemonSet、Deployment 等之一中创建卷装载和卷段。将卷映射到配置映射，并将卷装载到卷，如下所示:

```
**volumes:
  - name: cm-single-file-volume
    configMap:
      name: cm-single-file
volumeMounts:
  - name: cm-single-file-volume
    mountPath: /location/to/mount/config.yaml
    subPath: config.yaml**
```

上面的例子获取了一个名为 *config.yaml，*的文件，并通过 ConfigMap 将它映射为一个容器。然而，事情并不总是这么简单。

## 挑战

在扩展上述方法时，存在许多挑战:

*   使用不符合 YAML 规范的字符的文件名会导致 Helm 失败。此类文件名的示例:

```
**/some/directory/of/files/data,production.yaml
/some/directory/of/files/config-prod.yaml**
```

*   当试图映射一个有几十甚至几百个文件的目录时，Helm 崩溃了。虽然可以列出一个目录并将这些文件映射到一个配置映射中，但是不能迭代结果配置映射的内容，比如说，一个部署对象。这意味着，为了生成映射到配置映射的文件列表以便在文件目录中进行映射，必须再次完成源目录的目录列表。如果你有不符合 YAML 标准的文件名，你就不走运了。

不幸的是，我遇到了这些问题，在谷歌找到一些结果后，我不得不想出一个创造性的方法来解决这些问题。

# 使用 Helm 3.0 映射多个文件

鉴于上述问题，如何使用 Helm 将多个文件映射到一个容器中呢？答案在理论上有些简单，但对于初学者来说，生成它的 Helm 模板代码有些复杂。

## 第一步:

第一步是枚举文件列表，就像我们在简单的单个文件示例中开始做的那样:

```
**apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-multi-file
binaryData:
{{ range $path, $bytes := .Files.Glob (printf "/path/to/configfiles/**")}}
{{ $name := base $path }}
{{- sha256sum (printf "%s/%s" (index (regexSplit "path_to_files" (dir $path) -1) 1 ) $name ) | indent 2 }}{{ print ": "}}{{ $.Files.Get $path | b64enc }}
{{ end }}**
```

注意这次有一些不同:

*   首先，我们使用内置的 sprig 函数 [sha256sum](http://masterminds.github.io/sprig/crypto.html) 。这个函数允许我们创建文件名的 sha256 哈希，从而解决了上面列出的第一个挑战。由于 sha256 哈希包含 YAML 支持的字符作为键名，我们可以绕过任何奇怪的文件命名约定。
*   其次，我们使用正则表达式语法从完整路径中只提取文件名，因为我们在生成散列时不希望路径出现在文件名中。
*   最后，与单个文件示例一样，我们对文件的内容进行了 base64 加密，这使得我们也可以利用二进制数据文件。

就是这样。我们现在有了一个包含文件条目的 ConfigMap，可以在部署、StatefulSet、DaemonSet 或 job 中使用。

## 第二步:

与单个文件示例一样，第二步是通过引用我们的新 ConfigMap 将我们的文件映射到一个 VolumeMount，如下所示:

```
**volumes:
 - name: cm-multi-file-volume
   configMap:
     name: cm-multi-file
volumeMounts:
 {{ range $path, $bytes := .Files.Glob ( printf "/path/to/configfiles/**") }}
 {{ $name := base $path }}
  - name: cm-multi-file-volume
    mountPath: {{ printf "/etc/configuration/%s/%s" (index (regexSplit "/path/to/configfiles" (dir $path) -1) 1) $name | indent 2 }}
    subPath: {{- sha256sum (printf "%s/%s" (index (regexSplit "path_to_files" (dir $path) -1) 1 ) $name ) | indent 2 }}
 {{ end }}**
```

这里发生了很多事情，让我们来分解一下:

*   与单个文件示例一样，我们首先必须创建一个引用配置图的卷
*   然后，我们必须创建一组卷装载。这一次，我们需要从配置图中动态添加文件集，而不是手动指定文件。由于我们无法读取创建的配置图，我们再次读取文件的目录列表。
*   上例中的挂载路径只是文件所在的文件夹(也可以是其他文件夹)，映射到/etc/configuration。因此，完整的安装路径应该是:

```
**/etc/configuration/configfiles/<filename>**
```

*   对于每个文件，我们必须定义一个子路径。由于配置映射使用 SHA256 哈希引用文件名，我们必须以与配置映射相同的方式重新哈希文件名。也就是说，通过分离基本路径，只给我们留下文件名。

假设有一个文件目录，结果输出如下所示:

```
**apiVersion: v1
  kind: ConfigMap
  metadata:
   name: cm-multi-file
  binaryData:
481562103018784bef2571ef183a5284b8246f3e2f1fb86143f401c30602a3a9: sd3243=****6001ea194c183a6a42269c3d40334a87ade85de82999ec42814618df58a5176a: 2347dfklj37==****97acc83e0b4632fe551bd472aac750065b6b73817bbfc7ec8ca05742d9bd2ed8: jhkfv2348lj37==****0e39b2d867947dcb78613d4bb6be6cc714792db9671047dc0ef8efba88185f74: sdfk98gf37==**
```

*   上面我们有一个 ConfigMap 条目，其中映射了 4 个文件。每个文件的名称都经过了哈希处理。每个文件中的数据都经过了 base64 编码，因为我们使用的是 binaryData ConfigMap 条目。

```
**apiVersion: v1
 kind: Deployment
 ...
       volumes:
         - name: cm-multi-file-volume
           configMap:
             name: cm-multi-file
       containers:
           volumeMounts:
  **           - name: **cm-multi-file-volume
**                 mountPath:   **/etc/configuration/configfiles/file1.yaml**
                 subPath:  **481562103018784bef2571ef183a5284b8246f3e2f1fb86143f401c30602a3a9** - name: **cm-multi-file-volume
  **               mountPath:   **/etc/configuration/configfiles/file2.yaml**
                 subPath:  **6001ea194c183a6a42269c3d40334a87ade85de82999ec42814618df58a5176a** - name: **cm-multi-file-volume
  **               mountPath:   **/etc/configuration/configfiles/file3.yaml**
                 subPath:  **97acc83e0b4632fe551bd472aac750065b6b73817bbfc7ec8ca05742d9bd2ed8** - name: **cm-multi-file-volume
  **               mountPath:   **/etc/configuration/configfiles/file4.yaml**
                 subPath:  **0e39b2d867947dcb78613d4bb6be6cc714792db9671047dc0ef8efba88185f74**
```

*   在部署对象中，我们有 VolumeMount 条目，它们将文件名映射到它的最终挂载点。注意，文件名仍然是 SHA256 哈希，但是挂载路径现在包含了实际的文件名

# 替代解决方案—压缩文件

这是解决这个问题的一种方法。还考虑了 zip 文件解决方案，但是由于 Helm 没有内置的本地 zip/压缩功能，我不得不依赖 contain 来解压配置文件的目录。由于您不想将配置以 zip 文件的形式签入源代码管理，因此在升级 Helm 之前，其他一些进程需要生成一个 zip 文件。因此，就解决方案的优雅性和在这种情况下只依赖 Helm(至少对我来说)而言，基于散列的方法增加的复杂性是值得的。

# 摘要

如果文件的名称不符合 Helm YAML 密钥命名标准语法，则使用 Helm 3 无法轻松地将文件目录映射到容器中。为了解决这个问题并保持在 Helm 工具的范围内，可以由文件名组成 SHA256 散列来克服这个问题，因此可以使用配置映射、卷和卷装载的组合来映射到容器映像。