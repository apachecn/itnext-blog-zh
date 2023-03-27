# 用 kubebuilder 为 Kubernetes 构建一个操作符

> 原文：<https://itnext.io/building-an-operator-for-kubernetes-with-kubebuilder-17cbd3f07761?source=collection_archive---------2----------------------->

![](img/2f0c9ac75e5c30f5b357a447c17de201.png)

*   [本系列的第一篇文章](https://medium.com/p/b4204be9ad56)探索了样本控制器。
*   在本系列的第二篇文章中，我们将探索 kubebuilder。
*   [在本系列的第三篇文章](https://medium.com/p/40a029ea056)中，我们将探索 operator-sdk。

你可以在我的新书中了解更多关于使用 Kubernetes API 和操作符的信息:[https://bit.ly/3vsuwI9](https://bit.ly/3vsuwI9)

你也可以看看我在 2018 年伦敦 Velocity Conf 上关于这个主题的演讲:【https://youtu.be/Fp0QUf0Bwm0 

# 库伯 builder

现在让我们探索 kubebuilder 套件并创建相同的 CRD 和操作符。记住，我们想要编写一个操作符，它将在集群的节点上部署一个守护进程。它将使用`DaemonSet`对象来部署这个守护进程，我们希望能够指定一个标签，以便只在标记有该标签的节点上部署守护进程。我们还希望能够指定要部署的 Docker 映像。

## 开始一个项目

我们首先需要安装一些工具准备好:`kubebuilder`本身、`dep`和`kustomize`。安装这些工具的所有信息可以在[http://book.kubebuilder.io/quick-start.html](http://book.kubebuilder.io/quick-start.html)找到

让我们创建操作符。我们需要在 GOPATH 下创建我们的项目:

```
mkdir -p $GOPATH/src/mydomain.com/mygroup && cd $_
```

然后启动项目:

```
kubebuilder init --domain mydomain.com
```

并在提示运行`dep ensure`时回复`y`。

最后创造了 CRD:

```
kubebuilder create api --group mygroup --version v1beta1 --kind GenericDaemon
```

并在提示创建资源和控制器时回复`y`。

`kubebuilder`为我们创建了 API 的源代码，以在`pkg/apis/mygroup/v1beta1`下访问我们的 CRD。您可以看到创建的文件与我们之前在 sample-controller 中编辑的文件相似。

## 写一些代码

我们需要修改我们的`GenericDaemon`的结构，为我们的对象添加必要的字段。不要忘记记录字段，这样文档生成器可以创建一个好的文档:

```
// pkg/apis/mygroup/v1beta1/genericdaemon_types.go
[...]
// GenericDaemonSpec defines the desired state of GenericDaemon
type GenericDaemonSpec struct {
  // Label is the value of the 'daemon=' label to set on a node that should run the daemon
  **Label string `json:"label"`** // Image is the Docker image to run for the daemon
 **Image string `json:"image"`**
}// GenericDaemonStatus defines the observed state of GenericDaemon
type GenericDaemonStatus struct {
  // Count is the number of nodes the daemon is deployed to
  **Count int32 `json:"count"`**
}
[...]
```

然后让我们遵循`genericdaemon_controller.go`文件中的 TODO 指令。首先在`add`函数中，让我们听听`DaemonSet`而不是`Deployment`:

```
// pkg/controller/genericdaemon/genericdaemon_controller.go
func add(mgr manager.Manager, r reconcile.Reconciler) error {
  [...]
  // watch a Daemonset created by GenericDaemon
  err = c.Watch(&source.Kind{Type: &appsv1.DaemonSet{}}, &handler.EnqueueRequestForOwner{
    IsController: true,
    OwnerType:    &mygroupv1beta1.GenericDaemon{},
  })
  [...]
}
```

其次，我们来写`Reconcile`函数的代码。我们的 CRD 的具体部分以粗体显示:

```
// pkg/controller/genericdaemon/genericdaemon_controller.go
// Automatically generate RBAC rules to allow the Controller to read and write Deployments
// +kubebuilder:rbac:groups=apps,resources=**daemonsets**,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=mygroup.mydomain.com,resources=genericdaemons,verbs=get;list;watch;create;update;patch;delete
func (r *ReconcileGenericDaemon) Reconcile(request reconcile.Request) (reconcile.Result, error) {
  // Fetch the GenericDaemon instance
  instance := &mygroupv1beta1.GenericDaemon{}
  err := r.Get(context.TODO(), request.NamespacedName, instance)
  if err != nil {
    if errors.IsNotFound(err) {
      // Object not found, return.
      // Created objects are automatically garbage collected.
      // For additional cleanup logic use finalizers.
      return reconcile.Result{}, nil
    }
    // Error reading the object - requeue the request.
    return reconcile.Result{}, err
  }// Define the desired Daemonset object
  daemonset := &appsv1.DaemonSet{
    ObjectMeta: metav1.ObjectMeta{
      Name:      instance.Name + "-daemonset",
      Namespace: instance.Namespace,
    },
    Spec: appsv1.DaemonSetSpec{
      Selector: &metav1.LabelSelector{
        MatchLabels: map[string]string{"daemonset": instance.Name + "-daemonset"},
      },
      Template: corev1.PodTemplateSpec{
        ObjectMeta: metav1.ObjectMeta{
          Labels: map[string]string{"daemonset": instance.Name + "-daemonset"},
        },
        Spec: corev1.PodSpec{
          **NodeSelector: map[string]string{"daemon": instance.Spec.Label},
**          Containers: []corev1.Container{
            {
              Name:  "genericdaemon",
 **Image: instance.Spec.Image,**            },
          },
        },
      },
    },
  }
  if err := controllerutil.SetControllerReference(instance, daemonset, r.scheme); err != nil {
    return reconcile.Result{}, err
  }// Check if the Daemonset already exists
  found := &appsv1.DaemonSet{}
  err = r.Get(context.TODO(), types.NamespacedName{Name: daemonset.Name, Namespace: daemonset.Namespace}, found)
  if err != nil && errors.IsNotFound(err) {
    log.Printf("Creating Daemonset %s/%s\n", daemonset.Namespace, daemonset.Name)
    err = r.Create(context.TODO(), daemonset)
    if err != nil {
      return reconcile.Result{}, err
    }
  } else if err != nil {
    return reconcile.Result{}, err
  }
 **// Get the number of Ready daemonsets and set the Count status**  **if found.Status.NumberReady != instance.Status.Count {
    log.Printf("Updating Status %s/%s\n", isntance.Namespace, instance.Name)
    instance.Status.Count = found.Status.NumberReady
    err = r.Update(context.TODO(), instance)
    if err != nil {
      return reconcile.Result{}, err
    }
  }**
  // Update the found object and write the result back if there are any changes
  if !reflect.DeepEqual(daemonset.Spec, found.Spec) {
    found.Spec = daemonset.Spec
    log.Printf("Updating Daemonset %s/%s\n", daemonset.Namespace, daemonset.Name)
    err = r.Update(context.TODO(), found)
    if err != nil {
      return reconcile.Result{}, err
    }
  }
  return reconcile.Result{}, nil
}
```

我们还必须修改泛型守护进程的测试。重要的部分是正确地创建一个`GenericDaemon`实例:

```
// pkg/controller/genericdaemon/genericdaemon_controller_test.go
func TestReconcile(t *testing.T) {
  g := gomega.NewGomegaWithT(t)
 **instance := &mygroupv1beta1.GenericDaemon{
    ObjectMeta: metav1.ObjectMeta{Name: "foo", Namespace: "default"},
    Spec: mygroupv1beta1.GenericDaemonSpec{
      Label: "http",
      Image: "mydockerid/myimage",
    },
  }**  [...]
```

## 部署和播放

仅此而已！我们现在可以重新构建 API 和控制器，构建和推送 Docker 映像，并部署我们的项目:

```
make
make docker-build IMG=mydockerid/genericdaemon
make docker-push IMG=mydockerid/genericdaemon
make deploy
```

如果您检查`make deploy`命令的输出，您可以看到该命令为操作员部署了 CRD、RBAC 角色和角色绑定，以访问必要的对象，为操作员创建了名称空间，为操作员创建了服务和状态集。

此时，操作员应该正在运行:

```
$ kubectl get pods --namespace=mygroup-system 
NAME                           READY     STATUS    RESTARTS   AGE
mygroup-controller-manager-0   1/1       Running   0          7s
$ kubectl logs mygroup-controller-manager-0 --namespace=mygroup-system
2018/07/22 09:06:32 Registering Components.
2018/07/22 09:06:32 Starting the Cmd.
```

我们现在可以个性化生成的`GenericDaemon`样本:

```
// config/samples/mygroup_v1beta1_genericdaemon.yaml
apiVersion: mygroup.mydomain.com/v1beta1
kind: GenericDaemon
metadata:
  labels:
    controller-tools.k8s.io: "1.0"
  name: genericdaemon-sample
spec:
  **image: httpd
  label: http**
```

并创建它:

```
$ kubectl apply -f config/samples/mygroup_v1beta1_genericdaemon.yaml$ kubectl get genericdaemon
NAME                   AGE
**genericdaemon-sample**   5s$ kubectl get daemonset
NAME                             **READY** [...]  NODE SELECTOR
genericdaemon-sample-daemonset   **0**     [...]  **daemon=http**$ kubectl describe genericdaemons genericdaemon-sample
[...]
Spec:
  Image:  httpd
  Label:  http
**Status:
  Count:  0**$ kubectl label nodes mynode1 daemon=http$ kubectl get daemonset
NAME                             **READY** [...]  NODE SELECTOR
genericdaemon-sample-daemonset   **1**     [...]  **daemon=http**$ kubectl describe genericdaemons genericdaemon-sample
[...]
Spec:
  Image:  httpd
  Label:  http
**Status:
  Count:  1**
```

## 结论

与前一篇文章中探讨的样本控制器相比，kubebuilder 为我们提供了以下优势:

*   kubebuilder 仍然使用基本的 Kubernetes API:如果您已经使用过 sample-controller 或者您自己的控制器或操作符的实现，您将会认识到相同的模式，
*   可以在几个版本中创建几个 API，
*   有些测试是为我们编写的，
*   角色和角色绑定是为我们编写的(您必须用一些+kubebuilder:rbac 标记来修饰源代码)，
*   docker 映像的构建和部署由我们来处理。

接下来，[探索运营商——SDK](https://medium.com/p/40a029ea056)。