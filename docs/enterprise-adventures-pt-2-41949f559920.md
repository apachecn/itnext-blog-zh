# 企业冒险:从内部部署到云中

> 原文：<https://itnext.io/enterprise-adventures-pt-2-41949f559920?source=collection_archive---------2----------------------->

我在从遗产地到库伯内特天堂的应许之地的旅途中遇到了一个有趣的问题。

旧环境中有一个 CI 基础设施，包括一个 git 存储库、一个 Jenkins 构建服务器和一个 Nexus 工件存储库。

云中有一个环境，有一个很好的 Kubernetes 集群，还有一个保存部署描述符的 git 存储库、一个包含 Docker 映像的注册表和另一个负责持续部署的 Jenkins 实例(当然有一些手动步骤)。

问题是，由于一些非常重要的原因(每个企业中总有一些非常重要的原因让您的生活变得更轻松)，源代码不能被推到云中，这意味着工件是在遗留环境中构建的，并上传到那里的 Nexus 服务器。

当有人决定这样做时，工件被下载到开发人员的机器上，提交到第二版本控制中，然后上传到云环境中。在那里，Jenkins 服务器启动了一个 pod (docker，kubectl ),它将从 jar 文件创建一个 docker 映像，并将它推送到注册中心，Kubernetes 可以从那里访问和部署它。

你必须为网络流量支付额外的费用，因此你不可能每 10 分钟就下载一次工件——而且速度也会很慢。对于代码来说也是如此，我们不能在每次提交时都进行拉取和构建——这很昂贵。

对我来说，这是非常丑陋和痛苦的——尤其是手动将二进制文件提交到 git 并对它们进行版本控制，所以我想改变这一点。

总结:目标不是手动将 jar 文件签入 git，而是同时节省网络流量。

## Sha1 来救援

上面这些管道的创建者没有意识到的技巧是，当您创建 maven 工件并部署它们时，您会在二进制文件中获得一些额外的元数据，请看:

```
bnemeth@mash:~/.m2/repository/com/edudoo/sample/1.0-SNAPSHOT$ ls -la
total 104
drwxrwxr-x 2 bnemeth bnemeth  4096 szept 14 15:08 .
drwxrwxr-x 3 bnemeth bnemeth  4096 aug   27 12:46 ..
-rw-rw-r-- 1 bnemeth bnemeth 14345 szept  3 10:51 sample-1.0-20180903.072937-26.jar
**-rw-rw-r-- 1 bnemeth bnemeth    40 szept  3 10:51 sample-1.0-20180903.072937-26.jar.sha1**
-rw-rw-r-- 1 bnemeth bnemeth  1564 szept  3 10:51 sample-1.0-20180903.072937-26.pom
-rw-rw-r-- 1 bnemeth bnemeth    40 szept  3 10:51 sample-1.0-20180903.072937-26.pom.sha1
-rw-rw-r-- 1 bnemeth bnemeth 14399 szept 14 14:01 sample-1.0-20180914.083257-30.jar
**-rw-rw-r-- 1 bnemeth bnemeth    40 szept 14 14:01 sample-1.0-20180914.083257-30.jar.sha1**
-rw-rw-r-- 1 bnemeth bnemeth  1157 szept 14 14:01 sample-1.0-20180914.083257-30.pom
-rw-rw-r-- 1 bnemeth bnemeth    40 szept 14 14:01 sample-1.0-20180914.083257-30.pom.sha1
-rw-rw-r-- 1 bnemeth bnemeth 14387 szept 20 14:16 sample-1.0-SNAPSHOT.jar
-rw-rw-r-- 1 bnemeth bnemeth  1157 szept 13 10:23 sample-1.0-SNAPSHOT.pom
-rw-rw-r-- 1 bnemeth bnemeth   720 szept 20 14:16 maven-metadata-local.xml
-rw-rw-r-- 1 bnemeth bnemeth   787 szept 14 15:08 maven-metadata-ply-snapshots.xml
-rw-rw-r-- 1 bnemeth bnemeth    40 szept 14 15:08 maven-metadata-ply-snapshots.xml.sha1
-rw-rw-r-- 1 bnemeth bnemeth   424 szept 20 14:16 _remote.repositories
-rw-rw-r-- 1 bnemeth bnemeth   193 szept 14 15:08 resolver-status.properties
```

每次 jar 文件改变时，sha1 散列都会改变。

这是校验和。

而且从 Nexus 下载最新神器有一个非常简单的方法(格式:Jenkinsfile 中的 groovy):

```
def downloadJar(String groupId, String artifactId, String version, String folder){
jarUrl="**http://nexus.edudoo.com/service/local/artifact/maven/content?g=${groupId}&a=${artifactId}&v=${version}&r=snapshots&c=exec**"
    sh "**curl \"${jarUrl}\"** > ${folder}/${artifactId}-exec.jar"
}
```

通过添加限定符，您也可以获得散列文件:

```
def downloadHash(String groupId, String artifactId, String version) {
    shaUrl=**"http://nexus.edudoo.com/service/local/artifact/maven/content?g=${groupId}&a=${artifactId}&v=${version}&r=snapshots&c=exec&e=jar.sha1"**
    hash=sh(returnStdout: true, script: "**curl \"${shaUrl}\" -s**")
    hash
}
```

现在，不用每次都下载 jar 文件并与之前的文件进行比较，只需下载 hash 文件并进行比较就足够了！

(是的，这与依赖管理工具确定是否下载快照工件的方式相同。)

## 缓存校验和

我们可以像以前一样将 sha1 文件提交到 git 中，并总是将最新的 sha1 与那个 sha1 进行比较，但这感觉仍然非常可疑——请记住:我们的 Jenkins 节点是一个 Kubernetes pod，因此是无状态的，我们并不真正希望对卷进行修改。

如果我们能把元数据保存在第二个环境中的某个地方就好了…

确实有办法。而不是仅仅用版本号(这是一种非常过时的跟踪您部署的工件的方式，如果您使用 CD 的话——再想一想)和经典的`latest`(快照！)，为什么不干脆用 hash 本身来标记呢？这样我们就可以知道 Docker 工件是否已经从带有 sha1 标签的 jar 中构建出来了。

要检查这一点，只需:

```
def imageExists(String registryHost, String image, String tag) {
    echo "Checking ${image} in remote repository ${registryHost} for existing ${tag} tag..."
    exists=false
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'ARTIFACTORY_DOCKER_NDC', usernameVariable: 'USER', passwordVariable: 'PASSWD']]) {
        registryResponse=sh(returnStdout: true, script: "**curl --insecure -s -u ${USER}:${PASSWD} \"https://${registryHost}/v2/${image}/tags/list\"**")
        exists=registryResponse.contains(tag)
        echo "Image exists: ${exists}"
    }
    exists
}
```

有一些认证正在进行，但重点是 docker 注册中心也有一个端点，使用 curl 语句返回图像的所有标签。请注意，*registry response . contains(tag)*被简化了，在某些情况下，这种检查 json 对象是否在某处包含特定标记的方法是不正确的，但是我将把这个实现任务留给您(提示:使用 [jq](https://stedolan.github.io/jq/) 或一些 python 库)。

(用工件的提交 ID 进行标记可能是最好的，但是我们不能访问原始的 git，只能访问工件存储库。)

所以知道步骤是:

1.  从 Nexus 下载哈希
2.  检查是否存在带有该标签的图像
3.  如果是，什么也不做
4.  如果没有，将 jar 下载到一个临时目录，构建映像，用 hash 标记映像，用所有标记推送映像

现在我们可以一直运行这个作业，如果不是绝对必要，它永远不会下载 jar！