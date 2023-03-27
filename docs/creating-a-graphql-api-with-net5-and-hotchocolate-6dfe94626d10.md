# 用. NET5 和 HotChocolate 创建 GraphQL API

> 原文：<https://itnext.io/creating-a-graphql-api-with-net5-and-hotchocolate-6dfe94626d10?source=collection_archive---------2----------------------->

![](img/d5335b4135b7fe4f17c28186ff3eebba.png)

在这篇文章中，我将使用 HotChocolate 帮助您在. NET5 中开始使用 GraphQL。

# 介绍

GraphQL 是一种用于 API 的查询语言，也是一个用现有数据完成这些查询的运行时。

[热巧克力](https://chillicream.com/docs/hotchocolate)是微软的开源 GraphQL 服务器。NET 平台。这个库消除了构建完全成熟的 GraphQL 服务器的复杂性，让您专注于快速构建 API。

我们将创建一个简单的应用程序，可用于**创建**、**读取**、**更新**和**删除**一个客户。

这个示例的代码可以在 GitHub 上的 [GraphQLDemo](https://github.com/TSiustis/GraphQLDemo) 存储库中找到。

# 先决条件

*   [Visual Studio 2019](https://visualstudio.microsoft.com/vs/)
*   [SQL Server 2019](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

# 创建解决方案和项目

我们将需要一个 ASP.NET 网络应用编程接口项目。打开 VS 后，点击**新建项目**链接，选择**空白解决方案**，将项目和解决方案命名为 **GraphQLDemo** ，点击**创建**。接下来，右键单击该解决方案并选择**添加新项目**并选择**ASP.NET 核心 Web 应用程序，**选择**空项目**并将其命名为 **GraphQLDemo.Api.** 通过添加一个名为 **GraphQLDemo.Common.** 的**类库**来重复最后一步

# 安装依赖项

*   我们需要安装[热巧克力。AspNetCore(v11.3.8)](https://chillicream.com/) 包。这个包包含了 ASP.NET 的 GraphQL APIs。要安装这个包，在解决方案浏览器中右键单击这个解决方案，并选择**Manage nu get Packages for Solution**。在浏览部分，搜索**热巧克力。AspNetCore** 并点击它，然后在预览面板中点击安装按钮。我们还需要热巧克力。AspNetCore.Playground 和**热巧克力。数据**。对所有包重复该过程。
*   安装**微软。EntityFrameworkCore** ，**微软。EntityFrameworkCore . SQL server**和**微软。EntityFrameworkCore.Design** 分别封装。

# 实体

我们将使用 [EntityFrameworkCore](https://docs.microsoft.com/en-us/ef/core/) 作为 ORM。实体(类)用于定义数据库对象(表)，这些实体被映射到 [DbContext](https://docs.microsoft.com/en-us/ef/core/dbcontext-configuration/) 中的数据库表。

在这个演示中，我们将定义一个实体，即客户**，但是您可以拥有任意多个实体。
数据访问**代码、**存储库**和 **GraphQL** 相关类将被拆分到不同的文件夹中。****

# **客户模型**

**在 **GraphQLDemo 中创建一个文件夹 **Models** 。常见的**和新的 c#类 **Customer.cs** 。这个类的代码是:**

```
public class Customer{public long Id { get; set; }public string Email { get; set; }public string Name { get; set; }public int? Code { get; set; }public DateTime CreatedAt { get; set; }public bool IsBlocked { get; set; }}
```

**其余的类将被添加到 GraphQLDemo.API 中。**

# **数据库上下文**

**DbContext 将非常简单。大多数数据库配置都是在这个文件中完成的。在**数据库**文件夹中添加一个新的 C#类文件，命名为**graphqldemodbcontext . cs**。通常以应用程序的名字命名 **DbContext** 。该文件的代码是:**

```
public DbSet<Customer> Customers { get; set; }public GraphQLDemoDbContext(DbContextOptions<GraphQLDemoDbContext> options) : base(options){}
```

****DbSet<Customer>**定义客户属性，用于将 **Customer.cs** 实体映射到数据库中的 **Customer** 表。**

**从我的 github repo 中，您还可以获得一个样本数据播种器。**

# ****创建数据库****

**在 appsettings.json 中添加以下行:**

```
"ConnectionStrings": {"ConnectionString": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=GraphQL;Integrated Security=True;Connect Timeout=30;Encrypt=False;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False;"}
```

**在软件包管理器控制台中:**

*   **输入命令 Add-Migration<yourcustommigrationname></yourcustommigrationname>**
*   **更新-数据库**

**该数据库将被创建，名称为 **GraphQL** 。**

# **客户知识库**

**在服务文件夹中添加以下接口 **ICustomerRepository** :**

```
public interface  ICustomerRepository{void Add(Customer customer);void Delete(int id);IQueryable<Customer> GetAll();Customer GetById(int id);Customer Update(Customer customer);}
```

**将一个名为 **CustomerRepository.cs** 的实现 ICustomerRepository**的 C#类文件添加到 **Services** 文件夹中:****

```
public class CustomerRepository : ICustomerRepository{private readonly GraphQLDemoDbContext _dbContext;public CustomerRepository(GraphQLDemoDbContext dbContext){_dbContext = dbContext;}public void Add(Customer customer){_dbContext.Customers.Add(customer);_dbContext.SaveChanges();}public void Delete(int id){var customer = _dbContext.Customers.FirstOrDefault(c => c.Id == id);if (customer != null){_dbContext.Customers.Remove(customer);_dbContext.SaveChanges();}}public IQueryable<Customer> GetAll() => _dbContext.Customers.AsQueryable();public Customer GetById(int id) => _dbContext.Customers.FirstOrDefault(c => c.Id == id);public Customer Update(Customer customer){var newCustomer = new Customer();if (newCustomer.Name != null)newCustomer.Name = customer.Name;newCustomer.Code ??= customer.Code;newCustomer.CreatedAt = customer.CreatedAt;if (newCustomer.Email != null)newCustomer.Email = customer.Email;newCustomer.IsBlocked = customer.IsBlocked;newCustomer.Status = customer.Status;_dbContext.Customers.Update(customer);_dbContext.SaveChanges();return newCustomer;}
```

**该类定义了返回所有客户列表的 **GetAll()** 、 **GetById()** 、获取 Id 并返回客户(如果客户存在则返回 null，如果客户不存在则返回 null)、 **Add()** 方法将客户作为参数并将其添加到数据库中、 **Update()** 获取客户并更新客户，以及 **Delete** 方法获取 Id 并删除客户(如果客户不存在)。**

# **GraphQL**

**对于 REST API，我们使用 GET 来获取数据，使用 POST、PUT、DELETE 来修改数据。GET in REST API 与 GraphQL 中的 Query 相同。POST，PUT，DELETE 和突变是一样的。**

## **询问**

**将名为 **Query.cs** 的新 C#类文件添加到 **GraphQL** 文件夹中。这个类将包含我们需要执行的所有查询:**

```
public class Query{[UsePaging][UseFiltering][UseSorting]public IQueryable<Customer> Customers([Service] ICustomerRepository _customerRepository) => _customerRepository.GetAll();public Customer Customer([Service] ICustomerRepository _customerRepository, int id){var customer = _customerRepository.GetById(id);if (customer == null){throw new CustomerNotFoundException() { Id = id };}return customer;}}
```

**Customers()返回所有客户。Customer()返回 id 指定的客户，如果它存在于数据库中。**

**Query.cs 和 Mutation.cs 中的所有方法都使用**【服务】**标记注入了 **CustomerRepository** 。**

**我们还添加了分页、过滤和排序功能，这些功能由 HotChocolate 库负责。**

## **变化**

**将名为 **Mutation.cs** 的新 C#类文件添加到 **GraphQL** 文件夹中。这个类将包含我们需要执行的所有突变:**

```
public Customer AddCustomer([Service] ICustomerRepository _customerRepository, string name, string email, int? code, DateTime createdAt, bool isBlocked){var customer = new Customer{Name = name,Email = email,Code = code,CreatedAt = createdAt,IsBlocked = isBlocked};_customerRepository.Add(customer);return customer;}public Customer UpdateCustomer([Service] ICustomerRepository _customerRepository, int id, string? name, string? email, int? code, DateTime? createdAt, bool? isBlocked){var customerToUpdate = _customerRepository.GetById(id);if (customerToUpdate == null) throw new CustomerNotFoundException() { Id = id };if (name != null)customerToUpdate.Name = name;if (email != null)customerToUpdate.Email = email;if (code != null)customerToUpdate.Code = code;if (createdAt != null)customerToUpdate.CreatedAt = (DateTime)createdAt;if (isBlocked != null)customerToUpdate.IsBlocked = (bool)isBlocked;_customerRepository.Update(customerToUpdate);return customerToUpdate;}public Customer Delete(int id, [Service] ICustomerRepository _customerRepository){var customerToDelete = _customerRepository.GetById(id);if (customerToDelete == null)throw new CustomerNotFoundException() { Id = id };_customerRepository.Delete(id);return customerToDelete;} 
```

**AddCustomer()将客户添加到数据库中，UpdateCustomer()完全或部分更新客户，Delete()根据给定的 id(如果存在)删除客户。请注意，所有方法都需要返回要添加/更新/删除的实体，因为 GraphQL 需要返回它。**

# **配置 GraphQL**

**我们需要配置 **Startup.cs** 来使用 GraphQL。将文件中的代码替换为:**

```
using GraphQLDemo.Api.Database;using GraphQLDemo.Api.GraphQL;using GraphQLDemo.Api.Services;using HotChocolate;using HotChocolate.AspNetCore;using HotChocolate.AspNetCore.Playground;using Microsoft.AspNetCore.Builder;using Microsoft.AspNetCore.Hosting;using Microsoft.AspNetCore.Http;using Microsoft.EntityFrameworkCore;using Microsoft.Extensions.Configuration;using Microsoft.Extensions.DependencyInjection;using Microsoft.Extensions.Hosting;namespace GraphQLDemo.Api{public class Startup{public IConfiguration Configuration { get; }public Startup(IConfiguration configuration){Configuration = configuration;}public void ConfigureServices(IServiceCollection services){services.AddScoped<ICustomerRepository, CustomerRepository>();services.AddDbContext<GraphQLDemoDbContext>(context =>{context.UseSqlServer(Configuration.GetConnectionString("ConnectionString"),sqlServerOptionsAction: sqlOptions =>{sqlOptions.EnableRetryOnFailure();});});services.AddGraphQLServer().AddQueryType<Query>().AddFiltering().AddSorting().ModifyRequestOptions(opt => opt.IncludeExceptionDetails = true).AddMutationType<Mutation>();}// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.public void Configure(IApplicationBuilder app, IWebHostEnvironment env){if (env.IsDevelopment()){app.UseDeveloperExceptionPage();app.UsePlayground(new PlaygroundOptions{QueryPath = "/api",Path = "/playground"});}app.UseRouting();app.UseEndpoints(endpoints =>{endpoints.MapGraphQL("/api");});}}}
```

**在这个文件中，我们为 GraphQL、查询和变异配置服务。**

**我们还注册了 **Playground** ，一个你可以轻松测试你的 GraphQL API 的地方。**

# **运行应用程序**

**构建解决方案并运行应用程序。在您的浏览器中，导航到[https://localhost:44351/playground/](https://localhost:44351/graphql/)来测试您的 API。**

**Query.cs、Mutation.cs 中定义的方法已经分别归类在 Query 和 Mutation 下。**

# **测试查询**

**您可以使用以下查询获得前两个客户:**

```
query getCustomers{
  customers(first:2){
    edges{
      node{
        id,
        name,
        email,
        code,
        createdAt,
        isBlocked
      }
    }
  }
}
```

**由于我们使用分页，我们必须查询客户 [**连接**](https://chillicream.com/docs/hotchocolate/fetching-data/pagination) 而不是直接查询客户对象。直接通过 id 查询客户的例子如下:**

```
query getCustomer{
  customer(id:3){
        id,
        name,
        email,
        code,
        createdAt,
        isBlocked 
  }
}
```

# **测试突变**

**您可以添加具有以下变异的客户:**

```
mutation addCustomer{
  addCustomer(name: "John Doe", email:"[johndoe@gmail.com](mailto:johndoe@gmail.com)", code: 4, isBlocked: false createdAt:"2020-08-08"){
      id,
      name,
      email,
      createdAt,
      isBlocked
    }
  }
```

**教程到此结束。尝试 API 并添加您自己的特性！**