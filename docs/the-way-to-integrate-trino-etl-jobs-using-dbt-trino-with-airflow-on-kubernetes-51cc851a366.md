# 使用 dbt-trino 和 Kubernetes 上的 Airflow 集成 Trino ETL 作业的方法

> 原文：<https://itnext.io/the-way-to-integrate-trino-etl-jobs-using-dbt-trino-with-airflow-on-kubernetes-51cc851a366?source=collection_archive---------1----------------------->

![](img/8b260be24d559a311acbd2d6f4235446.png)

Todd Quackenbush 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

Trino 是数据仓库中最流行的查询引擎。最近，trino 可以用于运行长时间运行的 ETL 作业，具有容错执行配置以及交互式查询，这意味着，我认为，在大多数情况下，您可以用 trino 替换 Hive。

这种 trino etl 作业可以使用`[dbt-trino](https://github.com/starburstdata/dbt-trino)`来执行，但是如果您想要将 trino etl 作业的 dbt 模型与 kubernetes 上的 Airflow 之类的工作流集成，没有简单的方法可以做到这一点。在这里，我将讨论如何使用 dbt 将 trino etl 作业与 kubernetes 上运行的 airflow 集成。

为了理解本文中使用的代码，克隆下面的 git repo。

```
[https://github.com/mykidong/dbt-trino-example.git](https://github.com/mykidong/dbt-trino-example.git)
```

# 在本地运行 trino dbt 模型

让我们使用安装在本地机器上的 dbt 来执行 trino dbt 模型。

安装 dbt-trino。

```
sudo yum install python3-devel -y;
sudo yum groupinstall 'development tools' -y;
sudo pip3 install --upgrade pip;
sudo pip3 install dbt-core;
sudo pip3 install dbt-trino;
```

参见示例 dbt 配置文件`dbt-trino-example/components/dbt/example/profiles.yml`。

```
trino:
  target: dev
  outputs:
    dev:
      type: trino
      method: ldap
      user: anyuser
      password: anypassword
      host: trino-gateway-hostname
      port: 443
      database: iceberg
      schema: silver
      threads: 8
      http_scheme: https
      session_properties:
        query_max_run_time: 5d
        exchange_compression: True
```

要连接 trino 和 TLS，您可以使用 [trino gateway](/trino-gateway-8e654366df5e) 安装 trino cluster。看一下输出配置，`database`是指 trino 的目录，类似于`iceberg`目录，`schema`与 trino 的`schema`相同。也就是说，输出数据将在 trino 的`iceberg`目录中的`silver`模式中创建或更新。

我们来看看示例模型`dbt-trino-example/components/dbt/example/silver/models/hive_to_iceberg.sql`。

```
{{
  config(
    pre_hook = "set session query_max_run_time='10m'",
    materialized = "incremental",
    incremental_strategy = "append",
    on_table_exists = "drop",
    format = "ORC",
    using = "ICEBERG"
  )
}}
SELECT
   baseproperties.eventtype,
   itemid, 
   price 
FROM hive.default.test_parquet
```

执行该模型后，将从`hive`目录和`default`模式中的表`test_parquet`创建并追加`iceberg`目录和`silver`模式中的表`hive_to_iceberg`。这是一种将 hive 数据转换为 iceberg 表的简单方法。

现在，您可以为 trino 运行 dbt 模型。

```
cd ~/dbt-trino-example/components/dbt/example/silver;

# debug.
dbt debug --project-dir ./ --profiles-dir ../;

# run
dbt run --project-dir ./ --profiles-dir ../

# run a specific model.
dbt run --project-dir ./ --profiles-dir ../ -m models/example.sql
```

# 在 Kubernetes 上运行 trino dbt 模型

要在 kubernetes 上运行 dbt 模型，首先要构建 dbt docker 映像。

```
FROM k8s.gcr.io/git-sync/git-sync:v3.6.1

ENV *LANG*='en_US.UTF-8' *LANGUAGE*='en_US:en' *LC_ALL*='en_US.UTF-8'

ENTRYPOINT []

USER root

ENV *DEBIAN_FRONTEND* noninteractive

RUN set -eux; \
    apt-get update; \
    apt-get install -y curl --no-install-recommends; \
    apt install python3-pip -y; \
    pip3 install --upgrade pip; \
    pip3 install dbt-core==1.2.1; \
    pip3 install dbt-trino==1.2.2;

# print dbt version.
RUN dbt --version

# print git sync version.
RUN /git-sync --version

# set time zone.
ENV *TZ*="Asia/Seoul"

# print date.
RUN echo "current date: $(date)"
```

正如在 docker 文件中看到的，这个 dbt docker 映像扩展了`[git-sync](https://github.com/kubernetes/git-sync)`。我的方法是首先克隆 dbt 模型代码所在的 git repo，并从这些代码运行 dbt 模型。像这样构建和推送 dbt docker 映像。

```
cd ~/dbt-trino-example/components/dbt/docker;

chmod a+x build-docker.sh;

export TAG=1.2.2;
./build-docker.sh \
--image=cloudcheflabs/dbt:${TAG} \
;
```

把码头报告换成你的。

运行 pod 并在 pod 中执行 dbt 模型。

```
kubectl run --rm dbt -it --image cloudcheflabs/dbt:1.2.2 --image-pull-policy='Always' -- sh;
```

从 git repo 克隆 dbt 代码。

```
/git-sync \
-repo https://yourgitrepo \
-branch main \
-username anyuser \
-password yourtoken \
-root /tmp/git/dbt-trino-example \
-one-time
```

并运行指定的 trino dbt 模型。

```
cd /tmp/git/dbt-trino-example/dbt-trino-example.git/components/dbt/example/silver && \
dbt clean && \
dbt run \
--profiles-dir ../ \
--project-dir ./ \
-m models/example.sql
```

您可以运行一个命令来克隆和运行模型代码，如下所示。

```
kubectl run --rm dbt -it --image cloudcheflabs/dbt:1.2.2 --image-pull-policy='Always' -- \
/bin/sh -c '
/git-sync \
-repo https://yourgitrepo \
-branch main \
-username anyuser \
-password yourtoken \
-root /tmp/git/dbt-trino-example \
-one-time && \
cd /tmp/git/dbt-trino-example/dbt-trino-example.git/components/dbt/example/silver && \
dbt clean && \
dbt run \
--profiles-dir ../ \
--project-dir ./ \
-m models/example.sql
'
```

然后，您将看到以下日志。

```
If you don't see a command prompt, try pressing enter.
I1014 10:39:38.539098       7 main.go:748] "level"=0 "msg"="syncing git" "rev"="HEAD" "hash"="98dfabb29a8f28834428dae5737a6fad5b48a3b2"
I1014 10:39:39.592321       7 main.go:783] "level"=0 "msg"="adding worktree" "path"="/tmp/git/dbt-trino-example/98dfabb29a8f28834428dae5737a6fad5b48a3b2" "branch"="origin/main"
I1014 10:39:39.598263       7 main.go:844] "level"=0 "msg"="reset worktree to hash" "path"="/tmp/git/dbt-trino-example/98dfabb29a8f28834428dae5737a6fad5b48a3b2" "hash"="98dfabb29a8f28834428dae5737a6fad5b48a3b2"
I1014 10:39:39.598290       7 main.go:849] "level"=0 "msg"="updating submodules"
01:39:41  Running with dbt=1.2.1
01:39:41  Partial parse save file not found. Starting full parse.
01:39:42  Found 2 models, 0 tests, 0 snapshots, 0 analyses, 271 macros, 0 operations, 0 seed files, 0 sources, 0 exposures, 0 metrics
01:39:42  
01:39:43  Concurrency: 8 threads (target='dev')
01:39:43  
01:39:43  1 of 1 START incremental model silver.example .................................. [RUN]
01:39:52  1 of 1 OK created incremental model silver.example ............................. [SUCCESS in 9.78s]
01:39:52  
01:39:52  Finished running 1 incremental model in 0 hours 0 minutes and 10.83 seconds (10.83s).
01:39:52  
01:39:52  Completed successfully
01:39:52  
01:39:52  Done. PASS=1 WARN=0 ERROR=0 SKIP=0 TOTAL=1
Session ended, resume using 'kubectl attach dbt -c dbt -i -t' command when the pod is running
pod "dbt" deleted
```

# 从 kubernetes 上的气流运行 trino dbt 模型

我们已经准备好在 kubernetes 上运行 trino etl 作业的 trino dbt 模型了。要使用 helm chart 在 kubernetes 上安装 airflow，请参见[https://air flow . Apache . org/docs/helm-chart/stable/index . html](https://airflow.apache.org/docs/helm-chart/stable/index.html)。

要从 airflow 的任务在 kubernetes 上创建 dbt pod，您需要将 RBAC 添加到在 kubernetes 上运行的 airflow 的服务帐户中。

```
cat <<EOF > rbac.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: airflow
rules:
  - apiGroups:
      - '*'
    resources:
      - '*'
    verbs:
      - '*'
  - nonResourceURLs:
      - '*'
    verbs:
      - '*'

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: airflow-scheduler
  labels:
    app.kubernetes.io/name: airflow
  namespace: airflow

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: airflow-triggerer
  labels:
    app.kubernetes.io/name: airflow
  namespace: airflow

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: airflow-webserver
  labels:
    app.kubernetes.io/name: airflow
  namespace: airflow

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: airflow-worker
  labels:
    app.kubernetes.io/name: airflow
  namespace: airflow

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: airflow-scheduler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: airflow
subjects:
  - kind: ServiceAccount
    name: airflow-scheduler
    namespace: airflow

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: airflow-triggerer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: airflow
subjects:
  - kind: ServiceAccount
    name: airflow-triggerer
    namespace: airflow

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: airflow-webserver
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: airflow
subjects:
  - kind: ServiceAccount
    name: airflow-webserver
    namespace: airflow

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: airflow-worker
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: airflow
subjects:
  - kind: ServiceAccount
    name: airflow-worker
    namespace: airflow
EOF

kubectl apply -f rbac.yaml;
```

现在使用`KubernetesPodOperator`创建 DAG 来运行 trino dbt 模型。

```
dbt_generate_command = f"""
        git-sync \
        -repo https://yourgitrepo \
        -branch main \
        -username anyuser \
        -password yourtoken \
        -root /tmp/git/dbt-trino-example \
        -one-time && \
        cd /tmp/git/dbt-trino-example/dbt-trino-example.git/components/dbt/example/silver && \
        dbt clean && \
        dbt run \
        --profiles-dir ../ \
        --project-dir ./ \
        -m models/example.sql
        """

t = KubernetesPodOperator(
    task_id="run-dbt-model",
    namespace='dbt',
    image='cloudcheflabs/dbt:1.2.2',
    name="dbt-trino-job",
    is_delete_operator_pod=True,
    get_logs=True,
    cmds=["/bin/sh", "-c", dbt_generate_command],
    dag=dag,
)
```

`dbt_generate_command`与前一节中用于从 git repo 克隆 dbt 模型和运行 dbt 模型的命令相同。任务`t`将创建 dbt pod 并运行命令在 kubernetes 上执行 trino dbt 模型。

就是这样。