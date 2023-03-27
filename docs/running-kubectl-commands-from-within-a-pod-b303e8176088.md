# 从 Pod 中运行 kubectl 命令

> 原文：<https://itnext.io/running-kubectl-commands-from-within-a-pod-b303e8176088?source=collection_archive---------0----------------------->

*镜像自*[https://trstringer.com/kubectl-from-within-pod/](https://trstringer.com/kubectl-from-within-pod/)。

通常在内部 pod 中使用来自*的 Kubernetes 资源。该平台显然是一个很好的调度器和指挥器，但是您可能需要定制逻辑来从 Kubernetes 资源中做出决策，或者为 Kubernetes 资源做出决策。*

这篇文章将使用一个简单的例子。假设您需要列出所有的 pods，但是您需要从集群中的 pods 完成这项工作(可能您必须根据资源的状态以编程方式做出决策)。这是一个简单的 docker 文件，用于示例图像…

# Dockerfile 文件

```
FROM debian:busterRUN apt update && \
      apt install -y curl && \
      curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl && \
      chmod +x ./kubectl && \
      mv ./kubectl /usr/local/bin/kubectlCMD kubectl get po
```

这是一种创建包含`kubectl` bin 的 docker 图像的方法。最后，从这个映像创建的任何容器都会运行`kubectl get po`。

您的直觉可能是创建一个具有以下配置的 pod

# pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: internal-kubectl
spec:
  containers:
    - name: internal-kubectl
      image: trstringer/internal-kubectl:latest
```

尝试运行这个 pod ( `$ kubectl apply -f pod.yaml`)，您会在日志中看到以下错误…

> 来自服务器的错误(禁止):pods 被禁止:用户" system:service account:default:default "无法在命名空间" default "中列出 API 组""中的资源" pods "

除非另有说明(如上所述)，否则 pods 将在`default`服务帐户下运行，这对于每个名称空间都是现成的。从错误消息中可以看出，`default`没有列出 pod 的正确权限。

阻止我们的潜在力量是 [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) ，我们需要正确配置它。告诉 Kubernetes 我们希望这个 pod 有一个可以列出 pod 的身份的方法是通过一些不同资源的组合…

# service-account.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: internal-kubectl
```

我们希望分配给 pod 的身份对象将是一个服务帐户。但是它本身没有权限。这就是角色出现的地方。

# role.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: modify-pods
rules:
  - apiGroups: [""]
    resources:
      - pods
    verbs:
      - get
      - list
      - delete
```

上面的角色指定了我们希望能够获取、列出和删除窗格。但是，我们需要一种方法将我们的新服务客户与我们的新角色联系起来。角色绑定是这方面的桥梁…

# 角色绑定. yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: modify-pods-to-sa
subjects:
  - kind: ServiceAccount
    name: internal-kubectl
roleRef:
  kind: Role
  name: modify-pods
  apiGroup: rbac.authorization.k8s.io
```

这个角色绑定将我们的服务帐户连接到拥有我们需要的权限的角色。现在，我们只需修改 pod 配置，以包括服务帐户…

# pod.yaml(新)

```
apiVersion: v1
kind: Pod
metadata:
  name: internal-kubectl
spec:
  serviceAccountName: internal-kubectl
  containers:
    - name: internal-kubectl
      image: trstringer/internal-kubectl:latest
```

通过指定`spec.serviceAccountName`，这将我们从使用`default`服务帐户更改为具有正确权限的新帐户。运行我们的新 pod，我们应该会看到正确的输出…

```
$ kubectl logs internal-kubectl
NAME               READY   STATUS    RESTARTS   AGE
internal-kubectl   1/1     Running   1          5s
```

尽情享受吧！