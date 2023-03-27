# 在 Go 中构建一个基本服务容器

> 原文：<https://itnext.io/build-a-rudimentary-dependency-container-in-go-d0c9800ccb24?source=collection_archive---------2----------------------->

![](img/5ddeaff79ff8a6c05766b243b4eee3a6.png)

我特别是在 PHP 和 Symfony/Laravel 上工作了很长时间，我非常喜欢这种框架:应用程序容器——一个包含正确运行应用程序所需的所有依赖项的对象。

Go 中的大多数 RESTful APIs 示例使用普通闭包来处理请求，但是我发现在构建比简单端点更复杂的东西时，这有一点限制。

对于最近的一个任务，我想出了一个基本的依赖容器，我相信它可以很好地满足这个目的。

# 启动服务器

首先，让我们启动一个 HTTP 服务器来处理请求:

```
package cmdimport (
 "github.com/lucavallin/goat-webshop/pkg/api"
 "github.com/lucavallin/goat-webshop/pkg/herd"
 "github.com/spf13/cobra"
 "os"
 "path/filepath"
)var serveCmd = &cobra.Command{
 Use:   "serve",
 Short: "Start API server for the Goat Webshop",
 Long: "Start API server for the Goat Webshop",
 Run: func(cmd *cobra.Command, args []string) {
  Serve()
 },
}func init() {
 rootCmd.AddCommand(serveCmd)
}// Serve stars the APIs for the goat-webshop
func Serve() {
 herdPath, _ := filepath.Abs("data/herd.xml")
 herdRepo := herd.NewXMLFileRepository(herdPath)
 app := api.NewApp(herdRepo) app.Run(os.Getenv("PORT"))
}
```

在这个代码片段中，我使用 [Cobra](https://github.com/spf13/cobra) 构建一个基于命令的应用程序，而 **Serve()** 函数是它的入口点。调用 **api 时。NewApp()** 创建了一个新的应用程序容器——它保存了我在执行过程中需要的依赖项(在本例中是一个存储库，基于 XML)。

**app。Run()** 有效地启动应用程序，使用**操作系统从环境中读取所需的端口。Getenv("PORT")** —我在 Heroku 上托管这些 API，这就是它如何在那里工作的**。**

应用程序的主入口可能会因为更多的依赖项而看起来很混乱，但根据这条推文，这似乎没什么问题:

# 集装箱

当服务器运行时，让我们看看在 **api 内部发生了什么。NewApp()** :

```
package apiimport (
 "github.com/gin-gonic/gin"
 "github.com/lucavallin/goat-webshop/pkg/herd"
 "log"
)// App contains all the dependencies needed for the API
type App struct {
 herdRepo herd.Repository
 router   *gin.Engine
}// NewApp creates a new app with the needed dependencies
func NewApp(herdRepo herd.Repository) *App {
 router := gin.Default() app := &App{herdRepo, router}
 app.router.GET("/herd/:days", app.GetHerdHandler()) return app
}// Run starts the APIs
func (app *App) Run(port string) {
 log.Fatal(app.router.Run(":" + port))
}
```

**NewApp()** 接收我们需要的存储库，创建并返回一个 **App** 对象，其中包含依赖项和路由器。我用**杜松子酒**做这个，因为我发现它使用起来既简单又干净——它也是最常见的。

有趣的是端点的处理程序:它是一个必须在 **app** 对象上调用的方法，这样可以确保在处理请求时，我们可以访问我们需要的依赖项。

# 处理程序示例

```
package apiimport (
 "github.com/gin-gonic/gin"
 "github.com/lucavallin/goat-webshop/pkg/herd"
 "strconv"
)...// GetHerdHandler handlers GET to /goat-webshop/herd/:days
func (app *App) GetHerdHandler() func(c *gin.Context) {
 return func(c *gin.Context) {
  elapsedDays, _ := strconv.Atoi(c.Param("days"))
  newHerd := app.herdRepo.Get()
  newHerd.Age(elapsedDays) c.JSON(http.StatusOK, newHerd)
 }
}
```

忽略业务逻辑，因为它与目的无关，专注于这一行:

```
newHerd := app.herdRepo.Get()
```

由于依赖关系包含在 **app** 对象中，我们现在可以访问它们，但避免了紧密耦合的代码:存储库(或任何其他您可能需要的服务)驻留在依赖关系容器中，从不直接公开——这使得在需要时很容易被替换(例如，它目前是基于 XML 的，但如果您足够清醒，您可能希望使用真正的数据库)。

同样，端点处理程序只能在 app 对象(实际上是一个指向它的指针)上调用，以确保我们需要的依赖项总是可用的。

***想了解更多围棋和创新知识？关注@lucavallin 上的*** [***推特***](https://twitter.com/lucavallin) ***和***[***Github***](https://github.com/lucavallin)***或者来 Xebia 工作室***[****和我们见面吧！****](https://www.xebia.studio/)