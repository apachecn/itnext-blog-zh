# 仅用 Kubectl 构建 YAML

> 原文：<https://itnext.io/constructing-yaml-with-kubectl-only-15b3fbff7485?source=collection_archive---------1----------------------->

![](img/80d3891854301bd50ebee387b2d5bfbe.png)

我迫切需要在一个空气间隙的环境中创造一些 K8s 资源，可惜我的备忘单也不在身边:(

“不可能完成的任务”仍然可以在 kubectl 命令行工具的帮助下完成。

让我们看看，我们需要限制名称空间中的最大 Pod 资源限制，这通常是通过“limits”K8s 资源来实现的。似乎没有办法通过命令行用 kubectl 工具创建它，我们必须使用通用的 YAML 文件。但是我记不起确切的语法了。

首先，让我们通过找出资源的正式名称

```
**kubectl api-resources | grep -i limit**
```

这让我，

```
limitranges                       limits                                      true         LimitRange
```

所以我知道资源的全名是 limitranges，简称是 limits，种类名是 LimitRange。所以至少现在，我可以开始我的 YAML 了

```
kind: LimitRange
metadata:
  name: mylimit
spec:
```

apiVersion 和确切的规格是什么？第二个 kubectl 命令会告诉我们，

```
**kubectl explain limits**
```

这给了我，

```
KIND:     LimitRange
VERSION:  v1DESCRIPTION:
     LimitRange sets resource usage limits for each kind of resource in a
     Namespace.FIELDS:
   apiVersion <string>
     APIVersion defines the versioned schema of this representation of an... ...SKIPPED MANY LINES... spec <Object>
     Spec defines the limits enforced. More info:
     [https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)
```

所以 apiVersion 应该设置为“v1”。

我可以进一步寻求帮助

```
**kubectl explain limits.spec**
```

我的结果是，

```
KIND:     LimitRange
VERSION:  v1RESOURCE: spec <Object>DESCRIPTION:
     Spec defines the limits enforced. More info:
     [https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)LimitRangeSpec defines a min/max usage limit for resources that match on
     kind.FIELDS:
   limits <[]Object> -required-
     Limits is the list of LimitRangeItem objects that are enforced.
```

所以对于规范，我必须用键“limits”定义一个数组对象。

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mylimit
spec:
  limits:
    - 
```

遵循树形结构，运行`kubectl explain limits.spec.limits`将返回，

```
KIND:     LimitRange
VERSION:  v1RESOURCE: limits <[]Object>DESCRIPTION:
     Limits is the list of LimitRangeItem objects that are enforced. LimitRangeItem defines a min/max usage limit for any resource that matches on kind.FIELDS:
   default <map[string]string>
     Default resource requirement limit value by resource name if resource limit is omitted. defaultRequest <map[string]string>
     DefaultRequest is the default resource requirement request value by resource name if resource request is omitted. max <map[string]string>
     Max usage constraints on this kind by resource name. maxLimitRequestRatio <map[string]string>
     MaxLimitRequestRatio if specified, the named resource must have a request and limit that are both non-zero where limit divided by request is less than or equal to the enumerated value; this represents the max burst for the named resource. min <map[string]string>
     Min usage constraints on this kind by resource name. type <string>
     Type of resource that this limit applies to.
```

所以现在极限需要定义类型和极限的不同组合。极限范围的知识现在可以联系起来了。所以我可以写我的 YAML，

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mylimit
spec:
  limits:
    - type: Pod
      max:
        cpu: 1
        memory: 2Gi
```

现在我可以将 YAML 应用于 K8s 来实现我的“使命”。

事实上，进行“解释”的一种快捷方式是添加“—递归”选项，

```
kubectl explain limits --recursive
```

这就给出了，

```
KIND:     LimitRange
VERSION:  v1DESCRIPTION:
     LimitRange sets resource usage limits for each kind of resource in a
     Namespace.FIELDS:
   apiVersion <string>
   kind <string>
   metadata <Object> ...SKIPPED MANY LINES... spec <Object>
      limits <[]Object>
         default <map[string]string>
         defaultRequest <map[string]string>
         max <map[string]string>
         maxLimitRequestRatio <map[string]string>
         min <map[string]string>
         type <string>
```

它保持缩进，使它更直观。