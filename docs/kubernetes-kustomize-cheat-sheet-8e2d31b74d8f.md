# Kubernetes Kustomize 备忘单

> 原文：<https://itnext.io/kubernetes-kustomize-cheat-sheet-8e2d31b74d8f?source=collection_archive---------0----------------------->

![](img/77ddc3d1b9009cefe279999b04996350.png)

> kustomize 是一个命令行工具，支持面向 k8s 风格对象的声明性配置的无模板、结构化定制。
> 
> 以 k8s 为目标意味着 kustomize 对 API 资源、k8s 概念如名称、标签、命名空间等有所了解。，以及资源修补的语义。
> 
> kustomize 是 DAM 的一个实现。

*   [Github](https://github.com/kubernetes-sigs/kustomize)
*   [Kustomize 文档](https://kustomize.io/)

# 装置

```
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

# 基本用法

## 内置于 kubectl

Kustomize 内置于`kubectl`。`kubectl apply -k`行为与`kustomize build path/to/some/app | kubectl apply -f -`如出一辙。

## 创建基础

从 Kubernetes 清单中
搜索当前目录中的所有 Kubernetes 资源，并将它们添加到`resources`

```
kustomize create --autodetect
```

从刈割基础
使用另一个刈割覆盖的`path/to/base`作为新覆盖的基础

```
kustomize create --resources path/to/base
```

搜索当前目录和所有子目录

```
kustomize create --resources path/to/base --recursive
```

# 构建资源

## 统一资源定位器

```
kustomize build [https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld?ref=v1.0.6](https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld?ref=v1.0.6)
```

# 改性碱

## 标签和注释

从远程资源创建新的 kustomization.yaml，添加到所有资源标签`app=hello-world`和`cloud=gcp`

```
$ kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6 --labels app:hello-world,cloud:gcp
$ kustomize build .
```

# 命名空间

从远程资源创建新的 kustomization.yaml，向其所有资源添加`namespace`属性

```
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6 --namespace dev
kustomize build .
```

# 资源名称

更新所有资源的名称前缀

```
$ kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
$ kustomize edit set nameprefix temp
$ kustomize build .
```

# 形象

将图像`monopole/hello:1`更新为`monopole/hello:latest`

```
$ kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
$ kustomize edit set image monopole/hello:1=monopole/hello:latest
$ kustomize build .
```

# 配置图和机密

从变量生成配置映射清单

```
$ kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
$ kustomize edit add configmap hello-world --behavior create --from-literal=host=google.com
$ kustomize build .
```

# JSON 补丁

更新副本的数量

```
$ cat <<EOF > patch.yaml
- op: replace
  path: /spec/replicas
  value: 1
EOF
$ kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
$ kustomize edit add patch --kind Deployment --path patch.yaml
$ kustomize build .
```

# 战略合并

添加`emptyDir`音量

```
$ cat <<EOF > patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  template:
    spec:
      containers:
      - name: the-container
        volumeMounts:
        - name: emptyDir
          mountPath: /appdata 
    volumes:
    - name: emptyDir
      emptyDir: {}
EOF
$ kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
$ kustomize edit add patch --kind Deployment --path patch.yaml
$ kustomize build .
```

# 移除资源

从最终清单集中完全省略资源

```
$ cat <<EOF > patch.yaml
\$patch: delete
apiVersion: v1
kind: ConfigMap
metadata:
  name: the-map
EOF
$ kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
$ kustomize edit add patch --kind ConfigMap --path patch.yaml
$ kustomize build .
```