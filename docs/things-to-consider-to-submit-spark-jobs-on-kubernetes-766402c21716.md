# 以集群模式在 Kubernetes 上提交 Spark 作业需要考虑的事项

> 原文：<https://itnext.io/things-to-consider-to-submit-spark-jobs-on-kubernetes-766402c21716?source=collection_archive---------1----------------------->

![](img/d5afe91bb23402adb3ec115d1c5e6e81.png)

照片由[30 日在](https://unsplash.com/@30daysreplay?utm_source=medium&utm_medium=referral) [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上播放社交媒体营销

很难在 kubernetes 上提交 spark jobs。正如之前在 kubernetes 的 spark 上的 [Hive 的帖子](/hive-on-spark-in-kubernetes-115c8e9fa5c1)中提到的，其中显示 spark thrift server 作为一个普通的 Spark 作业提交到 Kubernetes 上，有许多事情需要考虑。

在本文中，我将向您展示在 kubernetes 上以`cluster`模式提交 spark 作业时，应该考虑哪些事情来避免问题。

从下面的 git repo 中克隆源代码，以理解本文中显示的源代码。

```
[https://github.com/mykidong/hard-to-submit-spark-jobs-on-kubernetes.git](https://github.com/mykidong/hard-to-submit-spark-jobs-on-kubernetes.git)
```

# 为自己创造火花

在大多数情况下，可以下载的预构建的 spark 发行版并不适合您的需求。您必须构建 spark 来满足您的需求，例如，使用正确的 hadoop 版本，支持 hive 和 kubernetes 等。

## 检查火花源

用正确的标签检查火花源。

```
export SPARK_VERSION=3.2.2;
git clone https://github.com/apache/spark.git .;
git checkout v$SPARK_VERSION;
```

## 在`pom.xml`中添加与 S3 相关的依赖项

如今，S3 是流行的存储，所有的数据，如拼花地板，冰山表将被保存为数据湖。要使用 spark jobs 处理 S3 的数据，您必须在 spark source 中将 S3 相关的依赖项添加到`pom.xml`中，以避免 spark jobs 在`cluster`模式下提交到 kubernetes 时出现依赖项缺失问题。

```
<dependency>
   <groupId>org.apache.hadoop</groupId>
   <artifactId>hadoop-aws</artifactId>
   <version>3.2.2</version>
   <exclusions>
      <exclusion>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-core</artifactId>
      </exclusion>
      <exclusion>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-databind</artifactId>
      </exclusion>
      <exclusion>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-annotations</artifactId>
      </exclusion>
   </exclusions>
</dependency>
<dependency>
   <groupId>com.amazonaws</groupId>
   <artifactId>aws-java-sdk-s3</artifactId>
   <version>1.11.563</version>
</dependency>
```

这些依赖项用于提交带有`— packages com.amazonaws:aws-java-sdk-s3:1.11.563,org.apache.hadoop:hadoop-aws:3.2.2`选项的 spark 作业。

## 更改火花-提交

即使 spark 作业驱动程序 pod 用`exit code: 1`返回了作业失败，spark 提交也会用`exit code 0`返回作业成功。为了避免这种情况，需要将`$SPARK_HOME/bin`目录中的`spark-submit` shell 文件修改如下。

```
**#!/usr/bin/env bash** #
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

# disable randomized hash for string in Python 3.3+
export PYTHONHASHSEED=0

# check deployment mode.
if echo "$@" | grep -q "\-\-deploy-mode cluster";
then
    echo "cluster mode..";
    # temp log file for spark job.
    export TMP_LOG="/tmp/spark-job-log-$(date '+%Y-%m-%d-%H-%M-%S').log";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@" |& tee ${TMP_LOG};
    # when exit code 1 is contained in spark log, then return exit 1.
    if cat ${TMP_LOG} | grep -q "exit code: 1";
    then
        echo "exit code: 1";
        rm -rf ${TMP_LOG};
        exit 1;
    else
        echo "job succeeded.";
        rm -rf ${TMP_LOG};
        exit 0;
    fi
elif echo "$@" | grep -q "\-\-conf spark.submit.deployMode=cluster";
then
    echo "cluster mode..";
    # temp log file for spark job.
    export TMP_LOG="/tmp/spark-job-log-$(date '+%Y-%m-%d-%H-%M-%S').log";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@" |& tee ${TMP_LOG};
    # when exit code 1 is contained in spark log, then return exit 1.
    if cat ${TMP_LOG} | grep -q "exit code: 1";
    then
        echo "exit code: 1";
        rm -rf ${TMP_LOG};
        exit 1;
    else
        echo "job succeeded.";
        rm -rf ${TMP_LOG};
        exit 0;
    fi
else
    echo "client mode..";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
fi
```

看看下面的部分被替换成了`exec “${SPARK_HOME}”/bin/spark-class org.apache.spark.deploy.SparkSubmit “$@”`的 spark-submit 文件的行尾。

```
# check deployment mode.
if echo "$@" | grep -q "\-\-deploy-mode cluster";
then
    echo "cluster mode..";
    # temp log file for spark job.
    export TMP_LOG="/tmp/spark-job-log-$(date '+%Y-%m-%d-%H-%M-%S').log";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@" |& tee ${TMP_LOG};
    # when exit code 1 is contained in spark log, then return exit 1.
    if cat ${TMP_LOG} | grep -q "exit code: 1";
    then
        echo "exit code: 1";
        rm -rf ${TMP_LOG};
        exit 1;
    else
        echo "job succeeded.";
        rm -rf ${TMP_LOG};
        exit 0;
    fi
elif echo "$@" | grep -q "\-\-conf spark.submit.deployMode=cluster";
then
    echo "cluster mode..";
    # temp log file for spark job.
    export TMP_LOG="/tmp/spark-job-log-$(date '+%Y-%m-%d-%H-%M-%S').log";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@" |& tee ${TMP_LOG};
    # when exit code 1 is contained in spark log, then return exit 1.
    if cat ${TMP_LOG} | grep -q "exit code: 1";
    then
        echo "exit code: 1";
        rm -rf ${TMP_LOG};
        exit 1;
    else
        echo "job succeeded.";
        rm -rf ${TMP_LOG};
        exit 0;
    fi
else
    echo "client mode..";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
fi
```

如果 spark 作业以`cluster`模式提交，`spark-submit` shell 将检查从 spark 作业驱动程序箱返回的日志行是否包含`exit code: 1`。在`client`模式下提交 spark 作业时，会执行`exec “${SPARK_HOME}”/bin/spark-class org.apache.spark.deploy.SparkSubmit “$@”`原来的 spark 提交命令。

## 包装火花分布

你必须用选项构建 spark 源代码，例如，hive，spark thrift server，kubernetes 对 3.2.2 版本 hadoop 的支持。在 JDK 1.8 中运行以下命令来构建和打包 spark。

```
export MAVEN_OPTS="-Xss64m -Xmx2500m -XX:ReservedCodeCacheSize=1g"
./dev/make-distribution.sh --name custom-spark --pip --tgz -DskipTests=true -Dhadoop.version=3.2.2 -Phive -Phive-thriftserver -Pkubernetes;
```

## 构建并推送 Spark Docker 映像

经过上面的包装火花，`spark-3.2.2-bin-custom-spark.tgz`就会被创造出来。Untar 它，并建立这样的火花码头形象。把码头报告换成你的。

```
# spark version.
export SPARK_VERSION=3.2.2;

# create and push spark and spark-py docker image.
bin/docker-image-tool.sh -r docker.io/cloudcheflabs -t v$SPARK_VERSION -p ./kubernetes/dockerfiles/spark/bindings/python/Dockerfile build;
bin/docker-image-tool.sh -r cloudcheflabs -t v$SPARK_VERSION -p ./kubernetes/dockerfiles/spark/bindings/python/Dockerfile push;
```

# 上传火花 uberjar 到 S3 桶

要提交 spark 作业，你编译的 java 代码需要像这样打包成 uberjar。

```
mvn -e -DskipTests=true clean install shade:shade;
```

将打包的 spark uberjar 上传到 S3 bucket，以便在向 kubernetes 提交 spark 作业时使用类似于`s3a://your-bucket/spark-jobs/your-spark-uberjar.jar`的文件选项。例如，可以使用 MinIO CLI `mc`将 uberjar 上传到 S3 bucket。

```
mc cp your-spark-uberjar.jar your-s3/your-bucket/spark-jobs/;
```

现在，你可以像这样在 kubernetes 上以`cluster`模式提交 spark jobs。

```
export SPARK_VERSION=3.2.2
export SPARK_IMAGE=cloudcheflabs/spark:v${SPARK_VERSION};
export SPARK_MASTER=k8s://https://your-kubernetes-master
export NAMESPACE=spark;
export S3_BUCKET=your-bucket;
export S3_ACCESS_KEY=xxx;
export S3_SECRET_KEY=xxx;
export S3_ENDPOINT=https://s3-endpoint-xxx;
...

spark-submit \
--master $SPARK_MASTER \
--deploy-mode cluster \
--name your-spark-job-name \
--class YourClass \
--packages com.amazonaws:aws-java-sdk-s3:1.11.563,org.apache.hadoop:hadoop-aws:3.2.2 \
...
--conf spark.hadoop.fs.defaultFS=s3a://${S3_BUCKET} \
--conf spark.hadoop.fs.s3a.access.key=${S3_ACCESS_KEY} \
--conf spark.hadoop.fs.s3a.secret.key=${S3_SECRET_KEY} \
--conf spark.hadoop.fs.s3a.connection.ssl.enabled=true \
--conf spark.hadoop.fs.s3a.endpoint=$S3_ENDPOINT \
--conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
--conf spark.hadoop.fs.s3a.fast.upload=true \
--conf spark.hadoop.fs.s3a.path.style.access=true \
--conf spark.hadoop.fs.s3a.aws.credentials.provider=org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider \
--conf spark.driver.extraJavaOptions="-Divy.cache.dir=/tmp -Divy.home=/tmp" \
--conf spark.executor.instances=3 \
--conf spark.executor.memory=10G \
--conf spark.executor.cores=2 \
--conf spark.driver.memory=5G \
s3a://your-bucket/spark-jobs/your-spark-uberjar.jar;
```

看一下 spark job uberjar `s3a://your-bucket/spark-jobs/your-spark-uberjar.jar`的执行文件。

# 将 Kubernetes 上提交的 Spark 作业与工作流集成

Spark 作业可以从运行在 Kubernetes 上的类似`Airflow`的工作流任务中提交。对于在 kubernetes 上从 workflow 提交 spark 作业的更方便的方法，我们将使用在 kubernetes 上运行的 Livy。

## 使用 Livy

Spark 作业可以通过 Livy 提供的 REST API 提交。

来看看李维的`Dockerfile`。

```
FROM cloudcheflabs/spark:v3.2.2

# env.
ENV *LIVY_HOME* /opt/livy
ENV *LIVY_USER* livy
ENV *KUBECONFIG* ${*LIVY_HOME*}/.kube/config

RUN useradd -ms /bin/bash -d ${*LIVY_HOME*} ${*LIVY_USER*}

# install livy.
RUN set -eux; \
    apt install -y unzip curl; \
    mkdir -p ${*LIVY_HOME*}/.kube; \
    cd ${*LIVY_HOME*}; \
    curl -L -O https://dlcdn.apache.org/incubator/livy/0.7.1-incubating/apache-livy-0.7.1-incubating-bin.zip; \
    unzip apache-livy-0.7.1-incubating-bin.zip; \
    cp -rv apache-livy-0.7.1-incubating-bin/* .; \
    rm -rf apache-livy-0.7.1-incubating-bin/; \
    rm -rf apache-livy-0.7.1-incubating-bin.zip;

# add run shell.
ADD run-livy.sh ${*LIVY_HOME*}

# add  kubeconfig.
ADD config ${*LIVY_HOME*}/.kube

# add permissions.
RUN chown ${*LIVY_USER*}: -R ${*LIVY_HOME*}

# change work directory.
USER ${*LIVY_USER*}
RUN chmod +x ${*LIVY_HOME*}/*.sh
WORKDIR ${*LIVY_HOME*}
```

Livy `Dockerfile`扩展了前面部分创建的`cloudcheflabs/spark:v3.2.2`的映像，这意味着通过 REST API 的 spark 提交请求将在前面部分修改并构建为 spark docker 映像的`spark-submit`的情况下执行。

要在 kubernetes 上运行 Livy，首先要像这样构建 livy docker 映像。

```
cd ~/hard-to-submit-spark-jobs-on-kubernetes/components/livy/docker;

chmod a+x build-docker.sh;

export TAG=0.7.1;

./build-docker.sh \
--image=cloudcheflabs/livy:${TAG} \
;
```

用你的替换 docker repo。

像这样用舵轮图部署李维。

```
## install livy with helm.
cd ~/hard-to-submit-spark-jobs-on-kubernetes/components/livy/chart;

export TAG=0.7.1;

helm upgrade \
livy \
--install \
--create-namespace \
--namespace livy \
--set image=cloudcheflabs/livy:${TAG} \
--set livy.storageClass=nfs \
.;
```

`ReadWriteMany`livy PVC 需要使用支持的存储类，如`nfs`。

检查 livy pod 是否正在运行。

```
kubectl get po -n livy
NAME                   READY   STATUS    RESTARTS   AGE
livy-c6d54ff44-7lr45  1/1     Running   0          10h
```

并确保修改后的`spark-submit`将用于在 Livy pod 的 kubernetes 上提交 spark 作业。

```
kubectl exec -it livy-c6d54ff44-7lr45 -n livy -- cat /opt/spark/bin/spark-submit#!/usr/bin/env bash

#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

# disable randomized hash for string in Python 3.3+
export PYTHONHASHSEED=0

# check deployment mode.
if echo "$@" | grep -q "\-\-deploy-mode cluster";
then
    echo "cluster mode..";
    # temp log file for spark job.
    export TMP_LOG="/tmp/spark-job-log-$(date '+%Y-%m-%d-%H-%M-%S').log";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@" |& tee ${TMP_LOG};
    # when exit code 1 is contained in spark log, then return exit 1.
    if cat ${TMP_LOG} | grep -q "exit code: 1";
    then
        echo "exit code: 1";
        rm -rf ${TMP_LOG};
        exit 1;
    else
        echo "job succeeded.";
        rm -rf ${TMP_LOG};
        exit 0;
    fi
elif echo "$@" | grep -q "\-\-conf spark.submit.deployMode=cluster";
then
    echo "cluster mode..";
    # temp log file for spark job.
    export TMP_LOG="/tmp/spark-job-log-$(date '+%Y-%m-%d-%H-%M-%S').log";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@" |& tee ${TMP_LOG};
    # when exit code 1 is contained in spark log, then return exit 1.
    if cat ${TMP_LOG} | grep -q "exit code: 1";
    then
        echo "exit code: 1";
        rm -rf ${TMP_LOG};
        exit 1;
    else
        echo "job succeeded.";
        rm -rf ${TMP_LOG};
        exit 0;
    fi
else
    echo "client mode..";
    exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
fi
```

## 创建 DAG 来调用气流中的 Livy

我们需要创建一个 DAG 来调用 Livy REST API 提交 kubernetes 上的 spark 作业。`LivyOperator`用于在 kubernetes 上提交 spark 作业。

```
now = datetime.now()
date_time = now.strftime("%m/%d/%Y, %H:%M:%S")
livy_job_name = "your-spark-job-example-" + date_time
t2 = LivyOperator(
        task_id="spark-job-example",
        livy_conn_id='livy_default',
        polling_interval=5,
        file="s3a://your-bucket/spark-jobs/your-spark-uberjar.jar",
        class_name="YourClass",
        args=[
            "xxx",
            "xxx",
            "xxx",
            "xxx",
            "xxx"
        ],
        name=livy_job_name,
        driver_memory="5G",
        driver_cores=1,
        executor_memory="10G",
        executor_cores=2,
        num_executors=3,
        conf={
            "spark.submit.deployMode": "cluster",
            "spark.jars.packages": "com.amazonaws:aws-java-sdk-s3:1.11.563,org.apache.hadoop:hadoop-aws:3.2.2",
            ...
            "spark.hadoop.fs.defaultFS": "s3a://your-bucket",
            "spark.hadoop.fs.s3a.access.key": "xxx",
            "spark.hadoop.fs.s3a.secret.key": "xxx",
            "spark.hadoop.fs.s3a.connection.ssl.enabled": "true",
            "spark.hadoop.fs.s3a.endpoint": "https://xxx",
            "spark.hadoop.fs.s3a.impl": "org.apache.hadoop.fs.s3a.S3AFileSystem",
            "spark.hadoop.fs.s3a.fast.upload": "true",
            "spark.hadoop.fs.s3a.path.style.access": "true",
            "spark.hadoop.fs.s3a.aws.credentials.provider": "org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider",
            "spark.driver.extraJavaOptions": "-Divy.cache.dir=/tmp -Divy.home=/tmp"
        }
    )
```

您需要添加部署模式`“spark.submit.deployMode”: “cluster”`的配置，并添加`livy_default`与`livy-service.livy.svc`主机和`8998`端口的连接。

就是这样。