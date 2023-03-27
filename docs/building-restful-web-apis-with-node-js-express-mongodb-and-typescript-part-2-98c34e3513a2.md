# 用 Node.js、Express、MongoDB 和 TypeScript 构建 RESTful Web APIs 第 2 部分

> 原文：<https://itnext.io/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-2-98c34e3513a2?source=collection_archive---------0----------------------->

![](img/7f972589990eaeaee66d122279b1f756.png)

(图片来自 OctoPerf)

有一个关于[如何在 Lynda](https://www.lynda.com/Node-js-tutorials/Next-steps/633869/671263-4.html) 上构建 Web APIs 的课程，但是他们没有使用 TypeScript。所以我决定用 TypeScript 做一个。这个项目中有许多需要改进的地方。如果你找到了，请留下评论。我很感激。)

[**第一部分:设置项目**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-1-2-195bdaf129cf)

[**第二部分:实现路由和 CRUD**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-2-98c34e3513a2)

[**第 3 部分:为 Web APIs 使用控制器和模型**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-3-d545b243541e)

[**第 4 部分:将 Web APIs 连接到 MongoDB 或其他**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-4-954c8c059cd4)

[**第 5 部分:我们的 Web APIs 的安全性**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-5-a80e5a7f03db)

[**奖励:用云函数、Firestore 和 Express**](/building-a-serverless-restful-api-with-cloud-functions-firestore-and-express-f917a305d4e6) 构建“无服务器”RESTful API

[**奖励:在 Nodejs**](/handling-long-running-api-requests-in-nodejs-403bd566d47) 中处理长时间运行的 API 请求

在第 2 部分中，我将为 API 构建路由。

**步骤 1:创建用于路由的 TS 文件**

记得在这个项目的第一部分。我们将所有内容保存在 **lib** 文件夹中。因此，我将使用名为 **crmRoutes.ts** 的文件创建 **routes** 文件夹，该文件将保存这个项目的所有路线。

```
// /lib/routes/crmRoutes.tsimport {Request, Response} from "express";

export class Routes {       
    public routes(app): void {          
        app.route('/')
        .get((req: Request, res: Response) => {            
            res.status(200).send({
                message: 'GET request successfulll!!!!'
            })
        })               
    }
}
```

创建完第一条路径后，我们需要将其导入到 **lib/app.ts** 中。

```
// /lib/app.tsimport * as express from "express";
import * as bodyParser from "body-parser";
**import { Routes } from "./routes/crmRoutes";**

class App {

    public app: express.Application;
    **public routePrv: Routes = new Routes();**

    constructor() {
        this.app = express();
        this.config();        
        **this.routePrv.routes(this.app);**     
    }

    private config(): void{
        this.app.use(bodyParser.json());
        this.app.use(bodyParser.urlencoded({ extended: false }));
    }
}
```

现在，您可以直接或通过使用 [Postman](https://www.getpostman.com/apps) 向您的应用程序(http://localhost:3000)发送 GET 请求。

**第二步:为 Web APIs 构建 CRUD**

我假设您对 HTTP 请求(GET、POST、PUT 和 DELETE)有基本的了解。如果没有，那很简单:

*   获取:用于检索数据
*   发布:用于创建新数据
*   PUT:用于更新数据
*   删除:用于删除数据

现在，我们将为构建联系人 CRM 构建路由，以便保存、检索、更新和删除联系人信息。

```
// /lib/routes/crmRoutes.tsimport {Request, Response} from "express";

export class Routes {    

    public routes(app): void {   

        app.route('/')
        .get((req: Request, res: Response) => {            
            res.status(200).send({
                message: 'GET request successfulll!!!!'
            })
        })

        // Contact 
        app.route('/contact') 
        // GET endpoint 
        .get((req: Request, res: Response) => {
 **// Get all contacts**            
            res.status(200).send({
                message: 'GET request successfulll!!!!'
            })
        })        
        // POST endpoint
        .post((req: Request, res: Response) => {   
 **// Create new contact**         
            res.status(200).send({
                message: 'POST request successfulll!!!!'
            })
        })

        // Contact detail
        app.route('/contact/:contactId')
        // get specific contact
        .get((req: Request, res: Response) => {
 **// Get a single contact detail**            
            res.status(200).send({
                message: 'GET request successfulll!!!!'
            })
        })
        .put((req: Request, res: Response) => {
 **// Update a contact**           
            res.status(200).send({
                message: 'PUT request successfulll!!!!'
            })
        })
        .delete((req: Request, res: Response) => {       
 **// Delete a contact**     
            res.status(200).send({
                message: 'DELETE request successfulll!!!!'
            })
        })
    }
}
```

现在，路由已经准备好接收 HTTP 请求了。这是[的结尾**第二部**的结尾](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-2-98c34e3513a2)。我将在短期内更新 [**部分 3**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-3-d545b243541e)[**部分** **部分 4**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-4-954c8c059cd4) **和** [**部分 5**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-5-a80e5a7f03db) 。以防你需要跳过一个头。请访问我的 [github 库](https://github.com/dalenguyen/rest-api-node-typescript)获取完整代码。

[](https://github.com/dalenguyen/rest-api-node-typescript) [## GitHub-dalen guyen/rest-API-node-typescript:用 Node.js 构建 RESTful Web APIs，Express…

### 这是一个简单的 API，保存人们的联系信息。这个项目有两个版本。1.0.0 版:您可以…

github.com](https://github.com/dalenguyen/rest-api-node-typescript) 

[**在 Twitter 上关注我**](https://twitter.com/dale_nguyen) 了解 Angular、JavaScript & WebDevelopment 的最新内容👐