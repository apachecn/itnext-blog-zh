# Neo4j:奔跑在库伯内特斯

> 原文：<https://itnext.io/neo4j-running-in-kubernetes-79633944f213?source=collection_archive---------6----------------------->

![](img/1b1efd94550cc3b0c360821bccba6112.png)

在之前的帖子中— [Neo4j:图形数据库—用 Docker 和 Cypher QL 运行示例](https://rtfm.co.ua/en/neo4j-graph-database-run-with-docker-and-cypher-ql-examples/) —我们已经用в Docker 运行了 Neo4j 数据库。

下一个任务是在 Kubernetes 集群中运行它。

将使用 Neo4j Community Edition，它将作为单节点实例运行，因为 Neo4j 的群集功能在企业版中提供，其成本约为 190.000 美元/年。

对于社区版，我们可以应用一个掌舵图——[neo4j——社区](https://hub.helm.sh/charts/equinor-charts/neo4j-community)。

添加其存储库:

```
$ helm repo add equinor-charts [https://equinor.github.io/helm-charts/charts/](https://equinor.github.io/helm-charts/charts/)
“equinor-charts” has been added to your repositories$ helm repo update
Hang tight while we grab the latest from your chart repositories…
…Successfully got an update from the “equinor-charts” chart repository
…Successfully got an update from the “stable” chart repository
Update Complete. ⎈ Happy Helming!
```

部署到一个定制的名称空间，在本例中，它是 *eks-dev-1-neo4j* 。

用`--set`接受它的许可，并为它的[持久卷](https://rtfm.co.ua/en/kubernetes-persistentvolume-and-persistentvolumeclaim-an-overview-with-examples/)的`[StorageClass](https://rtfm.co.ua/en/kubernetes-persistentvolume-and-persistentvolumeclaim-an-overview-with-examples/#StorageClass)`添加值，并添加一个供管理员访问的密码:

```
$ helm upgrade — install — namespace eks-dev-1-neo4j — create-namespace neo4j-community equinor-charts/neo4j-community — set acceptLicenseAgreement=yes — set neo4jPassword=mySecretPassword — set persistentVolume.storageClass=gp2
Release “neo4j-community” does not exist. Installing it now.
NAME: neo4j-community
LAST DEPLOYED: Fri Jul 31 15:27:38 2020
NAMESPACE: eks-dev-1-neo4j
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
We’ll need to wait a few seconds for Neo4j to start.
We need to see this line in our pod log:
> Remote interface available at Remote interface available at [http://localhost:7474/](http://localhost:7474/)
We can see the content of the logs by running the following command:
kubectl logs -l “app=neo4j-community”
```

好的——部署完毕，检查吊舱:

```
$ kk -n eks-dev-1-neo4j get pod
NAME READY STATUS RESTARTS AGE
neo4j-community-neo4j-community-0 1/1 Running 0 29s
```

检查其 [PersistentVolumeClaim](https://rtfm.co.ua/en/kubernetes-persistentvolume-and-persistentvolumeclaim-an-overview-with-examples/) :

```
$ kk -n eks-dev-1-neo4j get pvc
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
datadir-neo4j-community-neo4j-community-0 Bound pvc-29c5d6a0–34b7–4b5c-94d2–778139790a2d 10Gi RWO gp2 15m
```

检查相应的 [PersistentVolume](https://rtfm.co.ua/en/kubernetes-persistentvolume-and-persistentvolumeclaim-an-overview-with-examples/) —注意，我们已经以动态方式创建了该 PVC，因此它的回收策略设置为 *Delete* :

```
$ kk -n eks-dev-1-neo4j get pv pvc-29c5d6a0–34b7–4b5c-94d2–778139790a2d
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
pvc-29c5d6a0–34b7–4b5c-94d2–778139790a2d 10Gi RWO Delete Bound eks-dev-1-neo4j/datadir-neo4j-community-neo4j-community-0 gp2 16m
```

现在，运行端口转发来从您的工作站访问 Neo4j 服务器:

```
$ kubectl -n eks-dev-1-neo4j port-forward neo4j-community-neo4j-community-0 7474:7474
Forwarding from 127.0.0.1:7474 -> 7474
Forwarding from [::1]:7474 -> 7474
```

检查连接:

```
$ curl [http://localhost:7474/db/data/](http://localhost:7474/db/data/)
{
“errors” : [ {
“code” : “Neo.ClientError.Security.Unauthorized”,
“message” : “No authentication header supplied.”
} ]
}
```

服务器已准备好工作。

# 有用的链接

*   [Neo4j-Helm 用户指南](https://github.com/neo4j-contrib/neo4j-helm/blob/master/user-guide/USER-GUIDE.md)
*   [编排环境中的 Neo4j 考虑事项](https://medium.com/neo4j/neo4j-considerations-in-orchestration-environments-584db747dca5)
*   [如何备份运行在 Kubernetes](https://medium.com/neo4j/how-to-backup-neo4j-running-in-kubernetes-3697761f229a) 的 Neo4j
*   [Neo4j 团簇的外部曝光](https://github.com/neo4j-contrib/neo4j-helm/blob/master/tools/external-exposure/EXTERNAL-EXPOSURE.md)

*最初发布于* [*RTFM: Linux，devo PSисистемноеадмнииитииовваниование*](https://rtfm.co.ua/en/neo4j-running-in-kubernetes/)*。*