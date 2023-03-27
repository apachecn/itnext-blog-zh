# 如何检查 Kubernetes 节点的配置

> 原文：<https://itnext.io/how-to-inspect-the-configuration-of-a-kubernetes-node-ca2b7fd2a9a3?source=collection_archive---------0----------------------->

查看 Kubernetes 节点的配置听起来是一件简单的事情，但是并不那么明显。

kubelet 接受的参数要么来自于[命令行](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)参数，要么来自于您通过`--config`传递的配置文件。似乎，直截了当地做一个`ps -ex | grep kubelet`并在文件中查看你看到的`--config`参数之后。简单吧？但是……你确定所有的论证都是正确的吗？如果 Kubernetes 默认了一个您不想要的值呢？如果您没有对节点的 shell 访问权限，该怎么办？

有一种方法可以向 Kubernetes API 查询节点正在运行的配置:`api/vi/nodes/<node_name>/proxy/cofigz`。让我们在实际部署中看看这一点。

## 部署 Kubernetes 集群

我这里使用的是 AWS 上 Kubernetes (CDK) 的 [*规范发行版，但是你可以使用任何你喜欢的云和 Kubernetes 安装方法。*](https://jujucharms.com/canonical-kubernetes/)

```
juju bootstrap aws
juju deploy canonical-kubernetes
```

..并等待部署完成

```
watch juju status 
```

## 更改配置

[CDK](https://jujucharms.com/canonical-kubernetes/) 允许配置命令行参数和配置文件的额外参数。这里我们向配置文件添加参数:

```
juju config kubernetes-worker kubelet-extra-config='{imageGCHighThresholdPercent: 60, imageGCLowThresholdPercent: 39}'
```

一个很大的问题是我们如何得到`imageGCHighThreshholdPercent`的字面意思。在撰写本文时，官方的[上游文档](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/#create-the-config-file)会将您指向[类型定义](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/config/types.go)；一种相当丑陋的方法。在类型定义中有一个`EvictionHard`属性，但是，如果你看一下[上游文档](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/#create-the-config-file)的例子，你会看到相同的属性是小写的。

## 检查配置

我们需要两个弹壳。第一次我们将启动 API 代理，第二次我们将查询 API。在第一个壳上:

```
juju ssh kubernetes-master/0
kubectl proxy
```

现在我们在 kubernetes-master 上的`127.0.0.1:8001`处有了代理，使用第二个 shell 来获取节点名，我们查询 API:

```
juju ssh kubernetes-master/0
kubectl get no
curl -sSL "[http://localhost:8001/api/v1/nodes/](http://localhost:8001/api/v1/nodes/)<node_name>/proxy/configz" | python3 -m json.tool
```

这里是一个完整的运行:

```
juju ssh kubernetes-master/0
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-1023-aws x86_64)* Documentation:  [https://help.ubuntu.com](https://help.ubuntu.com)
 * Management:     [https://landscape.canonical.com](https://landscape.canonical.com)
 * Support:        [https://ubuntu.com/advantage](https://ubuntu.com/advantage)System information as of Mon Oct 22 10:40:40 UTC 2018System load:  0.11               Processes:              115
  Usage of /:   13.7% of 15.45GB   Users logged in:        1
  Memory usage: 20%                IP address for ens5:    172.31.0.48
  Swap usage:   0%                 IP address for fan-252: 252.0.48.1Get cloud support with Ubuntu Advantage Cloud Guest:
    [http://www.ubuntu.com/business/services/cloud](http://www.ubuntu.com/business/services/cloud)0 packages can be updated.
0 updates are security updates.Last login: Mon Oct 22 10:38:14 2018 from 2.86.54.15
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.ubuntu@ip-172-31-0-48:~$ kubectl get no
NAME               STATUS   ROLES    AGE   VERSION
ip-172-31-14-174   Ready    <none>   41m   v1.12.1
ip-172-31-24-80    Ready    <none>   41m   v1.12.1
ip-172-31-63-34    Ready    <none>   41m   v1.12.1
ubuntu@ip-172-31-0-48:~$ curl -sSL "[http://localhost:8001/api/v1/nodes/ip-172-31-14-174/proxy/configz](http://localhost:8001/api/v1/nodes/ip-172-31-14-174/proxy/configz)" | python3 -m json.tool
{
    "kubeletconfig": {
        "syncFrequency": "1m0s",
        "fileCheckFrequency": "20s",
        "httpCheckFrequency": "20s",
        "address": "0.0.0.0",
        "port": 10250,
        "tlsCertFile": "/root/cdk/server.crt",
        "tlsPrivateKeyFile": "/root/cdk/server.key",
        "authentication": {
            "x509": {
                "clientCAFile": "/root/cdk/ca.crt"
            },
            "webhook": {
                "enabled": true,
                "cacheTTL": "2m0s"
            },
            "anonymous": {
                "enabled": false
            }
        },
        "authorization": {
            "mode": "Webhook",
            "webhook": {
                "cacheAuthorizedTTL": "5m0s",
                "cacheUnauthorizedTTL": "30s"
            }
        },
        "registryPullQPS": 5,
        "registryBurst": 10,
        "eventRecordQPS": 5,
        "eventBurst": 10,
        "enableDebuggingHandlers": true,
        "healthzPort": 10248,
        "healthzBindAddress": "127.0.0.1",
        "oomScoreAdj": -999,
        "clusterDomain": "cluster.local",
        "clusterDNS": [
            "10.152.183.93"
        ],
        "streamingConnectionIdleTimeout": "4h0m0s",
        "nodeStatusUpdateFrequency": "10s",
        "nodeLeaseDurationSeconds": 40,
        "imageMinimumGCAge": "2m0s",
        "imageGCHighThresholdPercent": 60,
        "imageGCLowThresholdPercent": 39,
        "volumeStatsAggPeriod": "1m0s",
        "cgroupsPerQOS": true,
        "cgroupDriver": "cgroupfs",
        "cpuManagerPolicy": "none",
        "cpuManagerReconcilePeriod": "10s",
        "runtimeRequestTimeout": "2m0s",
        "hairpinMode": "promiscuous-bridge",
        "maxPods": 110,
        "podPidsLimit": -1,
        "resolvConf": "/run/systemd/resolve/resolv.conf",
        "cpuCFSQuota": true,
        "cpuCFSQuotaPeriod": "100ms",
        "maxOpenFiles": 1000000,
        "contentType": "application/vnd.kubernetes.protobuf",
        "kubeAPIQPS": 5,
        "kubeAPIBurst": 10,
        "serializeImagePulls": true,
        "evictionHard": {
            "imagefs.available": "15%",
            "memory.available": "100Mi",
            "nodefs.available": "10%",
            "nodefs.inodesFree": "5%"
        },
        "evictionPressureTransitionPeriod": "5m0s",
        "enableControllerAttachDetach": true,
        "makeIPTablesUtilChains": true,
        "iptablesMasqueradeBit": 14,
        "iptablesDropBit": 15,
        "failSwapOn": false,
        "containerLogMaxSize": "10Mi",
        "containerLogMaxFiles": 5,
        "configMapAndSecretChangeDetectionStrategy": "Watch",
        "enforceNodeAllocatable": [
            "pods"
        ]
    }
}
```

## 总结

有一种方法可以通过 Kubernetes API(*API/v1/nodes/<node _ name>/proxy/configz*)获得在线 Kubernetes 节点的配置。如果您想针对 Kubernetes 进行编码，或者不想了解特定集群设置的复杂性，这可能会很方便。

## 参考

*   Kubelet 配置:[https://kubernetes . io/docs/tasks/administrator-cluster/kube let-config-file/# create-the-config-file](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/#create-the-config-file)
*   库伯内特的典型分布:【https://jujucharms.com/canonical-kubernetes/ 