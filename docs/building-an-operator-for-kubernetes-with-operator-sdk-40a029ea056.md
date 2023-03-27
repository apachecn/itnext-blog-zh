# 用 operator-sdk 为 Kubernetes 构建一个操作符

> 原文：<https://itnext.io/building-an-operator-for-kubernetes-with-operator-sdk-40a029ea056?source=collection_archive---------4----------------------->

![](img/f63577133de729f6e05d3f5adf206ec1.png)

*   [本系列的第一篇文章](https://medium.com/p/b4204be9ad56)探索了样本控制器。
*   [该系列的第二篇文章](https://medium.com/p/17cbd3f07761)探索了 kubebuilder。
*   第三篇文章将探索 operator-sdk

你可以在我的新书中了解更多关于使用 Kubernetes API 和操作符的信息:[https://bit.ly/3vsuwI9](https://bit.ly/3vsuwI9)

# 操作员-sdk

现在让我们探索 operator-sdk 套件，并创建相同的 CRD 和操作员。记住，我们想要编写一个操作符，它将在集群的节点上部署一个守护进程。它将使用`DaemonSet`对象来部署这个守护进程，我们希望能够指定一个标签，以便只在标记有该标签的节点上部署守护进程。我们还希望能够指定要部署的 Docker 映像。

## 装置

按照以下步骤安装 operator-sdk:

```
go get github.com/operator-framework/operator-sdk
cd $GOPATH/src/github.com/operator-framework/operator-sdk
make dep
make install
```

## 开始一个项目

让我们创建操作符。我们需要在 GOPATH 下创建我们的项目:

```
mkdir -p $GOPATH/src/mydomain.com/mygroup2 && cd $_
```

然后启动项目:

```
operator-sdk new app-operator --api-version=mygroup2.mydomain.com/v1beta1 --kind=GenericDaemon
```

## 写一些代码

我们需要修改我们的`GenericDaemon`的结构，为我们的对象添加必要的字段。

```
// pkg/apis/mygroup2/v1beta1/types.go
[...]
type GenericDaemonSpec struct {
 // Label is the value of the 'daemon=' label to set on a node that should run the daemon
 Label string `json:"label"`
 // Image is the Docker image to run for the daemon
 Image string `json:"image"`
}
type GenericDaemonStatus struct {
 // Count is the number of nodes the daemon is deployed to
 Count int32 `json:"count"`
}
```

然后创建协调处理程序:

```
/// pkg/stub/handler.go
func (h *Handler) Handle(ctx context.Context, event sdk.Event) error {
  switch o := event.Object.(type) {
  case *v1beta1.GenericDaemon:
    genericdaemon := o
    logrus.Info("handle GenericDaemon")
    ds := newDaemonset(o)
    err := sdk.Create(ds)
    if err != nil && !errors.IsAlreadyExists(err) {
      logrus.Errorf("Failed to create daemonset : %v", err)
      return err
    }
    err = sdk.Get(ds)
    if err != nil {
      logrus.Errorf("failed to get daemonset: %v", err)
      return err
    }
    if genericdaemon.Status.Count != ds.Status.NumberReady {
      genericdaemon.Status.Count = ds.Status.NumberReady
      err = sdk.Update(genericdaemon)
      if err != nil {
        logrus.Errorf("failed to update genericdaemon status: %v", err)
        return err
      }
    }
  }
  return nil
}func newDaemonset(cr *v1beta1.GenericDaemon) *appsv1.DaemonSet {
  return &appsv1.DaemonSet{
    TypeMeta: metav1.TypeMeta{
      Kind:       "DaemonSet",
      APIVersion: "apps/v1",
    },
    ObjectMeta: metav1.ObjectMeta{
      Name:      cr.Name + "-daemonset",
      Namespace: cr.Namespace,
      OwnerReferences: []metav1.OwnerReference{
        *metav1.NewControllerRef(cr, schema.GroupVersionKind{
          Group:   v1beta1.SchemeGroupVersion.Group,
          Version: v1beta1.SchemeGroupVersion.Version,
          Kind:    "GenericDaemon",
        }),
      },
    },
    Spec: appsv1.DaemonSetSpec{
      Selector: &metav1.LabelSelector{
        MatchLabels: map[string]string{"daemonset": cr.Name + "-daemonset"},
      },
      Template: corev1.PodTemplateSpec{
        ObjectMeta: metav1.ObjectMeta{
          Labels: map[string]string{"daemonset": cr.Name + "-daemonset"},
        },
        Spec: corev1.PodSpec{
          NodeSelector: map[string]string{"daemon": cr.Spec.Label},
          Containers: []corev1.Container{
            {
              Name:  "genericdaemon",
              Image: cr.Spec.Image,
            },
          },
        },
      },
    },
  }
}
```

## 部署和播放

仅此而已！我们现在可以重新构建操作符和映像，并推送映像，最后部署我们的项目:

```
operator-sdk build mydockerid/genericdaemon2
docker push mydockerid/genericdaemon2
kubectl create -f deploy/rbac.yaml
kubectl create -f deploy/crd.yaml
kubectl create -f deploy/operator.yaml
```

此时，操作员应该正在运行:

```
$ kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
app-operator-65bf4777f7-8lx9m   1/1       Running   0          4s 
```

我们现在可以个性化生成的`GenericDaemon`样本:

```
// deploy/cr.yaml
apiVersion: mygroup2.mydomain.com/v1beta1
kind: GenericDaemon
metadata:
  name: example
spec:
  **image: httpd
  label: http**
```

并创建它:

```
$ kubectl apply -f deploy/cr.yaml$ kubectl get genericdaemon
NAME      AGE
**example**   5s$ kubectl get daemonset
NAME                **READY** [...]  NODE SELECTOR
example-daemonset   **0**     [...]  **daemon=http**$ kubectl describe genericdaemons example
[...]
Spec:
  Image:  httpd
  Label:  http
**Status:
  Count:  0**$ kubectl label nodes mynode1 daemon=http$ kubectl get daemonset
NAME                **READY** [...]  NODE SELECTOR
example-daemonset   **1**     [...]  **daemon=http**$ kubectl describe genericdaemons example
[...]
Spec:
  Image:  httpd
  Label:  http
**Status:
  Count:  1**
```

## 结论

operator-sdk 的使用与 kubebuilder 相当。它在原生 Kubernetes API 的基础上引入了一个新的 API，如果您习惯了这个 API，可能会感到困惑，但是它简化了代码和概念。

这是一个预 alpha 版本，缺少一些为我们的操作代码生成的测试。