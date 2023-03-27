# 在 Argo CD 中配置定制工具

> 原文：<https://itnext.io/configure-custom-tooling-in-argo-cd-a4948d95626e?source=collection_archive---------0----------------------->

![](img/1627dd913853e34e24d9d54a9aa0ecc0.png)

在写完第一篇文章之后的一段时间，我巧妙地使用了 jsonnet 和 gitlab，我意识到管道当然是好的，但是不必要的困难和不便。

在大多数情况下，一个典型的任务是需要:“生成 YAML，并把它放在 Kubernetes”。事实上，这是阿尔戈光盘做得很好。

Argo CD 允许您连接 Git 存储库，并将其状态同步到 Kubernetes。默认支持几种类型的应用程序:Kustomize、Helm charts、Ksonnet、raw Jsonnet 或带有 YAML/JSON 清单的简单目录。

大多数用户会为拥有这个工具集而高兴，但不是所有人。为了满足任何人的需求，Argo CD 有能力使用定制工具。

首先，我对有机会添加对 [qbec](/trying-new-tools-for-building-and-automate-the-deployment-in-kubernetes-f96f9684e580#4c4b) 和 [git-crypt](/trying-new-tools-for-building-and-automate-the-deployment-in-kubernetes-f96f9684e580#29ed) 的支持很感兴趣，这在之前的文章中有充分的讨论。

在开始配置之前，我们需要先了解 Argo 光盘是如何工作的。
对于每个添加的应用程序，它有两个阶段:

*   **init**—部署前的初始准备，任何东西都可以在这里:依赖项下载、解包秘密等等。
*   **generate** —执行生成清单的命令，输出必须是有效的 YAML 流，这正是将应用于集群的内容。

值得注意的是，Argo 将这种方法应用于任何类型的应用，包括 Helm。因此，在 Argo 中，CD Helm 不将任何版本部署到集群，而是仅用于生成清单。

从另一方面来说，Argo 能够处理头盔挂钩，这允许不打破应用释放的逻辑。

# QBEC

Qbec 允许您使用 jsonnet 方便地描述应用程序，此外，它具有渲染舵图的能力，并且由于 Argo CD 可以处理舵钩，使用 Argo CD 的这一功能可以让您获得更正确的结果。

为了给 argocd 添加 qbec 支持，您需要两件东西:

*   您的 Argo CD 配置必须包含自定义插件定义以及用于生成清单的特定命令。
*   所需的二进制文件必须在 **argocd-repo-server** 映像中可用。

第一个任务[很简单地解决了](https://argoproj.github.io/argo-cd/user-guide/config-management-plugins/):

```
# cm.yaml
data:
  configManagementPlugins: |
    - name: qbec
      generate:
        command: [sh, -xc]
        args: ['qbec show "$ENVIRONMENT" -S --force:k8s-namespace "$ARGOCD_APP_NAMESPACE"']
```

*(命令* ***初始化*** *不使用)*

```
$ kubectl -n argocd patch cm/argocd-cm -p "$(cat cm.yaml)"
```

为了添加二进制文件，建议构建一个新的映像，或者使用带有 init-container 的[技巧:](https://argoproj.github.io/argo-cd/operator-manual/custom_tools/#adding-tools-via-volume-mounts)

```
# deploy.yaml
spec:
  template:
    spec:
      # 1\. Define an emptyDir volume which will hold the custom binaries
      volumes:
      - name: custom-tools
        emptyDir: {}
      # 2\. Use an init container to download/copy custom binaries into the emptyDir
      initContainers:
      - name: download-tools
        image: alpine:3.12
        command: [sh, -c]
        args:
        - wget -qO- https://github.com/splunk/qbec/releases/download/v0.12.2/qbec-linux-amd64.tar.gz | tar -xvzf - -C /custom-tools/
        volumeMounts:
        - mountPath: /custom-tools
          name: custom-tools
      # 3\. Volume mount the custom binary to the bin directory (overriding the existing version)
      containers:
      - name: argocd-repo-server
        volumeMounts:
        - mountPath: /usr/local/bin/qbec
          name: custom-tools
          subPath: qbec
        - mountPath: /usr/local/bin/jsonnet-qbec
          name: custom-tools
          subPath: jsonnet-qbec$ kubectl -n argocd patch deploy/argocd-repo-server -p "$(cat deploy.yaml)"
```

现在让我们看看我们的应用程序的清单是什么样子的:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: qbec-app
  namespace: argocd
spec:
  destination: 
    namespace: default
    server: [https://kubernetes.default.svc](https://kubernetes.default.svc)
  project: default
  source: 
    path: examples/test-app
    targetRevision: fix-example
    plugin: 
      env: 
        - name: ENVIRONMENT
          value: dev
      name: qbec
    repoURL: [https://github.com/kvaps/qbec](https://github.com/kvaps/qbec)
  syncPolicy: 
    automated: 
      prune: true
```

环境变量包含我们需要为其生成清单的环境的名称。

申请并看看我们得到了什么:

![](img/386040379cb6033b79065e08f16cf6fc.png)

应用程序已启动并运行，太棒了！

# git-crypt

Git-crypt 允许您设置存储库的透明加密。这是在 git 中存储敏感数据的一种简单而安全的方法。

git-crypt 实现变得更加困难。理论上，我们可以在自定义插件的初始化阶段运行`git-crypt unlock`,但是这不是很方便，因为它不允许使用本地部署方法。例如，在 Helm 和 Jsonnet 的情况下，我们失去了一个简化应用程序配置(值文件等)的灵活的 GUI 界面。).
这就是为什么我想在克隆的早期就解封仓库。

因为目前 Argo CD 不提供描述任何用于同步存储库的钩子的能力，所以我不得不使用一个包装 git 命令的复杂的 shell 脚本来绕过这个限制:

```
#!/bin/sh
$(dirname $0)/git.bin "$@"
ec=$?
[ "$1" = fetch ] && [ -d .git-crypt ] || exit $ec
GNUPGHOME=/app/config/gpg/keys git-crypt unlock 2>/dev/null
exit $ec
```

每次部署操作前，Argo CD 都会运行`git fetch`。Exaclty 这个命令我用来处理执行`git-crypt unlock`来解锁仓库。

对于测试，您可以使用我的 docker 图像，它已经有了所有需要的东西:

```
$ kubectl -n argocd set image deploy/argocd-repo-server argocd-repo-server=docker.io/kvaps/argocd-git-crypt:v1.7.3
```

现在我们需要考虑 Argo 将如何解密储存库。

让我们为它生成一个 gpg 密钥:

```
$ kubectl exec -ti deploy/argocd-repo-server -- bash$ printf "%s\n" \
    "%no-protection" \
    "Key-Type: default" \
    "Subkey-Type: default" \
    "Name-Real: YOUR NAME" \
    "Name-Email: YOUR EMAIL@example.com" \
    "Expire-Date: 0" \
    > genkey-batch $ gpg --batch --gen-key genkey-batch
gpg: WARNING: unsafe ownership on homedir '/home/argocd/.gnupg'
gpg: keybox '/home/argocd/.gnupg/pubring.kbx' created
gpg: /home/argocd/.gnupg/trustdb.gpg: trustdb created
gpg: key 8CB8B24F50B4797D marked as ultimately trusted
gpg: directory '/home/argocd/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/argocd/.gnupg/openpgp-revocs.d/9A1FF8CAA917CE876E2562FC8CB8B24F50B4797D.rev'
```

保存密钥名称`8CB8B24F50B4797D`以供后续步骤使用，然后导出密钥本身:

```
$ gpg --list-keys
gpg: WARNING: unsafe ownership on homedir '/home/argocd/.gnupg'
/home/argocd/.gnupg/pubring.kbx
-------------------------------
pub   rsa3072 2020-09-04 [SC]
      9A1FF8CAA917CE876E2562FC8CB8B24F50B4797D
uid           [ultimate] YOUR NAME <YOUR EMAIL@example.com>
sub   rsa3072 2020-09-04 [E]$ gpg --armor --export-secret-keys 8CB8B24F50B4797D
```

并将其添加为单独的秘密:

```
# argocd-gpg-keys-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: argocd-gpg-keys-secret
  namespace: argocd
stringData:
  8CB8B24F50B4797D: |-
    -----BEGIN PGP PRIVATE KEY BLOCK----- lQVYBF9Q8KUBDACuS4p0ctXoakPLqE99YLmdixfF/QIvXVIG5uBXClWhWMuo+D0c
    ZfeyC5GvH7XPUKz1cLMqL6o/u9oHJVUmrvN/g2Mnm365nTGw1M56AfATS9IBp0HH
    O/fbfiH6aMWmPrW8XIA0icoOAdP+bPcBqM4HRo4ssbRS9y/i
    =yj11
    -----END PGP PRIVATE KEY BLOCK-----$ kubectl apply -f argocd-gpg-keys-secret.yaml
```

剩下的唯一一件事就是将它连接到 **argocd-repo-server** 容器，以实现这个编辑部署:

```
$ kubectl -n argocd edit deploy/argocd-repo-server
```

并将现有的 **gpg-keys** 卷类型替换为`projected`，并在那里指定我们的秘密:

```
spec:
  template:
    spec:
      volumes:
      - name: gpg-keys
        projected:
          sources:
          - secret:
            name: argocd-gpg-keys-secret
          - configMap:
            name: argocd-gpg-keys-cm
```

Argo CD 在启动时自动从这个目录加载 gpg 密钥，所以它也加载我们的私钥。

让我们检查一下:

```
$ kubectl -n argocd exec -ti deploy/argocd-repo-server -- bash
$ GNUPGHOME=/app/config/gpg/keys gpg --list-secret-keys
gpg: WARNING: unsafe ownership on homedir '/app/config/gpg/keys'
/app/config/gpg/keys/pubring.kbx
--------------------------------
sec   rsa2048 2020-09-05 [SC] [expires: 2021-03-04]
      ED6285A3B1A50B6F1D9C955E5E8B1B16D47FFC28
uid           [ultimate] Anon Ymous (ArgoCD key signing key) <noreply@argoproj.io>sec   rsa3072 2020-09-03 [SC]
      9A1FF8CAA917CE876E2562FC8CB8B24F50B4797D
uid           [ultimate] YOUR NAME <YOUR EMAIL@example.com>
ssb   rsa3072 2020-09-03 [E]
```

太好了，钥匙装好了！现在我们只需要将 Argo CD 作为合作者添加到我们的存储库中。这将实现动态自动解密。

将密钥导入本地计算机:

```
$ GNUPGHOME=/app/config/gpg/keys gpg --armor --export 8CB8B24F50B4797D > 8CB8B24F50B4797D.pem
$ gpg --import 8CB8B24F50B4797D.pem
```

信任密钥:

```
$ gpg --edit-key 8CB8B24F50B4797D
trust
5
```

将 argo 作为协作者添加到您的 git 项目中:

```
$ git-crypt add-gpg-user 8CB8B24F50B4797D
```

**相关链接**:

*   [GitHub:带有修改图像的存储库](http://github.com/kvaps/argocd-git-crypt)
*   [Argo CD:定制工具](https://argoproj.github.io/argo-cd/operator-manual/custom_tools/)
*   [Argo 光盘:配置管理插件](https://argoproj.github.io/argo-cd/user-guide/config-management-plugins/)
*   [GitHub 要点:通过批处理文件生成 gpg 密钥](https://gist.github.com/vrillusions/5484422)