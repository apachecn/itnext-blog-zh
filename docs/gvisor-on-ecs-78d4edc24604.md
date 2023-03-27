# ECS 上的监视器

> 原文：<https://itnext.io/gvisor-on-ecs-78d4edc24604?source=collection_archive---------10----------------------->

谷歌的 [gVisor](https://github.com/google/gvisor) 的存在是为了给你的 Docker 容器提供一个**真正的沙箱**。它取代了最近有严重漏洞的默认 Docker 运行时`runc`。

理论上，gVisor 是`runc`的替代产品**，但它真的能与[亚马逊 ECS](https://aws.amazon.com/ecs/) 一起工作吗？**

![](img/023cebbd838abe0724bf75a5005e0368.png)

克里斯·帕纳斯[https://unsplash.com/@chrispanas](https://unsplash.com/@chrispanas)

# 什么是 gVisor？

> gVisor 是容器的用户空间内核。它限制了应用程序可访问的主机内核表面，同时仍然允许应用程序访问它期望的所有特性。
> 
> [*github.com/google/gvisor*](https://github.com/google/gvisor)

Docker 不像 VMs 那样在容器之间提供严格的安全边界。同一主机上的容器共享内核，并且可以直接对主机进行系统调用。当多租户(在单个主机上运行多个应用程序)您的虚拟机时，这是一个问题——当您的一个应用程序受到威胁时，您的所有应用程序**都会受到威胁。**

最近，在 Docker 中发现了其中一个漏洞。

> *当以 root (UID 0)身份在容器中运行进程时，该进程可以利用 runc 中的错误来获得运行容器的主机上的 root 权限。这允许他们无限制地访问服务器以及该服务器上的任何其他容器。*
> 
> [*kubernetes.io/blog/2019/02/11/…*](https://kubernetes.io/blog/2019/02/11/runc-and-cve-2019-5736/)

gVisor 通过替换易受攻击的组件`runc`来缓解此漏洞。

# 安装遮阳板

作为替代，安装 gVisor 很简单:

```
curl -LsO  [https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc](https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc)
chmod +x runsc
mv runsc /usr/local/bin
cat <<EOF >> /etc/docker/daemon.json
{
    "runtimes": {
        "runsc": {
            "path": "/usr/local/bin/runsc"
       }
    }
}
EOF
systemctl restart docker
```

现在，您可以通过在 Docker 命令中添加`--runtime=runsc`标志来使用 gVisor 运行容器。

**这本身并不是特别有用**，因为我们依赖 ECS 来为我们编排和运行容器。不幸的是，我们不能直接从 ECS 中挑选运行时。[此功能请求仍未完成](https://github.com/aws/amazon-ecs-agent/issues/1084)。[其中一条注释](https://github.com/aws/amazon-ecs-agent/issues/1084#issuecomment-357689366)为我们提供了一个解决方法的提示:**覆盖默认运行时**。我们需要做的只是向 Docker 配置文件添加一个额外的值:

```
{
    "default-runtime": "runsc",
    "runtimes": {
        "runsc": {
            "path": "/usr/local/bin/runsc"
       }
    }
}
```

重启 Docker 后，我们可以看到容器现在使用 gVisor 沙箱化了:

```
$ docker run -d nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
f7e2b70d04ae: Pull complete
08dd01e3f3ac: Pull complete
d9ef3a1eb792: Pull complete
Digest: sha256:98efe605f61725fd817ea69521b0eeb32bef007af0e3d0aeb6258c6e6fe7fc1a
Status: Downloaded newer image for nginx:latest
74cdfe0c0c1b1520b662d946c98883190b885f456ab3c082104521b88460a37c
$ ps aux | grep '[r]unc'
$ # no results
$ ps aux | grep '[r]unsc'
root     29540  0.0  0.0  10800  5004 ?        Sl   10:29   0:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/74cdfe0c0c1b1520b662d946c98883190b885f456ab3c082104521b88460a37c -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runsc
root     29553  0.0  0.0 119304 11404 ?        Sl   10:29   0:00 runsc-gofer --root=/var/run/docker/runtime-runsc/moby --debug=false --log=/run/containerd/io.containerd.runtime.v1.linux/moby/74cdfe0c0c1b1520b662d946c98883190b885f456ab3c082104521b88460a37c/log.json --log-format=json --debug-log= --debug-log-format=text --file-access=exclusive --overlay=false --network=sandbox --log-packets=false --platform=ptrace --strace=false --strace-syscalls= --strace-log-size=1024 --watchdog-action=LogWarning --panic-signal=-1 --log-fd=3
gofer --bundle /run/containerd/io.containerd.runtime.v1.linux/moby/74cdfe0c0c1b1520b662d946c98883190b885f456ab3c082104521b88460a37c --spec-fd=4 --io-fds=5 --io-fds=6 --io-fds=7
--io-fds=8 --apply-caps=false --setup-root=false
nfsnobo+ 29557  2.6  0.1 166480 22572 ?        Ssl  10:29   0:01 runsc-sandbox --root=/var/run/docker/runtime-runsc/moby --debug=false --log=/run/containerd/io.containerd.runtime.v1.linux/moby/74cdfe0c0c1b1520b662d946c98883190b885f456ab3c082104521b88460a37c/log.json --log-format=json --debug-log= --debug-log-format=text --file-access=exclusive --overlay=false --network=sandbox --log-packets=false --platform=ptrace --strace=false --strace-syscalls= --strace-log-size=1024 --watchdog-action=LogWarning --panic-signal=-1 --log-fd=3 boot --bundle=/run/containerd/io.containerd.runtime.v1.linux/moby/74cdfe0c0c1b1520b662d946c98883190b885f456ab3c082104521b88460a37c --controller-fd=4 --spec-fd=5 --start-sync-fd=6 --io-fds=7 --io-fds=8 --io-fds=9 --io-fds=10 --stdio-fds=11 --stdio-fds=12 --stdio-fds=13 --cpu-num 8 74cdfe0c0c1b1520b662d946c98883190b885f456ab3c082104521b88460a37c
```

# 正在运行`ecs-agent`

因为 gVisor 只实现了有限的一组 Linux 系统调用，所以有些东西不能用它。任何期望在较低层次上与系统交互的东西都可能无法按预期工作。

一个显而易见的例子是 ECS 用来管理 EC2 实例的代理。在 [Amazon ECS 优化的 Amazon Linux 2](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/al2ami.html) 上， [ECS 代理](https://github.com/aws/amazon-ecs-agent)通常由 ecs-init systemd 服务启动，并运行 Docker 容器本身。因为它需要对运行它的系统进行低级别的访问，所以它不能很好地与 gVisor 兼容。

这是可以的，因为我们的目标不是用沙箱保护 ECS 代理本身。`docker`客户端允许我们指定另一个运行时，因此我们可以强制 ECS 代理使用`runc`，Docker 的默认运行时运行:

`docker run --runtime=runc [...]`

不幸的是，ecs-init 服务相当不透明，所以我们不能以优雅的方式覆盖它。完全禁用它并手动启动 ECS 代理(使用额外的运行时标志)可以达到预期的效果。

```
systemctl stop ecs  # the ecs-init service will try to start the ecs-agent with gVisor, which doesn't work. We must force stop it and run ecs-agent with runc
systemctl disable ecs
docker run --name ecs-agent \
    --runtime=runc \
    --detach=true \
    --restart=on-failure:10 \
    --volume=/var/run/docker.sock:/var/run/docker.sock \
    --volume=/var/log/ecs:/log \
    --volume=/var/lib/ecs/data:/data \
    --net=host \
    --env-file=/etc/ecs/ecs.config \
    --env=ECS_LOGFILE=/log/ecs-agent.log \
    --env=ECS_DATADIR=/data/ \
    --env=ECS_ENABLE_TASK_IAM_ROLE=true \
    --env=ECS_ENABLE_TASK_IAM_ROLE_NETWORK_HOST=true \
    amazon/amazon-ecs-agent:latest
```

通常，手动调用一个通常由 init 系统管理的进程并期望它保持运行是一个坏主意。在这种情况下，Docker 守护程序管理 ECS 代理的正常运行时间，并在失败时重新启动它。

# 把所有的放在一起

总之，您的 userdata 或 AMI 烘焙脚本将如下所示:

```
curl -LsO  [https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc](https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc)
chmod +x runsc
mv runsc /usr/local/bin
cat <<EOF >> /etc/docker/daemon.json
{
    "default-runtime": "runsc",
    "runtimes": {
        "runsc": {
            "path": "/usr/local/bin/runsc"
       }
    }
}
EOF
systemctl restart docker
systemctl stop ecs  # the ecs-init service will try to start the ecs-agent with gVisor, which doesn't work. We must force stop it and run ecs-agent with runc
systemctl disable ecs
docker run --name ecs-agent \
    --runtime=runc \
    --detach=true \
    --restart=on-failure:10 \
    --volume=/var/run/docker.sock:/var/run/docker.sock \
    --volume=/var/log/ecs:/log \
    --volume=/var/lib/ecs/data:/data \
    --net=host \
    --env-file=/etc/ecs/ecs.config \
    --env=ECS_LOGFILE=/log/ecs-agent.log \
    --env=ECS_DATADIR=/data/ \
    --env=ECS_ENABLE_TASK_IAM_ROLE=true \
    --env=ECS_ENABLE_TASK_IAM_ROLE_NETWORK_HOST=true \
    amazon/amazon-ecs-agent:latest
```

结果是完全透明的。您在 ECS 上运行的任何服务都将正常启动，并出现在 AWS 控制台中，就好像您使用的是开箱即用的 Docker 一样，但实际上您运行的是`runsc`而不是`runc`。

# 奖励:资源预留

我让 gVisor 在几个集群上运行了几个星期，没有出现任何明显的问题。在此期间， [runc 漏洞](https://www.openwall.com/lists/oss-security/2019/02/11/2)被公布，我对自己相当满意。

有一天，我注意到我们的一个非 prod 集群出现了一些奇怪的行为。我们的 EC2 实例从 ECS 的左右两边断开，导致容器无法调度。我开始调查，发现我们的一个应用程序已经失控，正在消耗 100%的**主机的** CPU。ECS 代理处于如此严重的资源争用状态，以至于它无法保持与 ECS 控制平面的连接。

这应该是不可能的。我们充分利用[多租户和资源预留](https://aarongorka.com/blog/ecs-autoscaling-tips/#enforce-reservations-on-ecs-services)来防止这类问题。一个应用程序就能关闭群集破坏了多租户的理念。

结果是 gVisor 没有强制执行资源配额。拒绝服务是一种安全风险，所以修补一个小漏洞而在其他地方打开一个大漏洞是没有意义的。我最终不得不从集群中删除 gVisor，以使其再次稳定。

此后，gVisor 中增加了对 [cgroup 设置](https://github.com/google/gvisor/commit/29cd05a7c66ee8061c0e5cf8e94c4e507dcf33e0)的支持。

# 结论

对于仍然希望利用容器和多租户的高风险环境，gVisor 绝对值得一试。

由于这不是一个完全成熟的项目，您将需要测试任何缺失的功能。您还需要测试[是否适用于您组织的应用](https://github.com/google/gvisor#will-my-container-work-with-gvisor)。

感谢 [Melchi Salins](https://medium.com/@melchi.salins) 的文章[使用 Google 的 gVisor](https://medium.com/momenton/securing-your-caas-using-googles-gvisor-d6e0cd0ae230) 保护您的 CaaS，这启发了我开始研究这个问题。

*原载于*[](https://aarongorka.com/blog/gvisor-on-ecs/)**。**