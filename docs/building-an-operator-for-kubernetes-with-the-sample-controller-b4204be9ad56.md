# 用样本控制器为 Kubernetes 构建一个操作符

> 原文：<https://itnext.io/building-an-operator-for-kubernetes-with-the-sample-controller-b4204be9ad56?source=collection_archive---------1----------------------->

![](img/c43be57bb1ba3febfbb77aee155f4289.png)

在这一系列文章中，我们将探索一些工具来为 Kubernetes 创建一个操作符。

*   这第一篇文章将探索样本控制器。
*   [本系列的第二篇文章](https://medium.com/p/17cbd3f07761)将探索 kubebuilder。
*   [本系列的第三篇文章](https://medium.com/p/40a029ea056)将探讨 operator-sdk。

你可以在我的新书中了解更多关于使用 Kubernetes API 和操作符的信息:[https://bit.ly/3vsuwI9](https://bit.ly/3vsuwI9)

# 样品控制器

我们将用来实验创建操作符的第一个工具是样本控制器，你可以在这里找到:[https://github.com/kubernetes/sample-controller](https://github.com/kubernetes/sample-controller)。

这个项目为`Foo`类型实现了一个简单的操作符，当我们创建一个定制对象`foo`时，它将创建一个带有一些公共 Docker 映像和特定数量副本的部署。

要安装和构建它，一定要定义 GOPATH，然后运行:

```
go get github.com/kubernetes/sample-controller
cd $GOPATH/src/k8s.io/sample-controller
go build -o ctrl .
```

然后，我们可以为`Foo`类型创建定制资源定义，文件位于`artifacts/examples`子目录中:

```
kubectl apply -f artifacts/examples/crd-validation.yaml
```

最后运行控制器:

```
./ctrl -kubeconfig ~/.kube/config  -logtostderr=true
```

现在从另一个终端，我们可以操纵`Foo`对象，看看控制器会发生什么:

```
$ kubectl apply -f artifacts/examples/example-foo.yaml
$ kubectl get pods
NAME                           READY  STATUS    RESTARTS   AGE
example-foo-6cbc69bf5d-j8lhx   1/1    Running   0          18s$ kubectl delete -f artifacts/examples/example-foo.yaml
$ kubectl get pods
NAME                           READY  STATUS        RESTARTS   AGE
example-foo-6cbc69bf5d-j8lhx   0/1    Terminating   0          38s
```

> 在编写和使用 Kubernetes 1.11.0 时，控制器在创建部署后更新`foo`对象的状态时将进入无限循环:在`updateFooStatus`函数中，您必须通过调用`UpdateStatus(fooCopy)`来更改对`Update(fooCopy)`的调用。

到目前为止，控制器完成了工作:当我们创建一个`foo`对象时，它创建一个部署，当我们删除对象时，它停止部署。

现在，我们可以进一步修改 CRD 和控制器，以使用我们自己的自定义资源定义。

## 调整样品控制器

假设我们的目标是编写一个在集群节点上部署守护进程的操作符。它将使用`DaemonSet`对象来部署这个守护进程，我们希望能够指定一个标签，以便只在用这个标签标记的节点上部署守护进程。我们还希望能够指定要部署的 Docker 映像，而不是像 sample-controller 那样的静态映像。

让我们首先为我们的`GenericDaemon`类型创建定制资源定义:

```
// artifacts/generic-daemon/crd.yaml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: genericdaemons.mydomain.com
spec:
  group: mydomain.com
  version: v1beta1
  names:
    kind: Genericdaemon
    plural: genericdaemons
  scope: Namespaced
  validation:
    openAPIV3Schema:
      properties:
        spec:
          properties:
            label:
              type: string
            image:
              type: string
          required:
            - image
```

第一个要部署的守护程序示例:

```
// artifacts/generic-daemon/syslog.yaml
apiVersion: mydomain.com/v1beta1
kind: Genericdaemon
metadata:
  name: syslog
spec:
  label: logs
  image: mbessler/syslogdocker
```

我们现在必须为 API 构建 go 文件，这是从我们的操作符访问这个新的定制资源定义所必需的。为此，让我们创建一个新的目录`pkg/apis/genericdaemon`，在其中我们将复制在`pkg/apis/samplecontroller`中找到的文件(但是是`zz_generated.deepcopy.go`中的文件)

```
$ tree pkg/apis/genericdaemon/
pkg/apis/genericdaemon/
├── register.go
└── v1beta1
    ├── doc.go
    ├── register.go
    └── types.go
```

并修改其内容(更改的部分以粗体显示):

```
////////////////
// register.go
////////////////package **genericdaemon**const (
 GroupName = "**mydomain.com**"
)/////////////////////
// v1beta1/doc.go
/////////////////////// +k8s:deepcopy-gen=package// Package **v1beta1** is the **v1beta1** version of the API.
// +groupName=**mydomain.com**
package **v1beta1**/////////////////////////
// v1beta1/register.go
/////////////////////////package **v1beta1**import (
 metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
 "k8s.io/apimachinery/pkg/runtime"
 "k8s.io/apimachinery/pkg/runtime/schema"**genericdaemon** "k8s.io/sample-controller/pkg/apis/**genericdaemon**"
)// SchemeGroupVersion is group version used to register these objects
var SchemeGroupVersion = schema.GroupVersion{Group: **genericdaemon**.GroupName, Version: "**v1beta1**"}// Kind takes an unqualified kind and returns back a Group qualified GroupKind
func Kind(kind string) schema.GroupKind {
 return SchemeGroupVersion.WithKind(kind).GroupKind()
}// Resource takes an unqualified resource and returns a Group qualified GroupResource
func Resource(resource string) schema.GroupResource {
 return SchemeGroupVersion.WithResource(resource).GroupResource()
}var (
 SchemeBuilder = runtime.NewSchemeBuilder(addKnownTypes)
 AddToScheme   = SchemeBuilder.AddToScheme
)// Adds the list of known types to Scheme.
func addKnownTypes(scheme *runtime.Scheme) error {
 scheme.AddKnownTypes(SchemeGroupVersion,
  &**Genericdaemon**{},
  &**GenericdaemonList**{},
 )
 metav1.AddToGroupVersion(scheme, SchemeGroupVersion)
 return nil
}//////////////////////
// v1beta1/types.go
//////////////////////package **v1beta1**import (
 metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)// +genclient
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object// **Genericdaemon** is a specification for a **Generic Daemon** resource
type **Genericdaemon** struct {
 metav1.TypeMeta   `json:",inline"`
 metav1.ObjectMeta `json:"metadata,omitempty"` Spec   **GenericdaemonSpec**   `json:"spec"`
 Status **GenericdaemonStatus** `json:"status"`
}// **GenericDaemonSpec** is the spec for a **GenericDaemon** resource
type **GenericdaemonSpec** struct {
 **Label string `json:"label"`
 Image string `json:"image"`**
}// **GenericDaemonStatus** is the status for a **GenericDaemon** resource
type **GenericdaemonStatus** struct {
 **Installed int32 `json:"installed"`** }// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object// **GenericDaemonList** is a list of **GenericDaemon** resources
type **GenericdaemonList** struct {
 metav1.TypeMeta `json:",inline"`
 metav1.ListMeta `json:"metadata"`Items []**Genericdaemon** `json:"items"`
}
```

一个脚本`hack/update-codegen.sh`可用于围绕我们用这些先前的文件定义的新的定制资源定义生成代码。我们必须修改这个脚本来为我们的新 CRD 生成文件:

```
// hack/update-codegen.sh
#!/usr/bin/env bashset -o errexit
set -o nounset
set -o pipefailSCRIPT_ROOT=$(dirname ${BASH_SOURCE})/..
CODEGEN_PKG=${CODEGEN_PKG:-$(cd ${SCRIPT_ROOT}; ls -d -1 ./vendor/k8s.io/code-generator 2>/dev/null || echo ../code-generator)}# generate the code with:
# --output-base    because this script should also be able to run inside the vendor dir of
#                  k8s.io/kubernetes. The output-base is needed for the generators to output into the vendor dir
#                  instead of the $GOPATH directly. For normal projects this can be dropped.
${CODEGEN_PKG}/generate-groups.sh "deepcopy,client,informer,lister" \
  k8s.io/sample-controller/pkg/client k8s.io/sample-controller/pkg/apis \
  **genericdaemon:v1beta1** \
  --output-base "$(dirname ${BASH_SOURCE})/../../.." \
  --go-header-file ${SCRIPT_ROOT}/hack/boilerplate.go.txt
```

然后执行它:

```
$ ./hack/update-codegen.sh 
Generating deepcopy funcs
Generating clientset for genericdaemon:v1beta1 at k8s.io/sample-controller/pkg/client/clientset
Generating listers for genericdaemon:v1beta1 at k8s.io/sample-controller/pkg/client/listers
Generating informers for genericdaemon:v1beta1 at k8s.io/sample-controller/pkg/client/informers
```

我们现在可以调整我们的操作符。首先，我们必须用`Genericdaemon`类型改变所有对先前`Foo`类型的引用。其次，当创建一个新的通用守护进程时，我们必须创建一个守护进程集，而不是一个部署(这里没有显示)。

## 将操作员部署到 Kubernetes 集群

当我们根据需要修改完 sample-controller 之后，我们需要将它部署到 kubernetes 集群中。事实上，此时，我们已经通过使用我们的凭证从我们的开发系统运行它来测试它。

下面是一个简单的 Docker 文件，用于使用操作符构建 Docker 映像(要构建映像，您必须删除原始 sample-controller 中的所有代码):

```
FROM golangRUN mkdir -p /go/src/k8s.io/sample-controller
ADD . /go/src/k8s.io/sample-controller
WORKDIR /goRUN go get ./...
RUN go install -v ./...CMD ["/go/bin/sample-controller"]
```

我们现在可以构建映像并将其推送到 Docker Hub:

```
docker build . -t mydockerid/genericdaemon
docker push mydockerid/genericdaemon
```

最后，使用这个新映像开始部署:

```
// deploy.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: sample-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample
  template:
    metadata:
      labels:
        app: sample
    spec:
      containers:
      - name: sample
        image: "mydockerid/genericdaemon:latest"
```

和`kubectl apply -f deploy.yaml`

操作员现在正在运行，但是如果我们检查 pod 的日志，我们可以看到授权存在问题；pod 没有对不同资源的访问权限:

```
$ kubectl logs sample-controller-66b79c7d5f-2qnft
E0721 14:34:50.499584       1 reflector.go:134] k8s.io/sample-controller/pkg/client/informers/externalversions/factory.go:117: Failed to list *v1beta1.Genericdaemon: genericdaemons.mydomain.com is forbidden: User "system:serviceaccount:default:default" cannot list genericdaemons.mydomain.com at the cluster scope
E0721 14:34:50.500385       1 reflector.go:134] k8s.io/client-go/informers/factory.go:131: Failed to list *v1.DaemonSet: daemonsets.apps is forbidden: User "system:serviceaccount:default:default" cannot list daemonsets.apps at the cluster scope
[...]
```

我们需要创建一个`ClusterRole`和一个`ClusterRoleBinding`来给操作员必要的特权:

```
// rbac_role.yaml
kind: ClusterRole
metadata:
  name: operator-role
rules:
- apiGroups:
  - apps
  resources:
  - daemonsets
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - patch
  - delete
- apiGroups:
  - mydomain.com
  resources:
  - genericdaemons
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - patch
  - delete// rbac_role_binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: operator-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: operator-role
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
```

并部署它:

```
kubectl apply -f rbac_role.yaml
kubectl delete -f deploy.yaml
kubectl apply -f deploy.yaml
```

现在，您的操作员应该被部署到您的 Kubernetes 集群中并处于活动状态。

# 下一步是什么

在下一篇文章中，我们将探索 kubebuilder，它将为我们提供自动化所有这些步骤的工具。