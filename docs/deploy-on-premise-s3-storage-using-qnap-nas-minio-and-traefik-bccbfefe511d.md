# 使用 QNAP NAS、Minio 和 Traefik 部署本地 S3 存储

> 原文：<https://itnext.io/deploy-on-premise-s3-storage-using-qnap-nas-minio-and-traefik-bccbfefe511d?source=collection_archive---------0----------------------->

# 为什么？

任何受欢迎的云提供商，如[甲骨文](https://www.oracle.com/cloud/storage/object-storage.html)、[亚马逊](https://aws.amazon.com/s3/)或 [Azure](https://azure.microsoft.com/en-us/services/storage/blobs/) 都可以为您提供兼容的 S3 存储，因此我们的想法是部署一个本地 S3 兼容存储，用于开发、测试或存档，同时进行完整的云迁移。

# 配置图

让我们开始看看我们的本地 Docker 集群的部署图

![](img/120cd18728cef0b3128a818b9d2f06ba.png)

集群部署图

我们有一个 6 节点集群，外加一个 QNAP TS-831X，它作为集群的另一个节点，但具有不同的架构(ARM Cortex-A15 CPU)。

![](img/9a3c9c0d5700888c08cc18e4463c34c4.png)

# docker 节点

持久卷存储或者使用 [GlusterFS](https://hub.docker.com/repository/docker/mochoa/glusterfs-volume-plugin) 实现，或者使用 QNAP NAS 基于 NFS v4 实现。

# QNAP 集装箱站

QNAP 提供了一个基于浏览器的工具来管理名为[容器站](https://www.qnap.com/solution/container_station/en/)的 Docker 容器，如下所示:

![](img/53b462c838f36f90e85c5c294b20130e.png)

集装箱站

但是，通过将 QNAP NAS 加入我们的 Docker Swarm 集群，我们获得了另一个好处:

*   使用 [Portainer](https://www.portainer.io/) 与其他堆栈和容器一起管理 Docker

![](img/a6b064b52ae0ce1b046b5671c3540905.png)

Portainer。IO 应用群集群图形视图

*   使用 Grafana/Prometheus 进行监控

![](img/97ca1f085b56e9f654b35fca87a45ada.png)

Grafana/Prometheus 接口

*   最后是使用 ssh 对 QNAP NAS 的命令行访问

![](img/e4945f22a43e595c225699f6581f2e33.png)

# ssh 管理@10.254.0.158

# 部署我们的 S3 兼容存储

正如我们上面提到的，我们的想法是使用 [Minio 对象存储](https://min.io/)作为我们的本地 S3 后端，因此一旦 QNAP NAS 加入 Docker Swarm 集群并完全集成到其中，启动 Minio 服务器就非常容易了，但让我们看看两个不同的选项:

*   部署为群集堆栈，为 MinIO 服务器选择目标节点 NAS-DC
*   作为独立容器部署在 QNAP NAS 中

第一个选项很简单，只需在 Portainer 中放置一个 docker-compose.yml，部署包括 MinIO 在内的堆栈，并使用 Traefik 公开 UI，此处的缺点是所有到 S3 存储的 IO 都将使用 Traefik 路由，并且无法使用 NAS 硬件上可用的互连接口；默认情况下，这种 QNAP 模型带有两个网络接口，我们使用一个连接到机架专用主干，该主干连接到 swarm 的所有节点。由于未知的原因，如果 MinIO 作为 stack Docker 容器的一部分部署，它不会正确地公开 MinIO 端口 9000。

第二个选项允许 MinIO 接收 QNAP NAS 的两个网络接口的流量，这非常适合选择:

*   公共接口(MinIO Web UI 的流量，使用 [Caddy](https://caddyserver.com/) 反向代理和 Traefik 公开)
*   私有互连(来自使用 [S3 文件系统卷插件](https://hub.docker.com/repository/docker/mochoa/s3fs-volume-plugin)的 Docker 群任务的流量)

在这两种解决方案中，SSL 加密都是从 MinIO 服务器上卸载的，将流量留给使用 LetsEncrypt 证书的 [Traefik](https://traefik.io/) 。

# 在 QNAP NAS 部署 MinIO

通过使用 SSH 访问 QNAP NAS，创建一个简单的 shell 脚本来启动 MinIO 服务器:

```
[~] # cat start-minio.sh
  docker run -d \
  ***--restart=always*** \
  --health-cmd='curl -f [http://localhost:9000/minio/health/live'](http://localhost:9000/minio/health/live') \
  --health-interval=2s \
  --name minio-server \
  -p ***9000***:9000/tcp \
  -e "***MINIO_ACCESS_KEY***=AKIAIOSFODNN7EXAMPLE" \
  -e "***MINIO_SECRET_KEY***=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  -v /share/CACHEDEV1_DATA/homes/s3:/data \
  ***minio/minio:RELEASE.2020-11-25T22-36-25Z*** \
  server /data
```

注意事项:

*   -p 9000:9000/tcp 将在两个网络接口(端口 9000 tcp)上显示 MinIO 服务器
*   必须为生产更改 MINIO_ACCESS_KEY、MINIO_SECRET_KEY
*   /share/CACHEDEV1_DATA/homes/s3 是 QNAP NAS 上的本地目录，请为您的数据寻找一个有足够空间的目录，或者创建一个新目录
*   [minio/minio:release . 2020–11–25t 22–36–25Z](https://hub.docker.com/_/minio)Docker Hub 上的 Docker 映像是否为 ARM 编译
*   — restart=always 将确保 MinIO 始终处于开启状态，并在 NAS 重新启动后继续运行

# 公开 MinIO Web 用户界面

如上所述，MinIO Web UI 将使用 Caddy 和 [Traefik](https://traefik.io/) 暴露于互联网或内部网络，让我们来看一个简单的堆栈来实现这一点

```
version: '3.6'services:
  server:
    image: caddy:alpine
    command: caddy reverse-proxy --from :80 --to 10.254.0.158:9000
  networks:
    - lb_network
  deploy:
    mode: replicated
    placement:
      constraints:
        - node.hostname != NAS-DC
    labels:
      - traefik.enable=true
      - traefik.docker.network=lb_network
      - traefik.constraint-label=traefik-public
      - traefik.http.routers.minio.rule=Host(`***s3.mydomain.com***`)
      - traefik.http.routers.minio.entrypoints=http
      - traefik.http.services.minio.loadbalancer.server.port=80networks:
  lb_network:
    external: true
```

注意事项:

*   10.254.0.158 是连接到公共网络的 NAS QNAP 的 IP
*   球童:阿尔卑斯山是球童反向代理的 Docker 图像
*   s3.mydomain.com 是 Traefik 揭露的外部名称
*   lb_network 是由 Traefik stack 和 Caddy 共享的 Docker Swarm 网络
*   node .主机名！= NAS-DC 确保 Caddy 在其他主机上运行，而不是在 QNAP NAS 上运行

MinIO Web 用户界面将看起来像:

![](img/431a42b4d22dd9fa4f4464410b37e609.png)

MinIO Web 用户界面

注意，我使用的是普通 HTTP 访问，因为 Traefik 没有 LetsEncrypt 证书。

# 使用 Docker 堆栈中的 S3 存储

正如我前面提到的，要访问 MinIO S3 对象存储，需要安装一个 [S3 文件系统插件](https://hub.docker.com/repository/docker/mochoa/s3fs-volume-plugin)的实例，步骤如下:

```
# docker plugin install --alias s3fs  [mochoa/s3fs-volume-plugin](https://hub.docker.com/repository/docker/mochoa/s3fs-volume-plugin) --grant-all-permissions --disable
# docker plugin set s3fs ***AWSACCESSKEYID***="AKIAIOSFODNN7EXAMPLE"
# docker plugin set s3fs ***AWSSECRETACCESSKEY***="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
# docker plugin set s3fs DEFAULT_S3FSOPTS="use_path_request_style,url=http://***10.1.253.50***:9000/"
# docker plugin enable s3fs
```

注意事项:

*   mochoa/s3fs-volume-plugin 是 Docker Hub 上提供的 Docker 映像
*   AWSACCESSKEYID，AWSSECRETACCESSKEY 必须等于上述值
*   DEFAULT_S3FSOPTS 具有使用专用接口连接到 QNAP NAS 的信息。
    通过使用私有接口，使用此插件的 Docker 任务将被路由到第二个接口，从而获得除 MinIO Web UI 访问的流量之外的额外流量

# 使用 Docker 容器的样本测试

```
root@iguana:~# docker volume create -d ***s3fs*** ***test-bucket***
test-bucket
root@iguana:~# docker run -ti --rm -v ***test-bucket***:/mnt ubuntu bash
root@0db09c90684b:/# ls -l /mnt
total 1
drwxr-x--- 1 root root 0 Jan  1  1970 test-dir
root@0db09c90684b:/# ls -l /mnt/test-dir/
total 9
-rw-r----- 1 root root 8626 Jan 15 23:43 README.md
```

# 示例 Docker 堆栈

```
version: '3.7'services:
  test:
    image: ubuntu
    command: tail -f /dev/null
    deploy:
      mode: replicated
      placement:
        constraints:
          - node.hostname == iguana
    volumes:
      - ***data***:/mntvolumes:
  ***data***:
    driver: s3fs
    name: "***test-bucket***"
```

由以上堆栈启动的 Docker 容器内的日志记录:

```
root@iguana:~# docker exec -ti minio_test.1.zbun9oeqsrvsn1rrfy2knh0j9 bash
root@2aec797db597:/# ls -l /mnt/
total 1
drwxr-x--- 1 root root 0 Jan  1  1970 test-dir
root@2aec797db597:/# ls -l /mnt/test-dir/
total 9
-rw-r----- 1 root root 8626 Jan 15 23:43 README.md
root@2aec797db597:/# dd if=/dev/zero of=/mnt/test.img bs=1G count=1
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 20.868 s, ***52.8 MB/s*** root@2aec797db597:/# dd if=/mnt/test.img of=/dev/null
2097152+0 records in
2097152+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 13.4096 s, ***84.9 MB/s***
```

正如你所看到的足够快了，但是 ***NFS v4*** 对于写( ***102 > 52，8*** )来说快了两倍，对于读( ***114 > 84，9*** )来说快了 34%，同样的磁盘，同样的接口，同样的物理服务器托管 Swarm 任务，但是可能我必须更深入地研究 s3fs 参数才能获得最佳值。

无论如何， [MinIO](https://min.io/) 部署在 [QNAP NAS](https://www.qnap.com/en/) 上，作为我们内部部署的 S3 兼容存储，正是我们开始开发、测试和投入生产云原生应用所需要的。