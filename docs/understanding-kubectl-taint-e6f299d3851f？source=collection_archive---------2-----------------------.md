# 了解$ kubectl 污点(演示)

> 原文：<https://itnext.io/understanding-kubectl-taint-e6f299d3851f?source=collection_archive---------2----------------------->

![](img/275f923f412b890003e1ce349135dc99.png)

kubernetes.io

默认情况下，出于安全原因，kubernetes 集群不会在主节点上调度 pods。但是，如果我们希望能够在主节点上调度 pods，例如:对于用于测试和开发目的的单节点 kubernetes 集群，我们可以运行“$ kubectl taint”命令。

```
$ kubectl taint *--help*

Update the taints on one or more nodes. 

  * A taint consists of a key, value, and effect. As an argument here, it is expressed as key=value:effect.  

  * The key must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and underscores, up to

253 characters.  

  * Optionally, the key can begin with a DNS subdomain prefix and a single '/', like example.com/my-app  

  * The value must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and underscores, up

to  63 characters.  

  * The effect must be NoSchedule, PreferNoSchedule or NoExecute.  

  * Currently taint can only apply to node.

Examples:

  # Update node 'foo' with a taint with key 'dedicated' and value 'special-user' and effect 'NoSchedule'.

  # If a taint with that key and effect already exists, its value is replaced as specified.

  kubectl taint nodes foo dedicated=special-user:NoSchedule

  # Remove from node 'foo' the taint with key 'dedicated' and effect 'NoSchedule' if one exists.

  kubectl taint nodes foo dedicated:NoSchedule-

  # Remove from node 'foo' all the taints with key 'dedicated'

  kubectl taint nodes foo dedicated-

  # Add a taint with key 'dedicated' on nodes having label mylabel=X

  kubectl taint node -l myLabel=X  dedicated=foo:PreferNoSchedule

Options:

      *--all=false: Select all nodes in the cluster*

      *--allow-missing-template-keys=true: If true, ignore any errors in templates when a field or map key is missing in*

the template. Only applies to golang and jsonpath output formats.

      *--include-extended-apis=true: If true, include definitions of new APIs via calls to the API server. [default true]*

      *--no-headers=false: When using the default or custom-column output format, don't print headers (default print*

headers).

  -o, *--output='': Output format. One of:*

json|yaml|wide|name|custom-columns=...|custom-columns-file=...|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...

See custom columns [http://kubernetes.io/docs/user-guide/kubectl-overview/#custom-columns], golang template

[http://golang.org/pkg/text/template/#pkg-overview] and jsonpath template

[http://kubernetes.io/docs/user-guide/jsonpath].

      *--overwrite=false: If true, allow taints to be overwritten, otherwise reject taint updates that overwrite existing*

taints.

  -l, *--selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2)*

  -a, *--show-all=false: When printing, show all resources (default hide terminated pods.)*

      *--show-labels=false: When printing, show all labels as the last column (default hide labels column)*

      *--sort-by='': If non-empty, sort list types using this field specification.  The field specification is expressed*

as a JSONPath expression (e.g. '{.metadata.name}'). The field in the API resource specified by this JSONPath expression

must be an integer or a string.

      *--template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The*

template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

      *--validate=true: If true, use a schema to validate the input before sending it*

Usage:

  kubectl taint NODE NAME KEY_1=VAL_1:TAINT_EFFECT_1 ... KEY_N=VAL_N:TAINT_EFFECT_N [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

我们来演示一下。

这里我们有一个 kubernetes 集群，运行 1 个主节点和 2 个工作节点。

```
$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
node1     Ready     master    2h        v1.10.0
node2     Ready     <none>    2h        v1.10.0
node3     Ready     <none>    2h        v1.10.0
```

让我们验证主节点上污点的状态。

```
Taints:             node-role.kubernetes.io/master:NoSchedule
```

```
$ kubectl describe nodes node1
Name:               node1
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=node1
                    node-role.kubernetes.io/master=
Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:             node-role.kubernetes.io/master:NoSchedule
CreationTimestamp:  Tue, 01 May 2018 01:19:22 +0000
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason   Message
  *----             ------  -----------------                 ------------------                ------   -------*
  OutOfDisk        False   Tue, 01 May 2018 03:41:43 +0000   Tue, 01 May 2018 01:19:18 +0000   KubeletHasSufficientDisk   kubelet has sufficient disk space available
  MemoryPressure   False   Tue, 01 May 2018 03:41:43 +0000   Tue, 01 May 2018 01:19:18 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Tue, 01 May 2018 03:41:43 +0000   Tue, 01 May 2018 01:19:18 +0000   KubeletHasNoDiskPressure   kubelet has no disk pressure
  PIDPressure      False   Tue, 01 May 2018 03:41:43 +0000   Tue, 01 May 2018 01:19:18 +0000   KubeletHasSufficientPID   kubelet has sufficient PID available
  Ready            True    Tue, 01 May 2018 03:41:43 +0000   Tue, 01 May 2018 01:32:23 +0000   KubeletReady   kubelet is posting ready status
Addresses:
  InternalIP:  192.168.0.18
  Hostname:    node1
Capacity:
 cpu:                8
 ephemeral-storage:  10Gi
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             32917692Ki
 pods:               110
Allocatable:
 cpu:                8
 ephemeral-storage:  9663676400
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             32815292Ki
 pods:               110
System Info:
 Machine ID:                 7a452373b9954c958c1dab29c6c5db5c
 System UUID:                542371A0-83E2-C64A-8380-ED7CE0D127B5
 Boot ID:                    7efd235f-3c08-4e8a-aa51-b393bf25020d
 Kernel Version:             4.13.0-38-lowlatency
 OS Image:                   CentOS Linux 7 (Core)
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.3.0
 Kubelet Version:            v1.10.0
 Kube-Proxy Version:         v1.10.0
ExternalID:                  node1
Non-terminated Pods:         (7 in total)
  Namespace                  Name                             CPU Requests  CPU Limits  Memory Requests  Memory Limits
  *---------                  ----                             ------------  ----------  ---------------  -------------*
  kube-system                calico-etcd-q9qkt                0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                calico-node-cnm5s                250m (3%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                etcd-node1                       0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-apiserver-node1             250m (3%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-controller-manager-node1    200m (2%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-proxy-7crmd                 0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-scheduler-node1             100m (1%)     0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  *------------  ----------  ---------------  -------------*
  800m (10%)    0 (0%)      0 (0%)           0 (0%)
Events:         <none>
```

解开节点并验证如下:

```
$ kubectl taint nodes *--all node-role.kubernetes.io/master-*
node "node1" untainted

$ kubectl describe nodes node1 | grep -i taint
Taints:             <none>
```

让我们用名为“testsvr”的 7 个副本创建 nginx 部署。

```
$ kubectl run testsvr *--image=nginx --replicas=7*
deployment "testsvr" created
```

验证“testsvr”单元是否在主节点(即节点 1)上运行。

```
$ kubectl get pods -o wide | grep testsvr
testsvr-596d477cdd-9n2qj   1/1       Running   0          40m       192.168.135.9     node3
testsvr-596d477cdd-bnxd6   1/1       Running   0          40m       192.168.104.8     node2
testsvr-596d477cdd-csnn5   1/1       Running   0          40m       192.168.135.8     node3
testsvr-596d477cdd-hfb2w   1/1       Running   0          40m       192.168.104.9     node2
testsvr-596d477cdd-l9gdj   1/1       Running   0          40m       192.168.104.10    node2
testsvr-596d477cdd-pfhzz   1/1       Running   0          40m       192.168.166.130   node1
testsvr-596d477cdd-wwdc8   1/1       Running   0          40m       192.168.166.129   node1
```

酷！！！

我们将开始一个新的集群和污点设置如下。请注意，pod 不允许在主节点上进行调度。

```
$ kubectl describe nodes node1 | grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
$ kubectl describe nodes node2 | grep Taint
Taints:             <none>
$ kubectl describe nodes node3 | grep Taint
Taints:             <none>
```

让我们运行 10 个名为“websvr”的 nginx web 服务器，它们将运行在 node2 和 node3 上，因为它们没有配置任何污点。

```
$ kubectl run websvr --image=nginx --replicas=10
deployment "websvr" created

$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP          NODE
websvr-6699d7548b-299rb   1/1       Running   0          3m        10.44.0.2   node3
websvr-6699d7548b-5w97p   1/1       Running   0          3m        10.40.0.3   node2
websvr-6699d7548b-85dmm   1/1       Running   0          3m        10.40.0.2   node2
websvr-6699d7548b-8p8kk   1/1       Running   0          3m        10.40.0.4   node2
websvr-6699d7548b-crs2j   1/1       Running   0          3m        10.40.0.5   node2
websvr-6699d7548b-jhfsq   1/1       Running   0          3m        10.40.0.1   node2
websvr-6699d7548b-kr4m2   1/1       Running   0          3m        10.44.0.6   node3
websvr-6699d7548b-mlfmh   1/1       Running   0          3m        10.44.0.3   node3
websvr-6699d7548b-s5rp2   1/1       Running   0          3m        10.44.0.4   node3
websvr-6699d7548b-wn95r   1/1       Running   0          3m        10.44.0.5   node3
```

现在我们要在节点 2 上设置“NoExecute”污点效果。

```
$ kubectl taint nodes node2 node2=DoNotSchedulePods:NoExecute
node "node2" tainted[node1 ~]$ kubectl describe nodes node2 | grep Taint
Taints:             node2=DoNotSchedulePods:NoExecute
```

因此，node2 上的所有 pod 都在 node3 上终止和创建。

```
$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP           NODE
websvr-6699d7548b-299rb   1/1       Running   0          5m        10.44.0.2    node3
websvr-6699d7548b-6rh7r   1/1       Running   0          33s       10.44.0.9    node3
websvr-6699d7548b-bsksd   1/1       Running   0          33s       10.44.0.11   node3
websvr-6699d7548b-jrqdl   1/1       Running   0          33s       10.44.0.10   node3
websvr-6699d7548b-kmpbt   1/1       Running   0          33s       10.44.0.7    node3
websvr-6699d7548b-kr4m2   1/1       Running   0          5m        10.44.0.6    node3
websvr-6699d7548b-mlfmh   1/1       Running   0          5m        10.44.0.3    node3
websvr-6699d7548b-qjjxm   1/1       Running   0          33s       10.44.0.8    node3
websvr-6699d7548b-s5rp2   1/1       Running   0          5m        10.44.0.4    node3
websvr-6699d7548b-wn95r   1/1       Running   0          5m        10.44.0.5    node3
```

取消节点 2 上的设置。

```
$ kubectl taint nodes node2 node2:NoExecute-
node "node2" untainted

$ kubectl describe nodes node2 | grep Taint
Taints:             <none>
```

让我们在 node3 上设置“NoSchedule”污点效果。并运行 8 个名为“nginx”的 nginx web 服务器。

```
$ kubectl taint nodes node3 node3=DoNotSchedulePods:NoSchedule
node "node3" tainted

$ kubectl describe nodes node3 | grep Taint
Taints:             node3=DoNotSchedulePods:NoSchedule

$ kubectl run nginx --image=nginx --replicas=8
deployment "nginx" created
```

由于 node3 被配置为“no schedule”tain effect，8 个 nginx 服务器被安排在 node2 上运行。

```
$ kubectl get pods -o wide | grep nginx
nginx-7c87f569d-c4cnp     1/1       Running   0          28s       10.40.0.7    node2
nginx-7c87f569d-c4s45     1/1       Running   0          28s       10.40.0.10   node2
nginx-7c87f569d-ftp4k     1/1       Running   0          28s       10.40.0.13   node2
nginx-7c87f569d-n6htm     1/1       Running   0          28s       10.40.0.11   node2
nginx-7c87f569d-n99lz     1/1       Running   0          28s       10.40.0.8    node2
nginx-7c87f569d-s92ng     1/1       Running   0          28s       10.40.0.6    node2
nginx-7c87f569d-trpwv     1/1       Running   0          28s       10.40.0.9    node2
nginx-7c87f569d-x5jqw     1/1       Running   0          28s       10.40.0.12   node2
```

解开 node3 上的设置并进行一些测试。

```
$ kubectl taint nodes node3 node3:NoSchedule-
node "node3" untainted

$ kubectl describe nodes node3 | grep Taint
Taints:             <none>

$ kubectl run mypods --image=nginx --replicas=10
deployment "mypods" created

$ kubectl get pods -o wide | grep mypods
mypods-5bb566cb6-8nhsz    1/1       Running   0          21s       10.40.0.17   node2
mypods-5bb566cb6-8sjs6    1/1       Running   0          21s       10.44.0.15   node3
mypods-5bb566cb6-99rkw    1/1       Running   0          21s       10.40.0.15   node2
mypods-5bb566cb6-9ccmt    1/1       Running   0          21s       10.44.0.13   node3
mypods-5bb566cb6-9xkrv    1/1       Running   0          21s       10.44.0.12   node3
mypods-5bb566cb6-bl2jz    1/1       Running   0          21s       10.40.0.16   node2
mypods-5bb566cb6-d77hk    1/1       Running   0          21s       10.40.0.18   node2
mypods-5bb566cb6-hdwdz    1/1       Running   0          21s       10.44.0.14   node3
mypods-5bb566cb6-jtlsq    1/1       Running   0          21s       10.40.0.14   node2
mypods-5bb566cb6-lfwz7    1/1       Running   0          21s       10.40.0.19   node2
```

恭喜你！！！