# 在 K8s 中创建一个 DB2 客户机容器作为 Prometheus 抓取目标

> 原文：<https://itnext.io/creating-a-db2-client-container-as-prometheus-scraping-target-in-k8s-b36e5cdbd394?source=collection_archive---------4----------------------->

![](img/ddd23014ca412f83062014049f4613da.png)

我需要从 DB2 数据库中挖掘一些业务指标，并将其呈现在 Grafana 仪表板中。一种普罗米修斯刮靶即将研制出来。

大约 3 年前，我发表了一篇中型论文,通过使用 DB2 ODBC/CLI 驱动程序在 Golang 中运行 DB2 查询。接下来，让我们创建一个容器映像作为 Prometheus 抓取目标，并在 Kubernetes 中运行它。

# 作为抓取目标的应用程序

一些 Golang 代码摘录描述了数据收集的工作原理，

数据库和度量结构

```
type MetricConfig struct {
 Name      string `yaml:"name"`
 Desc      string `yaml:"desc"`
 Sql       string `yaml:"sql"`
 Frequency string `yaml:"frequency"`
}type DBMetricsConfig struct {
 Dsn      string         `yaml:"dsn,omitempty"`
 User     string         `yaml:"user,omitempty"`
 Password string         `yaml:"password,omitempty"`
 Metrics  []MetricConfig `yaml:"metrics,omitempty"`
}
```

Dsn 是 DB2 的 ODBC 数据源名称。公制是一种普罗米修斯规格。度量值将从返回单行和单列的 SQL 查询中设置。我们使用 [jobrunner 库](https://github.com/bamzi/jobrunner)将数据收集作为 cron 作业运行。

初始化功能，

```
func ScheduleDBMetricJob(mc DBMetricsConfig) error {
 user := os.Getenv(mc.User)
 password := os.Getenv(mc.Password)
 db, err := sqlx.Open("odbc", fmt.Sprintf("DSN=%s;uid=%s;pwd=%s", mc.Dsn, user, password))
 if err != nil {
  logrus.Errorf("Error opening database: %v", err)
  return err
 } db.SetMaxIdleConns(5)
 db.SetMaxOpenConns(5) for _, met := range mc.Metrics {
  gauge := prometheus.NewGauge(prometheus.GaugeOpts{
   Namespace: "db2",
   Subsystem: "mon",
   Name:      met.Name,
   Help:      met.Desc,
  })
  prometheus.MustRegister(gauge)
  jobrunner.Schedule(fmt.Sprintf("[@every](http://twitter.com/every) %s", met.Frequency), DBMetricJob{
   db:    db,
   sql:   met.Sql,
   gauge: gauge,
  })
 } return nil
}
```

由于有了`sqlx`助手函数，下面显示的实际数据收集函数非常简单。

```
type DBMetricJob struct {
 db    *sqlx.DB
 sql   string
 gauge prometheus.Gauge
}func (job DBMetricJob) Run() {
 var i int err := job.db.Get(&i, job.sql)
 if err != nil {
  logrus.Errorf("failed to run sql:%v", err)
  return
 } job.gauge.Set(float64(i))
}
```

# 构建容器映像

从 IBM 网站[获取 DB2 ODBC/CLI 驱动程序](https://www.ibm.com/support/pages/node/323035)。说我们有的是`v11.5.5fp1_linuxx64_odbc_cli.tar.gz`的 tar 文件。

构建图像的 Dockerfile 文件如下所示，

```
FROM golang as builder
RUN apt update && apt install -y unixodbc-dev
WORKDIR /build
COPY ./src /build
RUN go build -o serving *.goFROM ubuntu
WORKDIR /app
COPY --from=builder /build/serving /app
ENTRYPOINT ["./serving"]ADD v11.5.5fp1_linuxx64_odbc_cli.tar.gz ./
RUN apt update && \
    apt install -y curl unixodbc libxml2-dev libpam0g-dev && \
    rm -rf /var/lib/apt/lists/*ENV DB2_CLI_DRIVER_INSTALL_PATH /app/odbc_cli/clidriver
ENV LD_LIBRARY_PATH /app/odbc_cli/clidriver/lib
ENV LIBPATH /app/odbc_cli/clidriver/lib
ENV DB2BINPATH /app/odbc_cli/clidriver/bin:/app/odbc_cli/clidriver/adm
```

这是一个多阶段的容器构建。golang builder 需要`unixodbc-dev`库进行构建。由于 ODBC 库的依赖性，应用程序必须在不禁用 CGO 的情况下构建。因此不能使用 Alpine base 图像，因为它不提供 glibc。

而是用 Ubuntu 作为 app 镜像库。添加编译好的 app 和下载的 ODBC 驱动。在 Dockerfile 中添加时，驱动程序 tar 文件会自动展开。安装 DB2 驱动程序的运行时依赖项，包括`unixodbc, ibxml2-dev, libpam0g-dev`

如 docker 文件所示，最后一部分是 DB2 驱动程序的环境变量。如果需要，可以在容器中使用 DB2BINPATH 来设置路径。这些环境变量是静态的。

还有另外两个环境变量。`ODBCINI`告诉 ODBC 驱动程序用户 odbc.ini 文件在哪里。`DB2CLIINIPATH`告诉 db2 CLI 驱动程序 cli ini 文件的目录在哪里。当我们将应用程序部署到 K8s 中时，我们将动态地使用这两个环境变量。

用 Buildah 构建并推送图像，将图像标签设置为 db2mon:v1.0

# 部署到 K8s 中

在我们部署 K8s 部署之前，创建以下 odbc.ini 文件。

```
[mydb2]
Protocol=TCPIP
Hostname=192.168.10.102
ServiceName=50001
Database=dbname
CurrentSchema=dbschema
Driver=/app/odbc_cli/clidriver/lib/libdb2o.so
```

这个文件将用于 odbc.ini 和 db2cli.ini。在名称空间中创建它们相应的配置映射，

```
kubectl create cm cm-db2odbc --from-file=odbc.ini=odbc.ini
kubectl create cm cm-db2cli --from-file=db2cli.ini=obdc.ini
```

配置图内容相同，但密钥不同。

如下所示创建 K8s 部署，

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db2mon
  labels:
    app: db2mon
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db2mon
  template:
    metadata:
      labels:
        app: db2mon
    spec:
      containers:
      - name: db2mon
        image: db2mon:v1.0
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /app/config.yaml
          subPath: config.yaml
        - name: odbcini
          mountPath: /app/odbc.ini
          subPath: odbc.ini
        - name: db2cliini
          mountPath: /app/db2cli.ini
          subPath: db2cli.ini
        env:
        - **name: ODBCINI**
          value: /app/odbc.ini
        - **name: DB2CLIINIPATH**
          value: /app
        - name: DB_USER
          value: db2inst1
        - name: DB_PASSWORD
          value: password
      volumes:
      - name: config
        configMap:
          name: cm-db2monconfig
      - name: odbcini
        configMap:
          name: cm-db2odbc
      - name: db2cliini
        configMap:
          name: cm-db2cli
```

在这里，我们使用 mountPath 和子路径将 configMap 挂载到/app 目录，并使用其特定的文件名，这样就不会覆盖/app 的原始结构。

定义分别指向`/app/odbc.ini`和`/app`的环境变量`ODBCINI`和`DB2CLIINIPATH`。

通过这些设置，ODBC DSN 被正确设置，golang 可以将 DB2 数据作为 Prometheus gauge 数据类型进行查询。

关于如何公开服务并为 Prometheus 操作者设置服务监视器以收集数据的其余内容在这里跳过。