# 探索使用 Tekton 管道构建映像并将其推送到私有注册表的不同方法

> 原文：<https://itnext.io/explore-different-methods-to-build-and-push-image-to-private-registry-with-tekton-pipelines-5cad9dec1ddc?source=collection_archive---------3----------------------->

![](img/52c8761576bf9d004749bb19919e44ca.png)

[Tekton](https://tekton.dev/) 项目使管道资源能够被宣布为 Kubernetes CRDs，并因此以 Kubernetes 本地方式进行管理。本地 Kubernetes 集群的一个常见用例是构建 Docker 映像并将其推入私有注册中心。针对 OpenShift 3.11 docker 注册表，本文探讨了在 Tekton 管道中构建和推送映像的不同方式。

## 泰克顿管道资源

Tekton 管道的目标是从 Git 存储库中提取一个简单的 hello world Golang HTTP 处理程序的源代码，构建可执行文件和 docker 映像，并将其推送到 OpenShift 映像注册中心。

文档附在下面，

```
FROM golang as builder
WORKDIR /build
COPY src/*go /buildRUN CGO_ENABLED=0 go build -o demoApp *.goFROM alpine
WORKDIR /appCOPY --from=builder /build/demoApp /appCMD ["./demoApp"]
```

为了概括这个过程，为 git repo 和 docker registry repo 创建了以下管道资源。

```
---
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: cicd-demo-git-source
spec:
  type: git
  params:
    - name: revision
      value: master
    - name: url
      value: http://gitea.apps.ocp.fyre.io.cpak/wenzm/cicd-demo.git
---
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: cicd-demo-registry-image
spec:
  type: image
  params:
    - name: url
      value: docker-registry.default.svc.cluster.local:5000/cicd-demo/cicd-demo
```

git 资源是 Gitea 的 Git repo，托管在 OpenShift Kubernetes 集群上。图像资源是用名称空间和 repo 名称定义的 OpenShift 注册表。

## 访问 OpenShift 图像注册表

由于 Tekton 管道运行在 Kubernetes 集群内部，我们可以通过它的服务名来访问 OpenShift 图像注册中心，服务名是

```
docker-registry.default.svc.cluster.local:5000
```

我们使用服务帐户及其令牌进行身份验证。在名称空间中，创建名为`pipeline`的服务帐户。将集群角色`system:image-pusher`分配给这个服务帐户，以便 RBAC 允许它创建图像流对象并将图像推送到这个名称空间。

```
oc create sa pipeline
oc adm policy add-role-to-user system:image-builder -z pipeline
```

密码是服务帐户的令牌，可以通过，

```
oc get secret $(oc get secret | grep pipeline-token | head -1 | awk '{print $1}') -o jsonpath="{.data.token}" | base64 -d
```

通过连接到 OpenShift 路由器暴露的注册表来验证它，在我的例子中是`docker-registry-default.apps.ocp.fyre.io.cpak`，

```
docker login docker-registry-default.apps.ocp.fyre.io.cpak
Username: sa
Password: <The token shown with above command>
Login Succeeded
```

## 方法一。用 Docker 构建

在 Tekton 管道中构建映像的直接方法是使用 docker 容器(docker in docker)进行构建。然而，要使它在 OpenShift 集群中工作，需要注意一些要点。

> 在这种方法中，主机 docker 插座的安装是一个很大的安全问题。不要在生产环境中这样做。

以下是为此任务创建的 Tekton 任务对象。

```
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: cicd-demo-build-with-docker
  namespace: cicd-demo
spec:
  inputs:
    resources:
      - name: git-source
        type: git
    params:
      - name: pathToDockerFile
        default: /workspace/git-source/compile.Dockerfile
      - name: pathToContext
        default: /workspace/git-source
      - name: registryHost
        default: "docker-registry.default.svc.cluster.local:5000"
  outputs:
    resources:
      - name: docker-image
        type: image
  steps:
    - name: build
      image: docker
      securityContext:
        runAsUser: 0
        privileged: true
      command:
        - sh
        - -c
        - "cd ${inputs.params.pathToContext} && \
           docker build -f int-build.dockerfile -t int-build . && \
           docker stop int-container || echo ignore not running && \ 
           docker rm int-container || echo ignore non-exists && \
           docker run --name int-container -d int-build sh -c 'while true; do sleep 60; done' && \
           docker cp int-container:/go/demoApp . && \
           docker stop int-container && \
           docker build -f final-build.dockerfile -t ${outputs.resources.docker-image.url} . && \
           docker login ${inputs.params.registryHost} -u sa -p $(cat /var/run/secrets/kubernetes.io/serviceaccount/token) && \
           docker push ${outputs.resources.docker-image.url}
          "
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run/docker.sock
  volumes:
    - name: docker-socket
      hostPath:
        path: /var/run/docker.sock
        type: Socket
```

在构建步骤中，我们使用`docker`图像。然而，为了在 docker (dind)中的 docker 中构建映像，我们将 docker 套接字从节点挂载到 pod 中，并且容器必须在特权模式下运行。因此，必须将相应的安全上下文约束(SCC)分配给运行 pod 的服务帐户。

```
oc adm policy add-scc-to-user privileged -z pipeline
```

因此，容器定义中的安全上下文设置了以下内容，

```
 securityContext:
        runAsUser: 0
        privileged: true
```

正如在 OpenShift 3.11 中，Doker 的版本不支持多阶段构建，我们必须切换回一些旧的 Docker 技术来相应地构建映像。首先，我们用名为 int-build 的 dockerfile 文件编译和构建二进制文件。Dockerfile，

```
FROM golang
COPY src/* src
RUN CGO_ENABLED=0 go build -o demoApp src/*.go
```

然后，我们使用循环 shell 命令将容器作为守护进程运行。使用“`*docker cp*`”将二进制文件复制到构建上下文中。最后，我们用下面的 dockerfile 文件构建发布映像，

```
FROM alpine
WORKDIR /app
COPY demoApp /appCMD ["./demoApp"]
```

在我们将映像推送到注册表之前，我们需要使用`docker login`命令登录。用户名可以是任意的，密码是服务帐户的令牌，Kubernetes 将它安装到`/var/run/secrets/kubernetes.io/serviceaccount/token`的位置。用这个文件的内容登录，我们就可以推送图像了。

我们可以通过创建任务运行 CRD 对象来运行任务。它的内容被跳过。

总的来说建造速度很快。然而，安装节点的 Docker 套接字和特权安全上下文需求是这种方法最大的安全问题。

## **方法二。用 Buildah 构建**

Buildah 是一个命令行工具，有助于构建开放容器倡议(OCI)容器映像。

虽然在主机中的集群之外运行命令行工具相当顺利，但是在 OpenShift 的 Kubernetes 集群内部运行该工具相当具有挑战性。我们使用预先构建的 Docker 映像`quay.io/buildah/stable`来完成 Tekton 任务。最终的工作版本如下所示，

```
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: cicd-demo-build-with-buildah
  namespace: cicd-demo
spec:
  inputs:
    resources:
      - name: git-source
        type: git
    params:
      - name: pathToDockerFile
        default: /workspace/git-source/Dockerfile
      - name: pathToContext
        default: /workspace/git-source
      - name: registryHost
        default: "docker-registry.default.svc.cluster.local:5000"
      - name: imageTag
        default: cicd-demo
  outputs:
    resources:
      - name: docker-image
        type: image
  steps:
    - name: build
      image: quay.io/buildah/stable
      securityContext:
        runAsUser: 0
        privileged: true
      command:
        - sh
        - -c
        - "cd ${inputs.params.pathToContext} && \
           buildah --storage-driver=vfs build-using-dockerfile -f ${inputs.params.pathToDockerFile} -t ${inputs.params.imageTag} . && \           
           buildah --storage-driver=vfs push --tls-verify=false --creds=anyone:$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) ${inputs.params.imageTag} docker://${outputs.resources.docker-image.url}
          "
```

理想情况下，我们不应该期望安全上下文的特权要求。但是，如果没有这些，我会遇到下面的错误，

```
Error during unshare(CLONE_NEWUSER): Invalid argumentUser namespaces are not enabled in /proc/sys/user/max_user_namespaces.
```

我们应该能够运行如下所示的普通构建命令，

```
buildah build-using-dockerfile -f ${inputs.params.pathToDockerFile} -t ${inputs.params.imageTag} .
```

但是，我遇到了以下错误，(*是从容器日志*中复制的)

```
process exited with error: fork/exec /bin/sh: no such file or directorysubprocess exited with status 1
```

最终我们不得不使用`buildah — storage-driver=vfs`选项来构建图像。还要注意的是，为了让`WORKDIR /build`工作，我们必须在 order 文件中显式地创建该目录，如下所示，

```
FROM golang as builder
**RUN mkdir  /build**
WORKDIR /build
COPY src/*go /buildRUN CGO_ENABLED=0 go build -o demoApp *.goFROM alpine
**RUN mkdir /app**
WORKDIR /appCOPY --from=builder /build/demoApp /appCMD ["./demoApp"]
```

对于身份验证部分，我们使用令牌文件的内容作为登录注册中心的密码。

总的来说，Buildah 不需要从主机挂载任何东西，但是我们仍然需要特权安全上下文，并且构建速度相对较慢，因为我们必须使用 vfs 存储驱动程序来使它在集群内工作。

## 方法三。与 Kaniko 一起构建

让我们创建以下 Tekton 任务，

```
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: cicd-demo-build-with-kaniko
  namespace: cicd-demo
spec:
  inputs:
    resources:
      - name: git-source
        type: git
    params:
      - name: pathToDockerFile
        default: /workspace/git-source/Dockerfile
      - name: pathToContext
        default: /workspace/git-source
  outputs:
    resources:
      - name: docker-image
        type: image
  steps:
    - name: build
      image: gcr.io/kaniko-project/executor:debug
      securityContext:
        runAsUser: 0
      command:
        - sh
        - "-c"
        - " 
            /kaniko/executor \
            --dockerfile=${inputs.params.pathToDockerFile} \
            --destination=${outputs.resources.docker-image.url} \
            --context=${inputs.params.pathToContext}  \
            --skip-tls-verify 
          "
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      configMap: 
        name: docker-config
```

我正在使用`gcr.io/kaniko-project/executor:debug`图像，这样我可以在中间插入睡眠命令来排除故障。

注意，我们不需要特权安全上下文。

一个特殊的事情是，对于 Kaniko 将图像推送到私有注册中心，我们需要以 Docker 的 config.json 格式提供认证。

创建以下 JSON 文件，

```
{
  "auths": {
    "docker-registry.default.svc.cluster.local:5000": {
      "auth": "c2E6ZXlKaGJHY2lPaUp<...skipped many lines ...>"
    }
  }
}
```

使用以下命令获得`auth`的值

```
secret=$(oc get secret  | grep pipeline-token | head -1 | awk '{print $1}')token=$(oc get secret $secret -o jsonpath="{.data.token}" | base64 -d -i)
echo "sa:$token" | base64
```

*您可能需要努力删除新行来加入 base64 编码行。一个更好的方法可能是用一些 golang 魔法来自动化它，比如* `base64.StdEncoding.EncodeToString([]byte(fmt.Sprintf(“sa:%s”, token)))`

使用名为 config.json 的键创建配置映射，

```
kubectl -n cicd-demo create configmap docker-config --from-file=config.json=docker.config.json
```

使用这个配置图将它挂载到 Pod 的默认目录`/kaniko/.docker`

## 结论

基于此时的比较，Kaniko 提供了一种更安全但足够快速的方法来在 Tekton 管道中构建和推送图像。