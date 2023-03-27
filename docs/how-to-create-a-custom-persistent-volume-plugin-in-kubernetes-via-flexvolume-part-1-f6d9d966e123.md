# 如何通过 FlexVolume 在 Kubernetes 中创建自定义持久卷插件(第 1 部分)

> 原文：<https://itnext.io/how-to-create-a-custom-persistent-volume-plugin-in-kubernetes-via-flexvolume-part-1-f6d9d966e123?source=collection_archive---------0----------------------->

> 这是一个两部分的系列。 [**第一部分**](/how-to-create-a-custom-persistent-volume-plugin-in-kubernetes-via-flexvolume-part-1-f6d9d966e123) :司机。 [**第二部**](https://medium.com/@liangrog/how-to-create-a-custom-persistent-volume-plugin-in-kubernetes-via-flexvolume-part-2-c6390f9f94e0) :置备者

![](img/0411a6078084f9056a520a54273b2ca8.png)

Kubernetes 提供了丰富的内置[持久卷类型](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)，使用这些类型，您可以轻松地与您的物理或基于云的持久卷挂钩。然而，有时候持久卷(PV)的类型不是 Kubernetes 中内置的，或者 PV 的生命周期管理(如提供卷)不是标准的。例如，您必须对第三方供应商执行 API 调用来配置卷，而 API 调用会返回 Kubernetes 的相应 PV 驱动程序无法处理的安装规格。

从 Kubernetes 1.8 开始，它已经停止接受“树内”卷插件。对于这个可能暴露自定义存储系统的问题，建议采用两种解决方案:

1.  [**【集装箱存储接口】**](https://kubernetes.io/docs/concepts/storage/volumes/#csi)**【1】**。这种卷类型在 kubernetes 1.9 中被称为 alpha，在 1.10 中被称为 beta。
2.  [**flex volume**](https://kubernetes.io/docs/concepts/storage/volumes/#flexvolume)**【2】**。这种卷类型是从 kubernetes 1.2 开始引入的

在本文中，我将带您完成使用 **FlexVolume** 编写您自己的定制持久卷插件的步骤。

FlexVolume 工作的基本要素是一个驱动程序(可执行文件),该驱动程序可以执行附加/分离操作，在主机上装载/卸载永久卷。

```
HOST+------------------+
|                  |
|                  |
|                  |
|                  |
|     Kubelet      |
|        +         |
|        |         |
|        |         |
+------------------+                     +-----------------------+
|        |         |                     |                       |
|        |         +----------------->   |      Third party      |
|        v         |                     |      persistent       |
|  Custom driver   |  <------------------+      volume           |
|                  |                     |                       |
+------------------+   Attach/Detach     +-----------------------+
                       Mount/Un-mount
```

一旦你在你的主机上安装了驱动程序，你就可以**静态地**配置你的 PVs。

如果您想要动态配置 PVs，会发生什么情况？为了回答这个问题，在第 2 部分文章的后面，我还将指导您如何使用[外部存储](https://github.com/kubernetes-incubator/external-storage)【4】库来创建动态供应器。

静态和动态供应器之间的差异可以在这里找到[。](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#provisioning)

## 司机

Flexvolume 驱动程序可以用任何语言编写，但必须是可执行的，并且必须符合以下调用 API 规范[2][5]:

```
----------- Mandatory ------------
<driver executable> init // Performs driver initialisation----------- Implementation option 1 -----------<driver executable> attach <json options> <node name> // Attaching persistent volume to the host<driver executable> waitforattach <mount device> <json options> // Waiting for volume to be attached (10 minutes timeout)<driver executable> isattached <json options> <node name> // Check if the volume is attached<driver executable> detach <mount device> <node name> // Detaching persistent volume from the host<driver executable> mountdevice <mount dir> <mount device> <json options> // Mount the device to a global path so the pod can bind to it by Kubelet<driver executable> unmountdevice <mount device> // Un-mount the device from global path----------- Implementation option 2-----------<driver executable> mount <mount dir> <json options> // This call-out implements both attach and mountdevice functions<driver executable> unmount <mount dir> // This call-out implements both unmountdevice and detach functions
```

参数如`<mount dir>`和`<json options>`将由 kubelet 传入。您基本上使用这些信息来执行您的装载/卸载逻辑。json 选项格式将在下面的“PV 和 PVC 清单”一节中详细介绍。

每次调用都必须通过 stdout 和 stderr 将具有以下格式的消息返回给 kubelet:

```
{
	"status": "<Success/Failure/Not supported>",
	"message": "<Reason for success/failure>",
	"device": "<Path to the device attached. This field is valid only for attach & waitforattach call-outs>"
	"volumeName": "<Cluster wide unique name of the volume. Valid only for getvolumename call-out>"
	"attached": <True/False (Return true if volume is attached on the node. Valid only for isattached call-out)>
    "capabilities": <Only included as part of the Init response>
    {
        "attach": <True/False (Return true if the driver implements attach and detach)>
    }
}
```

“init”调出是必需的。返回消息会将“attach”值设置为 true 或 false，以告知 kubelet 是使用实现选项 1(使用单独的 attach/detach 调用)还是 2。

在 Kubernetes 的卷插件实现中，有多种类型的卷插件，如`RecyclableVolumePlugin`和`ProvisionableVolumePlugin`(更多详情查看此处[)。FlexVolume 实现了两种类型，即`AttachableVolumePlugin`和`PersistentVolumePlugin`。因此，按照插件接口的要求，有两种驱动程序实现的选择。](https://github.com/kubernetes/kubernetes/blob/master/pkg/volume/plugins.go)

请注意，实现 2 中只有“ **mount** ”方法会在 json 参数中传递秘密。更多详情可在[这里](https://github.com/kubernetes/community/blob/master/contributors/devel/flexvolume.md)找到。

如果使用 secret，则机密类型必须是“供应商名称/驱动程序名称”。

一旦你准备好驱动程序，只需将可执行文件放入每个主机的文件夹`<kubernetes volume plugin-dir>/<vendorname~drivername>/<driver name>`。默认的 kubernetes 卷插件目录是`/usr/libexec/kubernetes/kubelet-plugins/volume/exec/`。可以通过配置 kubelet 的参数`volume-plugin-dir`来改变。

有很多方法可以把你的驱动程序放到主机上，比如使用配置工具，比如 Ansible，Puppets。然而，由于集群的动态特性，节点可以上下伸缩，因此使用这些配置工具来维护稳定的集群的开销不容忽视。

更好的解决方案是使用 Daemonset[3]部署驱动程序，它可以:

*   促进驱动程序更新
*   自动部署到新节点
*   没有持续开销

## PV 和 PVC 货单

Flexvolume PV 具有以下格式

```
apiVersion: v1 
kind: PersistentVolume 
metadata:   
  name: pv0001  
spec:   
  capacity:     
    storage: 5Gi    
  accessModes:     
    - ReadWriteOnce
  storageClassName: custom-class
  flexVolume:     
    driver: vendorname/drivername     
    fsType: "ext4"      
    secretRef: 
      name: foo-secret      
    readOnly: true      
    options:        
      fooServer: 192.168.0.1:1234       
      fooVolumeName: bar
```

“选项”键将允许你传递任何自定义值到你的驱动程序。上述 PV 将被转换为 json，并通过 json 选项以如下格式传递到您的驱动程序中:

```
"kubernetes.io/fsType":"ext4",
"kubernetes.io/readwrite":"rwo",
"kubernetes.io/secret/key1":"<secret1>",
...
"kubernetes.io/secret/keyN":"<secretN>",
"fooServer": "192.168.0.1:1234",
"fooVolumeName": "bar"
```

注意:您必须手动设置物理持久性卷，并在“选项”中传递所有必要的信息，以便您的驱动程序了解如何装载和卸载。

现在，在您创建了具有类名“custom-class”的 PV 之后，您可以通过指定“storageClassName”或使用标签选择器在您的任何 PVC 中使用它。举个例子，

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec: 
  storageClassName: custom-class
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

## 参考:

[1] [Kubernetes 容器存储接口(CSI)](https://github.com/container-storage-interface/spec/blob/master/spec.md)

[2] [Flexvolume 规格](https://github.com/kubernetes/community/blob/master/contributors/devel/flexvolume.md)

[3] [动态 Flexvolume 插件发现](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/flexvolume-deployment.md)

[4][【GitHub】外部存储](https://github.com/kubernetes-incubator/external-storage)

[5] [Openshift —使用 Flexvolume 插件的持久卷](https://docs.openshift.org/latest/install_config/persistent_storage/persistent_storage_flex_volume.html)