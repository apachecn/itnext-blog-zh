# kubernetes Ingress——ku bectl 插件简化了 Nginx 故障排除

> 原文：<https://itnext.io/kubernetes-ingress-nginx-troubleshooting-made-easy-with-kubectl-plugins-dbc8fdff35fd?source=collection_archive---------0----------------------->

随着部署规模越来越大，网络问题越来越难解决。

本指南将提供使用 *ingress-nginx* kubectl 插件通过 ingress 控制器调试应用程序访问的基本信息。

![](img/7a408ccca7850335301073745988db90.png)

由[鲍里斯·斯莫克罗维奇](https://unsplash.com/@borisworkshop?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

从 Kubernetes 1.12 开始，可以为 kubectl 编写定制插件。ingress 社区，具体来说是 [**alexkursell**](https://github.com/alexkursell) 在创建一个插件来帮助我们轻松调试 Ingress 问题方面做了大量工作。

在本教程中，我们将使用 *minikube* ，但是所有的故障排除示例都可以在任何具有 *ingress-nginx* 部署(0.23.0+)的 Kubernetes 集群(1.12+)上执行。

## 要求

为了开始使用本指南，必须安装以下软件:

*   [**VirtualBox**](https://www.virtualbox.org/wiki/Downloads)
*   [**Minikube**](https://kubernetes.io/docs/tasks/tools/install-minikube/)
*   [**Kubectl**](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)
*   **必须在 BIOS 中启用 VT-x/AMD-v 虚拟化**
*   **首次运行时的互联网连接**

注:如果你是第一次使用 *ingress-nginx* 控制器，我推荐查看我的教程[**Minikube**](https://medium.com/@awkwardferny/getting-started-with-kubernetes-ingress-nginx-on-minikube-d75e58f52b6c)上的 Kubernetes Ingress-Nginx 入门。

## 装置

*ingress-nginx* 插件可以通过 ***krew*** 安装，这是一个 kubectl 插件依赖管理器。为了安装*扳手*，只需遵循[https://github.com/kubernetes-sigs/krew](https://github.com/kubernetes-sigs/krew)中提供的安装指南。

一旦您安装了 *krew* ，您就可以通过执行以下操作来获取信息并安装***ingress-nginx***插件:

```
**$ kubectl krew info ingress-nginx**NAME: ingress-nginx
URI: [https://github.com/kubernetes/ingress-nginx/releases/download/nginx-0.24.0/kubectl-ingress_nginx-darwin-amd64.tar.gz](https://github.com/kubernetes/ingress-nginx/releases/download/nginx-0.24.0/kubectl-ingress_nginx-darwin-amd64.tar.gz)
SHA256: 34e999ddc4e1926bbd7f4bdbb5fc40d6c3a5b2ffd711131e4f62e2d24cb2a179
DESCRIPTION:
The official kubectl plugin for ingress-nginx.VERSION: 0.24.0**$ kubectl krew install ingress-nginx**Updated the local copy of plugin index.
Installing plugin: ingress-nginx
Installed plugin: ingress-nginx
```

更多关于 *krew* 和 kubectl 插件的一般信息，请查看 [**为所有人编写 kubectl 插件:开发、打包和分发**](https://youtu.be/83ITOTsXsHU) 来自 KubeCon Europe 2019。

## 使用

首先，您必须创建一个 *minikube* 集群并启用入口控制器。我们还可以验证是否生成了 *ingress-nginx* pods。

```
**$ minikube start**😄  minikube v1.0.1 on darwin (amd64)
🤹  Downloading Kubernetes v1.14.1 images in the background ...
🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
📶  "minikube" IP address is 192.168.99.100
🐳  Configuring Docker as the container runtime ...
🐳  Version of container runtime is 18.06.3-ce
⌛  Waiting for image downloads to complete ...
✨  Preparing Kubernetes environment ...
🚜  Pulling images required by Kubernetes v1.14.1 ...
🚀  Launching Kubernetes v1.14.1 using kubeadm ...
⌛  Waiting for pods: apiserver proxy etcd scheduler controller dns
🔑  Configuring cluster permissions ...
🤔  Verifying component health .....
💗  kubectl is now configured to use "minikube"
🏄  Done! Thank you for using minikube!**$ minikube addons enable ingress**✅  ingress was successfully enabled**$ kubectl get pods -n kube-system | grep ingress**nginx-ingress-controller-586cdc477c-74vlb   1/1     Running   0          97s
```

现在我们可以通过直接从 kubectl 运行 *ingress-nginx* 命令来排除故障。

```
**$ kubectl ingress-nginx --help**....
Available Commands:
backends  Inspect the dynamic backend information of an ingress-nginx instance
certs     Output the certificate data stored in an ingress-nginx pod
conf      Inspect the generated nginx.conf
exec      Execute a command inside an ingress-nginx pod
general   Inspect the other dynamic ingress-nginx information
help      Help about any command
info      Show information about the ingress-nginx service
ingresses Provide a short summary of all of the ingress definitions
lint      Inspect kubernetes resources for possible issues
logs      Get the kubernetes logs for an ingress-nginx pod
ssh       ssh into a running ingress-nginx pod
...
```

## 添加用于测试的入口资源

让我们快速创建一个在本教程中使用的应用程序、服务和入口。

**部署应用**

让我们使用部署来部署我们的应用程序单元(容器)。关于部署的更多信息可以在[这里](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)找到。

```
**$ echo "
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: meow
spec:
  replicas: 2
  selector:
    matchLabels:
      app: meow
  template:
    metadata:
      labels:
        app: meow
    spec:
      containers:
      - name: meow
        image: gcr.io/kubernetes-e2e-test-images/echoserver:2.1
        ports:
        - containerPort: 8080
" | kubectl apply -f -**deployment.extensions/meow created
```

**通过服务公开 pod**

这里，我们在集群的内部 IP 上公开应用程序。更多关于服务的信息，请看这里。

```
**$ echo "
apiVersion: v1
kind: Service
metadata:
  name: meow-svc
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: meow
" | kubectl apply -f -**service/meow-svc created
```

**创建入口资源**

这允许我们通过 */meow* 路径访问服务 *meow-svc* 。由于我们使用的是 minikube，*cats.com*是我们的虚拟主机，所以我们必须通过*主机*头来访问应用程序。

```
**$ echo "
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: meow-ingress
  namespace: default
spec:
  rules:
  - host: cats.com
    http:
      paths:
      - backend:
          serviceName: meow-svc
          servicePort: 80
        path: /meow
" | kubectl apply -f -**ingress.extensions/meow-ingress created**$ curl $(minikube ip)/meow -H "host: cats.com"**....
Request Information:
client_address=172.17.0.7
method=GET
real path=/meow
query=
request_version=1.1
request_scheme=http
request_uri=http://cats.com:8080/meow
....
```

# 排除故障

现在让我们开始一些调试工作🐛。我们可以采取一些步骤来解决常见的入侵问题。

## 验证部署、进入和后端

通过验证部署、进入和后端，我们可以看到我们实际上是否设置正确。

```
**$ kubectl ingress-nginx ingresses -n default**INGRESS_NAME HOST+PATH ADDRESSES TLS SERVICE SERVICE_PORT ENDPOINTS
meow-ingress  cats.com/meow  10.0.2.15  NO  meow-svc  80  2**$ kubectl ingress-nginx backends -n kube-system --list**default-meow-svc-80
upstream-default-backend**$ kubectl ingress-nginx backends -n kube-system --backend upstream-default-meow-svc-80**{
  "endpoints": [
    {
      "address": "172.17.0.8",
      "port": "8080"
    },
    {
      "address": "172.17.0.9",
      "port": "8080"
    }
  ],
  "name": "default-meow-svc-80",
  "noServer": false,
  "port": 80,
  ....**$ kubectl ingress-nginx lint -n default**Checking ingresses...
Checking deployments...
```

首先，通过检查入口资源，我们可以看到我们是否正确地配置了入口。我们还可以验证特定的后端配置是否正确。

最后，我们可以执行 lint，检查是否存在任何潜在的配置问题。

**注意:**对于后端命令，如果您的 ingress-nginx 部署未命名为 *nginx-ingress-controller，您将需要传递名称空间以及 **- *部署*** 标志。*

## 检查日志

如果存在特定问题，并且我们知道服务和入口资源配置正确，我们应该继续查看日志。日志将给出处理入口资源问题的细节。

```
**$ curl $(minikube ip)/meow -H "host: cats.com"**....
Request Information:
client_address=172.17.0.7
method=GET
real path=/meow
query=
request_version=1.1
request_scheme=http
request_uri=http://cats.com:8080/meow
....**$ kubectl ingress-nginx logs -n kube-system**....
I0623 04:49:34.058860       6 controller.go:172] Configuration changes detected, backend reload required.I0623 04:49:34.061302       6 event.go:221] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"meow-ingress", UID:"4bb59ae1-9572-11e9-b459-080027ab110d", APIVersion:"extensions/v1beta1", ResourceVersion:"1333", FieldPath:""}): type: 'Normal' reason: 'CREATE' Ingress default/meow-ingressI0623 04:49:34.226892       6 controller.go:190] Backend successfully reloaded.I0623 04:50:01.122763       6 status.go:388] updating Ingress default/meow-ingress status from [] to [{10.0.2.15 }]192.168.99.1 - [192.168.99.1] - - [23/Jun/2019:05:13:17 +0000] "GET /meow HTTP/1.1" 200 638 "-" "curl/7.54.0" 76 0.001 [default-meow-svc-80] 172.17.0.8:8080 638 0.001 200 176c54287bf24c650c132e6ff3b7aed8
```

这基本上与检查*入口-nginx* 吊舱输出的日志功能相同。

我们发送一个 curl，然后检查日志，看看是否一切正常。正如您所看到的，有关于后端成功加载的信息，以及 curl 通过的信息。我们可以使用这些日志来确保我们的入口被正确加载，以及查看流量是否通过。

## 检查 NGINX 配置

我们可以检查 nginx 配置，看看是否所有的指令和路径都配置正确。我们可以使用 [**nginx 文档**](http://nginx.org/en/docs/) 来看看指令和变量是做什么的。

```
**$ kubectl ingress-nginx conf -n kube-system**# Configuration checksum: 4909015860030157750
# setup custom paths that do not require root access
pid /tmp/nginx.pid;
load_module /etc/nginx/modules/ngx_http_modsecurity_module.so;
daemon off;
worker_processes 2;
worker_rlimit_nofile 523264;
worker_shutdown_timeout 10s ;
....**$ kubectl ingress-nginx conf -n kube-system --host cats.com**server {
  server_name cats.com ;
  listen 80;
  set $proxy_upstream_name "-";
  location /meow {
    set $namespace      "default";
    set $ingress_name   "meow-ingress";
    set $service_name   "meow-svc";
    set $service_port   "80";
    set $location_path  "/meow";
  rewrite_by_lua_block {
    balancer.rewrite()
  }
  header_filter_by_lua_block {}
  body_filter_by_lua_block {}
  log_by_lua_block {
    balancer.log()
    monitor.call()
  }
  port_in_redirect off;
  set $proxy_upstream_name    "default-meow-svc-80";
  set $proxy_host             $proxy_upstream_name;
  client_max_body_size                    1m;
  ....
```

## 使用入口控制器盒运行命令

如果我们希望解决任何具体问题，我们总是可以在 pod 中运行命令。

```
**$ kubectl ingress-nginx exec -n kube-system -- curl http://127.0.0.1/meow -H "host: cats.com"**....
Server values:
server_version=nginx: 1.12.2 - lua: 10010
Request Information:
client_address=172.17.0.7
method=GET
real path=/meow
query=
request_version=1.1
request_scheme=http
request_uri=http://cats.com:8080/meow
....
```

这与*执行*到*入口-nginx* 吊舱的功能基本相同。上面你可以看到，我们正在测试我们的应用程序是否可以从我们的入口 pod 访问。

感谢阅读，希望你喜欢！😸

要了解更多关于 *ingress-nginx* kubectl 插件的信息，你可以查看它的[官方文档](https://kubernetes.github.io/ingress-nginx/kubectl-plugin/)。

要了解更多关于解决入口问题的信息，你可以查看丹尼尔·马丁斯的【痛苦(较少)NGINX 入口】一文。这是一篇写得非常好的文章。