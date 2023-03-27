# 使用 Spark 运算符在 Spark 上配置蜂箱

> 原文：<https://itnext.io/hive-on-spark-with-spark-operator-9a43ea7ebe06?source=collection_archive---------1----------------------->

![](img/2bf471756d936e47046f07114ef0ef73.png)

照片由[亚伦·伯顿](https://unsplash.com/@aaronburden?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

Spark 节俭服务器用作 Hive 服务器，其执行引擎是 spark。正如 Kubernetes 中 Spark 上的 [Hive 所提到的，Spark Thrift Server 可以部署到 Kubernetes 上。对于这种情况，安装在本地机器上的`spark-submit`已经被用来向 kubernetes 提交 spark thrift 服务器。还有一种方法可以向 kubernetes 提交 spark 应用程序。在这里，我将向您展示如何使用 spark Operator 将 spark thrift server 作为 Spark 应用程序部署到 kubernetes。安装在容器中的`spark-submit , spark-submit`将用于将 spark 应用程序部署到 kubernetes，而不是本地安装，也就是说，Spark Operator 将用于此。](/hive-on-spark-in-kubernetes-115c8e9fa5c1)

有许多 spark 操作符，但我写了一个 spark 操作符( [DataRoaster Spark 操作符](https://github.com/cloudcheflabs/dataroaster/tree/master/operators/spark))来控制 Spark 应用程序，如与开源数据平台 [DataRoaster](https://github.com/cloudcheflabs/dataroaster) 相关的 spark thrift server。

# 先决条件

在使用 spark operator 部署 spark thrift server 之前，需要安装 hive metastore。Hive metastore 需要卷来保存 mysql 中的元数据。还需要将卷安装到 kubernetes 上运行的 spark 应用程序上。对于本文，将安装 NFS 服务器来提供卷，即持久卷。

## 安装 NFS 服务器

首先安装 nfs 二进制文件。

```
sudo yum install nfs-utils -y;
```

创建 nfs 共享目录。

```
sudo mkdir -p /var/nfs_share_dir/nfs-client;
sudo chmod -R 755 /var/nfs_share_dir
sudo chown nfsnobody:nfsnobody /var/nfs_share_dir
```

启动 nfs 服务。

```
sudo systemctl enable rpcbind
sudo systemctl enable nfs-server
sudo systemctl enable nfs-lock
sudo systemctl enable nfs-idmap
sudo systemctl start rpcbind
sudo systemctl start nfs-server
sudo systemctl start nfs-lock
sudo systemctl start nfs-idmap
```

配置导出文件并重新启动 nfs 服务器。

```
# Configuring the exports file for sharing.
sudo vi /etc/exports;
...
/var/nfs_share_dir  *(rw,sync,no_root_squash,insecure)
/var/nfs_share_dir/nfs-client *(rw,sync,no_root_squash,insecure)
...# restart nfs server.
sudo systemctl restart nfs-server;# reexport.
sudo exportfs -rav;# firewall.
sudo firewall-cmd --permanent --zone=public --add-service=nfs
sudo firewall-cmd --permanent --zone=public --add-service=mountd
sudo firewall-cmd --permanent --zone=public --add-service=rpc-bind
sudo firewall-cmd --reload
```

现在，已经安装了 nfs 服务器来共享卷。

## 安装 NFS 存储类

为了在 kubernetes 中动态供应卷，将在 kubernetes 上安装 NFS 存储类。

添加舵回购。

```
helm repo add nfs-subdir-external-provisioner [https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/](https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/)
helm repo update
```

创建值文件。

```
cat <<EOF > nfs-values.yaml
nfs:
  server: <nfs-server-ip>
  path: /var/nfs_share_dir/nfs-client
storageClass:
  name: nfs-external
  accessModes: ReadWriteMany
EOF
```

`<nfs-server-ip>`需要设置您的 nfs 服务器节点的 ip 地址。

现在，用 helm 安装 nfs 存储类。

```
helm install \
nfs-external \
--create-namespace \
--namespace nfs-external \
--values ./nfs-values.yaml \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner;
```

让我们检查一下是否安装了 nfs 服务器存储类。

```
kubectl get sc
NAME                 PROVISIONER                                                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-external         cluster.local/nfs-external-nfs-subdir-external-provisioner   Delete          Immediate           true                   3d3h...
```

NFS 存储类`nfs-external`已成功创建。

## 准备 S3 水桶

S3 水桶是必要的上传星火申请工作文件到 S3 水桶。Hive metastore 也需要访问 S3 存储桶。有许多 S3 兼容的对象存储。你可以从安装在你机器上的 MinIO 或 Ceph S3，或者从 AWS S3、OCI 对象存储等公共云服务获得 S3 桶。应获得以下具有凭据的 S3 存储桶信息。

*   存储桶名称。
*   访问键。
*   秘钥。
*   端点。

# 在 Kubernetes 上安装 Hive Metastore

有一个 github repo 可以看到本文中显示的所有代码。克隆源代码。

```
cd ~;
git clone [https://github.com/mykidong/hive-on-spark-with-spark-operator](https://github.com/mykidong/hive-on-spark-with-spark-operator);
```

修改 metastore 图表的值文件。

```
cd ~/hive-on-spark-with-spark-operator/hive-metastore/metastore;
vi dataroaster-values.yaml;
...namespace: dataroaster-hivemetastore
s3:
  bucket: <bucket-name>
  accessKey: <access-key>
  secretKey: <secret-key>
  endpoint: [https://<](https://cnobgk2u8blu.compat.objectstorage.ap-seoul-1.oraclecloud.com)s3-endpoint>
jdbc:
  user: root
  password: icarus0337
```

需要配置`<bucket-name>`、`<access-key>`、`<secret-key>`和`<s3-endpoint>`。

现在，用下面的 shell 在 kubernetes 上安装 hive metastore。

```
cd ~/hive-on-spark-with-spark-operator/hive-metastore;./create.sh
```

这个 shell 将安装 mysql 服务器，运行作业以在 mysql 中创建初始模式，并在 kubernetes 上安装 hive metastore 服务器。

成功安装 hive metastore 后，它看起来类似于下面这样。

```
kubectl get po -n dataroaster-hivemetastore
NAME                         READY   STATUS      RESTARTS   AGE
hive-initschema--1-4wwgt     0/1     Error       0          73m
hive-initschema--1-js2mk     0/1     Completed   0          73m
metastore-5c65559cb7-pbbv5   1/1     Running     0          73m
mysql-statefulset-0          1/1     Running     0          73m
```

并且，可以看到 mysql 服务器使用的 PVC。

```
kubectl get pvc -n dataroaster-hivemetastore
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-data-disk   Bound    pvc-c2ce07f1-0a6b-4a91-b478-4f5f30f0d83e   1Gi        RWO            nfs-external   74m
```

# 安装 DataRoaster Spark 运算符

让我们安装数据焙烧炉火花运营商与舵图。

添加舵库。

```
helm repo add dataroaster-spark-operator [https://cloudcheflabs.github.io/helm-repository/](https://cloudcheflabs.github.io/helm-repository/)
helm repo update
```

部署火花操作员。

```
helm install \
spark-operator \
--create-namespace \
--namespace spark-operator \
--version v1.0.0 \
dataroaster-spark-operator/dataroaster-spark-operator;
```

查看 spark 操作器的 pod 状态。

```
kubectl get po -n spark-operator
NAME                              READY   STATUS    RESTARTS   AGE
spark-operator-756fbf4479-s2zjt   1/1     Running   0          52s
```

安装 spark operator 非常简单。

# 使用 Spark Operator 在 Kubernetes 上部署 Spark Thrift 服务器

## 准备运行 spark thrift 服务器

让我们与 RBAC 创建服务帐户运行火花节俭服务器。

```
kubectl create namespace spark;
kubectl create serviceaccount spark -n spark;
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=spark:spark --namespace=spark;
```

使用 dataroaster spark operator 部署 spark thrift server 需要准备以下事项。

*   S3 秘密是强制性的。它用于将应用程序文件上传到 S3 存储桶，并访问 S3 存储桶中的数据。
*   spark 本地目录的卷，用于存储 spark 驱动程序和执行器将产生的本地临时数据。spark 本地目录有三种卷类型，如`hostPath`、`emptyDir`、`persistentVolumeClaim`。`persistentVolumeClaim`将用于这个 spark 节俭服务器。

让我们创造 S3 的秘密。

```
cat <<EOF > s3-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: s3-secret
  namespace: spark-operator
type: Opaque
data:
  accessKey: $(echo -n "<access-key>" | base64)
  secretKey: $(echo -n "<secret-key>" | base64)
EOFkubectl apply -f s3-secret.yaml -n spark-operator;
```

`<access-key>`和`<secret-key>`需要配置。

我们需要两个用于 spark 节俭服务器的 spark driver 和 executor pods 的 PVC。来看看下面。

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc-driver
  namespace: spark
  labels: {}
  annotations: {}
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-external
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc-executor
  namespace: spark
  labels: {}
  annotations: {}
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-external
```

`storageClassName`的值是`nfs-external`，这是之前创建的 nfs 存储类。使用以下命令，将创建`nfs-pvc-driver`和`nfs-pvc-executor`PVC。

```
kubectl apply -f ~/hive-on-spark-with-spark-operator/spark-pvc.yaml;
```

看一下 PVC 的名称空间`spark`，因为我们的 spark thrift 服务器将部署在名称空间`spark`中。与此相反，上面创建的 S3 秘密将被运行在名称空间`spark-operator`中的 Spark 操作符使用。

## 安装 spark 节俭服务器的自定义资源

现在，使用定制资源`SparkApplication`部署 spark thrift 服务器的准备工作已经就绪。让我们来看看 spark 节俭服务器的定制资源。

```
apiVersion: "spark-operator.cloudchef-labs.com/v1alpha1"
kind: SparkApplication
metadata:
  name: spark-thrift-server-minimal
  namespace: spark-operator
spec:
  core:
    applicationType: EndlessRun
    deployMode: Cluster
    container:
      image: "cloudcheflabs/spark:v3.0.3"
      imagePullPolicy: Always
    class: com.cloudcheflabs.dataroaster.hive.SparkThriftServerRunner
    applicationFileUrl: "s3a://mykidong/spark-app/spark-thrift-server-3.0.3-spark-job.jar"
    namespace: spark
    s3:
      bucket: mykidong
      accessKey:
        valueFrom:
          secretKeyRef:
            name: s3-secret
            key: accessKey
      secretKey:
        valueFrom:
          secretKeyRef:
            name: s3-secret
            key: secretKey
      endpoint: "https://any-s3-endpoint"
    hive:
      metastoreUris:
        - "thrift://metastore.dataroaster-hivemetastore.svc:9083"
  driver:
    serviceAccountName: spark
    label:
      application-name: spark-thrift-server-driver
    resources:
      cores: "1"
      limitCores: "1200m"
      memory: "512m"
    volumeMounts:
      - name: driver-local-dir
        mountPath: "/tmp/local-dir"
  executor:
    instances: 1
    label:
      application-name: spark-thrift-server-executor
    resources:
      cores: "1"
      limitCores: "1200m"
      memory: "1g"
    volumeMounts:
      - name: executor-local-dir
        mountPath: "/tmp/local-dir"
  volumes:
    - name: driver-local-dir
      type: SparkLocalDir
      persistentVolumeClaim:
        claimName: nfs-pvc-driver
    - name: executor-local-dir
      type: SparkLocalDir
      persistentVolumeClaim:
        claimName: nfs-pvc-executor
```

`spec.applicationFileUrl`是下载 spark 应用文件(例如 spark fat jar 文件)的 url 路径。在运行 spark 应用程序的自定义资源之前，需要将这样的 spark 应用程序文件上传到 S3 bucket。

Spark thrift 服务器 jar 文件可以从这里下载。

```
curl -L -O [https://github.com/cloudcheflabs/spark/releases/download/v3.0.3/spark-thrift-server-3.0.3-spark-job.jar](https://github.com/cloudcheflabs/spark/releases/download/v3.0.3/spark-thrift-server-3.0.3-spark-job.jar);
```

要将 spark thrift 服务器 jar 文件上传到 S3，可以使用 MinIO CLI(mc)。

```
# install mc.
sudo yum install wget -y; 
sudo wget [https://dl.min.io/client/mc/release/linux-amd64/mc](https://dl.min.io/client/mc/release/linux-amd64/mc); 
sudo cp mc /usr/local/bin; 
sudo chmod +x /usr/local/bin/mc;# add s3 alias.mc alias set my-s3 \
[https://<](https://cnobgk2u8blu.compat.objectstorage.ap-seoul-1.oraclecloud.com)s3-endpoint> \
<access-key> \
<secret-key> \
--api S3v4;# upload spark thrift server jar to s3.
mc cp [spark-thrift-server-3.0.3-spark-job.jar](https://github.com/cloudcheflabs/spark/releases/download/v3.0.3/spark-thrift-server-3.0.3-spark-job.jar) my-s3/<bucket>/spark-app/spark-thrift-server-3.0.3-spark-job.jar
```

`spec.s3.accessKey.valueFrom.secretKeyRef.name`是 S3 以前创造的秘密名称。用 s3 端点更改`spec.s3.endpoint`的值。

现在，让我们创建 spark thrift 服务器的自定义资源。

```
cd ~/hive-on-spark-with-spark-operator;kubectl apply -f spark-thrift-server-minimal.yaml;
```

spark thrift server 的 spark driver 和 executor pods 将在名称空间`spark`中创建。

```
kubectl get po -n spark
NAME                                                  READY   STATUS    RESTARTS   AGE
spark-thrift-server-minimal-8daaab7dad4be14a-driver   1/1     Running   0          95s
spark-thrift-server-minimal-bf049d7dad4ce154-exec-1   1/1     Running   0          35s
```

## 连接到 Spark Thrift 服务器以查询数据

要查询与 spark thrift 服务器连接的数据，需要运行 spark 示例作业，以便在 hive 表中创建拼花数据。

```
# clone dataroaster sources.
cd ~;
git clone [https://github.com/cloudcheflabs/dataroaster.git](https://github.com/cloudcheflabs/dataroaster.git);
cd ~/dataroaster;# build all.
mvn -e -DskipTests=true clean install;
```

因为 spark 示例作业需要从本地机器访问 hive metastore，所以需要使用`port-forward`来访问 hive metastore 服务。

```
kubectl port-forward svc/metastore  9083 -n dataroaster-hivemetastore;
```

在本地机器上运行 spark 示例作业。

```
cd ~/dataroaster/components/hive/spark-thrift-server;
mvn -e -Dtest=JsonToParquetTestRunner \
-DmetastoreUrl=localhost:9083 \
-Ds3Bucket=<bucket> \
-Ds3AccessKey=<access-key> \
-Ds3SecretKey=<secret-key> \
-Ds3Endpoint=[https://<](https://cnobgk2u8blu.compat.objectstorage.ap-seoul-1.oraclecloud.com)s3-endpoint> \
test;
```

需要配置与 S3 相关的属性。

spark 实例工作成功后，我们可以用 spark thrift 服务器查询数据。让我们公开 spark 节俭服务器的服务。

```
# create spark thrift server service.
cat <<EOF > spark-thrift-server-service.yaml
kind: Service
apiVersion: v1
metadata:
  name: spark-thrift-server-service
  namespace: spark
spec:
  type: ClusterIP
  selector:
    spark-role: driver
  ports:
    - name: jdbc-port
      port: 10016
      protocol: TCP
      targetPort: 10016
EOFkubectl apply -f spark-thrift-server-service.yaml;
```

因为已经安装在 spark operator 容器中的 spark 中包含的直线将用于连接 spark thrift 服务器，所以让我们列出 spark operator 的 pod 状态来访问 spark operator pod。

```
kubectl get po -n spark-operator;
NAME                              READY   STATUS    RESTARTS   AGE
spark-operator-756fbf4479-s2zjt   1/1     Running   0          14m
```

让我们通过访问 spark operator 容器来连接 spark thrift 服务器。

```
kubectl exec -it spark-operator-756fbf4479-s2zjt -n spark-operator -- /opt/spark/spark-3.0.3/bin/beeline -u jdbc:hive2://spark-thrift-server-service.spark.svc:10016;
```

运行查询以激活节俭服务器。

```
0: jdbc:hive2://spark-thrift-server-service.s> show tables;
+-----------+---------------+--------------+
| database  |   tableName   | isTemporary  |
+-----------+---------------+--------------+
| default   | test_parquet  | false        |
+-----------+---------------+--------------+
1 row selected (1.19 seconds)
0: jdbc:hive2://spark-thrift-server-service.s> select * from test_parquet;
+----------------------------------------------------+---------------+--------+-----------+
|                   baseProperties                   |    itemId     | price  | quantity  |
+----------------------------------------------------+---------------+--------+-----------+
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
| {"eventType":"cart-event","ts":1527304486873,"uid":"any-uid0","version":"7.0"} | any-item-id0  | 1000   | 2         |
+----------------------------------------------------+---------------+--------+-----------+
8 rows selected (11.519 seconds)
0: jdbc:hive2://spark-thrift-server-service.s> select count(*) from test_parquet;
+-----------+
| count(1)  |
+-----------+
| 8         |
+-----------+
1 row selected (2.541 seconds)
```

好的，工作正常。

## 关闭 spark 节俭服务器

要关闭 spark thirft 服务器，只需删除 spark thirft 服务器的自定义资源。

```
kubectl delete sparkapp spark-thrift-server-minimal -n spark-operator;
```

删除 spark thrift server 自定义资源后，spark operator 会自动杀死 spark thrift server 的 driver pod，从而触发删除 spark thrift server 的 executor pods。

正如这里所看到的，spark thrift server 作为 spark 应用程序可以很容易地部署到 kubernetes 上。

使用 spark operator 不仅可以部署 spark 批处理应用程序，还可以轻松部署无休止运行的 spark 应用程序，如 spark 流应用程序。好好享受吧！

# 参考

*   本文源代码:[https://github . com/mykidong/hive-on-spark-with-spark-operator](https://github.com/mykidong/hive-on-spark-with-spark-operator)
*   DataRoaster Spark 操作员:[https://github . com/cloudcheflabs/data roaster/tree/master/operators/Spark](https://github.com/cloudcheflabs/dataroaster/tree/master/operators/spark)