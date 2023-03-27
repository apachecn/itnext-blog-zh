# 用 Node.js、Express、MongoDB 和 TypeScript 构建 RESTful Web APIs 第 3 部分

> 原文：<https://itnext.io/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-3-d545b243541e?source=collection_archive---------0----------------------->

![](img/6f6e6758220da47de8657a6864fa0ea9.png)

(图片来自 OctoPerf)

有一个关于[如何在 Lynda](https://www.lynda.com/Node-js-tutorials/Next-steps/633869/671263-4.html) 上构建 Web APIs 的课程，但是他们没有使用 TypeScript。所以我决定用 TypeScript 做一个。这个项目中有许多需要改进的地方。如果你找到了，请留下评论。我很感激。)

[**第一部分:设置项目**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-1-2-195bdaf129cf)

[**第二部分:实现路由和 CRUD**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-2-98c34e3513a2)

[**第 3 部分:为 Web APIs 使用控制器和模型**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-3-d545b243541e)

[**第 4 部分:将 Web APIs 连接到 MongoDB 或其他**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-4-954c8c059cd4)

[**第 5 部分:我们的 Web APIs 的安全性**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-5-a80e5a7f03db)

[**奖励:用云函数、Firestore 和 Express**](/building-a-serverless-restful-api-with-cloud-functions-firestore-and-express-f917a305d4e6) 构建“无服务器”RESTful API

[**奖励:在 Nodejs**](/handling-long-running-api-requests-in-nodejs-403bd566d47) 中处理长时间运行的 API 请求

在这一部分中，我将向您展示如何使用控制器和模型来创建、保存、编辑和删除数据。记得在你前进之前阅读前面的部分。

**第一步:为您的数据创建模型**

所有模型文件将保存在 **/lib/models** 文件夹中。我们将通过使用来自 mongose 的[模式来定义联系人的结构。](http://mongoosejs.com/docs/guide.html)

```
//   /[lib](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)/[models](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/models)/crmModel.tsimport * as mongoose from 'mongoose';

const Schema = mongoose.Schema;

export const ContactSchema = new Schema({
    firstName: {
        type: String,
        required: 'Enter a first name'
    },
    lastName: {
        type: String,
        required: 'Enter a last name'
    },
    email: {
        type: String            
    },
    company: {
        type: String            
    },
    phone: {
        type: Number            
    },
    created_date: {
        type: Date,
        default: Date.now
    }
});
```

这个模型将在我们将创建数据的控制器内部使用。

**第二步:创建你的第一个控制器**

记得在[第 2 部分](/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-2-98c34e3513a2)中，我们创建了 CRUD 占位符用于与服务器通信。现在我们将把真正的逻辑应用到路由和控制器上。

1.  **创建新联系人(发布请求)**

所有的逻辑都会保存在**/**[**lib**](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)**/**[**控制器**](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/controllers)**/CRM controller . ts**中

```
//   /[lib](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)/[controllers](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/controllers)/crmController.tsimport * as mongoose from 'mongoose';
import { ContactSchema } from '../models/crmModel';
import { Request, Response } from 'express';

const Contact = mongoose.model('Contact', ContactSchema);export class ContactController{
...public addNewContact (req: Request, res: Response) {                
        let newContact = new Contact(req.body);

        newContact.save((err, contact) => {
            if(err){
                res.send(err);
            }    
            res.json(contact);
        });
    }
```

在路线中，我们不必通过任何东西。

```
 // /lib/routes/crmRoutes.tsimport { ContactController } from "../controllers/crmController";public contactController: ContactController = new ContactController();// Create a new contact
app.route('/contact')
   .post(this.contactController.addNewContact);
```

**2。获取所有联系人(获取请求)**

所有的逻辑都会保存在**/**[**lib**](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)**/**[**控制器**](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/controllers)**/CRM controller . ts**中

```
//   /[lib](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)/[controllers](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/controllers)/crmController.ts...public getContacts (req: Request, res: Response) {           
        Contact.find({}, (err, contact) => {
            if(err){
                res.send(err);
            }
            res.json(contact);
        });
    }
}
```

之后，我们将导入 **ContactController** 并应用 **getContacts** 方法。

```
// /lib/routes/crmRoutes.ts// Get all contacts
app.route('/contact')
   .get(this.contactController.getContacts)
```

**3。查看单个联系人(获取方法)**

我们需要联系人的 ID 才能查看联系信息。

```
//   /[lib](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)/[controllers](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/controllers)/crmController.tspublic getContactWithID (req: Request, res: Response) {           
        Contact.findById(req.params.contactId, (err, contact) => {
            if(err){
                res.send(err);
            }
            res.json(contact);
        });
    }
```

在路由中，我们简单地通过 **'/contact/:contactId'**

```
// /lib/routes/crmRoutes.ts// get a specific contact
app.route('/contact/:contactId')
   .get(this.contactController.getContactWithID)
```

**4。更新单个联系人(PUT 方法)**

请记住，如果没有{new: true}，更新的文档将不会被返回。

```
//   /[lib](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)/[controllers](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/controllers)/crmController.tspublic updateContact (req: Request, res: Response) {           
        Contact.findOneAndUpdate({ _id: req.params.contactId }, req.body, { new: true }, (err, contact) => {
            if(err){
                res.send(err);
            }
            res.json(contact);
        });
    }
```

在路线中，

```
// /lib/routes/crmRoutes.ts// update a specific contact
app.route('/contact/:contactId')
   .put(this.contactController.updateContact)
```

**5。删除单个联系人(删除方法)**

```
//   /[lib](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib)/[controllers](https://github.com/dalenguyen/rest-api-node-typescript/tree/master/lib/controllers)/crmController.tspublic deleteContact (req: Request, res: Response) {           
        Contact.remove({ _id: req.params.contactId }, (err, contact) => {
            if(err){
                res.send(err);
            }
            res.json({ message: 'Successfully deleted contact!'});
        });
    }
```

在路线中，

```
// /lib/routes/crmRoutes.ts// delete a specific contact
app.route('/contact/:contactId')
   .delete(this.contactController.deleteContact)
```

*请记住，您不必每次都调用 app.route('/contact/:contactId ')来获取、放置或删除单个联系人。你可以把它们结合起来

```
// /lib/routes/crmRoutes.tsapp.route('/contact/:contactId')
   // edit specific contact
   .get(this.contactController.getContactWithID)
   .put(this.contactController.updateContact)
   .delete(this.contactController.deleteContact)
```

现在，你的模型和控制器已经准备好了。我们将挂钩到 MongoDB 并测试 Web APIs。这是 [**的结尾第三部**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-3-d545b243541e) 。我将很快更新 [**第四部**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-4-954c8c059cd4) **和** [**第五部**](https://medium.com/@dalenguyen/building-restful-web-apis-with-node-js-express-mongodb-and-typescript-part-5-a80e5a7f03db) 。以防你需要跳过一个头。请访问我的 [github 库](https://github.com/dalenguyen/rest-api-node-typescript)获取完整代码。

[](https://github.com/dalenguyen/rest-api-node-typescript) [## GitHub-dalen guyen/rest-API-node-typescript:用 Node.js 构建 RESTful Web APIs，Express…

### 这是一个简单的 API，保存人们的联系信息。这个项目有两个版本。1.0.0 版:您可以…

github.com](https://github.com/dalenguyen/rest-api-node-typescript) 

[**在 Twitter 上关注我**](https://twitter.com/dale_nguyen) 了解 Angular、JavaScript & WebDevelopment 的最新内容👐