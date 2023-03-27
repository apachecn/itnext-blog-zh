# 用 Kubernetes Pods 扩展 Jenkins 构建代理

> 原文：<https://itnext.io/scaling-jenkins-build-agents-with-kubernetes-pods-8c89f87ba0d3?source=collection_archive---------2----------------------->

## 关于跨 Kubernetes 集群扩展 Jenkins 构建作业的教程。

![](img/4bc34505fe8029e148cc421fbd881f50.png)

根据上一篇关于如何在 Kubernetes 集群中运行 Jenkins 的教程,现在是时候利用 Kubernetes 基础设施在集群中扩展构建工作了。

## 示例构建作业

首先，需要两个项目来测试设置。这些作业不会做任何有用的事情，它们只会等待 10 秒钟，然后继续。

这些工作是通过点击仪表板中的“新项目”，然后选择“自由式项目”来创建的。

![](img/612987361874ebecb9981466f25fbc36.png)

在“构建”部分，需要添加“执行 shell”构建步骤。

![](img/ae4b166475ebb210ce2b50817f57005b.png)

构建作业应该执行的命令是

```
sleep 10
```

![](img/834ed6a5dee2ba60556479edaf288c28.png)

保存后，新作业将显示在仪表板中。需要创建另一个工作，直到有两个。

当两个作业同时触发时，它们将显示在“构建执行者状态”下。

![](img/fb78d5bbb125f0ef35b5e4878c4590b5.png)

现在，这两个任务都在 pod 中执行。从长远来看，这是无法扩展的。

实际目标是在新创建的 pod 中运行正在运行的构建作业的每个实例。pods 生命周期与构建工作的生命周期紧密相关。

这意味着，当新作业被触发时，将创建一个新的 pod，该作业在该 pod 内运行，当该作业完成时，该 pod 将被删除。

为此，必须进行一些额外的配置。

## 安装 Kubernetes 插件

Kubernetes 插件是通过

*   "管理詹金斯"
*   "管理插件"

![](img/08fd91bb5929b0e9de2641d85f49b43a.png)

然后

*   "可用"
*   “Kubernetes”精选
*   按下“安装但不重启”

![](img/c00f87167c08930fd88df3fe78309eb1.png)

然后所有东西都将被安装:

![](img/d0154f048241a09416406c00ab70bb3e.png)

## 添加 Kubernetes 的秘密

两个 Kubernetes-需要秘密。

第一个是`jenkins-robot.yaml`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-robot
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-robot
  namespace: jenkins
  labels:
    "app.kubernetes.io/name": 'jenkins'
rules:
- apiGroups: [""]
  resources: ["pods", "pods/exec", "pods/log", "persistentvolumeclaims", "events"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods", "pods/exec", "persistentvolumeclaims", "events"]
  verbs: ["create", "apply", "delete", "deletecollection", "patch", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-robot-binding
  namespace: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins-robot
subjects:
- kind: ServiceAccount
  name: jenkins-robot
  namespace: jenkins
```

第二个秘密是`jenkins-robot-global.yaml`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-robot-global
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-robot-global
  namespace: jenkins
  labels:
    "app.kubernetes.io/name": 'jenkins'
rules:
- apiGroups: [""]
  resources: ["pods", "pods/exec", "pods/log", "persistentvolumeclaims"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods", "pods/exec", "persistentvolumeclaims"]
  verbs: ["create", "apply", "delete", "deletecollection", "patch", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-robot-global-binding
  namespace: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-robot-global
subjects:
- kind: ServiceAccount
  name: jenkins-robot-global
  namespace: jenkins
```

这两个秘密都适用于:

```
kubectl apply -n jenkins -f jenkins-robot.yaml
kubectl apply -n jenkins -f jenkins-robot-global.yaml
```

这些秘密需要添加到詹金斯。

第一个秘密通过以下方式获取:

```
kubectl -n jenkins get serviceaccount jenkins-robot --template='{{range .secrets}}{{.name}}{{"\n"}}{{end}}' | xargs -n 1 kubectl -n jenkins get secret --template='{{ if .data.token }}{{ .data.token }}{{end}}' | head -n 1 | base64 -d -
```

这需要复制到剪贴板，以便在下一步中粘贴。

在 Jenkins 中，凭据是通过以下方式创建的

*   管理詹金斯
*   管理凭据

![](img/abdc8e11b217718a2c6fce7a03185b53.png)

然后，按下

*   新项目

![](img/de0bef1bda237419d1b254e134ea3212.png)

另外两个链接:

*   全局凭据(无限制)

![](img/04cb543257478ea3dfd265321504b7ea.png)

然后

*   添加凭据

![](img/ea6f977f32def7af0768839a4059d368.png)

必须添加以下值:

*   种类:秘密文本
*   范围:全球
*   秘密:
*   ID:詹金斯-机器人

![](img/2659e79d9b0dfaec79486c7d425eab7d.png)

秘密`Jenkins-Robot-Global`也需要这样做，可以通过以下方式检索:

```
kubectl -n jenkins get serviceaccount jenkins-robot-global --template='{{range .secrets}}{{.name}}{{"\n"}}{{end}}' | xargs -n 1 kubectl -n jenkins get secret --template='{{ if .data.token }}{{ .data.token }}{{end}}' | head -n 1 | base64 -d -
```

![](img/af2099404b4aa3310f1f6fdaae17631d.png)

最后，两个证书都出现在詹金斯:

![](img/8b87f4a010275f98167bf4c67a6b9129.png)

## 将 Kubernetes 云添加到 Jenkins

Jenkins 需要知道有一个可以使用的 Kubernetes 集群。下面解释了如何进行配置。

通过以下方式添加 Kubernetes 集群:

*   管理詹金斯
*   管理节点和云
*   配置云
*   添加一个新的云:Kubernetes

![](img/a859abb3e2abbe4f1d385663303b6487.png)

在“Kubernetes Cloud Details”下，需要添加以下值:

*   名称:Kubernetes
*   Kubernetes 网址 [https://:6443](https://:6443)
*   Kubernetes 名称空间:jenkins
*   证书:詹金斯-机器人
*   詹金斯隧道::50000

具有角色主机的群集节点的 IP 地址可以使用以下命令检索:

```
kubectl get nodes -o wide | grep master
```

Jenkins 隧道的 IP 是名为`jenkins-jnlp`的服务的 IP，可以通过以下方式检索:

```
kubectl get service jenkins-jnlp -n jenkins
```

![](img/f435f76693d4b80c7b64b648a13cf241.png)

其余的值保留默认值:

![](img/1e95600c8e15c5dfdd6d286b61ded1a2.png)

可以通过按下“测试连接”按钮来测试连接。

## 添加窗格和容器模板

Pod 模板需要填写如下:

Pod 模板:

*   名称:jnlp
*   用法:尽可能多地使用该节点

![](img/94373ea63d575bf6444050cd69017253.png)

对于本教程，这些代理的容器基于`jenkins/jnlp-agent-alpine`

*   名称:jnlp
*   Docker 图片:詹金斯/jnlp-代理-阿尔卑斯

![](img/9194ccd9588bf16c257c394a78be1de0.png)

需要添加“主机路径”卷:

![](img/05654021e7f29e56d54aef553468ac56.png)

*   主机路径:/var/run/docker.sock
*   挂载路径:/var/run/docker.sock

![](img/0d80af6c1506de4ab0cfa85b5e87b494.png)

还需要一个“空目录”卷

![](img/4ee6164021f8bf721eb14ad8d54342a0.png)

*   挂载路径:/home/jenkins/agent

![](img/d1d8641c6f31dbacbd559a455a10c87f.png)

按下“保存”按钮即可完成配置。

最后，Jenkins 需要配置为只有代理节点应该运行构建作业。这是通过以下方式实现的:

*   管理詹金斯
*   管理节点和云
*   掌握
*   安装ˌ使成形

需要提供以下值:

*   执行人数量:0
*   用法:仅构建标签表达式与此节点匹配的作业

![](img/431b992f7ada7e704ed56223a283ff85.png)

同样，完成后，需要按下“保存”按钮。

## 测试 Pod 构建代理

当两个构建作业同时被触发时，可以观察到在“构建执行者状态”下启动了两个 pod:

![](img/244f34df5d419bb303c2ac9b002d6b13.png)

然后，当 pod 运行时，作业就在其中:

![](img/4bc34505fe8029e148cc421fbd881f50.png)

感谢阅读！喜欢吗？ [**给我买杯咖啡！**](https://www.buymeacoffee.com/twissmueller)

## 资源

*   [裸机 ARM 上的可扩展 Jenkins CI/CD Kubernetes](https://otterwerks.medium.com/scalable-jenkins-ci-cd-on-bare-metal-arm-kubernetes-c71dfe5645f8)
*   [Kubernetes 上 Jenkins slave 的连接中断](https://stackoverflow.com/a/42124959/1065468)