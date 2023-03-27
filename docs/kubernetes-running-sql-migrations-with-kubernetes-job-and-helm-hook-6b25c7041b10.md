# Kubernetes:使用 Kubernetes Job 和 Helm hook 运行 SQL 迁移

> 原文：<https://itnext.io/kubernetes-running-sql-migrations-with-kubernetes-job-and-helm-hook-6b25c7041b10?source=collection_archive---------0----------------------->

![](img/e8b650911a5563fa094f6d923d4a4a95.png)

我们有一个在 Kubernetes 中运行的项目，它需要在部署期间运行 SQL 迁移。

要运行迁移，需要克隆一个 Github 存储库，并实际运行存储在其中的迁移。

目前，这是通过 [Kubernetes initContainers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) 完成的，有两个——第一个使用`git`将带有迁移文件的存储库克隆到 [Kubernetes 卷](https://kubernetes.io/docs/concepts/storage/volumes/)中，然后另一个使用`[sql-migrate](https://github.com/rubenv/sql-migrate)`从这个共享卷运行这些迁移。

尽管如此，这种方法还是有一些问题:

1.  每次当我们运行一个新的 pod 时，它都会运行它的 initContainers 和迁移
2.  如果一个部署启动了几个单元，每个单元将运行迁移
3.  如果迁移将运行一段时间，并且不会对 Kubernetes 的就绪状态做出响应，那么它可能会在完成迁移之前就被终止

为了避免上述所有情况，让我们重新配置这个过程，使用一个 [Kubernetes 作业](https://kubernetes.io/docs/concepts/workloads/controllers/job/)来只运行一个 pod，并添加 [Helm Hooks](https://helm.sh/docs/topics/charts_hooks/) 来在部署期间触发迁移。

*注意:这里的 kk 是 kubectl 的别名。*

# 准备

## Docker 图像

首先，让我们用 21 点和 https://github.com/rubenv/sql-migrate`git`和[创造我们自己的码头工人形象。](https://github.com/rubenv/sql-migrate)

创建 Dockerfile 文件:

```
FROM golang:alpine AS builder
RUN apk add --no-cache git gcc g++
RUN go get -v github.com/rubenv/sql-migrate/sql-migrate
RUN mv /go/bin/sql-migrate /bin/sql-migrate
```

构建并推动它:

```
$ docker build -t projectname/sql-migrate-git .
$ docker push projectname/sql-migrate-git
```

## Git 认证

第二件事是 Github 认证。

此时，我们的 git-container 通过一个保存在 Kubernetes Secrets 中的 RSA 密钥进行身份验证，然后它通过一个环境变量传递给一个 pod，bash 脚本`/opt/git/git.sh`从这个环境变量获取这个密钥，这个脚本用于在容器内部创建一个密钥文件`/root/.ssh/id_rsa`，这个密钥最终用于身份验证。

我们部署中的`initContainers`目前看起来像是下一个:

```
...
      initContainers:
      - name: git-clone
        image: projectname/git-cloner
        env:
        - name: SSH_PRIVATE_KEY
          valueFrom:
            secretKeyRef:
              name: git-ssh-key
              key: id_rsa
        - name: REPOSITORY_URL
          value: {{ .Values.git.repo }}
        - name: GIT_BRANCH
          value: {{ .Values.git.branch }}
        command: ['sh', '-c', '/opt/git/git.sh']
        volumeMounts:
          - name: git-volume
            mountPath: "/git"
            readOnly: false
      - name: init-migration
        image: fufuhu/sql-migrate:latest
        command: ['sh', '-c', 'while [ ! -d /git/db/migrations ]; do sleep 2; done && sleep 2; /bin/sql-migrate up -config=/config/config.yaml -env=main']
        volumeMounts:
          - name: migration-config
            mountPath: "/config/"
            readOnly: true
          - name: git-volume
            mountPath: "/git"
            readOnly: false
...
```

很多步骤，很多对象，复杂的过程。

相反，让我们使用一个登录名和一个 [Github 令牌](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token)，它将从环境变量传递到我们的容器中，然后我们可以通过 HTTPS 克隆这个库。

让我们来测试一下:

```
~ # export GIT_AUTHUSER=backend-user
~ # export GIT_AUTHKEY=cdc***0fe
~ # git clone
[https://$GIT_AUTHUSER:$GIT_AUTHKEY@github.com/projectname-dev/backend-services.git](https://$GIT_AUTHUSER:$GIT_AUTHKEY@github.com/projectname-dev/backend-services.git)
Cloning into ‘backend-services’…
…
Receiving objects: 100% (5115/5115), 846.55 KiB | 1.30 MiB/s, done.
Resolving deltas: 100% (2826/2826), done.
```

很好，“有用”

# Kubernetes 中的 SQL 迁移

现在，我们可以开始编写一个清单文件`templates/appname-api-migrations.yaml`来描述我们的 Kubernetes 任务，这个任务稍后将由一个 Helm hook 触发。

## 库伯内特工作

## `git clone`

首先，为了确保作业正常运行——让我们在不使用 Helm 变量和值的情况下编写它，所有 pod 的环境变量都将被设置为明文值，这里的操作暂时为`git clone`:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: "migration-job"
  labels:
  annotations:
spec: 
  backoffLimit: 0
  template:
    metadata:
      name: "migration-job-pod"
    spec:
      restartPolicy: Never
      containers:
      - name: db-migrations
        image: projectname/sql-migrate-git:latest
        command: ["/bin/sh", "-c"]
        args:
          - git clone --single-branch --branch develop [https://backend-user:cdc***0fe@github.com/projectname/backend-services.git](https://backend-user:cdc***0fe@github.com/projectname/backend-services.git) &&
            ls -l backend-services/db/migrations
```

在这里的`[restartPolicy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)`中，我们已经设置为如果失败就不重启容器，因为我们希望看到迁移失败，并且在`backoffLimit=0`中也是如此——如果失败就不重新创建 pod，并且只让作业处于*失败*状态。

在`git clone`中，将从 Jenkins 作业中设置一个分支，用户和 URL 将被设置在`values.yaml`中，认证令牌将与 [Helm secrets](https://rtfm.co.ua/en/helm-helm-secrets-sensitive-data-encryption-with-aws-kms-and-use-it-from-jenkins/) 一起保存，稍后会将其移动到一个环境变量中。

创建作业:

```
$ kk -n eks-dev-1-appname-api-ns apply -f appname-api-jobs.yaml
job.batch/migration-job created
```

检查其日志:

```
$ kk -n eks-dev-1-appname-api-ns logs job/migration-job
Cloning into ‘backend-services’…
total 20
-rw-r — r — 1 root root 538 Oct 24 12:20 BS_1_init_schema.up.sql
-rw-r — r — 1 root root 180 Oct 24 12:20 BS_2_add_brand_field.up.sql
-rw-r — r — 1 root root 225 Oct 24 12:20 BS_3_alter_table.up.sql
-rw-r — r — 1 root root 194 Oct 24 12:20 BS_4_add_created_at_field.sql
-rw-r — r — 1 root root 272 Oct 24 12:20 BS_5_alter_table_nourishment_diet.up.sql
```

存储库已克隆，现在可以访问迁移文件。

检查 pod 的状态:

```
$ kk -n eks-dev-1-appname-api-ns get pod
NAME READY STATUS RESTARTS AGE
migration-job-f72vs 0/1 Completed 0 9s
```

以及作业的状态:

```
$ kk -n eks-dev-1-appname-api-ns get job
NAME COMPLETIONS DURATION AGE
migration-job 1/1 2s 5s
```

现在可以继续进行准确的迁移过程了。

## `Secret`

要运行迁移，我们需要创建一个配置文件，该文件将存储在 [Kubernetes ConfigMap](https://rtfm.co.ua/kubernetes-configmaps-i-secrets-na-primere-gorush-servera/) 中，但是在这个文件中，必须设置数据库的密码。

将它明文存储在配置图和文件中并不是一个好主意，但是`sql-migrate`允许我们在文件中使用一个环境变量，查看它的文档-[https://github.com/rubenv/sql-migrate#as-a-standalone-tool](https://github.com/rubenv/sql-migrate#as-a-standalone-tool)

因此，我们将为 pod 创建一个名为`$DB_PASSWORD`的变量，并将实际密码保存在 Kubernetes Secrets 中，稍后在 Helm 中，我们将使用 [Helm secrets](https://rtfm.co.ua/en/helm-helm-secrets-sensitive-data-encryption-with-aws-kms-and-use-it-from-jenkins/) 将其加密存储在图表值中。

同样在这个秘密中，我们将为在`git clone`命令中使用的`$GIT_TOKEN`环境变量存储一个值。

还有一个`templates/appname-api-migrations.yaml`加一个`Secret`:

```
...
---
apiVersion: v1
kind: Secret
metadata:
  name: backend-db-password
type: Opaque
stringData:
  db_password: password
  git_token: cdc***0fe
```

在作业的`spec.containers.env`中添加变量并更新`git clone`以使用`$GIT_TOKEN`变量:

```
...
      containers:
      - name: db-migrations
        image: projectname/sql-migrate-git:latest
        command: ["/bin/sh", "-c"]
        args:
          - git clone --single-branch --branch develop [https://backend-user:$GIT_TOKEN@github.com/projectnamev/backend-services.git](https://backend-user:$GIT_TOKEN@github.com/projectnamev/backend-services.git) &&
            ls -l backend-services/db/migrations;
            cat /config/config.yaml
        env:
        - name: GIT_TOKEN
          valueFrom:
            secretKeyRef:
              name: backend-db-password
              key: git_token
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: backend-db-password
              key: db_password
...
```

## `ConfigMap`

接下来，创建一个`ConfigMap`来保存`sql-migrate`的`/config/config.yaml`内容，并在其中使用`$DB_PASSWORD`变量:

```
...
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: migration-config
data:
  config.yaml: |
    main:
      dialect: mysql
      datasource: backend-user:${DB_PASSWORD}@tcp(stage.backend-db3-master.example.com:3306)/dbname?parseTime=true
      dir: backend-services/db/migrations
      table: backend_services_migrations
```

在任务的 pod 模板中添加`volumes`，通过`volumeMounts`在`spec.containers`中挂载配置图为`/config/config.yaml`文件的卷。

该作业的完整清单如下:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: "migration-job"
  labels:
  annotations:
spec: 
  backoffLimit: 0
  template: 
    metadata:
      name: "migration-job-pod"
    spec:
      restartPolicy: Never
      containers:
      - name: db-migrations
        image: projectname/sql-migrate-git:latest
        command: ["/bin/sh", "-c"]
        args:
          - git clone --single-branch --branch develop [https://backend-user:$GIT_TOKEN@github.com/projectname/backend-services.git](https://backend-user:$GIT_TOKEN@github.com/projectname/backend-services.git) &&
            ls -l backend-services/db/migrations;
            cat /config/config.yaml
        env:
        - name: GIT_TOKEN
          valueFrom:
            secretKeyRef:
              name: backend-db-password
              key: git_token
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: backend-db-password
              key: db_password
        volumeMounts:
        - name: migration-config
          mountPath: "/config/config.yaml"
          subPath: "config.yaml"
          readOnly: true
      volumes:
        - name: migration-config
          configMap: 
            name: migration-config
            items:
            - key: "config.yaml"
              path: "config.yaml"
...
```

运行它:

```
$ kk -n eks-dev-1-appname-api-ns apply -f appname-api-jobs.yaml
job.batch/migration-job created
secret/backend-db-password created
configmap/migration-config created
```

检查:

```
$ kk -n eks-dev-1-appname-api-ns logs job/migration-job
Cloning into ‘backend-services’…
total 20
-rw-r — r — 1 root root 538 Oct 24 13:41 BS_1_init_schema.up.sql
-rw-r — r — 1 root root 180 Oct 24 13:41 BS_2_add_brand_field.up.sql
-rw-r — r — 1 root root 225 Oct 24 13:41 BS_3_alter_table.up.sql
-rw-r — r — 1 root root 194 Oct 24 13:41 BS_4_add_created_at_field.sql
-rw-r — r — 1 root root 272 Oct 24 13:41 BS_5_alter_table_nourishment_diet.up.sql
main:
dialect: mysql
datasource: backend-user:${DB_PASSWORD}@tcp(stage.backend-db3-master.example.com:3306)/dbname?parseTime=true
dir: backend-services/db/migrations
table: backend_services_migrations
```

好—存储库已克隆，配置文件已创建。

## 运行迁移

现在我们可以用`-dryrun`选项和第二个命令——检查其状态——来描述迁移过程:

```
...
        args:
          - git clone --single-branch --branch develop [https://backend-user:$GIT_TOKEN@github.com/projectname/backend-services.git](https://backend-user:$GIT_TOKEN@github.com/projectname/backend-services.git) &&
            ls -l backend-services/db/migrations &&
            cat /config/config.yaml &&
            /bin/sql-migrate up -config=/config/config.yaml -env=main -dryrun &&
            /bin/sql-migrate status -config=/config/config.yaml -env=main
...
```

运行，并检查其日志:

```
$ kk -n eks-dev-1-appname-test-migrations-ns logs job/migration-job
Cloning into ‘backend-services’…
total 20
-rw-r — r — 1 root root 538 Oct 24 14:02 BS_1_init_schema.up.sql
-rw-r — r — 1 root root 180 Oct 24 14:02 BS_2_add_brand_field.up.sql
-rw-r — r — 1 root root 225 Oct 24 14:02 BS_3_alter_table.up.sql
-rw-r — r — 1 root root 194 Oct 24 14:02 BS_4_add_created_at_field.sql
-rw-r — r — 1 root root 272 Oct 24 14:02 BS_5_alter_table_nourishment_diet.up.sql
main:
dialect: mysql
datasource: backnd-user:${DB_PASSWORD}@tcp(stage.backend-db3-master.example.com:3306)/dbname?parseTime=true
dir: backend-services/db/migrations
table: backend_services_migrations
+ — — — — — — — — — — — — — — — — — — — — — + — — — — — — — — — — — — — — — -+
| MIGRATION | APPLIED |
+ — — — — — — — — — — — — — — — — — — — — — + — — — — — — — — — — — — — — — -+
| BS_1_init_schema.up.sql | 2020–05–07 12:21:25 +0000 UTC |
| BS_2_add_brand_field.up.sql | 2020–05–12 14:31:17 +0000 UTC |
| BS_3_alter_table.up.sql | 2020–05–13 06:17:25 +0000 UTC |
| BS_4_add_created_at_field.sql | 2020–07–21 09:55:49 +0000 UTC |
| BS_5_alter_table_nourishment_diet.up.sql | 2020–07–21 09:55:49 +0000 UTC |
+ — — — — — — — — — — — — — — — — — — — — — + — — — — — — — — — — — — — — — -+
```

现在可以去舵轮图了。

# 舵模板

在旧图表中我们需要做什么？

1.  拆下`initContainers`
2.  消除旧的秘密
3.  在`values.yaml`中移动新变量的值
4.  将令牌和数据库的密码移入`secrets.yaml`

这里的主要部分是添加注释，以在 Helm 部署期间触发迁移过程

将`annotations`添加到工作中:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Chart.Name }}-migration-job
  labels:
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-1"
    "helm.sh/hook-delete-policy": before-hook-creation
spec: 
  backoffLimit: 0
...
```

这里:

*   `"helm.sh/hook": pre-install,pre-upgrade`:在`helm install`或`upgrade`之前运行作业(在我们的 Jenkins 管道中，从`helm secrets upgrade --install`开始)
*   `"helm.sh/hook-weight": "-1"`:创建资源的优先级，因为首先我们需要创建我们的作业将使用的`ConfigMap`和`Secret`，所以将它们的权重设置为小于作业的权重
*   `"helm.sh/hook-delete-policy"`:默认值为*创建钩子前的*(查看[文档](https://helm.sh/docs/topics/charts_hooks/))，设置为 if 用于测试，然后可以更改为*钩子成功*(但是如果迁移失败，您将无法查看日志)

将`annotations`块添加到配置图中，并在作业中使用小于`hook-weight`的密码。

`ConfigMap`清单现在全部内容:

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: migration-config
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
data:         
  config.yaml: |
    main:
      dialect: {{ .Values.backendConfig.db.driver }}
      datasource: {{ .Values.backendConfig.db.user }}:${DB_PASSWORD}@tcp({{ .Values.backendConfig.db.host }}:{{ .Values.backendConfig.db.port }})/{{ .Values.backendConfig.db.database }}?parseTime=true
      dir: backend-services/db/migrations
      table: {{ .Values.backendConfig.db.migrationsTable }}
```

`Secret`一:

```
---     
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Chart.Name }}-migration-secrets
  annotations: 
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-10"
    "helm.sh/hook-delete-policy": before-hook-creation
type: Opaque
stringData:
  backend-db-password: {{ .Values.backendConfig.db.password }}
  git_token: {{ .Values.git.token }}
```

而`Job`:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Chart.Name }}-migration-job
  labels:
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-1"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  backoffLimit: 0
  template:
    metadata:
      name: {{ .Chart.Name }}-migration-job-pod
    spec:
      restartPolicy: Never
      containers:
      - name: {{ .Chart.Name }}-db-migrations
        image: projectname/sql-migrate-git:latest
        command: ["/bin/sh", "-c"]
        args: 
          - git clone --single-branch --branch {{ .Values.git.branch }} [https://{{](/{{) .Values.git.user }}:$GIT_TOKEN@{{ .Values.git.repo }} &&
            ls -l backend-services/db/migrations &&
            cat /config/config.yaml &&
            /bin/sql-migrate up -config=/config/config.yaml -env=main || exit 1;
            /bin/sql-migrate status -config=/config/config.yaml -env=main
        env:
        - name: GIT_TOKEN
          valueFrom:
            secretKeyRef:
              name: {{ .Chart.Name }}-migration-secrets
              key: git_token
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Chart.Name }}-migration-secrets
              key: backend-db-password
        volumeMounts:
        - name: migration-config
          mountPath: "/config/config.yaml"
          subPath: "config.yaml"
          readOnly: true
      volumes:
        - name: migration-config
          configMap: 
            name: migration-config
            items:
            - key: "config.yaml"
              path: "config.yaml"
```

在这里，我将`exit 1`添加到了`/bin/sql-migrate up`中，这样，如果在迁移过程中出现错误，作业将*失败*，从而不会启动部署过程。

在詹金斯运行它:

![](img/48ea1d8f3992c933939b78613bbc3f6c.png)![](img/a6498e362a48d431484514d19c5f7d97.png)

在*钩子*中，我们可以看到，首先创建了具有`hook-weight": "-10"`的秘密，然后是配置图，最后是作业。

现在，部署流程如下所示:

![](img/e227aaa2100a329cefad0826386eca90.png)

首先，秘密的 ConfigMap и作业资源被移除(根据`"helm.sh/hook-delete-policy": before-hook-creation`)，然后被创建。

检查作业的状态:

```
$ kk -n eks-stage-1-appname-api-ns get job
NAME COMPLETIONS DURATION AGE
appname-api-migration-job 1/1 3s 6m21s
```

它的日志:

```
$ kk -n eks-stage-1-appname-api-ns logs job/appname-api-migration-job
Cloning into ‘backend-services’…
total 20
-rw-r — r — 1 root root 538 Oct 26 11:32 BS_1_init_schema.up.sql
-rw-r — r — 1 root root 180 Oct 26 11:32 BS_2_add_brand_field.up.sql
-rw-r — r — 1 root root 225 Oct 26 11:32 BS_3_alter_table.up.sql
-rw-r — r — 1 root root 194 Oct 26 11:32 BS_4_add_created_at_field.sql
-rw-r — r — 1 root root 272 Oct 26 11:32 BS_5_alter_table_nourishment_diet.up.sql
main:
dialect: mysql
datasource: backend-user:${DB_PASSWORD}@tcp(stage.backend-db3-master.example.com:3306)/dbname?parseTime=true
dir: backend-services/db/migrations
table: backend_services_migrations
Applied 0 migrations
+ — — — — — — — — — — — — — — — — — — — — — + — — — — — — — — — — — — — — — -+
| MIGRATION | APPLIED |
+ — — — — — — — — — — — — — — — — — — — — — + — — — — — — — — — — — — — — — -+
| BS_1_init_schema.up.sql | 2020–05–07 12:21:25 +0000 UTC |
| BS_2_add_brand_field.up.sql | 2020–05–12 14:31:17 +0000 UTC |
| BS_3_alter_table.up.sql | 2020–05–13 06:17:25 +0000 UTC |
| BS_4_add_created_at_field.sql | 2020–07–21 09:55:49 +0000 UTC |
| BS_5_alter_table_nourishment_diet.up.sql | 2020–07–21 09:55:49 +0000 UTC |
+ — — — — — — — — — — — — — — — — — — — — — + — — — — — — — — — — — — — — — -+
```

***应用了 0 个迁移*** ，因为在最后一个`APPLIED`之后迁移文件没有变化。

完成了。

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/kubernetes-running-sql-migrations-with-kubernetes-job-and-helm-hook/) *。*