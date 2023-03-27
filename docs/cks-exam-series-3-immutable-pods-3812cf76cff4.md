# CKS 考试系列#3 不可变豆荚

> 原文：<https://itnext.io/cks-exam-series-3-immutable-pods-3812cf76cff4?source=collection_archive---------0----------------------->

## Kubernetes CKS 示例考试问题系列

![](img/0b4636daffcb81ed1b2b37e49b2316ae.png)

> [CKS 考试系列](https://killer.sh/r?d=cks-series) | [CKA 考试系列](https://killer.sh/r?d=cka-series) | [CKAD 考试系列](https://killer.sh/r?d=ckad-series)

**#####更新####更新**

**此挑战不会在此更新，将移至:**

[https://killercoda.com/killer-shell-cks](https://killercoda.com/killer-shell-cks)

**#####更新####更新**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

→查看 Udemy 上的 [**全 CKS 课程**](https://killer.sh/r?d=cks-course)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# 内容

1.  [创建集群&安全最佳实践](https://wuestkamp.medium.com/cks-exam-series-1-create-cluster-security-best-practices-50e35aaa67ae?source=friends_link&sk=8bc466dae0ea90412251e32d4eaf7539)
2.  [豆荚、秘密和服务账户](https://wuestkamp.medium.com/cks-exam-series-2-pods-and-secrets-3d92a6fba331?source=friends_link&sk=379fa6e196233c73ef7845d84a3aa34d)
3.  [不可变豆荚](https://wuestkamp.medium.com/cks-exam-series-3-immutable-pods-3812cf76cff4?source=friends_link&sk=ed1231a0382d97bd5c8267afe75f14ac)
4.  [崩溃 Apiserver &检查日志](https://wuestkamp.medium.com/cks-exam-series-4-crash-that-apiserver-5f4d3d503028?source=friends_link&sk=3ccd9bf1b728e85f86157ef1af23d455)
5.  [ImagePolicyWebhook/admission controller](https://wuestkamp.medium.com/cks-exam-series-5-imagepolicywebhook-8d09f1ceee70?source=friends_link&sk=93017beeae20f640f52db41d20d3ffcd)
6.  [用户和证书签名请求](https://wuestkamp.medium.com/cks-exam-series-6-users-and-certificatesigningrequests-368a5b2c6a3f)
7.  [服务帐户令牌安装](https://wuestkamp.medium.com/cks-exam-series-7-serviceaccount-tokens-1158c93612d4?source=friends_link&sk=1064eaf2f3d4d03576bcde207eaf7cfb)
8.  [基于角色的访问控制(RBAC)](https://wuestkamp.medium.com/cks-exam-series-8-rbac-db8a0984059e?source=friends_link&sk=8a1abe2d51275faed47f3d36858b14d5)
9.  [基于角色的访问控制(RBAC) v2](https://wuestkamp.medium.com/cks-exam-series-9-rbac-v2-23ee24dd77cd?source=friends_link&sk=2a6027eb75fbcf7876216cab222fa953)
10.  [容器硬化](https://wuestkamp.medium.com/cks-exam-series-10-container-hardening-177588b8bbfe?source=friends_link&sk=dbdddc1ee9321a946ee2e3f778c0711a)
11.  [网络策略(默认拒绝+允许列表)](https://wuestkamp.medium.com/cks-exam-series-11-networkpolicies-default-deny-and-allowlist-b2ce4186551f?source=friends_link&sk=bdcc071a32f26b93d6c4a51b9a9436a7)

# 规则！

1.  速度要快，避免从头开始手动创建 yaml
2.  仅使用[kubernetes.io/docs](https://kubernetes.io/docs/home/)进行帮助。
3.  完成您的解决方案后，请查看我们的解决方案。你可能有一个更好的！

# 今天的任务:让豆荚不变

1.  用图像`bash:5.1.0`的两个容器`c1`和`c2`创建*Pod*，确保容器保持运行
2.  用 3 个副本创建映像`nginx:1.19.6`的*部署* `snow`
3.  强制 *Pod* `holiday`的容器`c2`不可变运行:在运行期间不能更改任何文件
4.  确保*部署* `snow`的容器不可变运行。然后使必要的路径可写，以便 Nginx 工作。
5.  核实一切

.

.

.

.

.

# 解决办法

## 1.

```
alias k=kubectlk run holiday --image=bash:5.1.0 --command -oyaml --dry-run=client -- sh -c 'sleep 1d' > holiday.yamlvim holiday.yaml
```

添加第二个容器并更改容器名称:

## 2.

```
k create deploy snow --image=nginx:1.19.6 -oyaml --dry-run=client > snow.yamlvim snow.yaml
```

更改副本:

## 3.

```
vim holiday.yaml
```

在容器级别添加 SecurityContext:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: holiday
  name: holiday
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: bash:5.1.0
    name: c1
    resources: {}
  - command:
    - sh
    - -c
    - sleep 1d
    image: bash:5.1.0
    name: c2
    resources: {}
 **securityContext:
      readOnlyRootFilesystem: true**
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## 4.

这个想法是将所有文件系统设为只读，然后启动 *Pod* 并检查容器日志中的错误。基于这些错误，我们可以创建用于写入的 emptyDir 卷。错误可能如下所示:

```
"/var/cache/nginx/client_temp" failed (30: Read-only file system)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (30: Read-only file system)
```

只要有 Docker，我们就可以做类似这样的事情:

```
docker run -d --read-only --tmpfs /var/cache nginx
```

在*部署*中要做到这一点:

```
vim snow.yaml
```

添加卷和卷装载:

## 5.

```
k exec holiday -c c1 -- touch /tmp/test # works
k exec holiday -c c2 -- touch /tmp/test # errork get deploy snow # should show 3 ready replicas
k exec snow-575cd78c85-ldplw -- touch /tmp/test # error
k exec snow-575cd78c85-ldplw -- touch /var/cache/nginx/test # works
```

# 你有不同的解决方法？

请在下面留言告诉我们！

# — — —结尾————

本次会议到此为止。下次再见，祝学习愉快！

# 准备好加入黑仔壳牌了吗？

## 完整的 CKS 课程

[![](img/b16ab8b3bd72db1ad641b82f067bdd80.png)](https://killer.sh/r?d=cks-course)

[链接](https://killer.sh/r?d=cks-course)

## …或者 CKS 模拟器

[![](img/4c118b1471bd606fc46cd5569df4f944.png)](https://killer.sh/cks)

[https://killer.sh/cks](https://killer.sh/cks)