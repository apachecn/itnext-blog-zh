# kubernetes:Gorush 服务器上的配置图和秘密示例

> 原文：<https://itnext.io/kubernetes-configmaps-and-secrets-on-a-gorush-server-example-2a9539ac92e2?source=collection_archive---------3----------------------->

![](img/b507bdfa1910ba0d8b988a12cd2d2352.png)

我们有一个来自 [Kubernetes 的 Gorush 服务器:在 AWS LoadBalancer](https://rtfm.co.ua/en/kubernetes-running-a-push-server-with-gorush-behind-an-aws-loadbalancer/) 帖子后面运行带有 Gorush 的推送服务器，我想添加一个通过 Github 存储库配置它的功能，并使用一组不同的设置运行它——用于暂存和生产环境。

让我们使用 Kubernetes [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) 来存储配置文件内容，使用 Kubernetes [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) 来存储机密数据。

此时，在`gorush-configmap.yaml`中，我们有了 Gorush 与 Redis 连接设置的默认配置图:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: gorush-config
  namespace: gorush
data:
  # stat
  stat.engine: redis
  stat.redis.host: redis:6379
```

同样，需要补充的是:

*   用于 Apple 推送服务鉴定的 SSL 证书
*   此证书的密码

我们将添加两个秘密——一个用于密码，一个用于密钥文件。

## 库伯内特的秘密

创建一个新文件`gorush-secrets.yaml`并添加一个 p12-key 主体。

由于这是 p12 —使用`data`类型的密码，在`base64`中获取您的证书:

```
$ cat apns_wl.p12 | base64
MIIM9QIBAzCCDLwGCSqGSIb3DQEHAaCCDK0EggypMIIMpTCCBycGCSqGSIb3DQEHBqCCBxgwggcU
AgEAMIIHDQYJKoZIhvcNAQcBMBwGCiqGSIb3DQEMAQYwDgQI+BcNML0eE1oCAggAgIIG4KVvRBPp
…
```

添加一个名为 *ios-push-rsa-key* 的`data`地图:

```
---
apiVersion: v1
kind: Secret
metadata:
  name: ios-push-secret
  namespace: gorush
type: Opaque
data:
  ios-push-rsa-key: |
    MIIM9QIBAzCCDLwGCSqGSIb3DQEHAaCCDK0EggypMIIMpTCCBycGCSqGSIb3DQEHBqCCBxgwggcU
    AgEAMIIHDQYJKoZIhvcNAQcBMBwGCiqGSIb3DQEMAQYwDgQI+BcNML0eE1oCAggAgIIG4KVvRBPp
    ...
```

或者。如果它是一个 PEM 编码的密钥——您可以使用`stringData`而不使用`base64`,只是一个明文:

```
---
apiVersion: v1
kind: Secret
metadata:
  name: ios-push-secret
  namespace: gorush
type: Opaque
stringData:
  ios-push-rsa-key: |
    -----BEGIN CERTIFICATE-----
    MIIGJDCCBQygAwIBAgIIAtw7YRUahfYwDQYJKoZIhvcNAQELBQAwgZYxCzAJBgNV
    BAYTAlVTMRMwEQYDVQQKDApBcHBsZSBJbmMuMSwwKgYDVQQLDCNBcHBsZSBXb3Js
    ZHdpZGUgRGV2ZWxvcGVyIFJlbGF0aW9uczFEMEIGA1UEAww7QXBwbGUgV29ybGR3
...
```

创造秘密:

```
$ kubectl apply -f gorush-secrets.yaml
secret/ios-push-secret created
```

检查一下:

```
$ kubectl -n gorush describe secret ios-push-secret
Name: ios-push-secret
Namespace: gorush
Labels: <none>
Annotations:
Type: Opaque
Data
====
ios-push-rsa-key: 4586 bytes
```

现在，我们需要将这个证书安装到 pod 和容器上。

## Kubernetes 秘密和`volumeMounts` -秘密，作为文件

更新`gorush-deployment.yaml`文件-从章节添加`[volumes](https://kubernetes.io/docs/concepts/storage/volumes/)`:

```
...
      volumes:
      - name: ios-push-secret-vol
        secret:
          secretName: ios-push-secret
          items:
          - key: ios-push-p12-key
            path: apns-crt.p12
```

并将其添加到 pod 的模板中:

```
...
    spec:
      containers:
      - image: appleboy/gorush
        name: gorush
        imagePullPolicy: Always
        ports:
        - containerPort: 8088
...
        volumeMounts:
        - name: ios-push-secret-vol
          mountPath: /data/ssl/apns-crt.p12
          subPath: apns-crt.p12
          readOnly: true
```

为了说明它们是如何相互联系的:

*   `kind: Secret` : //只是一个秘密类型
*   `name: ios-push-secret` //一个秘密名称——将在*规格>卷>秘密>中使用 secretName*
*   `data`:
*   `ios-push-rsa-key` //证书主体
*   `volumes` : //在*规格*中设定，将添加卷
*   `name: ios-push-secret-vol` //一个创建的卷名，将在*规格>容器>卷装载>名称*中使用
*   `secret` : //定义要使用的数据
*   `secretName: ios-push-secret` //查找名为*的秘密:ios-push-secret* 名
*   `items`://andsideofthesecret
*   `- key: ios-push-rsa-key` //寻找*秘密>数据> ios-push-rsa-key*
*   `path: apns-rsa.pem` //以及它必须如何安装
*   `volumeMounts` : //用于*规格>容器*
*   `name: ios-push-secret-vol` //要挂载的卷名称- *规格>卷>名称:ios-push-secret-vol*
*   `mountPath: /data/ssl/apns-rsa.pem` //完整的挂载路径
*   `subPath: apns-rsa.pem` //如果秘密作为专用文件安装，则目录中的文件名

部署和检查:

```
kubectl -n gorush exec -ti gorush-7ff8fd9f4c-gds8r -c bastion cat /data/ssl/apns-rsa.pem
Bag Attributes
friendlyName: Apple Push Services: ***
localKeyID: C0 3A 96 69 CB C4 9E E6 14 EB 43 1F 31 30 97 4D 85 89 A0 8D
subject=/UID=***/CN=Apple Push Services: ***/OU=7MF8BB6LXN/O=***/C=VG
issuer=/C=US/O=Apple Inc./OU=Apple Worldwide Developer Relations/CN=Apple Worldwide Developer Relations Certification Authorit
 — — -BEGIN CERTIFICATE — — -
MIIGJDCCBQygAwIBAgIIAtw7YRUahfYwDQYJKoZIhvcNAQELBQAwgZYxCzAJBgNV
BAYTAlVTMRMwEQYDVQQKDApBcHBsZSBJbmMuMSwwKgYDVQQLDCNBcHBsZSBXb3Js
…
```

## Kubernetes 秘密和`env` -秘密，作为环境变量

下面的步骤是为该证书添加密码。

有两种选择——使用配置文件或通过变量。

在配置文件中，Gorush 有一个`ios.password`参数:

```
...
    ios:
      enabled: true
      key_path: "/data/ssl/apns-crt.p12"
      password: "p@ssw0rd"
      production: true
...
```

如果使用变量——Gorush 将添加一个`[GORUSH](https://github.com/appleboy/gorush/blob/master/config/config.go#L224)`前缀:

```
...
  viper.SetEnvPrefix("gorush") // will be uppercased automatically
...
```

因此，我们的变量名将是`GORUSH_IOS_PASSWORD`。

向 *ios-push-secret* 添加新值。

添加一个带有`stringData`类型的附加地图:

```
---
apiVersion: v1
kind: Secret
metadata:
  name: ios-push-secret
  namespace: gorush-stage
type: Opaque
data:
  ios-push-p12-key: |
    MIIM9QIBAzCCDLwGCSqGSIb3DQEHAaCCDK0EggypMIIMpTCCBycGCSqGSIb3DQEHBqCCBxgwggcU
    AgEAMIIHDQYJKoZIhvcNAQcBMBwGCiqGSIb3DQEMAQYwDgQI+BcNML0eE1oCAggAgIIG4KVvRBPp
...
stringData:
  ios-push-p12-pass: p@ssw0rd
```

检查一下:

```
$ kubectl apply -f gorush-secrets.yaml
secret/ios-push-secret configured
```

现在检查—秘密中必须有两个对象—使用`describe secret`查看它们:

```
$ kubectl -n gorush describe secret ios-push-secret
Name: ios-push-secret
Namespace: gorush
Labels: <none>
Annotations:
Type: Opaque
Data
====
ios-push-p12-key: 3321 bytes
ios-push-p12-pass: 13 bytes
```

使用`secretKeyRef`将`GORUSH_IOS_PASSWORD`变量添加到部署中:

```
...
    spec:
      containers:
      - image: appleboy/gorush
        name: gorush
        imagePullPolicy: Always
        ports:
        - containerPort: 8088
...
        env:
        - name: GORUSH_IOS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ios-push-secret
              key: ios-push-p12-pass
        volumeMounts:
        - name: ios-push-secret-vol
          mountPath: /data/ssl/apns-crt.p12
          subPath: apns-crt.p12
          readOnly: true
...
```

部署和检查:

```
$ kubectl -n gorush exec -ti gorush-58bcd746c4–67b9n -c gorush env | grep GORUSH_IOS
GORUSH_IOS_PASSWORD=p@ssw0rd
```

它在这里。

## 配置图

## Kubernetes 配置图—配置，作为文件

最后要做的是为 Gorush 本身添加一个专用的配置文件。

该文件将是相同的生产和暂存，但密钥将从静脉曲张的秘密。

同样，使用变量和配置图，我们可以为不同的环境传递任何其他参数。

因此，让我们使用 ConfigMap 来保存`config.yml`文件的内容，以便稍后将其映射到 pods。

Gorush 已经有一个`gorush-configmap.yaml`文件-添加一个名为*的新配置 gorush-config-file* :

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: gorush-config
  namespace: gorush
data:
  # stat
  stat.engine: redis
  stat.redis.host: redis:6379
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: gorush-config-file
  namespace: gorush
data:
  gorush-config-file: |
    core:
      enabled: true
      mode: "release"
      port: "8088"
      max_notification: 1000
      sync: false
...
```

将其添加到我们部署中的`volumes`:

```
...
      volumes:
      - name: ios-push-secret-vol
        secret:
          secretName: ios-push-secret
          items:
          - key: ios-push-p12-key
            path: apns-crt.p12
      - name: gorush-config-file-vol
        configMap:
          name: gorush-config-file
          items:
            - key: gorush-config-file
              path: config.yml
```

以及 pods 模板中`volumeMounts`上的:

```
...
    spec:
      containers:
      - image: appleboy/gorush
        name: gorush
        imagePullPolicy: Always
        ports:
        - containerPort: 8088
        ...
        env:
        - name: GORUSH_IOS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ios-push-secret
              key: ios-push-p12-pass
        volumeMounts:
        - name: ios-push-secret-vol
          mountPath: /data/ssl/apns-crt.p12
          subPath: apns-crt.p12
          readOnly: true
        - name: gorush-config-file-vol
          mountPath: /config.yml
          readOnly: true
          subPath: config.yml
...
```

更新配置映射:

```
$ kubectl apply -f gorush-configmap.yaml
configmap/gorush-config unchanged
configmap/gorush-config-file created
```

检查它们:

```
$ kubectl -n gorush get configmap
NAME DATA AGE
gorush-config 2 28h
gorush-config-file 1 76s
```

在 pod 中应用部署并检查配置:

```
$ kubectl -n gorush-stage exec -ti gorush-6bc78df55d-9w9m8 -c gorush cat /config.yml
core:
enabled: true
mode: “release”
port: “8088”
max_notification: 1000
sync: false
…
ios:
enabled: true
key_path: “/data/ssl/apns-crt.p12”
password: “”
production: true
```

要自动应用来自配置图或机密的新值，您可以使用重新加载器控制器，请参见 [Kubernetes:配置图和机密 pods 中的数据自动重新加载](https://rtfm.co.ua/en/kubernetes-configmap-and-secrets-data-auto-reload-in-pods/)帖子。

完成了。

*最初发布于* [*RTFM: Linux，devo PSисистемноеадмнииитииовваниованиде*](https://rtfm.co.ua/en/kubernetes-configmaps-and-secrets-on-a-gorush-server-example/)*。*