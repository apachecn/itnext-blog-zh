# AWS 放大器用于角度应用的第一步

> 原文：<https://itnext.io/first-steps-with-aws-amplify-for-an-angular-app-31c271ade5a6?source=collection_archive---------3----------------------->

![](img/7cd932bf99e9b0251f5a32f735a30b65.png)

在本文中，我将描述如何用 AWS Amplify 构建一个 Angular 应用程序，这是一个开发 web 和移动应用程序的解决方案。

我们将致力于一个小应用程序，可以帮助一个自由职业者编辑其客户的账单。该应用程序将基本上定义三个对象:客户，账单和行(自由职业者将能够在同一个账单中支付几个东西)。

你可以在 https://github.com/feloy/aws-amplify-demo 的[找到项目的来源。](https://github.com/feloy/aws-amplify-demo)

本页描述了 CLI 的安装和第一个项目的创建(我们将基于 Angular 前端创建一个项目):[https://docs . amplify . AWS/start/getting-started/installation/q/integration/Angular](https://docs.amplify.aws/start/getting-started/installation/q/integration/angular)

实际上，您必须首先安装 CLI，然后运行`amplify configure`来选择一个 AWS 区域，创建一个新的 AWS 用户，并将相关的凭证保存到您的本地配置中:

```
$ npm install -g @aws-amplify/cli
$ amplify configure
```

# 应用程序初始化

下一步是从头开始创建一个 Angular 应用程序，并初始化项目。全部在本页描述:[https://docs . amplify . AWS/start/getting-started/setup/q/integration/angular](https://docs.amplify.aws/start/getting-started/setup/q/integration/angular)

# API 创建

是时候创建一个 API 了，它将包含用于**客户**、**账单**和**行**的三个表。我们将使用 GraphQL 服务来访问 API，并将集成的 *Amazon Cognito 用户池*作为授权服务。

```
$ amplify add api
? Please select from one of the below mentioned services: **GraphQL**
? Provide API name: **billing**
? Choose the default authorization type for the API Amazon Cognito User Pool
Using service: **Cognito**, provided by: awscloudformation

 The current configured provider is Amazon Cognito. 

 Do you want to use the default authentication and security configuration? **Default configuration**
 Warning: you will not be able to edit these selections. 
 How do you want users to be able to sign in? **Username**
 Do you want to configure advanced settings? **No, I am done.**
Successfully added auth resource
? Do you want to configure advanced settings for the GraphQL API **No, I am done.**
? Do you have an annotated GraphQL schema? **No**
? Do you want a guided schema creation? Yes
? What best describes your project: **One-to-many relationship (e.g., “Blogs” with “Posts” and “Comments”)**
? Do you want to edit the schema now? **Yes** [...]
```

受给定示例和 https://docs.amplify.aws/cli/graphql-transformer/o[verview 文档的启发，我创建了以下模型，其中客户可以包含许多账单，账单可以包含许多行、账单和仅分别属于一个客户和账单的行:](https://docs.amplify.aws/cli/graphql-transformer/overview)

```
type Customer
@model
@auth(rules: [{allow: owner}])
{
  id: ID!
  name: String!
  bills: [Bill] @connection(keyName: "byCustomer", fields: ["id"])
}type Bill
@model
@auth(rules: [{allow: owner}])
@key(name: "byCustomer", fields: ["customerID"])
{
  id: ID!
  title: String!
  customerID: ID!
  customer: Customer @connection(fields: ["customerID"])
  lines: [Line] @connection(keyName: "byBill", fields: ["id"])
}type Line
@model
@auth(rules: [{allow: owner}])
@key(name: "byBill", fields: ["billID", "title"])
{
  id: ID!
  billID: ID!
  bill: Bill @connection(fields: ["billID"])
  title: String!
  quantity: Int!
  cost: Float!
}
```

该模型目前包含最少的字段。稍后，您可以通过向表中添加更多字段来完成它。

> 请注意`@auth(rules: [{allow: owner}])`，它将限制用户对他所创建的数据的访问。

在使用以下选项运行`amplify push`之后，您可以在 AWS 控制台中看到 API 已经部署，并且创建了一个`API.service.ts`文件来帮助您从 Angular 应用程序访问 API。

```
? Do you want to generate code for your newly created GraphQL API **Yes**
? Choose the code generation language target **angular**
? Enter the file name pattern of graphql queries, mutations and subscriptions **src/graphql/**/*.graphql**
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions **Yes**
? Enter maximum statement depth [increase from default if your schema is deeply nested] **2**
? Enter the file name for the generated code **src/app/API.service.ts**
```

你还可以在`src/graphql`中找到三个有趣的文件:`queries.graphql`、`mutations.graphql`和`subscriptions.graphql`。这些文件包含已定义的 GraphQL 查询、变异和订阅，将用于检索数据、创建/更新数据和监听数据更改。

# 测试 API

要从 AWS 控制台测试 API，您必须首先创建一个用户:转到 Cognito 控制台，选择“管理用户池”，选择已经由 amplify 创建的用户池，然后选择菜单项“用户和组”，最后选择按钮“创建用户”。键入用户名、临时密码和有效的电子邮件地址。取消选中“将电话号码标记为已验证？”并创建用户。您应该会收到一封包含您的登录名和临时密码的电子邮件。

您还需要一个应用程序客户端 Id 来连接，您可以在 Cognito 控制台的应用程序客户端菜单中找到它。

现在转到 Amplify 控制台，选择你的应用程序，然后“后端环境”>开发> API >在 AppSync 中查看>运行查询。您应该访问一个 GraphQL 查询编辑器。通过点击“使用用户池登录”首次登录，并使用应用程序客户端 ID、登录名和临时密码；您必须键入一个新密码。最后，你应该被认证。

您可以尝试创建第一个客户:

```
mutation add {
  createCustomer(input: {
    name: "Casino"
  }) {
    id
    name
  }
}{
  "data": {
    "createCustomer": {
      "id": "99a34936-e773-42ad-aba4-7935c3e5bcc1",
      "name": "Casino"
    }
  }
}
```

然后得到这个特定的客户:

```
{
 getCustomer (
   id: "99a34936-e773-42ad-aba4-7935c3e5bcc1"
  ) {
   id
   name
 }
}{
  "data": {
    "getCustomer": {
      "id": "99a34936-e773-42ad-aba4-7935c3e5bcc1",
      "name": "Casino"
    }
  }
}
```

或者获取客户列表:

```
{
 listCustomers {
    items {
      id
      name
    }
  }
}{
  "data": {
    "listCustomers": {
      "items": [
        {
          "id": "99a34936-e773-42ad-aba4-7935c3e5bcc1"
          "name": "Casino",
        }
      ]
    }
  }
}
```

您可以验证用户只能访问他们创建的数据，正如我们在每个表定义中添加的`@auth(rules: [{allow: owner}])`所指定的那样:如上所述创建第二个用户，断开与第一个用户的连接并与第二个用户连接，然后再次运行 listCustomers 查询。您应该得到一个空列表。然后创建一个新客户，并再次获取列表。您应该只能看到该用户的客户。

# 认证集成

要将身份验证集成到应用程序中，您必须在应用程序的根目录放置一个`amplify-authenticator`组件，并在用户登录后将应用程序结构包含在该组件中:

```
<amplify-authenticator>
 <main>
  <mat-toolbar color="primary">
   <span>Billing Application</span>
   <span class="fill-remaining-space"></span>
   <a mat-button [routerLink]="['/customers']">My customers</a>
   <a mat-button [routerLink]="['/bills']">My bills</a>
   <amplify-sign-out></amplify-sign-out>
  </mat-toolbar>
  <div class="content">
   <router-outlet></router-outlet>
  </div>
 </main>
</amplify-authenticator>
```

# **列出客户**

由于前面创建的`API.service.ts`,可以通过对 API 的唯一调用获得客户列表:

```
constructor(private api: APIService) { }ngOnInit(): void {
  this.api.ListCustomers().then((list: ListCustomersQuery) => {
    this.customers = list.items;
  });
}
```

并像往常一样显示模板中的列表:

```
<mat-list>
  <a *ngFor="let customer of customers" mat-list-item [routerLink]="['/customer', customer.id]">{{customer.name}}</a>
</mat-list>
```

# 编辑客户

`API.servicets`为您提供添加、更新和删除客户的功能:

```
loadCustomer(id: string) {
  this.api.GetCustomer(id).then((customer: GetCustomerQuery) => {
    this.form.patchValue({
      id: customer.id,
      name: customer.name,
    })
  });
}add() {
  this.api.CreateCustomer(this.form.value).then(() => {
    this.router.navigate(['customers']);
  })
}update() {
  this.api.UpdateCustomer(this.form.value).then(() => {
    this.router.navigate(['customers']);
  })
}delete() {
  this.api.DeleteCustomer({ id: this.form.value['id']}).then(() => {
    this.router.navigate(['customers']);
  })
}
```

# 编辑账单

账单和行的版本类似于客户的版本。

# 发布应用程序

当您准备好发布应用程序时，首先需要将应用程序的源代码推送到 Git 存储库中。

然后，您必须转到 AWS Amplify 控制台，选择“前端环境”，选择您的 Git 提供商，然后按照说明选择包含应用程序源代码和您要部署的分支的存储库。

选择已经创建的`dev`后端环境，并如上所述创建一个新的服务角色。

给出了一个`amplify.yml`的内容。您可以下载它并将其保存在项目的根目录下。您也可以按照[https://docs . AWS . Amazon . com/amplify/latest/user guide/build-settings . html](https://docs.aws.amazon.com/amplify/latest/userguide/build-settings.html)上的说明进行个性化设置。特别是，如果您使用 Angular v9，您将必须指定使用节点 12:

```
version: 1
backend:
 phases:
  build:
   commands:
   - '# Execute Amplify CLI with the helper script'
   - amplifyPush --simple
frontend:
 phases:
  preBuild:
   commands:
 **- nvm use $VERSION_NODE_12**    - npm ci
  build:
   commands:
    - npm run build
 artifacts:
  baseDirectory: dist/billing
  files:
  - '**/*'
 cache:
  paths:
  - node_modules/**/*
```

最后点击“保存并部署”。这将运行应用程序的首次构建和部署。

稍后，当您在主分支中推送内容时，将自动启动新的构建，如果构建成功，应用程序将被部署。

## 在构建期间运行单元测试

如果您想在构建期间运行单元测试，您必须使用无头 chrome 驱动程序:

首先修改`package.json`:

```
"scripts": {
   [...]
   "test": "ng test **--no-watch --no-progress --browsers=ChromeHeadlessNoSandbox**",
```

然后将驱动程序添加到`karma.conf.js,`，并使用`--no-sandbox`选项运行，因为测试是以 root 用户身份运行的，如果没有这个选项，Chrome 不会以 root 用户身份启动。

```
browsers: ['Chrome'**, 'ChromeHeadlessNoSandbox'**],
**customLaunchers: {
 ChromeHeadlessNoSandbox: {
  base: 'ChromeHeadless',
  flags: ['--no-sandbox']
 }
},**
```

最后，您可以安装 Chrome 并从`amplify.yml`文件运行测试，在`preBuild`阶段:

```
[...]
frontend:
 phases:
  preBuild:
   commands:
   - nvm use $VERSION_NODE_12
   - npm m ci
 **- wget** [**https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm**](https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm) **- yum install -y ./google-chrome-stable_current_*.rpm**   **- npm run test**
```

# 用赛普拉斯运行 e2e 测试

现在，Cypress 是运行图形用户界面端到端测试的一个非常好的工具。这是 AWS Amplify 文档中记录的工具。让我们用它来做 e2e 测试。首先为我们的应用程序安装它:

```
$ npm add cypress --dev
$ $(npm bin)/cypress open
```

这个最新的命令首先创建一个包含默认规范文件的`cypress`目录，然后打开一个命令窗口，帮助您交互式地运行测试。你可以暂时打开窗户。

下面是检查登录过程的第一个测试，保存在`cypress/integration/authenticator_spec.js`中:

```
describe('Authenticator:', function () {
 beforeEach(function () {
 cy.visit('/');
});describe('Sign In:', () => {
 it('allows a user to signin', () => {
  cy.get('amplify-authenticator')
   .find(selectors.usernameInput, { includeShadowDom: true })
   .type("test", { force: true });
  cy.get('amplify-authenticator')
   .find(selectors.signInPasswordInput, { includeShadowDom: true })
   .type("password", , { force: true });
  cy.get('amplify-authenticator')
   .find(selectors.signInSignInButton, { includeShadowDom: true })
   .contains('Sign In')
   .click();cy.get('amplify-authenticator')
   .find(selectors.signOutButton, { includeShadowDom: true })
   .contains('Sign Out').should('be.visible');
  });
 });
});export const selectors = {
 // Auth component classes
 usernameInput: 'input[data-test="sign-in-username-input"]',
 signInPasswordInput: '[data-test="sign-in-password-input"]',
 signInSignInButton: '[data-test="sign-in-sign-in-button"]',
 signOutButton: '[data-test="sign-out-button"]'
}
```

请注意，**认证器**组件使用影子 DOM，并且在撰写本文时(2020–06)，影子 DOM 支持仍处于试验阶段。您必须在`cypress.json`文件中添加一个选项来启用它:

```
{
  "baseUrl": "http://localhost:4200/",
  "experimentalShadowDomSupport": true
}
```

您需要为测试创建一个专用用户(这里是`test/password`)。请按照上述步骤创建用户。记得第一次手动加载时验证其密码。

要尝试本地测试，首先使用`ng serve`从另一个终端启动应用程序，然后运行:

```
$ $(npm bin)/cypress run
```

要在 CI/CD 期间运行 e2e 测试，您需要在`amplify.yml`文件中添加一个`test`部分:

```
test:
 phases:
  preTest:
   commands:
   - npm ci
   - npm install wait-on
   - npm install mocha@5.2.0 mochawesome mochawesome-merge mochawesome-report-generator
   - 'npm start & npx wait-on [http://localhost:4200'](http://localhost:4200')
  test:
   commands:
   - 'npx cypress run --reporter mochawesome --reporter-options "reportDir=cypress/report/mochawesome-report,overwrite=false,html=false,json=true,timestamp=mmddyyyy_HHMMss"'
  postTest:
   commands:
   - npx mochawesome-merge cypress/report/mochawesome-report/mochawesome*.json > cypress/report/mochawesome.json
 artifacts:
  baseDirectory: cypress
  configFilePath: '**/mochawesome.json'
  files:
  - '**/*.png'
  - '**/*.mp4'
```

# 维护应用程序

一旦你的应用被部署，你可能需要对它进行一些改进。为此，AWS Amplify 可以帮助您使用专用后端处理功能分支。

首先在新的 git 存储库中创建一个新的分支，例如`add-address`，并将其推到原点:

```
$ git checkout -b add-address
$ git push --set-upstream origin add-address
```

现在，您可以从 Amplify 控制台连接这个新分支。在常规应用程序设置中，列出了管理的分支机构列表，一个按钮**连接一个分支机构**让您有机会管理您的新分支机构。

在表单中，选择新的分支名称`add-address`，选择“创建新环境”作为后端环境，并将其命名为`appaddress` ( *环境名称必须在 2 到 10 个字符之间，仅小写*)。当点击**保存并部署**时，一个名为`addaddress`的新后端将被创建并附加到您的新分支，前端将被构建，并被参数化以访问该后端。此操作可能需要几分钟时间。

您现在可以通过类似于 [https://add-address 的地址连接到您的开发前端。*.amplifyapp.com/](https://add-address.d1054gnrjvy2b0.amplifyapp.com/) 。由于你的后台是空的，你必须创建一个新用户，这一次是从你的应用程序的登录页面。

要在本地处理新的后端，您必须使用后端环境中的命令在本地创建它的配置:

```
$ amplify pull --appId ***anAppId*** --envName addaddress
```

要向`Customer`表添加一个`address`字段，您必须编辑`amplify/backend/api/billing/schema.graphql`文件:

```
type Customer
[@model](http://twitter.com/model)
[@auth](http://twitter.com/auth)(rules: [{allow: owner}])
{
  id: ID!
  name: String!
 **address: String**  bills: [Bill] [@connection](http://twitter.com/connection)(keyName: "byCustomer", fields: ["id"])
}
```

接下来，您可以推送您在 API 上所做的更改:

```
$ amplify api push
Current Environment: **addaddress**| Category | Resource name | Operation | Provider plugin   |
| -------- | ------------- | --------- | ----------------- |
| Api      | billing       | Update    | awscloudformation |
? Are you sure you want to continue? **Yes**
? Do you want to update code for your updated GraphQL API **Yes**
? Do you want to generate GraphQL statements (queries, mutations and subscription) based on your schema types?
This will overwrite your current graphql queries, mutations and subscriptions **Yes**
```

验证这些更改将在`addaddress`环境中进行，然后验证这些更改。此操作可能需要几分钟时间。

完成后，您可以在 AppSync 控制台中验证`billing-addaddress` API 模式中的表`Customer`是否包含新的`address`字段:

```
input CreateCustomerInput {
 id: ID
 name: String!
 **address: String** }
```

现在，你可以用你的 Angular 应用来管理这个新领域。请注意，`API.service.ts`文件已经更新，以匹配后端模式中所做的更改。

一旦完成，您可以将更改提交到`add-address`分支，并将分支推送到原点。AWS Amplify 将检测到这些变化，应用程序将自动构建并再次部署。

一旦应用程序部署完毕，您或您的客户或产品所有者就可以验证应用程序是否按预期运行。当你满意的时候，你可以将这个分支合并到主分支中，并将最后一个分支拉到原点。这样，AWS Amplify 将检测到变化:它将更新生产后端以添加新字段，然后重建应用程序并部署它。

您现在可以分离开发分支并删除开发后端。

# 添加自定义计算

您可能希望在后端添加特定的计算。在我们的应用程序中，我们希望每个账单都被分配一个新的字段`serialnum`，根据账单创建的月份，自动计算出一个递增值。该字段将得到形式`YYYYmmNNNN` —例如，2020 年 6 月的第二张账单将是`2020060002`。

为此，我们需要定义一个 Lambda 函数，当在数据库中创建一个新的账单时，该函数将被触发，并且能够查询和更新现有的账单。我们可以通过以下方式创建它:

```
$ amplify function add
Using service: Lambda, provided by: awscloudformation
? Provide a friendly name for your resource to be used as a label for this category in the project: **serial**
? Provide the AWS Lambda function name: **serial**
? Choose the function runtime that you want to use: **NodeJS**
? Choose the function template that you want to use: **Lambda trigger**
? What event source do you want to associate with Lambda trigger? **Amazon DynamoDB Stream**
? Choose a DynamoDB event source option **Use API category graphql** [**@model**](http://twitter.com/model) **backed DynamoDB table(s) in the current Amplify project**
Selected resource billing
? Choose the graphql [@model](http://twitter.com/model)(s) **Bill**
? Do you want to access other resources created in this project from your Lambda function? **Yes**
? Select the category **storage**
? Storage has 3 resources in this project. Select the one you would like your Lambda to access **Bill:**[**@model**](http://twitter.com/model)**(appsync)**
? Select the operations you want to permit for Bill:[@model](http://twitter.com/model)(appsync) **update, read**You can access the following resource attributes as environment variables from your Lambda function
        API_BILLING_BILLTABLE_ARN
        API_BILLING_BILLTABLE_NAME
 API_BILLING_GRAPHQLAPIIDOUTPUT
 ENV
 REGION
? Do you want to invoke this function on a recurring schedule? **No**
? Do you want to edit the local lambda function now? **No**
Successfully added resource serial locally.
```

我们将创建一个`serialnum`字段来存储这个自动值。

为了能够按创建日期顺序查询账单，您需要向账单模型添加一个二级键，并定义两个字段`createdAt`和`owner`。这些字段由系统编写，但是您需要声明它们以便能够在键中引用它们:

```
type Bill
@model
@auth(rules: [{allow: owner}])
@key(name: "byCustomer", fields: ["customerID"])
**@key(name: "byOwnerCreation", fields: ["owner", "createdAt"])** {
 id: ID!
 **serialnum: String** title: String!
 customerID: ID!
 customer: Customer @connection(fields: ["customerID"])
 lines: [Line] @connection(keyName: "byBill", fields: ["id"])
 **createdAt: AWSDateTime!
 owner: String** }
```

函数的代码可以在这里找到:[https://github . com/feloy/AWS-amplify-demo/blob/master/amplify/back end/function/serial/src/index . js](https://github.com/feloy/aws-amplify-demo/blob/master/amplify/backend/function/serial/src/index.js)