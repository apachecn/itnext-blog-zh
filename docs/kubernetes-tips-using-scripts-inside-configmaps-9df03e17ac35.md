# Kubernetes 提示|在配置图中使用脚本

> 原文：<https://itnext.io/kubernetes-tips-using-scripts-inside-configmaps-9df03e17ac35?source=collection_archive---------0----------------------->

![](img/d8100e3fe169f140dcfd69abd050c981.png)

[https://cncf-branding.netlify.app/projects/kubernetes/](https://cncf-branding.netlify.app/projects/kubernetes/)

在过去十年的大部分时间里，我在德沃普斯/SRE 世界工作，我学到的一件事是，快速行动是生存的唯一途径。在一个行业、一个组织中，事情会在一瞬间发生变化，而你的工作量永远不会减少。显然，我们都是为了效率和质量而建设，但我们也必须牢记速度。在这个领域，脚本就是生命。没有脚本，自动化就无法成功。在我的整个职业生涯中，我真的想不出有哪一周我没有编写一些脚本来实现一些自动化。在你意识到之前，你已经有成百上千的东西要维护了。

当使用包含 Kubernetes 的架构时，我们如何管理和维护所有这些脚本来做所有这些美妙的事情？一种方法是创建一个完整的专用回购协议，然后构建一个包含所有这些脚本的容器映像，称为类似“自动化”的东西。这并不完全是个坏主意，因为我们都应该遵循 GitOps 实践，我永远不会反对这一点(GitOps always。一直都是。一直都是。).

在过去的几年里，我发现一个非常有用的方法是将这些脚本存储为*配置图*。这使我能够更轻松地管理自动化所需的所有这些定制脚本，并更快地进行部署。在接下来的文章中，我将演示一些在*配置图*中放置脚本的场景，来演示如何做到这一点。

*对于我们的 K8s 环境，我们将使用* [*种类*](https://kind.sigs.k8s.io/) *。有关 KiND 入门的更多信息，请参见文档。这是我比较喜欢的本土开发 K8s。*

## 场景 1:用 Bash 执行一个简单的任务

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: slim-shady-configmap
data:
  slim-shady.sh: |
    #!/bin/bash

    echo "Hi!"
    echo "My name is"
    echo "What?"
    echo "My name is"
    echo "Who?"
    echo "My name is"
    echo "Chika-chika"
    echo "Slim Shady"
```

在上面的 *configmap* 中，我们创建了一个名为“slim-shady.sh”的 bash 脚本，它模仿了 Eminem 的歌词[“我的名字是”](https://youtu.be/sNPnbI1arSE)(**请注意**这不是一个真实的用例🙂但却是我最喜欢的歌曲之一)。要使用*配置图*运行这个脚本，让我们看一下下面的例子 [Kubernetes Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) 并浏览一下。

```
apiVersion: batch/v1
kind: Job
metadata:
  name: chicka-chicka-slim-shady
spec:
  template:
    spec:
      containers:
        - name: shady
          image: centos
          command: ["/script/slim-shady.sh"]
          volumeMounts:
            - name: script
              mountPath: "/script"
      volumes:
        - name: script
          configMap:
            name: slim-shady-configmap
            defaultMode: 0500
      restartPolicy: Never
```

在上面的作业清单中，您可以看到我们为配置图创建了一个新的[卷。然后，我们将该卷安装在位于 *"/script"* 的 shady 容器下，然后执行脚本文件 *slim-shady.sh* ，这是我们的*配置图*中的文件名。](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

我想在这里指出两件事:

a)在卷内，我们必须指定一个至少为 *0500* 的 *defaultMode* (读取+执行给用户)。这定义了当 pod 运行时容器内的文件权限。如果我们不设置这个，我们会得到权限被拒绝的错误。这是因为默认情况下，configmap 将与 *0644* 一起挂载，您将会看到如下错误。

```
 Warning  Failed     7s    kubelet            Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/script/slim-shady.sh": permission denied: unknown
```

b)在这里，我们还能够使用现有的众所周知的容器映像，而不是创建我们自己的映像。这对我们来说更容易，因为我们不需要用新代码等构建新的映像，而是只需要部署*配置图*和 [*作业*](https://kubernetes.io/docs/concepts/workloads/controllers/job/) 。

在我们应用工作清单之后，让我们看一下日志。

```
> kubectl logs -f chicka-chicka-slim-shady-h7j5m
Hi!
My name is
What?
My name is
Who?
My name is
Chika-chika
Slim Shady
```

## 场景 2:在 MLflow 中推广最新的 ML 模型

现在是一个更真实的例子，但仍然是一个非常简单的设置。以下内容遵循与上述相同的方法，但该脚本的目的是从 MLflow Registry 中获取最新的 ML 模型，并将其升级到“生产”。

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: promote-model
data:
  promote_model.py: |
    """
    This file is used to demonstrate how to promote the latest version of a Model to "Production" stage so that it can be
    served out of the Model Registry.
    """
    import os
    import mlflow
    from mlflow.tracking import MlflowClient

    # Set the Remote Tracking Server Information
    mlflow.set_tracking_uri("http://mlflow")

    # Promote the latest Version of Staging to Production
    client = MlflowClient()
    model_name = os.getenv("MODEL_NAME")

    # Get the latest version in Stage "Staging" and promote to "Production"
    models = client.get_latest_versions(model_name, stages=["Staging"])
    for model in models:
        name = model.name
        latest_version = int(model.version)
        run_id = model.run_id
        current_stage = model.current_stage
        print(name, latest_version)
        # Transition
        client.transition_model_version_stage(
            name=name,
            version=latest_version,
            stage="Production"
        )
```

上面的代码接受一个*环境变量*作为模型名(***MODEL _ NAME = OS . getenv(" MODEL _ NAME ")***)来指定要提升哪个模型。要定义模型名，我们可以简单地将带有模型名的 [*env*](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/) 传递给作业中的容器:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: promote-model-a
spec:
  template:
    spec:
      containers:
        - name: promote
          image: wbassler/mlflow-utils:0.0.1
          command:
            - python
          args:
            - /mlflow/promote.py
          env: 
            - name: MODEL_NAME
              value: model-a
          volumeMounts:
            - name: promote
              mountPath: "/mlflow"
      volumes:
        - name: promote
          configMap:
            name: promote-model
            defaultMode: 0500
      restartPolicy: Never
```

正如你在上面看到的，我们使用“模型-a”作为我们的环境变量。当我们的代码被执行时，它将在 MLflow 模型注册表中找到“模型-a”的最新版本，并将其提升到“生产”。

这里要指出两件事:

a)通过使用现有的容器映像，我们又一次节省了大量时间，并且不必构建自己的映像来添加新代码。也许您在这里所需要的只是通过 GitOps 来部署您的 *configmap* 。

b)我们编写脚本的方式也可以很简单地被其他作业重用*。*在我们的例子中，其他团队/用户/管道等可以非常容易地重用脚本来升级他们自己的 *MODEL_NAME* ，只需简单地改变 *Job* manifest *中的 *env* 。*

## 场景 3:为新用户创建新目录

在最后一个场景中，我将演示如何执行存储在*配置图*中的脚本，该脚本从存储在*配置图*中的配置文件中读取。在解释这样做的目的之前，让我们看一下下面的*配置图*:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: create-users-dir
data:
  users.ini: |
    user1
    user2
    user3
    user5
  create-dir.sh: |
    #!/bin/bash
    set -ex
    cat /scripts/users.ini | while read line
    do
       mkdir -pv /nfs/folder/$line
    done
```

在上面的*配置图*中，我们正在创建两个文件。 *users.ini* 用于定义用户目录列表的配置文件。 *create-dir.sh* 是一个 bash 脚本，用于为 *users.ini* 文件中的每个条目创建一个文件夹。

```
---
apiVersion: v1
kind: Pod
metadata:
  generateName: create-users-dirs
spec:
  containers:
  - name: create-users
    image: centos
    command: ["/scripts/create-dir.sh"]
    volumeMounts:
    - name: script
      mountPath: "/scripts"
  volumes:
    - name: script
      configMap:
        name: create-users-dir
        defaultMode: 0500
        items:
          - key: create-dir.sh
            path: create-dir.sh
          - key: users.ini
            path: users.ini
```

运行时:

```
+ cat /scripts/users.ini
+ read line
+ mkdir -pv /nfs/folder/user1
mkdir: created directory '/nfs'
mkdir: created directory '/nfs/folder'
mkdir: created directory '/nfs/folder/user1'
+ read line
+ mkdir -pv /nfs/folder/user2
mkdir: created directory '/nfs/folder/user2'
+ read line
+ mkdir -pv /nfs/folder/user3
mkdir: created directory '/nfs/folder/user3'
+ read line
+ mkdir -pv /nfs/folder/user5
mkdir: created directory '/nfs/folder/user5'
+ read line
```

这个例子需要指出两点:

a)我们再次利用现有的容器映像，而不必创建自己的映像。

b)在上面的例子中，我们可以通过使用类似 Kustomize[*configMapGenerator*](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#configmapgenerator)*的东西，在几个不同的 K8s 环境中重用相同的代码(如果您想获得真正的乐趣，请将其集成到您的 GitOps 系统中😺).*

*作为 *Pod* 资源运行在这里显然不是一个好的选择，但是我真的很喜欢上面的场景，因为它可以以许多不同的方式被利用，比如作为*作业*或者作为[init container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)。作为 CI 管道的一部分，可以在每次向 *config.ini* 添加新用户时运行新的*作业*，或者可以在每次 pod 启动时运行 *initContainer* 以确保所有目录已经存在。*

*将脚本甚至代码存储为 *configmaps* 并不是什么新鲜事，也不是我自己想出来的。我多年来安装和使用的几个 Kubernetes 应用程序和操作程序也遵循这种方法。这种方法有许多优点和缺点，但是希望我已经给了那里的一些人一个想法，可以节省他们一些时间，并且给自动化带来一些方便。*