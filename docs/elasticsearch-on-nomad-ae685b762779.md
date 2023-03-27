# 游牧民族的弹性研究

> 原文：<https://itnext.io/elasticsearch-on-nomad-ae685b762779?source=collection_archive---------2----------------------->

![](img/5c8fa3c804f1a578ab61a8856d7e2b13.png)

由[瑞安·斯通](https://unsplash.com/@rstone_design?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

正如我在[之前的文章](https://mykidong.medium.com/install-nomad-cluster-d9a40d2206f5)中提到的，我一直在寻找 Kubernetes 的替代品，在容器编排器上部署有状态的应用程序。通常，有状态应用程序需要卷来保存数据。当提交有状态应用程序作业时，应该动态地提供卷。但是目前，Nomad 不支持 Kubernetes 所支持的这种动态卷供应。然而，Nomad 给了我操作有状态应用程序的优势。Kubernetes 上没有启动、停止和重新部署 Statefulsets 之类的有状态应用程序而不丢失数据的概念。如果在 Kubernetes 上删除了 statefulsets，恢复 PVs 上的数据真的很难，否则 PVs 上的数据就会丢失。如果您想在 Kubernetes 上部署 Statefulsets，您应该使用这样的补丁来保存 PVs 上的数据。另一方面，在 Nomad 上，您可以简单地停止正在运行的有状态应用程序作业，并在不丢失数据的情况下重新部署它们，这在最独立的环境中我们已经很熟悉了。由于这个原因，对于我来说，Nomad 是处理在容器编排器上运行的有状态应用程序的工作负载编排器的理想选择。当然，使用 Nomad 有利也有弊，例如，因为 Nomad 不支持动态卷供应，所以在运行分布式有状态应用程序之前，您必须创建和注册大量的卷，这对于编写 Nomad 作业来说是非常繁琐和低效的，您将在本文中看到这一点。

好了，我们来谈谈在 Nomad 上部署 Elasticsearch。Elasticsearch 是一个分布式搜索引擎，它以分布式方式保存卷上的数据。你将在下面看到如何在 Nomad 上部署 Elasticsearch，在这篇文章的最后，我们还将看到如何在 Nomad 上部署 Kibana。

# 开始之前

在继续之前，应该有几个组件可供您使用。

*   Ceph v14
*   Ceph CSI v3.3.1
*   nomad 1 . 0 . 4 版
*   咨询模板 v0.22.1

# 构建弹性搜索 Docker 图像

有一个预构建的 elasticsearch docker 图像，但我将构建一个 elasticsearch docker 图像用于我们的示例。

让我们创建弹性搜索的`Dockerfile`。

```
FROM centos:7

ENV *APP_HOME* /opt/elasticsearch
ENV *ES_TMPDIR*=${*APP_HOME*}/temp
ENV *ES_USER* elasticsearch

RUN useradd -ms /bin/bash -d ${*APP_HOME*} ${*ES_USER*}

RUN yum install nmap-ncat -y

RUN set -ex \
    && ES_PACK_NAME=elasticsearch-7.12.1 \
    && FILE_NAME=${*ES_PACK_NAME*}-linux-x86_64.tar.gz \
    && curl -O https://artifacts.elastic.co/downloads/elasticsearch/${*FILE_NAME*} \
    && tar -zxf ${*FILE_NAME*} \
    && cp -R ${*ES_PACK_NAME*}/* ${*APP_HOME*} \
    && rm -rf ${*APP_HOME*}/config/elasticsearch.yml \
    && rm -rf ${*APP_HOME*}/config/jvm.options \
    && rm -rf ${*FILE_NAME*} \
    && rm -rf ${*ES_PACK_NAME*}

RUN mkdir -p ${*APP_HOME*}/temp
RUN ls -al ${*APP_HOME*}

RUN chmod a+x -R ${*APP_HOME*}/bin
RUN chown ${*ES_USER*}: -R ${*APP_HOME*}

RUN set -ex \
    && echo 'elasticsearch   - nofile 65536' >> /etc/security/limits.d/elasticsearch.conf \
    && echo 'elasticsearch   - nproc  65536' >> /etc/security/limits.d/elasticsearch.conf \
    && echo 'root  soft  memlock unlimited' >> /etc/security/limits.d/elasticsearch.conf \
    && echo 'root  hard  memlock unlimited' >> /etc/security/limits.d/elasticsearch.conf
RUN cat /etc/security/limits.d/elasticsearch.conf

USER ${*ES_USER*}
WORKDIR ${*APP_HOME*}
EXPOSE 9200 9300
```

Elasticsearch `7.12.1`将安装在基于 CentOS 7 基础映像的 docker 映像上。

建立和推广弹性搜索的 docker 形象。

```
# image.
export ES_IMAGE=mykidong/elasticsearch:7.12.1# build.
docker build . -t ${ES_IMAGE};## push.
docker push ${ES_IMAGE};
```

# 在 Nomad 上部署弹性搜索

要在 Nomad 上部署有状态的应用程序作业，需要预先为作业提供卷。Elasticsearch 需要保存索引等数据的卷。如前所述，Nomad 不支持动态配置卷，所有卷都需要手动创建和注册。我希望 Nomad 在不久的将来支持这种动态卷供应。

我们将在 Nomad 上部署 3 个主节点和 2 个数据节点。每个节点将相互使用一个调配的卷来保存数据。

## 创建名称空间

创建一个运行所有 elasticsearch 任务的名称空间。

```
nomad namespace apply -description "Elasticsearch Cluster" elasticsearch;
```

## 创建和注册卷

如果你想知道如何在 Nomad 上提供卷，你应该事先看看[的博客](/provision-volumes-from-external-ceph-storage-on-kubernetes-and-nomad-using-ceph-csi-7ad9b15e9809)。

我们将为主节点和数据节点创建 5 个卷。

让我们为 elasticsearch 的第一个主节点创建一个卷。要创建 ceph 池的映像，您应该键入如下内容。

```
sudo rbd create csi-vol-00000000-1111-2222-bbbb-cacacacacae2 --size 2048 --pool myPool --image-feature layering;
```

要注册用创建的 ceph 映像映射的卷，如下所示。

```
cat <<EOF > es-volume-master-0.hcl
type = "csi"
id   = "es-master-0"
name = "es-master-0"
external_id     = "0001-0024-62c42aed-9839-4da6-8c09-9d220f56e924-0000000000000009-00000000-1111-2222-bbbb-cacacacacae2"
access_mode     = "single-node-writer"
attachment_mode = "file-system"
mount_options {
  fs_type = "ext4"
}
plugin_id       = "ceph-csi"
secrets {
  userID  = "admin"
  userKey = "AQAvo5JgP++oEhAAeZb1j/MTWyLGGJC6abCNFw=="
}
context {
  clusterID = "62c42aed-9839-4da6-8c09-9d220f56e924"
  pool      = "myPool"
  imageFeatures = "layering"
}
EOF
```

并运行以下命令在 Nomad 上注册该卷。

```
nomad volume register -namespace=elasticsearch es-volume-master-0.hcl;
```

作为上面注册的卷，您应该再创建和注册 4 个卷。为 elasticsearch 创建和注册所有卷的完整脚本可以在这里看到:[https://gist . github . com/myki dong/5 FCE 30 ad 4203807 AEA 6281 ee 57 CB 49 f 2](https://gist.github.com/mykidong/5fce30ad4203807aea6281ee57cb49f2)

完成卷注册后，您可以看到卷的状态如下。

```
nomad volume status -namespace elasticsearch;
Container Storage Interface
ID        Name         Plugin ID  Schedulable  Access Mode
es-data-  es-data-0    ceph-csi   true         single-node-writer
es-data-  es-data-1    ceph-csi   true         single-node-writer
es-maste  es-master-0  ceph-csi   true         single-node-writer
es-maste  es-master-1  ceph-csi   true         single-node-writer
es-maste  es-master-2  ceph-csi   true         single-node-writer
```

现在，它准备将这些卷附加到 Nomad 上的 elasticsearch 任务中。

## 创建弹性搜索游牧工作

我们在 Nomad 上部署了三个主节点和两个数据节点。我们可能会想象弹性搜索游牧工作会是这样的。

```
job "elasticsearch" {
   ...
  group "master" {
     count = 3
     ...
  }  
  group "data" {
     count = 2
     ...
  }
}
```

在`master`组中，将运行三个主节点任务，在`data`组中，将运行两个数据节点任务。

但问题是使用 CSI 进行卷装载。让我们在组和任务中添加`volume`和`volume_mount`节，看看为什么卷挂载是问题所在。

```
job "elasticsearch" {
   ...
  group "master" {
    count = 3
    ...
    volume "ceph-volume" {
      type = "csi"
      read_only = false
      source = "es-master-${NOMAD_ALLOC_INDEX}"
    }
    task "elasticsearch" {
      driver = "docker"
      ...
      volume_mount {
        volume      = "ceph-volume"
        destination = "/srv"
        read_only   = false
      }
      ...
    }
  } 
  group "data" {
     count = 2
     ...
     volume "ceph-volume" {
      type = "csi"
      read_only = false
      source = "es-data-${NOMAD_ALLOC_INDEX}"
    }
    task "elasticsearch" {
      driver = "docker"
      ...
      volume_mount {
        volume      = "ceph-volume"
        destination = "/srv"
        read_only   = false
      }
      ...
    }
  }
}
```

`NOMAD_ALLOC_INDEX`是分配指标，用来区分组中任务的实例，如 0、1、2 等。如果当前主节点任务的分配索引为 0，那么将挂载上一节注册的`es-master-0`的卷，分配索引为 1 的`es-master-1`的卷，分配索引为 2 的`es-master-2`的卷。以这种方式在 nomad 作业中挂载多个卷会很不错。但是因为`volume`节的值不能在运行时动态设置，所以上面的作业不能正常工作。

因为在`volume`节中只允许设置静态值，所以我们应该如下计划 elasticsearch nomad 作业。

```
job "elasticsearch" {
   ...
  group "master-0" {
     count = 1
     ...
  }
  group "master-1" {
     count = 1
     ...
  }
  group "master-2" {
     count = 1
     ...
  }
  group "data-0" {
     count = 1
     ...
  }
  group "data-1" {
     count = 1
     ...
  }
}
```

我认为，由于在`volume`节中缺乏支持变量名插值，写这么长的 nomad 作业真的没有效率。例如，如果您想在 nomad 上运行 100 个数据节点，您将放弃它来编写这么长的 Nomad 作业脚本。

让我们看看 elasticsearch job 中`master-0`的完整组。

```
job "elasticsearch" {
   ...
  group "master-0" {
    count = 1
    restart {
      attempts = 3
      delay = "30s"
      interval = "5m"
      mode = "fail"
    }
    network {
      port "request" {
      }
      port "communication" {
      }
    }
    volume "ceph-volume" {
      type = "csi"
      read_only = false
      source = "es-master-0"
    }
    task "elasticsearch" {
      driver = "docker"
      kill_timeout = "300s"
      kill_signal = "SIGTERM"
      volume_mount {
        volume      = "ceph-volume"
        destination = "/srv"
        read_only   = false
      }
      env {
        ES_TMPDIR = "/opt/elasticsearch/temp"
      }
      template {
        data = <<EOF
cluster:
  name: my-cluster
  publish:
    timeout: 300s
  join:
    timeout: 300s
  initial_master_nodes:
    - {{ env "NOMAD_IP_communication" }}:{{ env "NOMAD_HOST_PORT_communication" }}
node:
  name: es-master-0
  master: true
  data: false
  ingest: false
network:
  host: 0.0.0.0
discovery:
  seed_hosts:
    - {{ env "NOMAD_IP_communication" }}:{{ env "NOMAD_HOST_PORT_communication" }}
path:
  data:
    - /srv/data
  logs: /srv/log
bootstrap.memory_lock: true
indices.query.bool.max_clause_count: 10000
EOF
        destination = "local/elasticsearch.yml"
      }
      template {
        data = <<EOF
-Xms512m
-Xmx512m
8-13:-XX:+UseConcMarkSweepGC
8-13:-XX:CMSInitiatingOccupancyFraction=75
8-13:-XX:+UseCMSInitiatingOccupancyOnly
14-:-XX:+UseG1GC
-Djava.io.tmpdir=${ES_TMPDIR}
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=data
-XX:ErrorFile=logs/hs_err_pid%p.log
8:-XX:+PrintGCDetails
8:-XX:+PrintGCDateStamps
8:-XX:+PrintTenuringDistribution
8:-XX:+PrintGCApplicationStoppedTime
8:-Xloggc:logs/gc.log
8:-XX:+UseGCLogFileRotation
8:-XX:NumberOfGCLogFiles=32
8:-XX:GCLogFileSize=64m
9-:-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
EOF
        destination = "local/jvm.options"
      }
      config {
        image = "mykidong/elasticsearch:7.12.1"
        force_pull = false
        volumes = [
          "./local/elasticsearch.yml:/opt/elasticsearch/config/elasticsearch.yml",
          "./local/jvm.options:/opt/elasticsearch/config/jvm.options"
        ]
        command = "bin/elasticsearch"
        args = [
          "-Enetwork.publish_host=${NOMAD_IP_request}",
          "-Ehttp.publish_port=${NOMAD_HOST_PORT_request}",
          "-Ehttp.port=${NOMAD_PORT_request}",
          "-Etransport.publish_port=${NOMAD_HOST_PORT_communication}",
          "-Etransport.tcp.port=${NOMAD_PORT_communication}"
        ]
        ports = [
          "request",
          "communication"
        ]
        ulimit {
          memlock = "-1"
          nofile = "65536"
          nproc = "65536"
        }
      }
      resources {
        cpu = 100
        memory = 1024
      }
      service {
        name = "es-req"
        port = "request"
        check {
          name = "rest-tcp"
          type = "tcp"
          interval = "10s"
          timeout = "2s"
        }
        check {
          name     = "rest-http"
          type     = "http"
          path     = "/"
          interval = "5s"
          timeout  = "4s"
        }
      }
      service {
        name = "es-master-0-comm"
        port = "communication"
        check {
          type = "tcp"
          interval = "10s"
          timeout = "2s"
        }
      }
    }
  }
  ...
}
```

看一看`task` 中的`template` 节。使用这些模板，`elasticsearch.yml`和`jvm.options`的配置将被创建并链接到 elasticsearch 容器。有两个`service`节，用于请求 elasticsearch 的服务`es-req`和用于 elasticsearch 集群中节点间通信的服务`es-master-0-comm`将注册到 Consul。您可以使用 [Consul-Template](https://github.com/hashicorp/consul-template) 在服务`es-req`中获取 elasticsearch 节点的 IP 地址和端口。

```
cat <<EOF > service.tpl
{{ range service "es-req" }}
  - {{ .Address }}:{{ .Port }}{{ end }}
EOFconsul-template -template service.tpl -dry;
> - 10.0.0.200:30246
  - 10.0.0.200:22052
  - 10.0.0.200:26345
  - 10.0.0.200:20821
  - 10.0.0.200:30044
```

卷`es-master-0`已经被挂载到`/srv`的路径上。

```
 volume "ceph-volume" {
      type = "csi"
      read_only = false
      source = "es-master-0"
    }
    task "elasticsearch" {
      ...
      volume_mount {
        volume      = "ceph-volume"
        destination = "/srv"
        read_only   = false
      }
      ...
    }
```

在容器参数中，可以设置附加的弹性搜索配置。

```
 args = [
          "-Enetwork.publish_host=${NOMAD_IP_request}",
          "-Ehttp.publish_port=${NOMAD_HOST_PORT_request}",
          "-Ehttp.port=${NOMAD_PORT_request}",
          "-Etransport.publish_port=${NOMAD_HOST_PORT_communication}",
          "-Etransport.tcp.port=${NOMAD_PORT_communication}"
        ]
```

你可以得到分配任务的 IP 地址。变量`NOMAD_IP_<network-port-name>`是端口为`<network-port-name>`的已分配任务的 IP 地址，例如`NOMAD_IP_request`是监听`request`端口的已分配任务`elasticsearch`的 IP 地址。

`NOMAD_HOST_PORT_<network-port-name>`是端口名为`<network-port-name>`的已分配任务容器的发布端口，`NOMAD_PORT_<network-port-name>`是端口名为`<network-port-name>`的已分配任务容器正在监听的容器端口。例如，`http.publish_port`将被设置为已分配任务容器的已发布`request`端口，`transport.publish_port`将被设置为已分配任务容器的已发布`communication`端口。

好了，我们现在已经看到了`master-0`小组的详细情况。你可以在这里看到 elasticsearch nomad job `elasticsearch.nomad`的完整代码:[https://gist . github . com/myki dong/AEA 105d 29 e 47 fafb 1c caeaf 2 ed C5 c 183](https://gist.github.com/mykidong/aea105d29e47fafb1ccaeaf2edc5c183)

最后，让我们运行 elasticsearch nomad job。

```
nomad job run elasticsearch.nomad;
```

正确运行 elasticsearch 作业后，您将看到 elasticsearch 集群的健康状况。Elasticsearch 请求的 ip 地址和端口可以使用如上所述的 consul-template 获得。

```
curl [http://10.0.0.200:26345/_cluster/health?pretty](http://10.0.0.200:26345/_cluster/health?pretty)
{
  "cluster_name" : "my-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 5,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 9,
  "active_shards" : 18,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

现在，我们有了一个 elasticsearch 集群，它由运行在 Nomad 上的 3 个主节点和 2 个数据节点组成！

# 在游牧部落部署基巴纳

使用 Kibana，您可以在 elasticsearch 中查询索引，并轻松构建仪表板。

创建 Kibana docker 图像。

```
FROM centos:7

ENV *APP_HOME* /opt/kibana
ENV *KIBANA_USER* kibana

RUN useradd -ms /bin/bash -d ${*APP_HOME*} ${*KIBANA_USER*}

RUN yum install nmap-ncat -y

RUN set -ex \
    && PACK_NAME=kibana-7.12.1-linux-x86_64 \
    && FILE_NAME=${*PACK_NAME*}.tar.gz \
    && curl -O https://artifacts.elastic.co/downloads/kibana/${*FILE_NAME*} \
    && tar -zxf ${*FILE_NAME*} \
    && cp -R ${*PACK_NAME*}/* ${*APP_HOME*} \
    && rm -rf ${*APP_HOME*}/config/kibana.yml \
    && rm -rf ${*FILE_NAME*} \
    && rm -rf ${*PACK_NAME*}

RUN mkdir -p ${*APP_HOME*}/temp
RUN ls -al ${*APP_HOME*}

RUN chmod a+x -R ${*APP_HOME*}/bin
RUN chown ${*KIBANA_USER*}: -R ${*APP_HOME*}

USER ${*KIBANA_USER*}
WORKDIR ${*APP_HOME*}
EXPOSE 5601
```

为 Kibana 构建 docker 映像。

```
# image.
export KIBANA_IMAGE=mykidong/kibana:7.12.1# build.
docker build . -t ${KIBANA_IMAGE};## push.
docker push ${KIBANA_IMAGE};
```

为 Kibana 数据创建并注册一个卷。

```
## create an image in the pool of ceph.
sudo rbd create csi-vol-00000000-1111-2222-bbbb-cacacacacag1 --size 2048 --pool myPool --image-feature layering;## es kibana volume.
cat <<EOF > es-volume-kibana.hcl
type = "csi"
id   = "es-kibana"
name = "es-kibana"
external_id     = "0001-0024-62c42aed-9839-4da6-8c09-9d220f56e924-0000000000000009-00000000-1111-2222-bbbb-cacacacacag1"
access_mode     = "single-node-writer"
attachment_mode = "file-system"
mount_options {
  fs_type = "ext4"
}
plugin_id       = "ceph-csi"
secrets {
  userID  = "admin"
  userKey = "AQAvo5JgP++oEhAAeZb1j/MTWyLGGJC6abCNFw=="
}
context {
  clusterID = "62c42aed-9839-4da6-8c09-9d220f56e924"
  pool      = "myPool"
  imageFeatures = "layering"
}
EOF# register volume.
nomad volume register -namespace=elasticsearch es-volume-kibana.hcl;
```

现在，创建基巴纳游牧民工作`kibana.nomad`。

```
job "kibana" {
  namespace = "elasticsearch"
  datacenters = ["dc1"]
  type        = "service"
  update {
    max_parallel     = 1
    health_check     = "checks"
    min_healthy_time = "30s"
    healthy_deadline = "5m"
    auto_revert      = true
    canary           = 0
    stagger          = "30s"
  }
  group "kibana-server" {
    count = 1
    restart {
      attempts = 3
      delay = "30s"
      interval = "5m"
      mode = "fail"
    }
    network {
      port "http" {
        static = 5601
      }
    }
    volume "ceph-volume" {
      type = "csi"
      read_only = false
      source = "es-kibana"
    }
    task "await-es-req" {
      driver = "docker"
      config {
        image        = "busybox:1.28"
        command      = "sh"
        args         = ["-c", "echo -n 'Waiting for service'; until nslookup es-req.service.consul 2>&1 >/dev/null; do echo '.'; sleep 2; done"]
        network_mode = "host"
      }
      resources {
        cpu    = 200
        memory = 128
      }
      lifecycle {
        hook    = "prestart"
        sidecar = false
      }
    }
    task "kibana" {
      driver = "docker"
      kill_timeout = "300s"
      kill_signal = "SIGTERM"
      volume_mount {
        volume      = "ceph-volume"
        destination = "/srv"
        read_only   = false
      }
      template {
        data = <<EOF
elasticsearch:
  hosts:
    - [http://{{range](/{{range) $index, $element := service "es-req"}}{{if eq $index 0}}{{ .Address }}:{{ .Port }}{{end}}{{end}}
path:
  data: /srv/data
EOF
        destination = "local/kibana.yml"
      }
      config {
        image = "mykidong/kibana:7.12.1"
        force_pull = false
        volumes = [
          "./local/kibana.yml:/opt/kibana/config/kibana.yml",
        ]
        command = "bin/kibana"
        args = [
          "--host",
          "0.0.0.0",
          "--port",
          "${NOMAD_PORT_http}"
        ]
        ports = [
          "http"
        ]
        ulimit {
          memlock = "-1"
          nofile = "65536"
          nproc = "65536"
        }
      }
      resources {
        cpu = 100
        memory = 1024
      }
      service {
        name = "es-kibana-http"
        port = "http"
        check {
          name = "http-tcp"
          type = "tcp"
          interval = "10s"
          timeout = "2s"
        }
        check {
          name     = "http-http"
          type     = "http"
          path     = "/"
          interval = "5s"
          timeout  = "4s"
        }
      }
    }
  }
}
```

在运行 Kibana 任务之前，将运行预启动任务`await-es-req`。

```
 ... 
    task "await-es-req" {
      driver = "docker"
      config {
        image        = "busybox:1.28"
        command      = "sh"
        args         = ["-c", "echo -n 'Waiting for service'; until nslookup es-req.service.consul 2>&1 >/dev/null; do echo '.'; sleep 2; done"]
        network_mode = "host"
      }
      resources {
        cpu    = 200
        memory = 128
      }
      lifecycle {
        hook    = "prestart"
        sidecar = false
      }
    }
    ...
```

这个预启动任务将等待在 Consul 中注册的服务`es-req`准备就绪，也就是说，elasticsearch 准备好获取请求。

看看 kibana 配置中的 elasticsearch 主机`kibana.yml`。

```
elasticsearch:
  hosts:
    - [http://{{range](/{{range) $index, $element := service "es-req"}}{{if eq $index 0}}{{ .Address }}:{{ .Port }}{{end}}{{end}}
```

将获得服务`es-req`中第一个元素的 IP 地址和端口，该服务已经在 consul 上注册以处理 elasticsearch 请求。

让我们运行基巴纳游牧工作。

```
nomad job run kibana.nomad
```

要找到访问 Kibana UI 的 IP 地址和端口，请使用下面的 consul-template 脚本。

```
cat <<EOF > service.tpl
{{ range service "es-kibana-http" }}{{ .Address }}:{{ .Port }}{{ end }}
EOF# print result to standout.
consul-template -template service.tpl -dry;
>
10.0.0.200:5601
```

在 Kibana UI 中，可以在添加数据>示例数据中创建`Sample eCommerce orders`，然后看起来是这样的。

![](img/2240195b95641dbcbc8fb842c553d349.png)

我们可以再做一件事来检查在 Nomad 上重新部署 elasticsearch 后的数据丢失情况。让我们重新部署弹性搜索工作。

```
## stop elasticsearch.
nomad stop -purge -namespace elasticsearch elasticsearch;# run elasticsearch job.
nomad job run elasticsearch.nomad;
```

停止并再次部署 kibana 作业。

```
# stop kibana job.
nomad job stop -purge -namespace elasticsearch kibana# run kibana job.
nomad job run kibana.nomad
```

然后，您将在 Kibana UI 中的示例电子商务仪表板中看到没有丢失的数据。

正如我们现在所看到的，Nomad 目前不支持运行时的`volume`节中的变量名插值或动态的卷供应，这应该会在不久的将来得到改进。尽管如此，我认为，Nomad 能够以一种简单的方式更好地处理有状态应用程序，而不会丢失数据。