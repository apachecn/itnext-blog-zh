# OpenShift 的 Airgap 镜像

> 原文：<https://itnext.io/airgap-image-mirroring-for-openshift-5dc56b464c64?source=collection_archive---------2----------------------->

## 在气隙开放移位环境中的 Rook Ceph 算子设置

![](img/1f135b982f054b07a1cb85b560bff050.png)

在 airgap 环境中，获取容器图像的挑战始终存在。您可以直接填充到[本地容器存储](https://medium.com/@zhimin.wen/loading-of-oci-images-in-an-airgap-environment-a9c9ec01cbfe)中，或者更完整的解决方案是建立一个镜像注册表，让 OpenShift 从那里获取图像。

本文以 Rook Ceph 算子为例，记录了气隙镜像的步骤。

## 设置本地注册表

本地注册表用于托管从互联网下载的所有图像，供 airgap OpenShift 使用。很可能您已经准备好了这个用来设置 OpenShift 的注册表。您可以参考[本](https://medium.com/@zhimin.wen/airgap-disconnected-installation-of-openshift-4-2-abd7794fc7fe)了解当地注册表的更多详情。

如果你有 OpenShift 注册表，你可以把它用于本地注册表*。然而，由于我设置 Rook Ceph 是为了满足注册表的持久容量需求，这是一个先有鸡还是先有蛋的问题，所以我将使用在 OpenShift 之外运行的本地注册表。

** OpenShift 注册表实际上是基于 docker 注册表 API V2 的。设置好车之后，我用 Skopeo 工具测试了镜像图像。我没有一个问题来推动一个单拱图像。然而，如果是多拱图像，我在使用 Skopeo 复制图像时会遇到 500 HTTP 错误。测试在 OCP 4.5.15 上进行。*

## **使用 Skopeo 下载图像文件**

确定所需的图像。然后在面向互联网的服务器上，使用 Skopeo 将图像镜像到一个目录中。

对于当前车，需要以下图像。

```
docker.io/ceph/ceph:v15.2.4
docker.io/rook/ceph:master
quay.io/cephcsi/cephcsi:v3.1.1
k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.0.1
k8s.gcr.io/sig-storage/csi-resizer:v1.0.0
k8s.gcr.io/sig-storage/csi-provisioner:v2.0.0
k8s.gcr.io/sig-storage/csi-snapshotter:v3.0.0
k8s.gcr.io/sig-storage/csi-attacher:v3.0.0
```

对于其中的每一个，请执行以下操作:

```
mkdir -p $HOME/rookImages/ceph-v15.2.4skopeo copy --all docker://docker.io/ceph/ceph:v15.2.4 dir://$HOME/rookImages/ceph-v15.2.4
```

我使用标记名作为目录名的一部分。请注意“— all”标志，如果是多拱图像，它将下载所有图像，以便匹配正确的图像摘要值。

现在我们已经下载了所有的图像，然后我们可以复制并将其传输到本地注册表中。

## 将图像加载到 airgap 本地注册表

移动到本地注册服务器(`airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000)`)，将图像加载到其中。

```
skopeo login -u admin -p xxxxx airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000
```

对每一幅图像将其加载到注册表中，例如，

```
skopeo copy --all  dir:$HOME/rookImages/ceph-v15.2.4 docker://airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/ceph/ceph:v15.2.4
```

## 创建 open shift ImageContentSourcePolicy

用`oc apply`将下面的 YAML 文件应用到 OpenShift 中，

```
apiVersion: operator.openshift.io/v1alpha1
kind: ImageContentSourcePolicy
metadata:
  name: rook-ceph-images
spec:
  repositoryDigestMirrors:
  - mirrors:
    - airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/ceph
    source: docker.io/ceph
  - mirrors:
    - airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/rook
    source: docker.io/rook
  - mirrors:
    - airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/sig-storage
    source: k8s.gcr.io/sig-storage
  - mirrors:
    - airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/cephcsi
    source: quay.io/cephcsi
```

对于每个图像存储库，图像内容源策略定义了本地注册表的镜像。例如，docker.io/ceph 报告中的原始图像将映射到 airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/ceph,，因此图像 docker.io/ceph/ceph:v15.2.4 将从 airgap-45-lb . airgap-OCP 45 . IBM cloud . io . cpak:5000/ceph:v 15 . 2 . 4 的镜像中提取

一旦应用了 YAML，OpenShift machine-config 集群操作符将更新每个节点上的 crio 注册表配置设置，并让它重新加载配置。在此过程中，节点被标记为不可调度，直到它完成此过程。

自动生成并更新的`/etc/container/registries.conf`文件如下:

```
unqualified-search-registries = ["registry.access.redhat.com", "docker.io"][[registry]]
  prefix = ""
  location = "docker.io/ceph"
  mirror-by-digest-only = true [[registry.mirror]]
    location = "airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/ceph"[[registry]]
  prefix = ""
  location = "docker.io/rook"
  mirror-by-digest-only = true [[registry.mirror]]
    location = "airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/rook"[[registry]]
  prefix = ""
  location = "k8s.gcr.io/sig-storage"
  mirror-by-digest-only = true [[registry.mirror]]
    location = "airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/sig-storage"[[registry]]
  prefix = ""
  location = "quay.io/cephcsi"
  mirror-by-digest-only = true [[registry.mirror]]
    location = "airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/cephcsi"[[registry]]
  prefix = ""
  location = "quay.io/openshift-release-dev/ocp-release"
  mirror-by-digest-only = true [[registry.mirror]]
    location = "airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/ocp4/openshift4"[[registry]]
  prefix = ""
  location = "quay.io/openshift-release-dev/ocp-v4.0-art-dev"
  mirror-by-digest-only = true [[registry.mirror]]
    location = "airgap-45-lb.airgap-ocp45.ibmcloud.io.cpak:5000/ocp4/openshift4"
```

在这个 TOML 格式文件中，为每个源注册表定义了一个指向本地注册表的镜像。除了安装过程中定义的 OCP 回购协议之外，还包括新车回购协议。

注意标志 **mirror-by-digest-only** 被设置为 true，这告诉我们图像必须被摘要值引用，只有这样，图像才会从它的镜像(本地注册表)中取出。

## 引用部署中的映像

为了部署 Rook 操作符，我们需要更新它的对象定义，以使用 SHA256 摘要格式的图像。例如，原始图像

`docker.io/rook/ceph:master`

需要更新为

`*docker.io/ceph/ceph@sha256:*637a92f1b356674a434b13299077ec40cf23f70e0b000b944f4954bdec4eaca8`

让我们使用其原始 YAML 设置部署操作符，然后更新映像，

```
kubectl -n rook-ceph set image deploy/rook-ceph-operator rook-ceph-operator=docker.io/rook/ceph@sha256:637a92f1b356674a434b13299077ec40cf23f70e0b000b944f4954bdec4eaca8
```

对于 CSI 相关的图像，我们可以修补设置图像版本的配置图，

```
kubectl -n rook-ceph patch cm/rook-ceph-operator-config --type merge -p '{"data":{"ROOK_CSI_ATTACHER_IMAGE":"k8s.gcr.io/sig-storage/csi-attacher@sha256:bc6d3ca87c62a2587470ba7adca84e9e5e7db61be1a35176084cc9ec3b4a87d5","ROOK_CSI_CEPH_IMAGE":"quay.io/cephcsi/cephcsi@sha256:afd643c2805029f14862bf8976916ce6ec894b172334eae7c4a74bd717df8f2d","ROOK_CSI_PROVISIONER_IMAGE":"k8s.gcr.io/sig-storage/csi-provisioner@sha256:271b0049ba7885e536d575a72ef56ab3b94c6989f36812d3fe4d886c4d8aed96","ROOK_CSI_REGISTRAR_IMAGE":"k8s.gcr.io/sig-storage/csi-node-driver-registrar@sha256:e07f914c32f0505e4c470a62a40ee43f84cbf8dc46ff861f31b14457ccbad108","ROOK_CSI_RESIZER_IMAGE":"k8s.gcr.io/sig-storage/csi-resizer@sha256:5a8d85cdd1c80f43fb8fe6dcde1fae707a3177aaf0a786ff4b9f6f20247ec3ff","ROOK_CSI_SNAPSHOTTER_IMAGE":"k8s.gcr.io/sig-storage/csi-snapshotter@sha256:ece29e2217e0faf87973565c062c8508c2b78039169e855032bb357acefa3c98"}}'
```

当你定义车群 CR 时，也用摘要格式更新 Ceph 图像。

现在你将有气隙车 Ceph 设置和运行。