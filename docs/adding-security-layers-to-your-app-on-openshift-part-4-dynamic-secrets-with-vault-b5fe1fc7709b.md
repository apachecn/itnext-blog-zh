# 在 OpenShift 上向您的应用添加安全层—第 4 部分:使用 Vault 的动态秘密

> 原文：<https://itnext.io/adding-security-layers-to-your-app-on-openshift-part-4-dynamic-secrets-with-vault-b5fe1fc7709b?source=collection_archive---------1----------------------->

今天，保护你的应用程序是必须的，但是如果你没有在一开始就考虑它，那么在不修改代码的情况下引入它是很困难的。幸运的是，像 [Kubernetes](http://kubernetes.io) 这样的容器和平台带来的新的云原生模式提供了简单的方法来解决安全问题，而无需接触代码。

在本系列的前一部分中，我们已经看到了如何将 [Hashicorp Vault](https://www.vaultproject.io/) 轻松部署到 [Kubernetes](http://www.kubernetes.io) 上，并从托管在同一平台上的`fruits-catalog` 应用程序中使用。这次我们将使用一个叫做[动态秘密](https://learn.hashicorp.com/vault/getting-started/dynamic-secrets)的高级功能。根据保险库网站:

> 动态机密是短暂的，当它们被访问时以编程方式生成，并且直到它们被读取时才存在，从而降低了有人窃取它们或另一个客户端使用相同机密的风险。动态机密可以在使用后立即撤销，从而最大限度地缩短机密的生命周期。

我们将添加这个动态秘密特性来访问我们的 MongoDB 数据库。这一部分是前一部分的直接延续，估计你已经读过了。

![](img/9fd8d4b85278bfbd66e12fc3d4129ff1.png)

第 4 部分—在 MongoDB 中使用 Vault 动态秘密

这篇文章是一个更大的系列文章的一部分，讲述了如何给现有的应用程序增加安全层。目前公布的其他部分有:

*   [*第 1 部分—部署和 TLS 入口*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-1-deployment-and-tls-ingress-9ef752835599)
*   [*第二部分——使用 Keycloak 的认证和授权*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-2-8320018bcdd1)
*   [*第三部分——金库的秘密管理*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-3-secret-management-with-vault-8efd4ec29ec4)
*   *第四部分——带金库的动态秘密(本帖)*
*   [*第 5 部分——与 Istio*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-5-mutual-tls-with-istio-a8800c2e4df4) 的相互 TLS
*   [*第 6 部分—使用保管库和证书管理器的 PKI 服务*](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-part-6-pki-as-a-service-with-vault-and-cert-e6dbbe7028c7)

# 我们想在这部分实现什么？

因此，在这一部分中，我们将重用和重新配置已经安装的 Vault 实例，以便它能够在应用程序 pod 启动时为 MongoDB 生成动态机密。这里需要注意的重要一点是，将为每个 pod 生成一个新的不同的动态秘密！因此，如果您需要扩展您的应用程序，添加新的`fruits-catalog` pod 副本，Vault 将为每个实例创建新的秘密。

下面是我们的应用程序架构的新模式:

![](img/484bb6d02fc2202138af640e30b7937a.png)

这个架构转换之后的新流程将向我们在本系列第 3 部分中开始的步骤添加新的动态秘密生成步骤。总而言之，从新应用程序盒的初始化开始:

1.  我们将获得一个由 Kubernetes 颁发的特定令牌——以及相关的证书——在启动时注入到我们的应用程序 Pod 中，
2.  从一个名为`vault-init`的 [Init 容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)中，也运行在我们的 Pod 中，我们将使用这个特定的令牌对 Vault 进行身份验证调用，
3.  Vault 将能够根据 Kubernetes APIs 检查这个调用的有效性，
4.  如果成功，Vault 现在将使用其内部秘密连接到 MongoDB 数据库，并创建一个新的用户/密码对。这一对(由租约在保险库端标识)将被发送到具有租约标识符的 init 容器，
5.  然后，从 Vault 中检索到的秘密将通过一个共享文件系统提供给我们的主 Spring Boot 容器:一个配置文件将在`/deployment/config`上创建，租约标识符也将存储在本地。我们的主容器现在应该可以启动了，
6.  但是 Vault 提供的租约应该定期更新…为此我们现在有一个名为`vault-lease-refresher`的[边车容器](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#understanding-pods)来为我们处理这个任务…直到我们的 pod 死亡，
7.  当我们的 pod 将死亡时(由于异常或按比例缩小的决定)，由于有一个[预停止钩](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#container-hooks)，也将调用`vault-lease-refresher`。最后一次调用将负责撤销 Vault 端的租约，并销毁 MongoDB 端的用户/密码对。

轻松点。？好吧，让我们看看如何实现这一点…

# 如何应用？

如果您已经阅读了本系列的第一部分，那么您可能已经克隆了包含所有资源和资产的 [my GitHub repository](https://github.com/lbroudoux/secured-fruits-catalog-k8s) 。如果没有，我敦促你这样做；-)从您笔记本电脑上的克隆中，切换到`master`分支。

对于这第四部分，第一步将是激活称为`database`的金库机密引擎。在这个引擎中，我们将为我们的基础创建一个新的配置，以便将连接设置馈送到数据库的 Vault。注意，因为我们和数据库在同一个 Kubernetes 名称空间中，所以我们可以使用简短的`mongodb.fruits-catalog.svc.cluster.local`符号。最后，我们必须为我们的数据库创建一个新的角色；此角色将描述新租赁的生存时间以及应该用于发布该租赁的声明:

```
$ vault secrets enable -tls-skip-verify database
Success! Enabled the database secrets engine at: database/$ vault write -tls-skip-verify database/config/fruit-catalog-mongodb \
   plugin_name=mongodb-database-plugin \
   allowed_roles="fruit-catalog-role" \
   connection_url="mongodb://{{username}}:{{password}}@mongodb.fruits-catalog.svc.cluster.local:27017/admin" \
   username="admin" \
   password="$(oc get secret/mongodb -o jsonpath="{.data.database-admin-password}" | base64 -D )"
Success! Data written to: database/roles/fruit-catalog-mongodb$ vault write -tls-skip-verify database/roles/fruit-catalog-role \
   db_name=fruit-catalog-mongodb \
   creation_statements='{ "db": "sampledb", "roles": [{"role": "readWrite", "db": "sampledb"}] }' \
   default_ttl="1h" \
   max_ttl="24h" \
   revocation_statements='{ "db": "sampledb" }'
Success! Data written to: database/roles/fruit-catalog-role
```

使用 Vault CLI 或贵由还可以轻松检查我们刚刚创建的资源的配置:

```
$ vault read -tls-skip-verify database/config/fruit-catalog-mongodb
Key                                   Value
---                                   -----
allowed_roles                         [fruit-catalog-role]
connection_details                    map[connection_url:mongodb://{{username}}:{{password}}@mongodb.fruits-catalog.svc.cluster.local:27017/admin username:admin]
plugin_name                           mongodb-database-plugin
root_credentials_rotate_statements    []$ vault read -tls-skip-verify database/roles/fruit-catalog-role
Key                      Value
---                      -----
creation_statements      [{ "db": "sampledb", "roles": [{"role": "readWrite", "db": "sampledb"}] }]
db_name                  fruit-catalog-mongodb
default_ttl              1h
max_ttl                  24h
renew_statements         []
revocation_statements    [{ "db": "sampledb" }]
rollback_statements      []
```

与前一部分处理静态机密一样，我们需要在 Vault 中创建一个策略，允许读取 MongoDB 的动态机密。使用 Vault CLI 时，我们以管理员身份登录，但我们的服务帐户并非如此。在前面的部分中，我们已经激活了 Kubernetes 身份验证方法，这个策略还需要关联到服务帐户和 Kubernetes 名称空间。这就是以下两个命令的意义所在:

```
$ vault policy write -tls-skip-verify fruit-catalog-policy-dynamic ./k8s/fruit-catalog-policy-dynamic.hcl
Success! Uploaded policy: fruit-catalog-policy-static$ vault write -tls-skip-verify auth/kubernetes/role/fruits-catalog \
   bound_service_account_names=fruits-catalog-vault \
   bound_service_account_namespaces=fruits-catalog \
   policies=fruit-catalog-policy-static,fruit-catalog-policy-dynamic \
   ttl=24h
Success! Data written to: auth/kubernetes/role/fruits-catalog
```

最后，只需用一个包含`vault-init` Init 容器和`vault-lease-refresher` Sidecar 容器的新容器替换 OpenShift 中现有的`Deployment`。只是不要忘记在那之前用你的值替换我的路由值，使用一些无用的`sed`命令；-)

```
$ export VAULT_ROUTE=$(oc get route/vault -n fruits-catalog | grep vault | awk '{print $2}')
$ export KEYCLOAK_ROUTE=$(oc get route/keycloak -n fruits-catalog | grep keycloak | awk '{print $2}')$ sed -i '' 's=vault-fruits-catalog.apps.144.76.24.92.nip.io='"$VAULT_ROUTE"'=' k8s/fruits-catalog-dynamic-secret-deployment.yml
$ sed -i '' 's=keycloak-fruits-catalog.apps.144.76.24.92.nip.io='"$KEYCLOAK_ROUTE"'=' k8s/fruits-catalog-dynamic-secret-deployment.yml$ oc replace -f k8s/fruits-catalog-dynamic-secret-deployment.yml
```

> 我们在这里配置部署的方式很难理解——一切都是“手动”进行的。虽然检查以了解细节很有趣，但现在有一种更简单的方法可以做到这一点。我在那个上发布了一个[更新，使用了新的保险库代理注入器。](https://medium.com/@lbroudoux/adding-security-layers-to-your-app-on-openshift-update-welcome-vault-agent-injector-46cab161c366)

# 检查它是否按预期工作

当检查它是否工作时，首先要做的显然是使用`fruits-catalog`路径的 URL 打开一个浏览器，检查应用程序是否还在工作。但是如果你想/需要深潜，这里有一些其他的检查。

首先，我们可以使用以下命令从 Vault 本身重新读取每个配置项:

```
$ vault read -format json database/creds/fruit-catalog-role
{
  "request_id": "d48cea61-18b1-b34e-67ea-eea1850fb3cf",
  "lease_id": "database/creds/fruit-catalog-role/3bxsRmUzwtUu2r1m6q2zJLTM",
  "lease_duration": 3600,
  "renewable": true,
  "data": {
    "password": "A1a-hQUdn9qxGdSk0qpK",
    "username": "v-root-fruit-catalog-r-Oy05HvQHmZ0tu3LFZ8Lp-1560953738"
  },
  "warnings": null
}$ vault lease renew database/creds/fruit-catalog-role/3bxsRmUzwtUu2r1m6q2zJLTM
Key                Value
---                -----
lease_id           database/creds/fruit-catalog-role/3bxsRmUzwtUu2r1m6q2zJLTM
lease_duration     1h
lease_renewable    true$ vault lease revoke database/creds/fruit-catalog-role/3bxsRmUzwtUu2r1m6q2zJLTM
All revocation operations queued successfully!
```

然后，我们可以看一下`fruits-catalog`吊舱部署，并检查我们是否一切就绪。以下是 OpenShift web 控制台的屏幕截图，显示了:

*   我们已经成功完成了一个初始化容器。我们还可以访问这个`vault-init`容器发出的命令，
*   我们已经有了第一个名为`vault-lease-refresher`的容器，并且仍然可以访问后一个容器发出的命令。稍后会有更多的介绍，
*   我们最终得到了第二个也是主要的容器，它是我们的应用程序，使用由 Init 容器编写的`application.properties`配置。

![](img/8a31b08d02e1598bfedb78adb7048d1f.png)

需要注意的两件重要事情是`vault-lease-refresher`容器使用的命令。正如在关于`vault-init` 容器的前一篇文章中所解释的，这个容器使用了一个极简的[Red Hat Universal Base Image](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)，它刚刚用 [jq](https://stedolan.github.io/jq/) 工具进行了增强。当查看[部署](https://github.com/lbroudoux/secured-fruits-catalog-k8s/blob/master/k8s/fruits-catalog-dynamic-secret-deployment.yml#L61)的 YAML 时，您首先会发现这个命令是一个用 bash shell 编写的无限循环，它定期调用我们的 Vault 实例上的适当 API 来刷新租约:

```
X_VAULT_TOKEN=$(cat /etc/vault/token);
VAULT_LEASE_ID=$(cat /etc/app/creds.json | jq -j '.lease_id');while true; do
  curl -k -s --request PUT --header "X-Vault-Token: $X_VAULT_TOKEN" --data '{"lease_id": "'"$VAULT_LEASE_ID"'", "increment": 3600}' [https://vault-fruits-catalog.apps.x.x.x.x.nip.io/v1/sys/leases/renew;](https://vault-fruits-catalog.apps.144.76.24.92.nip.io/v1/sys/leases/renew;)
  echo 'Waiting for 360 sec before renewing lease'
  sleep 360;
done
```

第二个需要注意的命令是使用[预停止生命周期挂钩](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#container-hooks)删除 Pod 时调用的命令。这里，我们再次调用一个 Vault API 来使用租约 Id 撤销所使用的机密:

```
X_VAULT_TOKEN=$(cat /etc/vault/token);
VAULT_LEASE_ID=$(cat /etc/app/creds.json | jq -j '.lease_id');curl -k --request PUT --header "X-Vault-Token: $X_VAULT_TOKEN" --data '{"lease_id": "'"$VAULT_LEASE_ID"'"}' https://vault-fruits-catalog.apps.x.x.x.x.nip.io/v1/sys/leases/revoke;
```

检查这种动态行为的一个简单演示是为 MongoDB 数据库连接一个图形客户端，并在您扩大或缩小`fruits-catalog`部署时实时查看用户和秘密的出现和消失。例如，您可以使用管理员凭证使用机器人 3T 连接到 MongoDB 实例。为此，您可能希望 CLI 的`port-forward`特性带有类似`oc port-forward mongodb-1-t85wm 56866:27017 -n fruits-catalog`的命令

![](img/88e280831e34533136dc3d3a46acb41d.png)

然后连接本地端口并检查`system.users`集合，根据 pods 副本的数量查看`sampledb`数据库上的用户数量；-)

# 结论和下一步

我们已经在第四部分中看到，如何让 Hashicorp [Vault](http://vaultproject.io/) 为我们管理数据库凭证！由于上一篇文章演示了 Vault 的一般秘密管理，我们在这里看到了通过其专用引擎与数据库的高级集成。

这种集成非常强大，因为您不再需要存储、分发和管理应用程序的数据库凭据！使用带有生命周期挂钩的 [Init](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) 和 [Sidecar](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#understanding-pods) 容器，您现在就拥有了短暂的凭证，它的生命周期就是 pod 的生命周期。Pods 通过 Kubernetes 集成，使用与其自己的服务帐户相关联的证书进行 Vault 身份验证。更好的是:我们没有接触应用程序代码；-)

现在想象一下这个功能的强大之处，再加上数据库端的审计日志……由于用户名包含 pod 标识符，我们现在能够准确地知道哪个副本在数据库上完成了哪个操作……继续讨论这个问题可能会很有趣，我们将在下一篇文章中切换到另一个主题，讨论零信任网络和集群内传输安全性。敬请期待！