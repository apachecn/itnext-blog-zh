# K8s/OpenShift 上的分布式容器调试

> 原文：<https://itnext.io/distroless-container-debugging-on-k8s-openshift-e418fd66fdad?source=collection_archive---------0----------------------->

![](img/cd669e4ecd8e742e8a045f9cd0584e4f.png)

当人们越来越关注集装箱的安全性时，基于分布式的图像经常被用来减少攻击面。在这些映像中，包管理器、非依赖模块或库，甚至外壳都被剥离，只有应用程序及其所需的依赖项被保留。对于静态链接的可执行文件，例如 golang 生产的，我们甚至可以使用“scratch”作为基础。

因此，对漏洞的潜在利用大大减少。但是，另一方面，如果连 shell 都不可用，只留下应用程序的日志，就很难对应用程序进行故障诊断。

在本文中，我们将探索不同的选项，通过恢复 shell 来简化调试。

# 应用程序、Dockerfile 和 YAMLs

让我们使用一个玩具 hello world HTTP handler golang 程序。Dockerfile 文件如下所示，

```
FROM golang as builder
WORKDIR /build
COPY ./src /buildRUN go build -o serving *.go**FROM gcr.io/distroless/base**
WORKDIR /app
COPY --from=builder /build/serving /appENTRYPOINT ["./serving"]
```

构建并推送映像。我的测试环境是 OpenShift，使用的是 OpenShift 的私有注册表。使用以下部署 YAML 运行应用程序。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  labels:
    app: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        imagePullPolicy: Always
        image: image-registry.openshift-image-registry.svc:5000/debug/k-debug:v1.0
        ports:
        - containerPort: 80
```

如果我们对 pod 做一个`kubectl exec`(我用的是 OCP)，正如我们所料，我们得到了

```
$ oc exec -it app-7c66b76bb7-2s9gr -- sh
ERRO[0000] exec failed: container_linux.go:367: starting container process caused: exec: "sh": executable file not found in $PATH
command terminated with exit code 1
```

至此，让我们来探索如何将 shell 访问添加到 pod 中。

# 1.发行调试映像

GoogleContainerTool 提供了一个`:debug`图像标签。因此，用:debug 标记替换基本映像，当 exec 进入 pod 时，重新构建映像将为我们提供一个 busybox shell。

显然，它只对开发阶段有好处。

# 2.共享进程命名空间

当一个 pod 启用了进程命名空间共享时，一个容器中的进程可供同一 pod 中的其他容器使用。

让我们更新部署 YAML 文件，

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-share-ns
  labels:
    app: app-share-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-share-ns
  template:
    metadata:
      labels:
        app: app-share-ns
    spec:
      **shareProcessNamespace: true**
      serviceAccountName: debugger
      containers:
      - name: app-share-ns
        imagePullPolicy: Always
        image: image-registry.openshift-image-registry.svc:5000/debug/k-debug:v1.0
        ports:
        - containerPort: 80
        securityContext:
          runAsUser: 1000 - name: shell
        image: ubuntu
        stdin: true
        tty: true
        securityContext:
          runAsUser: 0
```

我们将 pod 选项 shareProcessNamespace 启用为 true。使用 ubuntu 添加额外的 shell 容器。因为图像的默认 CMD 是 bash，所以我们将 tty 和 stdin 都设置为 true。以 root 用户身份运行该容器。让应用程序容器作为非根用户运行，比如用户 id 1000。

在 OpenShift 下，默认服务帐户受到安全上下文约束(SCC)的限制。为了能够以不同的用户身份运行，我们创建了一个新的服务帐户，将 anyuid SCC 分配给它。

```
kubectl create sa debugger
oc adm policy add-scc-to-user anyuid -z debugger
```

应用 YAML 后，附加到外壳容器，我们得到了外壳。

```
$ kubectl attach -it app-share-ns-76b85f9cb5-zkjpc -c shell
If you don't see a command prompt, try pressing enter.
root@app-share-ns-76b85f9cb5-zkjpc:/#
```

用 ps -ef 列出流程，

```
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 10:34 ?        00:00:00 /usr/bin/pod
1000           7       0  0 10:34 ?        00:00:00 ./serving
root          18       0  0 10:34 pts/0    00:00:00 bash
root          44      18  0 10:57 pts/0    00:00:00 ps -ef
```

我们可以看到在 PID 为 7 的用户 1000 下运行的 app 进程。

如果我们安装 netstat 工具，我们可以确定应用程序的监听端口在 8080 上。而部署 YAML 指定了错误的端口号。

```
netstat -tna
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp6       0      0 :::8080                 :::*                    LISTEN
```

我们甚至可以访问/proc 文件系统，这是一个伪文件系统，为进程提供了内核数据结构的接口。例如，我们可以通过符号链接/proc/7/root 访问应用程序的文件系统。但是我们不能使用上面的部署 YAML 来访问它，如下所示，

```
# cd /proc/7/root
bash: cd: /proc/7/root: Permission denied
```

为了实现访问，我们必须分配 SYS_PTRACE Linux 功能，以允许 shell 容器控制其他进程。修改部署外壳容器部分，添加 Linux 功能，如下所示，

```
 - name: shell
            image: ubuntu
            stdin: true
            tty: true
            securityContext:
              runAsUser: 0
              **capabilities:
                add:
                - SYS_PTRACE**
```

同时，我们必须更新 SCC 以允许 SYS_PTRACE Linux 功能。基于默认的 anyuid 创建新的 SCC，

```
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: false
allowHostPID: false
allowHostPorts: false
allowPrivilegeEscalation: true
allowPrivilegedContainer: false
**allowedCapabilities:
- SYS_PTRACE**
apiVersion: security.openshift.io/v1
defaultAddCapabilities: null
fsGroup:
  type: RunAsAny
groups:
- system:cluster-admins
kind: SecurityContextConstraints
metadata:
  name: anyuid-ptrace
priority: 10
readOnlyRootFilesystem: false
requiredDropCapabilities:
- MKNOD
runAsUser:
  type: RunAsAny
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: RunAsAny
volumes:
- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- projected
- secret
```

将新的 SCC 分配给调试器服务帐户，

```
oc adm policy remove-scc-from-user anyuid -z debugger
oc adm policy add-scc-to-user anyuid-ptrace -z debugger
```

重新部署部署，附加到外壳容器。我们现在可以看到这个过程。

```
# ps -ef 
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 15:39 ?        00:00:00 /usr/bin/pod
1000           8       0  0 15:39 ?        00:00:00 ./serving
root          19       0  0 15:39 pts/0    00:00:00 bash
root          33      19  0 15:45 pts/0    00:00:00 ps -ef
```

让我们确认添加了 SYS_PTRACE 功能。添加`capsh`工具。

```
# apt update; apt install libcap2-bin# cat /proc/self/status | grep Cap
CapInh: 00000000000805fb
CapPrm: 00000000000805fb
CapEff: 00000000000805fb
CapBnd: 00000000000805fb
CapAmb: 0000000000000000
```

解码功能，SYS_PTRACE 功能可用。

```
# capsh --decode=00000000000805fb
WARNING: libcap needs an update (cap=39 should have a name).
0x00000000000805fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,**cap_sys_ptrace**
```

现在，我们可以访问它的文件系统。

```
# cd /proc/8/root
# ls
app  bin  boot  busybox  dev  etc  home  lib  lib64  proc  root  run  sbin  sys  tmp  usr  var# ls -lh
total 5.9M
-rwxr-xr-x. 1 root root 5.9M Sep 11 10:32 serving
```

在不更改应用程序容器映像的情况下，共享进程命名空间方法具有对应用程序进行故障排除和调试的完全外壳访问权限。但是，修改部署 YAML 文件，并更改 Pod 的安全设置，使得这种方法只适用于测试环境。

# 3.Kubectl 调试

临时容器是一种特殊类型的容器，它在现有 Pod 中临时运行，以完成用户启动的操作，如故障排除。与`kubectl debug`函数一起，我们可以调试应用程序容器。

注意这个特性，从 Kubernetes v1.22 开始，仍然是 alpha 版本。 *(Alpha 发布目标(1.16)，Beta 发布目标(1.23)，*[*https://github.com/kubernetes/enhancements/issues/277*](https://github.com/kubernetes/enhancements/issues/277)*)*

作为它的阿尔法，我们需要打开它的特征门。对于我的环境，OpenShift 容器平台( *OCP 4.8，K8s v1.21.1* )，创建并应用如下 YAML，让 OCP 支持临时容器。

```
apiVersion: config.openshift.io/v1
kind: FeatureGate
metadata:
  name: cluster
spec:
  customNoUpgrade:
    enabled:
      - EphemeralContainers
  featureSet: CustomNoUpgrade
```

然后，我们可以使用`kubectl debug`命令启动一个容器来进行故障排除。注意到`kubectl debug`与`oc debug`不同。在`oc debug`中，相同容器的命令被替换为`/bin/sh,`，但对于 distroless 容器，这不起作用。

## 3.1 临时容器

使用`kubectl debug`有两种方法。一种方法是使用如下命令在现有 pod 上创建一个临时容器，

```
kubectl debug app-654465676b-5pf44 -it --image=ubuntu --target=app
```

请注意目标参数，它将指示容器引擎使用原始容器的进程名称空间创建容器。当您在新的调试器 shell 中创建进程列表时，pid 1 应该是应用程序进程。调试器 shell 的 pid 应该是应用程序后面的某个数字。

遗憾的是，截至目前，OpenShift 的底层容器引擎 CRI-O 还不支持“— target”命令*(*[*【https://github.com/cri-o/cri-o/issues/4790*](https://github.com/cri-o/cri-o/issues/4790)*)。我们在调试外壳中看不到应用程序的进程。*

一种方法是在原始 pod 中启用`shareProcessNamespace: true`,然后调试 shell 可以看到如下所示的进程名称空间。

```
$ kubectl debug app-654465676b-hmt8z --image=ubuntu -it --target=app
Defaulting debug container name to debugger-zl9v5.
If you don't see a command prompt, try pressing enter.
root@app-654465676b-hmt8z:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 07:41 ?        00:00:00 /usr/bin/pod
1000640+       8       0  0 07:41 ?        00:00:00 ./serving
root          37       0  0 07:45 pts/0    00:00:00 bash
root          47      37  0 07:48 pts/0    00:00:00 ps -ef
```

## 3.2.复制并添加

除了在临时容器上工作，另一种方法是将 pod 复制到一个新的 pod，同时添加一个调试容器。

举个例子，

```
kubectl debug app-7c66b76bb7-5vvxg -it --image=ubuntu --share-processes --copy-to=app-debug
Defaulting debug container name to debugger-vkwns.
If you don't see a command prompt, try pressing enter.
root@app-debug:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 07:55 ?        00:00:00 /usr/bin/pod
1000640+       7       0  0 07:55 ?        00:00:00 ./serving
root          20       0  0 07:55 pts/0    00:00:00 bash
root          28      20  0 07:55 pts/0    00:00:00 ps -ef
```

注意`--share-processes`标志，这样我们可以在调试 shell 中看到进程。

如果我们做一个逃生舱，

```
$ oc get pods
NAME                            READY   STATUS    RESTARTS   AGE
app-7c66b76bb7-5vvxg            1/1     Running   0          4m22s
app-debug                       2/2     Running   1          3m2s
```

显示了复制的 app-debug 窗格，注意现在它不再使用临时容器，而是添加了一个普通容器，

```
 - image: ubuntu
    imagePullPolicy: Always
    name: debugger-vkwns
    resources: {}
    securityContext:
      capabilities:
        drop:
        - MKNOD
    stdin: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    tty: true
```

不出所料，由于缺少 Linux 功能，我们无法访问/proc 进程文件系统。

```
root@app-debug:/# cd /proc/7
root@app-debug:/proc/7# cd root
bash: cd: root: Permission denied
```

在当前版本的 kubectl debug 中，没有办法添加额外的 Linux 功能。希望这个问题*(*[*https://github.com/kubernetes/kubernetes/issues/97103*](https://github.com/kubernetes/kubernetes/issues/97103)*)*解决后，我们能够添加所需的功能。

# 摘要

我们已经研究了在 K8s 中排除 distroless 容器故障的不同方法。到目前为止，如果不修改最初的 pod 定义，它们中没有一个真正适合生产环境使用。

当问题和特性需求结束时，kubectl 调试方法似乎更有希望用于生产。