# ArgoCD:用户、访问和 RBAC

> 原文：<https://itnext.io/argocd-users-access-and-rbac-ddf9f8b51bad?source=collection_archive---------6----------------------->

![](img/1a446629e8c022ba478ecb739487ffac.png)

[ArgoCD](https://rtfm.co.ua/en/category/ci-cd-en/argocd-en/) 有两种类型的用户——在`argocd-cm`配置图中设置的本地用户和单点登录用户。

下面，我们将谈到本地用户管理，在下一章将看到如何集成 ArgoCD 和 Okta，因为本地用户不能分组。参见[本地用户/账户](https://argoproj.github.io/argo-cd/operator-manual/user-management/#local-usersaccounts-v15)页面上的文档。

对于任何用户，都可以使用角色来配置他们的权限，这些角色附加了描述对象的策略，以允许用户对其进行访问和操作。

有了这个，访问可以按集群进行全局配置，或者专用于[项目](https://rtfm.co.ua/en/?p=26033#ArgoCD_Projects)。

让我们从添加一个简单的本地用户开始，然后一步一步地配置剩下的部分。

*   [ArgoCD 中的用户和角色](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#Users_and_roles_in_ArgoCD)
*   [添加本地用户](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#Adding_a_local_user)
*   [角色和 RBAC](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#Roles_and_RBAC)
*   [ArgoCD 项目](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#ArgoCD_Projects)
*   [项目和角色](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#Projects_and_roles)
*   [全局角色](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#Global_roles)
*   [项目角色和认证令牌](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#Project_roles_and_authentication_tokens)
*   [ArgoCD 组](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#ArgoCD_groups)
*   [把所有的放在一起](https://rtfm.co.ua/en/argocd-users-access-and-rbac/#Putting_all_together)

# ArgoCD 中的用户和角色

## 添加本地用户

要添加本地用户，编辑`argocd-cm`配置图并添加一条`accounts.USERNAME`记录:

```
apiVersion: v1
data:
  accounts.testuser: apiKey,login
...
```

通过这里的`apiKey`，我们已经设置用户可以生成 [JWT 令牌](https://rtfm.co.ua/en/kubernetes-serviceaccounts-jwt-tokens-authentication-and-rbac-authorization/#JWT_token)进行身份验证，参见[安全](https://argoproj.github.io/argo-cd/operator-manual/security/#authentication)，通过`login`，允许该用户使用 ArgoCD WebUI 登录。

保存并检查用户列表:

```
$ argocd account list
NAME ENABLED CAPABILITIES
admin true login
testuser true apiKey, login
```

*admin* 用户是在 ArgoCD 实例设置期间创建的，它不能使用令牌。这可以通过在`argocd-cm`中设置该用户来配置，尽管建议在添加所有必需的用户后禁用管理员用户。

一般来说，这个想法是让用户访问 WebUI，让 [*项目角色*](https://rtfm.co.ua/en/?p=26033#Projects_and_roles)*——获得可以在 CI/CD 管道中使用的令牌。*

*测试用户是我们刚刚添加的，目前还没有设置密码。*

*要为他创建密码，您需要有当前管理员的密码:*

```
*$ argocd account update-password --account testuser --new-password 1234 --current-password admin-p@ssw0rd*
```

*密码已更新*

*不是最好的解决方案，但事实如此。详情请参见[无法通过 argocd CLI](https://github.com/argoproj/argo-cd/issues/4096) 更改用户密码的讨论。*

*现在，使用*测试用户*登录:*

```
*$ argocd login dev-1–18.argocd.example.com --username testuser --name testuser@dev-1–18.argocd.example.com
Password:
‘testuser’ logged in successfully
Context ‘testuser@dev-1–18.argocd.example.com’ updated*
```

*检查本地 ArgoCD CLI 上下文:*

```
*$ argocd login dcontext
CURRENT NAME SERVER
admin@dev-1–18.argocd.example.com dev-1–18.argocd.example.com
* testuser@dev-1–18.argocd.example.com dev-1–18.argocd.example.com*
```

*好了，现在我们在*测试用户*下。*

## *角色和 RBAC*

*默认情况下，所有新用户都使用来自`argocd-rbac-cm`配置图的`policy.default`:*

```
*$ kubectl -n dev-1–18-devops-argocd-ns get configmap argocd-rbac-cm -o yaml
apiVersion: v1
data:
policy.default: role:readonly
…*
```

*ArgoCD 有两个默认角色— `role:readonly`和`role:admin`。此外，您可以用`role: ''`设置`policy.default`来完全禁用访问。*

*此时，我们的用户可以查看任何资源:*

```
*$ argocd cluster list
SERVER NAME VERSION STATUS MESSAGE
[https://kubernetes.default.svc](https://kubernetes.default.svc) in-cluster 1.18+ Successful*
```

*但是不能创建任何新的，例如，尝试添加一个新的集群:*

```
*$ argocd cluster add config-aws-china-eks-account@aws-china-eks-account --kubeconfig ~/.kube/config-aws-china-eks-account@aws-china-eks-account
INFO[0002] ServiceAccount “argocd-manager” already exists in namespace “kube-system”
INFO[0003] ClusterRole “argocd-manager-role” updated
INFO[0004] ClusterRoleBinding “argocd-manager-role-binding” updated
FATA[0006] rpc error: code = PermissionDenied desc = permission denied: clusters, create, [https://21D***ECD.gr7.cn-northwest-1.eks.amazonaws.com.cn,](https://21D***ECD.gr7.cn-northwest-1.eks.amazonaws.com.cn,) sub: testuser, iat: 2021–05–12T14:03:12Z*
```

****“权限被拒绝:集群，创建***”—啊哈，不能。*

*要允许用户添加集群，编辑`argocd-rbac-cm` CondfigMap，添加一个具有`clusters, create`权限的新`role:test-role`:*

```
*...
data:
  policy.default: role:readonly
  policy.csv: |
    p, role:test-role, clusters, create, *, allow
    g, testuser, role:test-role
...*
```

*检查一下:*

```
*$ argocd account can-i create clusters ‘*’
yes*
```

*并添加一个集群:*

```
*$ argocd cluster add config-aws-china-eks-account@aws-china-eks-account --kubeconfig ~/.kube/config-aws-china-eks-account@aws-china-eks-account
INFO[0001] ServiceAccount “argocd-manager” already exists in namespace “kube-system”
INFO[0002] ClusterRole “argocd-manager-role” updated
INFO[0003] ClusterRoleBinding “argocd-manager-role-binding” updated
Cluster ‘https://21D***ECD.gr7.cn-northwest-1.eks.amazonaws.com.cn' added*
```

*但是，如果您希望允许名称空间操作，该怎么做呢？配置图中的 RBAC 不允许这样做。*

*例如，在我目前的项目中，我们有 Web 开发人员、后端开发人员，他们都有部署在不同名称空间中的不同应用程序，分离他们的访问是一个好主意，因此 Web 团队不会影响后端的团队资源。*

# *ArgoCD 项目*

*而在这里"*项目来拯救*"！*

*[项目](https://argoproj.github.io/argo-cd/user-guide/projects/)允许指定对名称空间、存储库、集群等的访问。然后，我们将能够只通过他们自己的名称空间来限制每个开发团队*。**

*让我们看看这是如何工作的。*

*因此，在安装过程中，ArgoCD 创建了默认的*项目:**

```
*$ argocd proj list
NAME DESCRIPTION DESTINATIONS SOURCES CLUSTER-RESOURCE-WHITELIST NAMESPACE-RESOURCE-BLACKLIST SIGNATURE-KEYS ORPHANED-RESOURCES
default *,* * */* <none> <none> disabled*
```

*项目是 Kubernetes 定制的资源对象，类型为:*

```
*$ kubectl -n dev-1–18-devops-argocd-ns get appproject
NAME AGE
default 166d*
```

*可以用一个普通的 Kubernetes 清单创建，如下所示:*

```
*kind: AppProject
metadata:
  name: example-project
  namespace: dev-1-18-devops-argocd-ns
spec:
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
  destinations:
  - namespace: argo-test-ns
    server: [https://kubernetes.default.svc](https://kubernetes.default.svc)
  orphanedResources:
    warn: false
  sourceRepos:
  - '*'*
```

*或使用 ArgoCD CLI:*

```
*$ argocd proj create test-project -d [https://kubernetes.default.svc,argo-test-ns](https://kubernetes.default.svc,argo-test-ns) -s [https://github.com/argoproj/argocd-example-apps.git](https://github.com/argoproj/argocd-example-apps.git)*
```

*检查一下:*

```
*$ argocd proj list
NAME DESCRIPTION DESTINATIONS SOURCES CLUSTER-RESOURCE-WHITELIST NAMESPACE-RESOURCE-BLACKLIST SIGNATURE-KEYS ORPHANED-RESOURCES
default *,* * */* <none> <none> disabled
test-project [https://kubernetes.default.svc,argo-test-ns](https://kubernetes.default.svc,argo-test-ns) [https://github.com/argoproj/argocd-example-apps.git](https://github.com/argoproj/argocd-example-apps.git) <none> <none> <none> disabled*
```

*现在，让我们将现有的应用程序*留言簿*从`argo-test-ns`移动到新项目:*

```
*$ argocd app set guestbook --project test-project*
```

*检查一下:*

```
*$ argocd app get guestbook
Name: guestbook
Project: test-project
Server: [https://kubernetes.default.svc](https://kubernetes.default.svc)
Namespace: argo-test-ns
URL: https://dev-1–18.argocd.example.com/applications/guestbook
Repo: [https://github.com/argoproj/argocd-example-apps.git](https://github.com/argoproj/argocd-example-apps.git)
…*
```

*现在，让我们检查一下名称空间限制是如何工作的。*

*当我们创建了上面的项目，我们已经设置了*目的地* — `[https://kubernetes.default.svc,argo-test-ns](https://kubernetes.default.svc,argo-test-ns.)` [。](https://kubernetes.default.svc,argo-test-ns.)*

*尝试在该项目中创建一个新的应用程序，但将其名称空间设置为 *argo-test-2-ns* :*

```
*$ argocd app create guestbook-2 — repo [https://github.com/argoproj/argocd-example-apps.git](https://github.com/argoproj/argocd-example-apps.git) — path guestbook — dest-server [https://kubernetes.default.svc](https://kubernetes.default.svc) — dest-namespace argo-test-2-ns — project test-project
FATA[0001] rpc error: code = InvalidArgument desc = application spec is invalid: InvalidSpecError: application destination {https://kubernetes.default.svc argo-test-2-ns} is not permitted in project ‘test-project’*
```

****“项目' test-project* '** '中不允许应用目的地{ https://kubernetes . default . SVC Argo-test-2-ns }”—酷！对于*测试项目*项目中的 *testuser* 权限，我们目前不能将应用程序添加到新的名称空间，因为项目对目的地有限制。*

*更新项目并添加一个新的目的地—同一个集群，但是另一个名称空间 *argo-test-2-ns* :*

```
*$ argocd proj add-destination test-project [https://kubernetes.default.svc](https://kubernetes.default.svc) argo-test-2-ns*
```

*立即检查项目:*

```
*$ argocd proj get test-project
Name: test-project
Description:
Destinations: [https://kubernetes.default.svc,argo-test-ns](https://kubernetes.default.svc,argo-test-ns)
[https://kubernetes.default.svc,argo-test-2-ns](https://kubernetes.default.svc,argo-test-2-ns)
…*
```

*尝试再次创建应用程序:*

```
*$ argocd app create guestbook-2 — repo [https://github.com/argoproj/argocd-example-apps.git](https://github.com/argoproj/argocd-example-apps.git) — path guestbook — dest-server [https://kubernetes.default.svc](https://kubernetes.default.svc) — dest-namespace argo-test-2-ns — project test-project
application ‘guestbook-2’ created*
```

*现在成功了。*

*因此，在我们的 Dev Kubernetes 集群上，我们可以将目的地设置为`'*'`，因为这里的名称空间是动态创建的——开发人员正在将他们的分支部署到专用的名称空间，以便为测试设置一个多环境，但是在我们的生产集群上，我们可以对允许的名称空间设置一个硬限制。*

## *项目和角色*

## *全球角色*

*对项目的访问可以通过 te `argocd-rbac-cm` ConfigMap 进行全局设置，也可以根据项目进行本地设置。*

*让我们回到我们的全局`role:test-role`，添加一个只允许从`test-project`访问应用程序的策略，并在`policy.default`中通过设置`role: ''`禁用只读访问:*

```
*...
  policy.csv: |
    p, role:test-role, clusters, create, *, allow
    p, role:test-role, applications, *, test-project/*, allow
    g, testuser, role:test-role
  policy.default: role:''
...*
```

*切换到*测试用户*:*

```
*$ argocd context testuser@dev-1–18.argocd.example.com
Switched to context ‘testuser@dev-1–18.argocd.example.comd’*
```

*检查可用的应用程序:*

```
*$ argocd app list
NAME CLUSTER NAMESPACE PROJECT STATUS HEALTH SYNCPOLICY CONDITIONS REPO PATH TARGET
guestbook [https://kubernetes.default.svc](https://kubernetes.default.svc) argo-test-ns test-project OutOfSync Missing <none> <none> [https://github.com/argoproj/argocd-example-apps.git](https://github.com/argoproj/argocd-example-apps.git) helm-guestbook HEAD*
```

*这里唯一的一个，正如我们在政策中规定的。*

## *项目角色和身份验证令牌*

*另一种方法是为项目创建一个专门的角色。然后，您将能够像为普通用户一样为这个角色创建一个令牌。*

*在*测试项目*项目中创建*测试角色*:*

```
*$ argocd proj role create test-project test-role*
```

*添加策略—对 *guestbo* ok 应用程序的任何操作:*

```
*$ argocd proj role add-policy test-project test-role — action '*' --permission allow --object guestbook*
```

*该政策将被添加到关于该项目的 RBAC 规则中:*

```
*$ kubectl -n dev-1–18-devops-argocd-ns get appproject test-project -o jsonpath=’{.spec.roles[].policies}’
[p, proj:test-project:test-role, applications, *, test-project/guestbook, allow]*
```

*获取令牌:*

```
*$ argocd proj role create-token test-project test-role
eyJ***sCA*
```

*或者将其设置为一个变量:*

```
*$ token=$(argocd proj role create-token test-project test-role)*
```

*Ans 使用令牌检查权限，例如，同步应用程序:*

```
*$ argocd account can-i sync applications test-project/guestbook --auth-token $token
yes*
```

*好吧，这个有用，*

*但是您不能使用此角色的令牌添加群集，因为我们没有在其策略中设置它:*

```
*$ argocd account can-i create clusters '*' --auth-token $token
no*
```

*尽管如此，您的 testuser 仍然能够做到这一点，因为它在`argocd-rbac-cm`中拥有权限 et，其策略为*测试角色* - `p, role:test-role, clusters, create, *, allow`:*

```
*$ argocd account can-i create projects '*'
yes*
```

# *ArgoCD 组*

*使用 RBAC 规则，您可以将角色分组到组中，但现在用户可以分组，例如:*

```
*...
  policy.csv: |
    g, argocd-admins, role:admin
...*
```

*稍后，使用 SSO，我们将把用户组映射到 ArgoCD 角色。*

# *把所有的放在一起*

*好了，现在我们已经熟悉了 ArgoCD 中的用户管理，让我们来计划一下如何使用它。*

*我们有什么？*

*两个团队 Web 和后端。*

*每个团队都有自己的一套应用程序。*

*从全局用户中，我们可以为 DevOps 团队创建一个具有完全访问权限的根用户，以及一个绑定到项目的管理员和只读用户。*

*在一个项目中，我们可以通过使用它的令牌来指定一个将在 Github Actions/Jenkins 管道中使用的角色。*

*稍后，我们将配置 Oktaa 和 SSO，这样用户将使用他们的 Okta 凭据和 Okta 组登录 ArgoCD WebUI，Okta 中的后端团队将访问 ArgoCD 中的后端项目，Okta 中的 Web 团队将访问 ArgoCD 中的 Web 项目。*

*就目前而言，仅此而已。*

*在下面的帖子中，我们将配置 Okta 和 ArgoCD，参见 [ArgoCD: Okta 集成和用户组](https://rtfm.co.ua/en/?p=26037)帖子。*

**最初发表于* [*RTFM: Linux、DevOps 和系统管理*](https://rtfm.co.ua/en/argocd-users-access-and-rbac/) *。**