# 使用 Go 进行集成测试的 MySQL Docker 容器

> 原文：<https://itnext.io/mysql-docker-container-for-integration-testing-using-go-f784b70a03b?source=collection_archive---------1----------------------->

![](img/04d6516c3cc5c2079183f7e1782742cf.png)

在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上由 [Erwan Hesry](https://unsplash.com/@erwanhesry?utm_source=medium&utm_medium=referral) 拍摄的照片

在生产中，bug 是最昂贵的。在开发过程中使用测试用例来捕捉它们是我们能做的降低成本的最好的事情之一。测试在所有软件中都非常重要。这有助于确保我们代码的正确性，并有助于减少回归。单元测试有助于在没有任何外部依赖性的情况下独立测试组件。单元测试不足以确保我们有一个稳定的、经过良好测试的系统。事实上，失败发生在不同组件的集成过程中。当我们从未在真实数据库上运行测试时，具有数据库后端的应用程序会面临这个问题，我们可能永远不会注意到由于事务未提交、数据库版本错误等问题而导致的事情无法工作。集成测试在端到端测试中扮演着重要的角色。

在当今世界，我们用数据库作为存储后端编写了许多软件应用程序。模仿这些数据库调用进行单元测试可能很麻烦。在模式中做一些小的改变会导致重写一些或者所有的模拟。由于查询不进入实际的数据库引擎，所以没有查询语法或约束的验证。需要模拟每个查询会导致重复工作。为了避免这种情况，我们应该使用一个真实的数据库进行测试，这个数据库在测试完成后会被销毁。 [**Docker**](https://www.docker.com/) 非常适合运行测试用例，因为我们可以在几秒钟内旋转容器，并在完成后杀死它们。

让我们了解如何启动 MySQL docker 容器，并使用 go 代码对其进行测试。我们首先需要确保运行我们的测试用例的系统安装了 docker，这可以通过运行命令" **docker ps** "来检查。如果没有安装 docker，从这里的[安装 docker](https://docs.docker.com/install/)。

```
func (d *Docker) isInstalled() bool {
  command := exec.Command("docker", "ps")
  err := command.Run()
  if err != nil {
    return false
  }
  return true
}
```

一旦 docker 安装完毕，我们需要用一个用户名和密码来运行 MySQL 容器，这个用户名和密码可以用来连接 MySQL 服务器。

```
docker run --name our-mysql-container -e MYSQL_ROOT_PASSWORD=root -e MYSQL_USER=gouser -e MYSQL_PASSWORD=gopassword -e MYSQL_DATABASE=godb -p 3306:3306 --tmpfs /var/lib/mysql mysql:5.7
```

这运行了一个 MySQL 5.7 版本的 docker 映像，容器名为“our-mysql-container”。“-e”指定了我们需要为 MySQL docker 容器设置的运行时变量。我们将 root 设置为我们的 root 密码。创建一个密码为“gopassword”的用户“gouser ”,我们用它从我们的应用程序连接到 MySQL 服务器。我们公开了 docker 容器的 3306 端口，这样我们就可以连接到 Docker 容器内部运行的 mysql 服务器。我们使用的是 [tmpfs 挂载](https://docs.docker.com/v17.09/engine/admin/volumes/tmpfs/)，它只在主机的内存中存储数据。当容器停止时，tmpfs 装载将被删除。因为我们正在运行它的测试目的，所以没有必要在永久存储数据。

```
type ContainerOption struct {
  Name              string
  ContainerFileName string
  Options           map[string]string
  MountVolumePath   string
  PortExpose        string
}func (d *Docker) getDockerRunOptions(c ContainerOption) []string {
  portExpose := fmt.Sprintf("%s:%s", c.PortExpose, c.PortExpose)
  var args []string
  for key, value := range c.Options {
    args = append(args, []string{"-e", fmt.Sprintf("%s=%s", key, value)}...)
  }
  args = append(args, []string{"--tmpfs", c.MountVolumePath, c.ContainerFileName}...)
  dockerArgs := append([]string{"run", "-d", "--name", c.Name, "-p", portExpose}, args...)
  return dockerArgs
}func (d *Docker) Start(c ContainerOption) (string, error) {
  dockerArgs := d.getDockerRunOptions(c)
  command := exec.Command("docker", dockerArgs...)
  command.Stderr = os.Stderr
  result, err := command.Output()
  if err != nil {
    return "", err
  }
  d.ContainerID = strings.TrimSpace(string(result))
  d.ContainerName = c.Name
  command = exec.Command("docker", "inspect", d.ContainerID)
  result, err = command.Output()
  if err != nil {
    d.Stop()
    return "", err
  }
  return string(result), nil
}func (m *MysqlDocker) StartMysqlDocker() {
  mysqlOptions := map[string]string{
    "MYSQL_ROOT_PASSWORD": "root",
    "MYSQL_USER":          "gouser",
    "MYSQL_PASSWORD":      "gopassword",
    "MYSQL_DATABASE":      "godb",
  }
  containerOption := ContainerOption{
    Name:              "our-mysql-container",
    Options:           mysqlOptions,
    MountVolumePath:   "/var/lib/mysql",
    PortExpose:        "3306",
    ContainerFileName: "mysql:5.7",
  }
  m.Docker = Docker{}
  m.Docker.Start(containerOption)
}
```

我们可以使用 containerId 来检查容器的细节。

```
docker inspect containerId
```

一旦我们运行了 docker 容器，我们需要等到我们的 Docker 容器启动并运行。我们可以使用下面的命令检查这一点。

```
docker ps -a
```

一旦 docker 启动并运行，我们就可以开始在我们的应用程序中使用它来运行与真实数据库的集成测试用例。

```
func (d *Docker) WaitForStartOrKill(timeout int) error {
  for tick := 0; tick < timeout; tick++ {
    containerStatus := d.getContainerStatus()
    if containerStatus == dockerStatusRunning {
     return nil
    }
    if containerStatus == dockerStatusExited {
     return nil
    }
    time.Sleep(time.Second)
  }
  d.Stop()
  return errors.New("Docker faile to start in given time period so stopped")
}func (d *Docker) getContainerStatus() string {
  command := exec.Command("docker", "ps", "-a", "--format", "{{.ID}}|{{.Status}}|{{.Ports}}|{{.Names}}")
  output, err := command.CombinedOutput()
  if err != nil {
    d.Stop()
    return dockerStatusExited
  }
  outputString := string(output)
  outputString = strings.TrimSpace(outputString)
  dockerPsResponse := strings.Split(outputString, "\n")
  for _, response := range dockerPsResponse {
    containerStatusData := strings.Split(response, "|")
    containerStatus := containerStatusData[1]
    containerName := containerStatusData[3]
    if containerName == d.ContainerName {
      if strings.HasPrefix(containerStatus, "Up ") {
        return dockerStatusRunning
      }
    }
  }
  return dockerStatusStarting
}
```

我们可以使用下面的连接字符串从 go 代码连接到 docker 中运行的 MySQL 服务器。

```
gouser:gopassword@tcp(localhost:3306)/godb?charset=utf8&parseTime=True&loc=Local
```

所有这些都有助于使用真实的数据库运行集成测试，该数据库可以在每次运行时重新创建。这有助于确保我们的应用程序为产品发布做好准备。

完整的代码可以在这个 git 资源库中找到:【https://github.com/MiteshSharma/DockerMysqlGo

***PS:如果你喜欢这篇文章，请鼓掌支持*** 👏 ***。欢呼***