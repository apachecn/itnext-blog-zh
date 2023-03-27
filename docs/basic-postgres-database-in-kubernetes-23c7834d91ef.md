# Kubernetes 中的基本 Postgres 数据库

> 原文：<https://itnext.io/basic-postgres-database-in-kubernetes-23c7834d91ef?source=collection_archive---------1----------------------->

![](img/45b0a47202648b93169e9f4e31bb6dc2.png)

在本文中，我们将在 Kubernetes 中创建一个 Postgres 数据库实例，然后使用 cronjob 连接到这个实例。目标是通过这个练习学习 Kubernetes 的基础知识。对于学习 K8s 基础，我强烈推荐[这门](https://kodekloud.com/)初学者课程。

# 设置

*   首先使用[这里的](https://minikube.sigs.k8s.io/docs/start/)中的 [i](https://minikube.sigs.k8s.io/docs/start/) 指令设置一个 **minikube** Kubernetes 集群(我们可以使用其他任何东西，比如 GKE、EKS、K3 等等。). **minikube** 让你在本地运行 Kubernetes。 **minikube** 在您的个人电脑上运行单节点 Kubernetes 集群。
*   安装 [helm](https://helm.sh/docs/intro/install/) 来管理 Kubernetes 应用程序。管理多个 Kubernetes YAML 文件变得复杂，这就是赫尔姆帮助缓解事情。

# **验证设置**

1.  使用，`minikube status`命令检查 minikube 是否启动并运行

```
Let's try K8 : minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

2.验证舵是否安装正确

```
Let's try K8 : helm version
version.BuildInfo{Version:"v3.3.4", GitCommit:"a61ce5633af99708171414353ed49547cf05013d", GitTreeState:"dirty", GoVersion:"go1.15.2"}
```

# 安装 Postgres

1.  [添加要使用的 helm](https://helm.sh/docs/helm/helm_repo_add/) 的回购。我将在这里使用 bitnami 回购。你可以在这里找到他们的文件。如果我们搜索 bitnami repo，我们会发现他们提供了两个版本的 PostgreSQL，让我们使用第一个版本

```
Let's try K8 : helm search repo postgres
NAME                                CHART VERSION APP VERSION DESCRIPTION
bitnami/postgresql                  9.8.3         11.9.0      Chart for PostgreSQL, an object-relational data...
bitnami/postgresql-ha               4.0.1         11.9.1      Chart for PostgreSQL with HA architecture (usin...
```

2.让我们安装图表

```
helm install postgres bitnami/postgresql
```

让我们看看图表现在是否显示出来

```
Let's try K8 : helm list
NAME     NAMESPACE REVISION UPDATED                              STATUS   CHART            APP VERSION
postgres default   1        2020-10-24 10:57:00.678762 -0700 PDT deployed postgresql-9.8.3 11.9.0
```

3.验证 PostgreSQL 是否按预期运行

```
helm get notes postgres
```

验证所有服务和单元是否都已启动并运行。有趣的是，我们安装了一个持久卷、持久卷声明和 StatefulSet。这就是 helm 的美妙之处，它简化了同时安装多个 Kubernetes 对象，如果我们不使用 helm，我们将不得不创建多个 Kubernetes YAML 文件，这可能是一场管理噩梦。

```
Every 2.0s: kubectl get all -o wide NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
pod/postgres-postgresql-0   1/1     Running   0          3m32s   172.17.0.3   minikube   <none>           <none>NAME                                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE    SELECTOR
service/kubernetes                     ClusterIP   10.x.x.x        <none>        443/TCP    43h    <none>
service/postgres-postgresql            ClusterIP   10.x.x.x
<none>        5432/TCP   5m5s   app.kubernetes.io/instance=postgres,app.kubernetes.io/name=pos
tgresql,role=master
service/postgres-postgresql-headless   ClusterIP   None             <none>        5432/TCP   5m5s   app.kubernetes.io/instance=postgres,app.kubernetes.io/name=pos
tgresqlNAME                                   READY   AGE    CONTAINERS            IMAGES
statefulset.apps/postgres-postgresql   1/1     5m5s   postgres-postgresql   docker.io/bitnami/postgresql:11.9.0-debian-10-r48
```

让我们连接到 PostgreSQL 并验证我们可以创建一个表并插入值

```
Let's try K8 : export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)Let's try K8 : kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:11.9.0-debian-10-r48 --env="PGPASSWORD=$POSTGRES_PASSWORD" --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432
```

看起来我们可以从数据库中插入和查询

```
postgres=# CREATE TABLE phonebook(phone VARCHAR(32), firstname VARCHAR(32), lastname VARCHAR(32), address VARCHAR(64));
CREATE TABLEpostgres=# INSERT INTO phonebook(phone, firstname, lastname, address) VALUES('+1 123 456 7890', 'John', 'Doe', 'North America');
INSERT 0 1postgres=# SELECT * FROM phonebook ORDER BY lastname;
      phone      | firstname | lastname |    address
-----------------+-----------+----------+---------------
 +1 123 456 7890 | John      | Doe      | North America
(1 row)
```

# 添加一个可以每分钟连接到数据库的 cronjob

1.  编写一个脚本来连接到数据库

让我们使用一个简单的 python 脚本`connect_db.py`来连接数据库。我们使用以下参数。

```
Username => postgres
Password => $POSTGRES_PASSWORD variable as found previosuly
DB name  => postgres
```

连接到数据库的 Python 脚本

```
import sqlalchemy as dbengine = db.create_engine('postgresql://postgres:mq4Jq3C8zK@postgres-postgresql:5432/postgres')connection = engine.connect()metadata = db.MetaData(bind=connection, reflect=True)print(metadata)
```

2.作为 Kubernetes 集群的一部分，我们如何运行这个脚本呢？

A.我们可以将它添加为 Docker 映像的一部分和在 POD 中运行的容器。让我们首先创建一个映像来运行这个脚本。

**requirements.txt**

```
sqlalchemy==1.3.16
psycopg2-binary
```

**Dockerfile**

```
FROM python:3.8-slimWORKDIR /appCOPY requirements.txt /appCOPY connect_db.py /appRUN pip install -r requirements.txtCMD python connect_db.py
```

**connect_db.py**

```
import sqlalchemy as dbengine = db.create_engine('postgresql://postgres:mq4Jq3C8zK@postgres-postgresql:5432/postgres')connection = engine.connect()metadata = db.MetaData(bind=connection, reflect=True)print(metadata)
```

B.现在让我们来构建图像。

```
docker build -t postgres_image .
Sending build context to Docker daemon  6.144kB
Step 1/6 : FROM python:3.8-slim
 ---> 41dcfe21e8fd
Step 2/6 : WORKDIR /app
 ---> Using cache
 ---> 00a8320e3c52
Step 3/6 : COPY requirements.txt /app
 ---> Using cache
 ---> e67a21cc9bb8
Step 4/6 : COPY connect_db.py /app
 ---> Using cache
 ---> 38a6058e0000
Step 5/6 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 07495bf00820
Step 6/6 : CMD python connect_db.py
 ---> Using cache
 ---> 97b09c86daa9
Successfully built 97b09c86daa9
Successfully tagged postgres_image:latest
```

C.让我们创建一个 cronjob，它尝试使用一个 YAML 文件在 Kubernetes 集群中每分钟运行我们刚刚编写的脚本(根据我们的要求)。 [Kubernetes 的文档](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)也有同样的细节。

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: connect
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: connect
            image: postgres_image
          restartPolicy: OnFailure
```

让我们使用 kubectl 应用同样的方法

```
kubectl apply -f c.yml
cronjob.batch/connect created
```

让我们验证 cronjob

```
kubectl get cronjobs
NAME      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
connect   */1 * * * *   False     1        13s             72s
```

让我们验证 pod，看起来它正在运行

```
kubectl get po
NAME                       READY   STATUS             RESTARTS   AGE
connect-1603564500-6dk2m   0/1     ImagePullBackOff   0          2m18s
connect-1603564560-q6d4t   0/1     ImagePullBackOff   0          78s
connect-1603564620-dtrgf   0/1     ImagePullBackOff   0          18s
postgres-postgresql-0      1/1     Running            0          29m
```

但是，如果我们查看 POD 日志，我们会发现找不到该映像

```
kubectl logs connect-1603564560-q6d4t
Error from server (BadRequest): container "connect" in pod "connect-1603564560-q6d4t" is waiting to start: image can't be pulled
```

这是因为我们需要在本地或公共注册表中托管图像，否则 cronJob 无法找到图像。为简单起见，我们将把它放在一个公共注册表中(您也可以放在一个本地注册表中)

让我们标记我们的图像

```
docker tag postgres_image:latest XXXX/postgres_images:latest
```

让我们将图像推送到公共注册中心

```
docker push XXXX/postgres_images:latest
The push refers to repository [docker.io/XXXX/postgres_images]
1ec918490d02: Pushed
c9daa0dc3fcb: Pushed
6a53aff9114d: Pushed
8be2969ec15c: Pushed
4375d6f45f83: Pushed
06b60c6e6ffd: Pushed
322c3996a80b: Pushed
225ef82ca30a: Pushed
d0fe97fa8b8c: Pushed
latest: digest: sha256:c01442f3efe0e31de148926ad6d6e619be998d6e15d20512ac9c7368ef26ac46 size: 2203
```

现在 cronjob 可以每分钟运行一次 python 脚本来连接我们刚刚安装的 PostgreSQL DB。

# 摘要

我们刚刚安装了一个 PostgreSQL 数据库，并添加了一个每分钟运行一次的 cronjob 来连接数据库，我们在 Kubernetes 集群中添加了相同的内容。这是一个简单的例子来理解一些基本知识。在生产中运行 PostgreSQL 数据库是另一回事，我将在即将发布的帖子中分享我对这个主题的看法。