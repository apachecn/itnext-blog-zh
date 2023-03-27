# Kubernetes 星火上的蜂巢

> 原文：<https://itnext.io/hive-on-spark-in-kubernetes-115c8e9fa5c1?source=collection_archive---------0----------------------->

![](img/6020153eebe2ad69fecbab9d75fb5db8.png)

照片由[威廉·布特](https://unsplash.com/@williambout?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

在 Kubernetes 上运行 Hive 并不容易。据我所知，Tez 是一个 hive 执行引擎，可以运行在 YARN 上，而不是 Kubernetes 上。

在 Kubernetes 上运行 Hive 还有一个替代方法。Spark 可以在 Kubernetes 上运行，兼容 Hive Server2 的 Spark Thrift Server 是一个很好的候选。也就是说，Spark 将作为 hive 执行引擎运行。

我将讨论如何在 kubernetes 集群上运行 Hive on Spark。

这里提到的所有代码都可以从我的 github repo 中克隆:[https://github.com/mykidong/hive-on-spark-in-kubernetes](https://github.com/mykidong/hive-on-spark-in-kubernetes)

# 假设 S3 桶和 NFS 作为库伯内特存储可用

在 kubernetes 上运行 Hive 之前，您的 S3 存储桶和 NFS 存储应该可用于您的 Kubernetes 集群。

您的 S3 桶将用于存储上传的 spark 依赖 jar、hive 表数据等。

NFS 存储将用于支持启动作业所需的 PVC `ReadWriteMany`访问模式。

如果您没有这样的 S3 桶和 NFS 可用，您可以像我一样在您的 kubernetes 集群上手动安装它们:

*   MinIO Direct CSI:[https://github.com/minio/direct-csi](https://github.com/minio/direct-csi)
*   https://github.com/minio/operator 的迷你 S3 对象存储:
*   NFS:[https://github . com/helm/charts/tree/master/stable/NFS-server-provisioner](https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner)

# 安装配置单元 Metastore

Spark Thrift Server 作为 Hive Server2 需要 Hive metastore。为了在 kubernetes 上安装 hive metastore，我引用了[这个链接](https://github.com/joshuarobinson/presto-on-k8s/tree/master/hive_metastore)。

Hive metastore 需要 mysql 来存储元数据。`hive-metastore/mysql.yaml`看起来是这样的:

```
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secrets
  namespace: my-namespace
type: Opaque
data:
  ROOT_PASSWORD: aWNhcnVzMDMzNw==
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-data-disk
  namespace: my-namespace
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: direct.csi.min.io
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  namespace: my-namespace
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          ports:
            - containerPort: 3306
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              subPath: "mysql"
              name: mysql-data
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: ROOT_PASSWORD
      volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql-data-disk
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: my-namespace
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

看一下 PVC 存储“Storage class name:direct . CSI . min . io ”,它应该被删除或更改以适合您的 kubernetes 集群。

创建 mysql 后，将运行 hive metastore 初始化作业，为 Hive Metastore 创建数据库和表。让我们看看`hive-metastore/init-schema.yaml`:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: hive-initschema
  namespace: my-namespace
spec:
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - mysql
              topologyKey: kubernetes.io/hostname
      containers:
        - name: hivemeta
          image: mykidong/hivemetastore:v3.0.0
          command: ["/opt/hive-metastore/bin/schematool"]
          args: ["--verbose" ,"-initSchema" , "-dbType", "mysql" , "-userName", "root",
                 "-passWord", "icarus0337" , "-url", "jdbc:mysql://mysql-service.my-namespace.svc.cluster.local:3306/metastore_db?createDatabaseIfNotExist=true&connectTimeout=1000"]
      restartPolicy: Never
  backoffLimit: 4
```

现在，配置 Hive metastore hadoop 站点 xml 配置，参见`hive-metastore/core-site.xml`和`hive-metastore/metastore-site.xml`:

```
<configuration>
    <property>
        <name>fs.s3a.connection.ssl.enabled</name>
       <value>true</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>s3a://mykidong</value>
    </property>
    <property>
        <name>fs.s3a.path.style.access</name>
        <value>true</value>
    </property>
    <property>
        <name>fs.s3a.access.key</name>
        <value>my-access-key</value>
    </property>
    <property>
        <name>fs.s3a.secret.key</name>
        <value>my-secret-key</value>
    </property>
    <property>
        <name>fs.s3a.impl</name>
        <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
    </property>
    <property>
        <name>fs.s3a.endpoint</name>
        <value>https://s3-endpoint</value>
    </property>
    <property>
        <name>fs.s3a.fast.upload</name>
        <value>true</value>
    </property>
</configuration>..............<configuration>
   <property>
      <name>metastore.task.threads.always</name>
      <value>org.apache.hadoop.hive.metastore.events.EventCleanerTask</value>
   </property>
   <property>
      <name>metastore.expression.proxy</name>
      <value>org.apache.hadoop.hive.metastore.DefaultPartitionExpressionProxy</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://mysql-service.my-namespace.svc.cluster.local:3306/metastore_db?useSSL=false</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>root</value>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>icarus0337</value>
   </property>
   <property>
      <name>metastore.warehouse.dir</name>
      <value>s3a://mykidong/warehouse/</value>
   </property>
   <property>
      <name>metastore.thrift.port</name>
      <value>9083</value>
   </property>
</configuration>
```

您必须更改 s3 相关的属性以适应您的环境。

配置配置单元 metastore 站点 xml 后，可以使用清单运行配置单元 metastore，`hive-metastore/metastore.yaml`:

```
apiVersion: v1
kind: Service
metadata:
  name: metastore
  namespace: my-namespace
spec:
  ports:
  - port: 9083
  selector:
    app: metastore
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metastore
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: metastore
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: metastore
    spec:
      containers:
      - name: metastore
        image: mykidong/hivemetastore:v3.0.0
        env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: my-s3-keys
              key: access-key
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: my-s3-keys
              key: secret-key
        ports:
        - containerPort: 9083
        volumeMounts:
        - name: metastore-cfg-vol
          mountPath: /opt/hive-metastore/conf/metastore-site.xml
          subPath: metastore-site.xml
        - name: metastore-cfg-vol
          mountPath: /opt/hadoop/etc/hadoop/core-site.xml
          subPath: core-site.xml
        command: ["/opt/hive-metastore/bin/start-metastore"]
        args: ["-p", "9083"]
        resources:
          requests:
            memory: "2G"
            cpu: 2
        imagePullPolicy: Always
      volumes:
        - name: metastore-cfg-vol
          configMap:
            name: metastore-cfg
```

要运行 installing hive metastore all in one，请参见 shell 脚本`hive-metastore/create.sh`:

```
**#!/bin/sh** export MY_NAMESPACE=my-namespace

kubectl create namespace ${MY_NAMESPACE};

# create mysql server.
kubectl apply -f mysql.yaml;

# wait for mysql pod being ready.
while [[ $(kubectl get pods -n ${MY_NAMESPACE} -l app=mysql -o 'jsonpath={..status.conditions[?(@.type=="Ready")].status}') != "True" ]]; do echo "waiting for mysql pod being ready" && sleep 1; done

# Update configmaps
kubectl create configmap metastore-cfg --dry-run --from-file=metastore-site.xml --from-file=core-site.xml -o yaml -n ${MY_NAMESPACE} | kubectl apply -f -

# create secret for aws keys.
kubectl create secret generic my-s3-keys --from-literal=access-key='my-access-key' --from-literal=secret-key='my-secret-key' -n ${MY_NAMESPACE};

# create db schemas.
kubectl apply -f init-schema.yaml;

# create metastore.
kubectl apply -f metastore.yaml;

# wait for finishing creating schemas.
while [[ $(kubectl get pods -n ${MY_NAMESPACE} -l job-name=hive-initschema -o jsonpath={..status.phase}) != *"Succeeded"* ]]; do echo "waiting for finishing init schema job" && sleep 2; done

# restart hive metastore: not efficient, but...
kubectl rollout restart deployment.apps/metastore -n ${MY_NAMESPACE};
```

最后，运行创建 shell:

```
cd hive-metastore;
./create.sh;
```

# 安装火花 SA，RBAC，聚氯乙烯

因为 spark thrift server 是一个 spark 作业，它需要服务帐户、角色、角色绑定、`ReadWriteMany`支持的 PVC 才能在 kubernetes 上运行，所以在将 Spark Thrift Server 作为 hive server2 运行之前，这样的服务帐户、RBAC 和用于 Spark 作业的 PVC 应该是可用的。

让我们看看`spark-env/rbac-pvc.yaml`:

```
---
kind: ServiceAccount
apiVersion: v1
metadata:
  namespace: my-namespace
  name: spark
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: my-namespace
  name: spark-role
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list", "create", "update", "patch", "delete"]
  - apiGroups:
      - ""
    resources:
      - secrets
    verbs:
      - get
      - list
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - list
      - watch
      - create
      - update
      - patch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
      - list
      - update
      - watch
  - apiGroups:
      - ""
    resources:
      - namespaces
    verbs:
      - get
      - list
  - apiGroups:
      - storage.k8s.io
    resources:
      - storageclasses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - persistentvolumeclaims
    verbs:
      - get
      - list
      - watch
      - update
  - apiGroups:
      - ""
    resources:
      - persistentvolumes
    verbs:
      - get
      - list
      - watch
      - update
      - create
  - apiGroups:
      - storage.k8s.io
    resources:
      - volumeattachments
    verbs:
      - get
      - list
      - watch
      - update
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: spark-rolebinding
  namespace: my-namespace
subjects:
  - kind: ServiceAccount
    name: spark
roleRef:
  kind: Role
  name: spark-role
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: spark-driver-pvc
  namespace: my-namespace
  labels: {}
  annotations: {}
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: nfs
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: spark-exec-pvc
  namespace: my-namespace
  labels: {}
  annotations: {}
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
  storageClassName: nfs
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: spark-driver-localdir-pvc
  namespace: my-namespace
  labels: {}
  annotations: {}
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: nfs
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: spark-exec-localdir-pvc
  namespace: my-namespace
  labels: {}
  annotations: {}
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
  storageClassName: nfs
```

PVC 接入模式必须是 NFS 支持的`ReadWriteMany`。

```
spec:
  accessModes:
    - ReadWriteMany
```

看一下存储类“storageClassName: nfs ”,应该对其进行更改以适应您的 kubernetes 集群。

现在，使用以下命令创建 spark sa、pvc 和 role，rolebinding:

```
cd spark-env;kubectl apply -f . ;
```

# 安装 Spark 节俭服务器

您可以下载预构建的 spark 版本来使用，但是我不打算使用预构建的 spark 包。因为想和`3.2.0`的版本有 hadoop 依赖，所以要从源代码重新构建 spark。让我们按照下面的步骤来重建 spark:

```
# build spark with source.
export SPARK_VERSION=3.0.0;
cd spark;
git clone [https://github.com/apache/spark.git](https://github.com/apache/spark.git);
cd spark;
git checkout v$SPARK_VERSION;export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=1g";# build.
./dev/make-distribution.sh --name custom-spark --pip --tgz -DskipTests=true -Dhadoop.version=3.2.0 -Phive -Phive-thriftserver -Pkubernetes;
```

但是建立火花真的需要很长时间。好在我已经建好了，带 hadoop 3.2.0 的 spark 包可以从我的 google drive 下载。

```
# download spark tar file from google drive.
# https://drive.google.com/file/d/1_hpk6p_mgQ3gCA3ZV_cSUuEdt_yZlAaX/view?usp=sharing
SPARK_FILE_NAME=spark-3.0.0-bin-custom-spark
fileId=1_hpk6p_mgQ3gCA3ZV_cSUuEdt_yZlAaX
fileName=${SPARK_FILE_NAME}.tgz

curl -sc /tmp/cookie "https://drive.google.com/uc?export=download&id=${fileId}" > /dev/null
code="$(awk '/_warning_/ {print $NF}' /tmp/cookie)"
curl -Lb /tmp/cookie "https://drive.google.com/uc?export=download&confirm=${code}&id=${fileId}" -o ${fileName}
```

还有一件事对我们来说是必要的，那就是建立 spark 的 docker 映像，让我们建立 spark docker 映像，它将用于运行 spark thrift server 和稍后的另一个 spark 作业:

```
# create spark docker image.
export SPARK_VERSION=3.0.0;
cd $SPARK_HOME;bin/docker-image-tool.sh -r your-repo -t v$SPARK_VERSION build;
bin/docker-image-tool.sh -r your-repo -t v$SPARK_VERSION push;
```

现在，几乎可以安装 spark thrift server 了，让我们创建 jdbc 客户端可以连接的 spark thrift server 服务:

```
kind: Service
apiVersion: v1
metadata:
  name: spark-thrift-server-service
  namespace: my-namespace
spec:
  type: LoadBalancer
  selector:
    spark-role: driver
  ports:
    - name: jdbc-port
      port: 10016
      protocol: TCP
      targetPort: 10016
```

Spark submit 不允许默认的 spark thrift 服务器在 kubernetes 上以集群模式运行。这里有一个技巧来避免这种情况，我写了一个简单的包装类，其中将调用 spark thrift server，让我们看看包装类“SparkThriftServerRunner ”:

```
package io.mykidong.hive;

public class SparkThriftServerRunner {

    public static void main(String[] args) {
     org.apache.spark.sql.hive.thriftserver.HiveThriftServer2.*main*(args);

 while (true) {
    try {
        Thread.*sleep*(Long.*MAX_VALUE*);
    } catch (Exception e) {
        e.printStackTrace();
    }
} }
}
```

这个类将被调用来在 spark submit 中运行 spark thrift server，如下所示:

```
spark-submit \
--master $MASTER \
--deploy-mode cluster \
--name spark-thrift-server \
--class io.mykidong.hive.SparkThriftServerRunner \
...
```

当你用 maven 构建为这个博客提供的源代码时,`org.pentaho:pentaho-aggdesigner-algorithm:jar:5.1.5-jhyde`存在依赖性问题。下面的 maven 的服务器配置要加在`settings.xml`里。

```
<servers>
   <server>
      <id>pentaho-public</id>
      <username>devreaduser</username>
      <password>{zIMyJWfHKfoHiBJAVsAgW4E5BcJzR+nhTtgPy0J+/rs=}</password>
   </server>
</servers>
```

要构建 spark thrift server uber jar，请在`examples/spark-thrift-server`中键入以下命令:

```
mvn -e -DskipTests=true clean install shade:shade;
```

如前所述，spark thrift server 只是运行在 kubernetes 上的一个 spark 作业，让我们在 kubernetes 上的`cluster mode`中看到 spark 提交来运行 spark thrift server。

```
# submit spark thrift server job.
MASTER=k8s://your-k8s-master-url;
NAMESPACE=${MY_NAMESPACE};
ENDPOINT=https://s3-endpoint;
BUCKET=mykidong;
HIVE_METASTORE=metastore:9083;

spark-submit \
--master $MASTER \
--deploy-mode cluster \
--name spark-thrift-server \
--class io.mykidong.hive.SparkThriftServerRunner \
--packages com.amazonaws:aws-java-sdk-s3:1.11.375,org.apache.hadoop:hadoop-aws:3.2.0 \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-driver-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-exec-pvc \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-driver-localdir-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-exec-localdir-pvc \
--conf spark.kubernetes.file.upload.path=s3a://${BUCKET}/spark-thrift-server \
--conf spark.kubernetes.container.image.pullPolicy=Always \
--conf spark.kubernetes.namespace=$NAMESPACE \
--conf spark.kubernetes.container.image=mykidong/spark:v3.0.0 \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.hadoop.hive.metastore.client.connect.retry.delay=5 \
--conf spark.hadoop.hive.metastore.client.socket.timeout=1800 \
--conf spark.hadoop.hive.metastore.uris=thrift://${HIVE_METASTORE} \
--conf spark.hadoop.hive.server2.enable.doAs=false \
--conf spark.hadoop.hive.server2.thrift.http.port=10002 \
--conf spark.hadoop.hive.server2.thrift.port=10016 \
--conf spark.hadoop.hive.server2.transport.mode=binary \
--conf spark.hadoop.metastore.catalog.default=spark \
--conf spark.hadoop.hive.execution.engine=spark \
--conf spark.hadoop.hive.input.format=io.delta.hive.HiveInputFormat \
--conf spark.hadoop.hive.tez.input.format=io.delta.hive.HiveInputFormat \
--conf spark.sql.warehouse.dir=s3a:/${BUCKET}/apps/spark/warehouse \
--conf spark.hadoop.fs.defaultFS=s3a://${BUCKET} \
--conf spark.hadoop.fs.s3a.access.key=my-access-key \
--conf spark.hadoop.fs.s3a.secret.key=my-secret-key \
--conf spark.hadoop.fs.s3a.connection.ssl.enabled=true \
--conf spark.hadoop.fs.s3a.endpoint=$ENDPOINT \
--conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
--conf spark.hadoop.fs.s3a.fast.upload=true \
--conf spark.hadoop.fs.s3a.path.style.access=true \
--conf spark.driver.extraJavaOptions="-Divy.cache.dir=/tmp -Divy.home=/tmp" \
--conf spark.executor.instances=2 \
--conf spark.executor.memory=2G \
--conf spark.executor.cores=1 \
--conf spark.driver.memory=1G \
--conf spark.jars=${tempDirectory}/${DELTA_CORE_FILE_NAME}.jar,${tempDirectory}/${HIVE_DELTA_FILE_NAME}.jar \
file://<src>/examples/spark-thrift-server/target/spark-thrift-server-1.0.0-SNAPSHOT-spark-job.jar \
> /dev/null 2>&1 &
```

您必须用源目录的完整路径替换<src>。</src>

看看 S3 相关属性、Kubernetes 主 URL、Hive Metastore 端点的配置，应该对其进行更改以满足您的需求。

您可以找到 PVC 的几种火花配置，这是火花驱动器和执行器保存临时数据所必需的:

```
...
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-driver-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-exec-pvc \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-driver-localdir-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-exec-localdir-pvc \
...
```

如果 spark 作业被提交，首先，依赖 jar 文件将被上传到上面配置的 s3 桶，然后，spark 驱动程序和执行器将从 S3 桶下载上传的依赖 jar，并动态地将它们添加到自己的类加载器中。

让我们看看运行 spark thrift server 的完整 shell 脚本。

```
**#!/bin/sh** export MY_NAMESPACE=my-namespace
export tempDirectory=~/temp/spark;
mkdir -p ${tempDirectory};

cd ${tempDirectory};# download spark tar file from google drive.
# https://drive.google.com/file/d/1_hpk6p_mgQ3gCA3ZV_cSUuEdt_yZlAaX/view?usp=sharing
SPARK_FILE_NAME=spark-3.0.0-bin-custom-spark
fileId=1_hpk6p_mgQ3gCA3ZV_cSUuEdt_yZlAaX
fileName=${SPARK_FILE_NAME}.tgz

curl -sc /tmp/cookie "https://drive.google.com/uc?export=download&id=${fileId}" > /dev/null
code="$(awk '/_warning_/ {print $NF}' /tmp/cookie)"
curl -Lb /tmp/cookie "https://drive.google.com/uc?export=download&confirm=${code}&id=${fileId}" -o ${fileName}

# download delta core jar file from google drive.
# https://drive.google.com/file/d/1WCzSnwXEYc3Q8VkvvJ5nuidq9yGYDIsa/view?usp=sharing
DELTA_CORE_FILE_NAME=delta-core-shaded-assembly_2.12-0.1.0
fileId=1WCzSnwXEYc3Q8VkvvJ5nuidq9yGYDIsa
fileName=${DELTA_CORE_FILE_NAME}.jar

curl -sc /tmp/cookie "https://drive.google.com/uc?export=download&id=${fileId}" > /dev/null
code="$(awk '/_warning_/ {print $NF}' /tmp/cookie)"
curl -Lb /tmp/cookie "https://drive.google.com/uc?export=download&confirm=${code}&id=${fileId}" -o ${fileName}

# download hive delta file from google drive.
# https://drive.google.com/file/d/1PcSraIo9Fc5sKIuDmDfFytcE8i5DcgN_/view?usp=sharing
HIVE_DELTA_FILE_NAME=hive-delta_2.12-0.1.0
fileId=1PcSraIo9Fc5sKIuDmDfFytcE8i5DcgN_
fileName=${HIVE_DELTA_FILE_NAME}.jar

curl -sc /tmp/cookie "https://drive.google.com/uc?export=download&id=${fileId}" > /dev/null
code="$(awk '/_warning_/ {print $NF}' /tmp/cookie)"
curl -Lb /tmp/cookie "https://drive.google.com/uc?export=download&confirm=${code}&id=${fileId}" -o ${fileName}

# install spark.
mkdir -p spark;
tar zxvf ${SPARK_FILE_NAME}.tgz -C spark/;
cd spark/;
cp -R ${SPARK_FILE_NAME}/* .;
rm -rf ${SPARK_FILE_NAME};

# set spark home.
SPARK_HOME=${tempDirectory}/spark;
PATH=$PATH:$SPARK_HOME/bin;

cd ${tempDirectory};

# submit spark thrift server job.
MASTER=k8s://your-k8s-master-url;
NAMESPACE=${MY_NAMESPACE};
ENDPOINT=https://s3-endpoint;
BUCKET=mykidong;
HIVE_METASTORE=metastore:9083;

spark-submit \
--master $MASTER \
--deploy-mode cluster \
--name spark-thrift-server \
--class io.mykidong.hive.SparkThriftServerRunner \
--packages com.amazonaws:aws-java-sdk-s3:1.11.375,org.apache.hadoop:hadoop-aws:3.2.0 \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-driver-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-exec-pvc \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-driver-localdir-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-exec-localdir-pvc \
--conf spark.kubernetes.file.upload.path=s3a://${BUCKET}/spark-thrift-server \
--conf spark.kubernetes.container.image.pullPolicy=Always \
--conf spark.kubernetes.namespace=$NAMESPACE \
--conf spark.kubernetes.container.image=mykidong/spark:v3.0.0 \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.hadoop.hive.metastore.client.connect.retry.delay=5 \
--conf spark.hadoop.hive.metastore.client.socket.timeout=1800 \
--conf spark.hadoop.hive.metastore.uris=thrift://${HIVE_METASTORE} \
--conf spark.hadoop.hive.server2.enable.doAs=false \
--conf spark.hadoop.hive.server2.thrift.http.port=10002 \
--conf spark.hadoop.hive.server2.thrift.port=10016 \
--conf spark.hadoop.hive.server2.transport.mode=binary \
--conf spark.hadoop.metastore.catalog.default=spark \
--conf spark.hadoop.hive.execution.engine=spark \
--conf spark.hadoop.hive.input.format=io.delta.hive.HiveInputFormat \
--conf spark.hadoop.hive.tez.input.format=io.delta.hive.HiveInputFormat \
--conf spark.sql.warehouse.dir=s3a:/${BUCKET}/apps/spark/warehouse \
--conf spark.hadoop.fs.defaultFS=s3a://${BUCKET} \
--conf spark.hadoop.fs.s3a.access.key=my-access-key \
--conf spark.hadoop.fs.s3a.secret.key=my-secret-key \
--conf spark.hadoop.fs.s3a.connection.ssl.enabled=true \
--conf spark.hadoop.fs.s3a.endpoint=$ENDPOINT \
--conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
--conf spark.hadoop.fs.s3a.fast.upload=true \
--conf spark.hadoop.fs.s3a.path.style.access=true \
--conf spark.driver.extraJavaOptions="-Divy.cache.dir=/tmp -Divy.home=/tmp" \
--conf spark.executor.instances=2 \
--conf spark.executor.memory=2G \
--conf spark.executor.cores=1 \
--conf spark.driver.memory=1G \
--conf spark.jars=${tempDirectory}/${DELTA_CORE_FILE_NAME}.jar,${tempDirectory}/${HIVE_DELTA_FILE_NAME}.jar \
file://<src>/examples/spark-thrift-server/target/spark-thrift-server-1.0.0-SNAPSHOT-spark-job.jar \
> /dev/null 2>&1 &

PID=$!
echo "$PID" > pid

# check if spark thrift server pod is running.

SPARK_THRIFT_SERVER_IS_RUNNING="False";
check_spark_thrift_server_is_running() {
    POD_STATUS=$(kubectl get pods -n ${MY_NAMESPACE} -l spark-role=driver -o jsonpath={..status.phase});
    POD_NAME=$(kubectl get pods -n ${MY_NAMESPACE} -l spark-role=driver -o jsonpath={..metadata.name});

    if [[ ${POD_STATUS} != "" ]]
    then
      # Set space as the delimiter
      IFS=' ';

      #Read the split words into an array based on space delimiter
      read -a POD_STATUS_ARRAY <<< "$POD_STATUS";
      read -a POD_NAME_ARRAY <<< "$POD_NAME";

      for ((i = 0; i < ${#POD_STATUS_ARRAY[@]}; ++i)); do
          pod_status=${POD_STATUS_ARRAY[i]};
          pod_name=${POD_NAME_ARRAY[i]};
          printf "status: %s, name: %s\n" "${pod_status}" "${pod_name}";

          if [[ $pod_status == "Running" ]]
          then
              if [[ $pod_name =~ "spark-thrift-server" ]]
              then
                  printf "selected pod - status: %s, name: %s\n" "${pod_status}" "${pod_name}";
                  SPARK_THRIFT_SERVER_IS_RUNNING="True";
              fi
          fi
      done
    fi
}

while [[ $SPARK_THRIFT_SERVER_IS_RUNNING != "True" ]];
do
    echo "waiting for spark thrift server being run...";
    sleep 2;
    check_spark_thrift_server_is_running;
done

# kill current spark submit process.
kill $(cat pid);

# create service.
kubectl apply -f spark-thrift-server-service.yaml;

unset SPARK_HOME;
```

这看起来有点复杂，但事实并非如此。

在第一部分，从 google drive 下载必要的 jars 和用 hadoop 3.2.0 重建的 spark 包:

```
# download spark tar file from google drive.
# https://drive.google.com/file/d/1_hpk6p_mgQ3gCA3ZV_cSUuEdt_yZlAaX/view?usp=sharing
SPARK_FILE_NAME=spark-3.0.0-bin-custom-spark
fileId=1_hpk6p_mgQ3gCA3ZV_cSUuEdt_yZlAaX
fileName=${SPARK_FILE_NAME}.tgz

curl -sc /tmp/cookie "https://drive.google.com/uc?export=download&id=${fileId}" > /dev/null
code="$(awk '/_warning_/ {print $NF}' /tmp/cookie)"
curl -Lb /tmp/cookie "https://drive.google.com/uc?export=download&confirm=${code}&id=${fileId}" -o ${fileName}
...
```

spark 提交在后台执行:

```
# submit spark thrift server job.
MASTER=k8s://your-k8s-master-url;
NAMESPACE=${MY_NAMESPACE};
ENDPOINT=https://s3-endpoint;
BUCKET=mykidong;
HIVE_METASTORE=metastore:9083;

spark-submit \
--master $MASTER \
--deploy-mode cluster \
--name spark-thrift-server \
--class io.mykidong.hive.SparkThriftServerRunner \
--packages com.amazonaws:aws-java-sdk-s3:1.11.375,org.apache.hadoop:hadoop-aws:3.2.0 \
...
```

并检查 spark thrift server pod 是否正在运行:

```
# check if spark thrift server pod is running.

SPARK_THRIFT_SERVER_IS_RUNNING="False";
check_spark_thrift_server_is_running() {
    POD_STATUS=$(kubectl get pods -n ${MY_NAMESPACE} -l spark-role=driver -o jsonpath={..status.phase});
    POD_NAME=$(kubectl get pods -n ${MY_NAMESPACE} -l spark-role=driver -o jsonpath={..metadata.name});

    if [[ ${POD_STATUS} != "" ]]
    then
      # Set space as the delimiter
      IFS=' ';

      #Read the split words into an array based on space delimiter
      read -a POD_STATUS_ARRAY <<< "$POD_STATUS";
      read -a POD_NAME_ARRAY <<< "$POD_NAME";

      for ((i = 0; i < ${#POD_STATUS_ARRAY[@]}; ++i)); do
          pod_status=${POD_STATUS_ARRAY[i]};
          pod_name=${POD_NAME_ARRAY[i]};
          printf "status: %s, name: %s\n" "${pod_status}" "${pod_name}";

          if [[ $pod_status == "Running" ]]
          then
              if [[ $pod_name =~ "spark-thrift-server" ]]
              then
                  printf "selected pod - status: %s, name: %s\n" "${pod_status}" "${pod_name}";
                  SPARK_THRIFT_SERVER_IS_RUNNING="True";
              fi
          fi
      done
    fi
}

while [[ $SPARK_THRIFT_SERVER_IS_RUNNING != "True" ]];
do
    echo "waiting for spark thrift server being run...";
    sleep 2;
    check_spark_thrift_server_is_running;
done
...
```

最后，在终止 spark 提交进程后，创建 spark thrift 服务器服务。

```
# kill current spark submit process.
kill $(cat pid);

# create service.
kubectl apply -f spark-thrift-server-service.yaml;
```

然后，您可以看到 kubectl 的名称空间中的 pod，如下所示:

```
kubectl get po -n my-namespace;
NAME                                          READY   STATUS      RESTARTS   AGE
metastore-5544f95b6b-cqmkx                    1/1     Running     0          3d10h
mysql-deployment-5b68bb45bc-hs7g5             1/1     Running     0          3d10h
spark-thrift-server-b35bcc74c46273c3-driver   1/1     Running     0          3d3h
spark-thrift-server-ce356974c4636711-exec-1   1/1     Running     0          3d3h
spark-thrift-server-ce356974c4636711-exec-2   1/1     Running     0          3d3h
spark-thrift-server-ce356974c4636711-exec-3   1/1     Running     0          3d3h
```

让我们看看 spark thrift 服务器的日志:

```
kubectl logs -f spark-thrift-server-b35bcc74c46273c3-driver -n my-namespace;
```

# 运行火花三角洲湖示例作业

您可以运行 spark delta lake 示例作业来测试通过 JDBC 对 spark thrift server 的查询。

让我们先看看代码:

```
public class DeltaLakeExample {

    public static void main(String[] args) throws Exception {

        OptionParser parser = new OptionParser();
        parser.accepts("master").withRequiredArg().ofType(String.class);

        OptionSet options = parser.parse(args);

        String master = (String) options.valueOf("master");

        SparkConf sparkConf = new SparkConf().setAppName(DeltaLakeExample.class.getName());
        sparkConf.setMaster(master);

        // delta lake log store for s3.
        sparkConf.set("spark.delta.logStore.class", "org.apache.spark.sql.delta.storage.S3SingleDriverLogStore");

        SparkSession spark = SparkSession
                .*builder*()
                .config(sparkConf)
                .enableHiveSupport()
                .getOrCreate();

        // read json.
        String json = StringUtils.*fileToString*("data/test.json");
        String lines[] = json.split("\\r?\\n");
        Dataset<Row> df = spark.read().json(new JavaSparkContext(spark.sparkContext()).parallelize(Arrays.*asList*(lines)));

        df.show(10);

        // write delta to ceph.
        df.write().format("delta")
                .option("path", "s3a://mykidong/test-delta")
                .mode(SaveMode.*Overwrite*)
                .save();

        // create delta table.
        spark.sql("CREATE TABLE IF NOT EXISTS test_delta USING DELTA LOCATION 's3a://mykidong/test-delta'");

        // read delta from ceph.
        Dataset<Row> delta = spark.sql("select * from test_delta");

        delta.show(10);

        // create persistent parquet table with external path.
        delta.write().format("parquet")
                .option("path", "s3a://mykidong/test-parquet")
                .mode(SaveMode.*Overwrite*)
                .saveAsTable("test_parquet");

        // read parquet from table.
        Dataset<Row> parquet = spark.sql("select * from test_parquet");

        parquet.show(10);
    }
}
```

在 S3 上创建拼花地板数据和三角洲湖数据，并在 hive metastore 中创建 hive 表是一项简单的 spark 工作。

现在，运行示例作业。

```
cd examples/spark;# build spark uber jar.
mvn -e -DskipTests=true clean install shade:shade;# submit spark job onto kubernetes.
export MASTER=k8s://[y](https://52.231.203.153:6443)our-k8-master-url;
export NAMESPACE=my-namespace;
export ENDPOINT=[s](https://mykidong-tenant.minio.cloudchef-labs.com)3-endpoint;
export HIVE_METASTORE=metastore:9083;spark-submit \
--master ${MASTER} \
--deploy-mode cluster \
--name spark-delta-example \
--class io.mykidong.spark.examples.DeltaLakeExample \
--packages com.amazonaws:aws-java-sdk-s3:1.11.375,org.apache.hadoop:hadoop-aws:3.2.0 \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-driver-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.path=/checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.subPath=checkpoint \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.checkpointpvc.options.claimName=spark-exec-pvc \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.driver.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-driver-localdir-pvc \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.path=/localdir \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.mount.readOnly=false \
--conf spark.kubernetes.executor.volumes.persistentVolumeClaim.spark-local-dir-localdirpvc.options.claimName=spark-exec-localdir-pvc \
--conf spark.kubernetes.file.upload.path=s3a://mykidong/spark-thrift-server \
--conf spark.kubernetes.container.image.pullPolicy=Always \
--conf spark.kubernetes.namespace=$NAMESPACE \
--conf spark.kubernetes.container.image=mykidong/spark:v3.0.0 \
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
--conf spark.hadoop.hive.metastore.client.connect.retry.delay=5 \
--conf spark.hadoop.hive.metastore.client.socket.timeout=1800 \
--conf spark.hadoop.hive.metastore.uris=thrift://${HIVE_METASTORE} \
--conf spark.hadoop.hive.server2.enable.doAs=false \
--conf spark.hadoop.hive.server2.thrift.http.port=10002 \
--conf spark.hadoop.hive.server2.thrift.port=10016 \
--conf spark.hadoop.hive.server2.transport.mode=binary \
--conf spark.hadoop.metastore.catalog.default=spark \
--conf spark.hadoop.hive.execution.engine=spark \
--conf spark.hadoop.hive.input.format=io.delta.hive.HiveInputFormat \
--conf spark.hadoop.hive.tez.input.format=io.delta.hive.HiveInputFormat \
--conf spark.sql.warehouse.dir=s3a:/mykidong/apps/spark/warehouse \
--conf spark.hadoop.fs.defaultFS=s3a://mykidong \
--conf spark.hadoop.fs.s3a.access.key=my-access-key \
--conf spark.hadoop.fs.s3a.secret.key=my-secret-key \
--conf spark.hadoop.fs.s3a.connection.ssl.enabled=true \
--conf spark.hadoop.fs.s3a.endpoint=$ENDPOINT \
--conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
--conf spark.hadoop.fs.s3a.fast.upload=true \
--conf spark.hadoop.fs.s3a.path.style.access=true \
--conf spark.driver.extraJavaOptions="-Divy.cache.dir=/tmp -Divy.home=/tmp" \
--conf spark.executor.instances=3 \
--conf spark.executor.memory=2G \
--conf spark.executor.cores=1 \
--conf spark.driver.memory=1G \
file:///<src>/examples/spark/target/spark-example-1.0.0-SNAPSHOT-spark-job.jar \
--master ${MASTER};
```

完成这项工作后，一些数据将被保存在 S3 桶，并在配置单元中创建拼花表和三角洲湖表进行查询。

# 通过 JDBC 连接到 Spark 节俭服务器

我们可以通过 JDBC 用直线连接到 Spark Thrift 服务器。

```
cd <spark-home>;bin/beeline -u jdbc:hive2://$(kubectl get svc spark-thrift-server-service -n my-namespace -o jsonpath={.status.loadBalancer.ingress[0].ip}):10016;
```

您可以在其中键入一些查询:

```
show tables;
select * from test_parquet;
select * from test_delta;
select count(*) from test_delta;
...
```

就是这样！

# 参考

*   参见[Spark Thrift Server with data roaster Spark Operator](/hive-on-spark-with-spark-operator-9a43ea7ebe06)
*   【https://github.com/cloudcheflabs/dataroaster 
*   [demo](https://youtu.be/AeqkkQDwPqY) 展示了如何使用 [DataRoaster](https://github.com/cloudcheflabs/dataroaster) 轻松创建一个由运行在 Kubernetes 上的 hive metastore、spark thrift server、trino、redash 和 jupyterhub 等组成的数据平台。