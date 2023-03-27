# 在 Symfony 中使用枚举

> 原文：<https://itnext.io/using-enum-in-symfony-8f32d9cfaa0f?source=collection_archive---------3----------------------->

PHP 8.1 引入了官方的 *Enum* 支持。 [Doctrine 在其 ORM 框架](https://www.doctrine-project.org/2022/01/11/orm-2.11.html)中引入了 Enum 类型支持， [Symfony 增加了对 Enum 类型的序列化和反序列化支持](https://symfony.com/blog/new-in-symfony-5-4-php-enumerations-support)。

![](img/2898f06ba8a8dfe1e2f53fbfc161d43c.png)

在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上 [te chan](https://unsplash.com/@kakachen?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

如果您正在使用第三方枚举解决方案，现在是时候将您的项目迁移到使用 PHP Enum 了。

要使用 PHP *Enum* ，必须升级到 PHP 8.1，并在 project composer 文件中将 PHP 版本设置为`8.1`。

```
{
    //...
    "require": {
        "php": ">=8.1",
        //...
    }
}
```

# 正在创建枚举类

例如，我们将在`Post`实体中添加一个`Status`，并定义了帖子状态的几个固定值。

```
<?phpnamespace App\Entity;enum Status: string
{
    case Draft = "DRAFT";
    case PendingModerated = "PENDING_MODERATED";
    case Published = "PUBLISHED";
}
```

这里我们使用一个*字符串*支持的枚举，在`Post`类中添加一个字段。

```
#[Column(type: "string", enumType: Status::class)]
private Status $status;
```

注意，将*枚举类型*设置为`Status`类。它将状态值作为字符串存储在数据库表中。

在`Post`构造函数中，为状态分配一个默认值。

```
public function __construct()
{
    $this->status = Status::Draft;
    //...
}
```

现在一切都好了。

# 创建 HttpMethod

当我们在控制器类上设置`Route`属性时，我们使用一个文字值来设置 HTTP 方法。

```
#[Route(path: "/{id}", name: "byId", methods: ["GET"])]
```

对于*方法*值，只有几个选项可供选择。显然，如果引入*枚举*，它将提供一种*类型安全的*方式来设置值并减少输入错误。

创建一个名为`HttpMethod`的枚举。

```
<?phpnamespace App\Annotation;enum HttpMethod
{
    case GET;
    case POST;
    case HEAD;
    case OPTIONS;
    case PATCH;
    case PUT;
    case DELETE;
}
```

然后重构`Route`属性，创建一系列属性(`Get`、`Post`、`Put`、`Delete`等)。)映射到不同的 HTTP 方法。

```
//file : src/Annotation/Get.php
#[Attribute]
class Get extends Route
{
    public function getMethods()
    {
        return [HttpMethod::GET->name];
    }}//file : src/Annotation/Head.php
#[Attribute]
class Head extends Route
{
    public function getMethods()
    {
        return [HttpMethod::HEAD->name];
    }}//file : src/Annotation/Options.php
#[Attribute]
class Options extends Route
{
    public function getMethods()
    {
        return [HttpMethod::OPTIONS->name];
    }}//file : src/Annotation/Patch.php
#[Attribute]
class Patch extends Route
{
    public function getMethods()
    {
        return [HttpMethod::PATCH->name];
    }
}//file : src/Annotation/Post.php
#[Attribute]
class Post extends Route
{
    public function getMethods()
    {
        return [HttpMethod::POST->name];
    }
}//file : src/Annotation/Put.php
#[Attribute]
class Put extends Route
{
    public function getMethods()
    {
        return [HttpMethod::PUT->name];
    }
}//file : src/Annotation/Delete.php
#[Attribute]
class Delete extends Route
{
    public function getMethods()
    {
        return [HttpMethod::DELETE->name];
    }
}
```

现在你可以润色`PostController`，使用这些属性代替。正如您所看到的，新属性的命名看起来更加清晰。

```
#[Route(path: "/posts", name: "posts_")]
class PostController extends AbstractController
{ // constructor... // #[Route(path: "", name: "all", methods: ["GET"])]
    #[Get(path: "", name: "all")]
    public function all(#[QueryParam] string $keyword,
                        #[QueryParam] int $offset = 0,
                        #[QueryParam] int $limit = 20): Response
    {
        //...
    } // #[Route(path: "/{id}", name: "byId", methods: ["GET"])]
    #[Get(path: "/{id}", name: "byId")]
    public function getById(Uuid $id): Response
    {
       //...
    } //#[Route(path: "", name: "create", methods: ["POST"])]
    #[Post(path: "", name: "create")]
    public function create(#[Body] CreatePostDto $data): Response
    {
        //...
    } //#[Route(path: "/{id}", name: "update", methods: ["PUT"])]
    #[Put(path: "/{id}", name: "update")]
    public function update(Uuid $id, #[Body] UpdatePostDto $data): Response
    {
        //...
    } // #[Route(path: "/{id}/status", name: "update_status", methods: ["PUT"])]
    #[Put(path: "/{id}/status", name: "update_status")]
    public function updateStatus(Uuid $id, #[Body] UpdatePostStatusDto $data): Response
    {
       //...
    } //#[Route(path: "/{id}", name: "delete", methods: ["DELETE"])]
    #[Delete(path: "/{id}", name: "delete")]
    public function deleteById(Uuid $id): Response
    {
        //...
    } // comments sub resources.
    //#[Route(path: "/{id}/comments", name: "commentByPostId", methods: ["GET"])]
    #[GET(path: "/{id}/comments", name: "commentByPostId")]
    public function getComments(Uuid $id): Response
    {
      //...
    } //#[Route(path: "/{id}/comments", name: "addComments", methods: ["POST"])]
    #[Post(path: "/{id}/comments", name: "addComments")]
    public function addComment(Uuid $id, Request $request): Response
    {
		//...
    }}
```

再次运行该应用程序以确保其正常工作。

## 从我的 Github 获取源代码。

## 顺便说一句，我已经把我原来的 Symfony 帖子更新成了一个分步指南，在线阅读:[https://hantsy.github.io/symfony-rest-sample/](https://hantsy.github.io/symfony-rest-sample/)