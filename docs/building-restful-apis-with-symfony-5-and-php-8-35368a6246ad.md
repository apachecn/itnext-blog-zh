# 用 Symfony 5 和 PHP 8 构建 Restful APIs

> 原文：<https://itnext.io/building-restful-apis-with-symfony-5-and-php-8-35368a6246ad?source=collection_archive---------1----------------------->

Symfony 是一个全功能的模块化 PHP 框架，用于构建各种应用程序，从传统的 web 应用程序到小型微服务组件。

![](img/f08daefead13fc15e5d706ddba36d212.png)

[te chan](https://unsplash.com/@kakachen?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/china-snow?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍照

# 把你的脚弄湿

安装 PHP 8 和 PHP Composer 工具。

```
# choco php composer
```

安装【Symfony CLI】(Symfony check:requirements)，检查系统需求。

```
# symfony check:requirementsSymfony Requirements Checker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~> PHP is using the following php.ini file:
C:\tools\php80\php.ini> Checking Symfony requirements:....................WWW......... [OK]                                         
 Your system is ready to run Symfony projects Optional recommendations to improve your setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ * intl extension should be available
   > Install and enable the intl extension (used for validators). * a PHP accelerator should be installed
   > Install and/or enable a PHP accelerator (highly recommended). * realpath_cache_size should be at least 5M in php.ini
   > Setting "realpath_cache_size" to e.g. "5242880" or "5M" in
   > php.ini* may improve performance on Windows significantly in some
   > cases. Note  The command console can use a different php.ini file
~~~~  than the one used by your web server.
      Please check that both the console and the web server
      are using the same PHP version and configuration.
```

根据*建议*信息，在 *php.ini* 中调整您的 PHP 配置。我们将在示例应用程序中使用 Postgres 作为数据库，确保启用了`pdo_pgsql`和`pgsql`模块。

最后，您可以通过以下命令确认启用的模块。

```
# php -m
```

创建一个新的 Symfony 项目。

```
# symfony new rest-sample// a classic website application
# symfony new web-sample --full
```

默认情况下，它将创建一个简单的 Symfony 骨架项目，只包含核心内核配置，这有利于启动一个轻量级 Restful API 应用程序。

或者，您可以使用 Composer 创建它。

```
# composer create-project symfony/skeleton rest-sample//start a classic website application
# composer create-project symfony/website-skeleton web-sample
```

进入生成的项目根文件夹，启动应用程序。

```
# symfony server:start [WARNING] run "symfony.exe server:ca:install" first if you want to run the web server with TLS support, or use "--no-  
 tls" to avoid this warning                                                                                             

Tailing PHP-CGI log file (C:\Users\hantsy\.symfony\log\499d60b14521d4842ba7ebfce0861130efe66158\79ca75f9e90b4126a5955a33ea6a41ec5e854698.log)
Tailing Web Server log file (C:\Users\hantsy\.symfony\log\499d60b14521d4842ba7ebfce0861130efe66158.log)

 [OK] Web server listening                                                                                              
      The Web server is using PHP CGI 8.0.10                                                                            
      [http://127.0.0.1:8000](http://127.0.0.1:8000) [Web Server ] Oct  4 13:33:01 |DEBUG  | PHP    Reloading PHP versions
[Web Server ] Oct  4 13:33:01 |DEBUG  | PHP    Using PHP version 8.0.10 (from default version in $PATH)
[Web Server ] Oct  4 13:33:01 |INFO   | PHP    listening path="C:\\tools\\php80\\php-cgi.exe" php="8.0.10" port=61738
```

# 你好，Symfony

在 HTTP 响应中为资源实体创建一个简单的类。

```
class Post
{
    private ?string $id = null; private string $title; private string $content;

    //getters and setters.
}
```

并使用工厂创建一个新的 Post 实例。

```
class PostFactory
{
    public static function create(string $title, string $content): Post
    {
        $post = new Post();
        $post->setTitle($title);
        $post->setContent($content);
        return $post;
    }
}
```

让我们创建一个简单的控制器类。

要使用最新的 PHP 8 属性来配置路由规则，请在项目配置中应用以下更改。

*   打开*config/packages/doctrine . YAML*，移除`doctrine/orm/mapping/App/type`或将其值改为`attribute`
*   打开 *composer.json* ，将 PHP 版本改为`>=8.0.0`。

要将响应主体呈现为 JSON 字符串，请使用一个`JsonReponse`来包装响应。

```
#[Route(path: "/posts", name: "posts_")]
class PostController
{ #[Route(path: "", name: "all", methods: ["GET"])]
    function all(): Response
    {
        $post1 = PostFactory::create("test title", "test content");
        $post1->setId("1"); $post2 = PostFactory::create("test title", "test content");
        $post2->setId("2");
        $data = [$post1->asArray(), $post2->asArray()];
        return new JsonResponse($data, 200, ["Content-Type" => "application/json"]);
        //return $this->json($data, 200, ["Content-Type" => "application/json"]);
    }
}
```

`JsonReponse`的第一个参数接受一个数组作为数据，因此在`Post`类中添加一个函数来实现这个目的。

```
class Post{
    //...
    public function asArray(): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'content' => $this->content
        ];
    }
}
```

运行应用程序，使用`curl`测试`/posts`端点。

```
# curl [http://localhost:8000/posts](http://localhost:8000/posts)
```

Symfony 提供了一个简单的`AbstractController`,它包括几个函数来简化响应并采用容器和依赖注入管理。

在上面的控制器中，从`AbstractController`扩展而来，只需调用`$this->json`以 JSON 格式渲染响应，无需在渲染响应前将数据转换为数组。

```
class PostController extends AbstractController
{ function all(): Response
    {
        //...
        return $this->json($data, 200, ["Content-Type" => "application/json"]);
    }
}
```

# 连接到数据库

Doctrine 是一个流行的 ORM 框架，它高度受现有 Java ORM 工具的启发，如 JPA spec 和 Hibernate 框架。教义中有两个核心组件，`doctrine/dbal`和`doctrine/orm`，前者是用于数据库操作的底层 API，如果你懂 Java 开发，就把它当成 *Jdbc* 层。后者是高级 ORM 框架，公共 API 类似于 JPA/Hibernate。

将原则纳入项目。

```
# composer require symfony/orm-pack
# composer require --dev symfony/maker-bundle
```

**包**是一个虚拟的 Symfony 包，它会安装一系列的包和基本配置。

打开项目根目录下的`.env`文件，编辑`DATABASE_URL`值，设置数据库名称、用户名、密码进行连接。

```
DATABASE_URL="postgresql://user:password@127.0.0.1:5432/blogdb?serverVersion=13&charset=utf8"
```

使用以下命令生成 docker 合成文件模板。

```
# php bin/console make:docker:database
```

我们将其更改为以下内容，以启动正在开发的 Postgres 数据库。

```
version: "3.5" # specify docker-compose version, v3.5 is compatible with docker 17.12.0+# Define the services/containers to be run
services: postgres:
    image: postgres:${POSTGRES_VERSION:-13}-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-blogdb}
      # You should definitely change the password in production
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
      POSTGRES_USER: ${POSTGRES_USER:-user}
    volumes:
      - ./data/blogdb:/var/lib/postgresql/data:rw
      - ./pg-initdb.d:/docker-entrypoint-initdb.d
```

我们将使用`UUID`作为主键的数据类型，添加一个脚本以在 Postgres 启动时启用`uuid-ossp`扩展。

```
-- file: pg-initdb.d/ini.sql
SET search_path TO public;
DROP EXTENSION IF EXISTS "uuid-ossp";
CREATE EXTENSION "uuid-ossp" SCHEMA public;
```

打开*config/packages/test/doctrine . YAML*，注释掉`dbname_suffix`行。我们使用 Docker 容器来引导数据库，以确保开发和生产之间的应用程序行为是相同的。

现在启动应用程序并确保控制台中没有异常，这意味着数据库连接成功。

```
symfony server:start
```

在启动应用程序之前，请确保数据库正在运行。运行以下命令在 Docker 中启动 Postgres。

```
# docker compose up postgres
# docker ps -a # to list all containers and make the postgres is running
```

# 构建数据模型

现在，我们将构建将在下一节中使用的实体。我们正在建模一个简单的博客系统，它包括以下概念。

*   A `Post`在博客系统中呈现文章帖子。
*   A `Comment`呈现特定帖子下的评论。
*   常用的`Tag`可以应用在不同的帖子上，按照主题、类别等对帖子进行分类。

您可以在头脑中或通过一些图形数据建模工具来绘制模型关系。

*   帖子和评论是一种`one-to-many`关系
*   发布和标记是一种`many-to-many`关系

通过教义很容易将想法转化为真正的代码。运行以下命令创建`Post`、`Comment`和`Tag`实体。

在 ORM 2.10.x 和 Dbal 3.x 中，UUID 类型 ID 生成器已被弃用。我们将切换到 Uuid 形式`symfony\uid`。

先安装`symfony\uid`。

```
# composer require symfony/uid
```

简单地说，您可以使用下面的命令快速创建实体。

```
# php bin/console make:entity  # following the interactive steps to create them one by one.
```

最后我们在 *src/Entity* 文件夹中得到三个实体。按照您的期望修改它们。

```
// src/Entity/Post.php
#[Entity(repositoryClass: PostRepository::class)]
class Post
{
    #[Id]
    //#[GeneratedValue(strategy: "UUID")
    //#[Column(type: "string", unique: true)]
    #[Column(type: "uuid", unique: true)]
    #[GeneratedValue(strategy: "CUSTOM")]
    #[CustomIdGenerator(class: UuidGenerator::class)]
    private ?Uuid $id = null; #[Column(type: "string", length: 255)]
    private string $title; #[Column(type: "string", length: 255)]
    private string $content; #[Column(name: "created_at", type: "datetime", nullable: true)]
    private DateTime|null $createdAt = null; #[Column(name: "published_at", type: "datetime", nullable: true)]
    private DateTime|null $publishedAt = null; #[OneToMany(mappedBy: "post", targetEntity: Comment::class, cascade: ['persist', 'merge', "remove"], fetch: 'LAZY', orphanRemoval: true)]
    private Collection $comments; #[ManyToMany(targetEntity: Tag::class, mappedBy: "posts", cascade: ['persist', 'merge'], fetch: 'EAGER')]
    private Collection $tags; public function __construct()
    {
        $this->createdAt = new DateTime();
        $this->comments = new ArrayCollection();
        $this->tags = new ArrayCollection();
    }
    //other getters and setters
}// src/Entity/Comment.php
#[Entity(repositoryClass: CommentRepository::class)]
class Comment
{
    #[Id]
    //#[GeneratedValue(strategy: "UUID")]
    #[Column(type: "uuid", unique: true)]
    #[GeneratedValue(strategy: "CUSTOM")]
    #[CustomIdGenerator(class: UuidGenerator::class)]
    private ?Uuid $id = null; #[Column(type: "string", length: 255)]
    private string $content; #[Column(name: "created_at", type: "datetime", nullable: true)]
    private DateTime|null $createdAt = null; #[ManyToOne(targetEntity: "Post", inversedBy: "comments")]
    #[JoinColumn(name: "post_id", referencedColumnName: "id")]
    private Post $post; public function __construct()
    {
        $this->createdAt = new DateTime();
    }
    //other getters and setters
}//src/Entity/Tag.php
#[Entity(repositoryClass: TagRepository::class)]
class Tag
{
    #[Id]
    //#[GeneratedValue(strategy: "UUID")
    //#[Column(type: "string", unique: true)]
    #[Column(type: "uuid", unique: true)]
    #[GeneratedValue(strategy: "CUSTOM")]
    #[CustomIdGenerator(class: UuidGenerator::class)]
    private ?Uuid $id = null; #[Column(type: "string", length: 255)]
    private ?string $name; #[ManyToMany(targetEntity: Post::class, inversedBy: "tags")]
    private Collection $posts; public function __construct()
    {
        $this->posts = new ArrayCollection();
    }
}
```

同时，它为这些实体生成了三个`Repository`类。

```
// src/Repository/PostRepsoitory.php
class PostRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Post::class);
    }
}// src/Repository/CommentRepsoitory.php
class CommentRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Comment::class);
    }
}//src/Repository/TagRepository.php
class TagRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Tag::class);
    }
}
```

您可以使用原则迁移来生成一个*迁移*文件，以在生产环境中维护数据库模式。

运行以下命令生成一个*迁移*文件。

```
# php bin/console make:migration
```

执行后，在*迁移*文件夹中生成一个迁移文件，其命名类似于`Version20211104031420`。这是一个简单的扩展`AbstractMigration`类，`up`函数用于升级到这个版本，`down`函数用于降级到以前的版本。

自动在数据库上应用迁移。

```
# php bin/console doctrine:migrations:migrate# return to prev version
# php bin/console doctrine:migrations:migrate prev# migrate to next
# php bin/console doctrine:migrations:migrate next# These alias are defined : first, latest, prev, current and next# certain version fully qualified class name
# php bin/console doctrine:migrations:migrate FQCN
```

Doctrine bundle 还包括一些维护数据库和模式命令。例如。

```
# php bin/console doctrine:database:create
# php bin/console doctrine:database:drop// schema create, drop, update and validate
# php bin/console doctrine:schema:create
# php bin/console doctrine:schema:drop
# php bin/console doctrine:schema:update
# php bin/console doctrine:schema:validate
```

# 添加样本数据

创建一个自定义命令来加载一些示例数据。

```
# php bin/console make:command add-post
```

它会在 *src/Command* 文件夹下生成一个`AddPostCommand`。

```
#[AsCommand(
    name: 'app:add-post',
    description: 'Add a short description for your command',
)]
class AddPostCommand extends Command
{
 public function __construct(private EntityManagerInterface $manager)
    {
        parent::__construct();
    } protected function configure(): void
    {
        $this
            ->addArgument('title', InputArgument::REQUIRED, 'Title of a post')
            ->addArgument('content', InputArgument::REQUIRED, 'Content of a post')
            //->addOption('option1', null, InputOption::VALUE_NONE, 'Option description')
        ;
    } protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $io = new SymfonyStyle($input, $output);
        $title = $input->getArgument('title'); if ($title) {
            $io->note(sprintf('Title: %s', $title));
        } $content = $input->getArgument('content'); if ($content) {
            $io->note(sprintf('Content: %s', $content));
        } $entity = PostFactory::create($title, $content);
        $this ->manager->persist($entity);
        $this ->manager->flush();//        if ($input->getOption('option1')) {
//            // ...
//        } $io->success('Post is saved: '.$entity); return Command::SUCCESS;
    }
}
```

教条`EntityManagerInterface`由 Symfony *服务容器*管理，用于数据持久化操作。

运行以下命令，将帖子添加到数据库中。

```
# php bin/console app:add-post "test title" "test content"
 ! [NOTE] Title: test title                                               
 ! [NOTE] Content: test content                                                             
 [OK] Post is saved: Post: [ id =1ec3d3ec-895d-685a-b712-955865f6c134, title=test title, content=test content, createdAt=1636010040, blishedAt=]
```

# 测试存储库

PHPUnit 是 PHP 世界中最流行的测试框架，Symfony 紧密地集成了 PHPUnit。

运行以下命令来安装 PHPUnit 和 Symfony **测试包**。**测试包**将安装所有用于测试 Symfony 组件的必要包，并添加 PHPUnit 配置，如 *phpunit.xml.dist* 。

```
# composer require --dev phpunit/phpunit symfony/test-pack
```

一个用纯 PHPUnit 编写的简单测试示例。

```
class PostTest extends TestCase
{ public function testPost()
    {
        $p = PostFactory::create("tests title", "tests content"); $this->assertEquals("tests title", $p->getTitle());
        $this->assertEquals("tests content", $p->getContent());
        $this->assertNotNull( $p->getCreatedAt());
    }
}
```

Symfony 提供了一些特定的基类(`KernelTestCase`、`WebTestCase`等)。)来简化 Symfony 项目中的测试工作。

下面是测试一个`Repository` - `PostRepository`的例子。`KernelTestCase`包含引导应用内核的工具，并提供服务容器。

```
class PostRepositoryTest extends KernelTestCase
{ private EntityManagerInterface $entityManager; private PostRepository $postRepository; protected function setUp(): void
    {
        //(1) boot the Symfony kernel
        $kernel = self::bootKernel();
        $this->assertSame('test', $kernel->getEnvironment());
        $this->entityManager = $kernel->getContainer()
            ->get('doctrine')
            ->getManager(); //(2) use static::getContainer() to access the service container
        $container = static::getContainer(); //(3) get PostRepository from container.
        $this->postRepository = $container->get(PostRepository::class);
    } protected function tearDown(): void
    {
        parent::tearDown();
        $this->entityManager->close();
    } public function testCreatePost(): void
    {
        $entity = PostFactory::create("test post", "test content");
        $this->entityManager->persist($entity);
        $this->entityManager->flush();
        $this->assertNotNull($entity->getId()); $byId = $this->postRepository->findOneBy(["id" => $entity->getId()]);
        $this->assertEquals("test post", $byId->getTitle());
        $this->assertEquals("test content", $byId->getContent());
    }}
```

在上面的代码中，在`setUp`函数中，启动应用内核，启动后，测试范围*服务容器*可用。然后从服务容器中取出`EntityManagerInterface`和`PostRepository`。

在`testCreatePost`函数中，持久化一个`Post`实体，通过 id 找到这个帖子，并验证*标题*和*内容*字段。

> *目前，PHPUnit 不包括 PHP 8 属性支持，测试代码类似于遗留的 JUnit 4 代码风格。*

# 创建后置控制器:公开第一个 Rest API

与其他 MVC 框架类似，我们可以通过 Symfony `Controller`组件公开 RESTful APIs。遵循 REST 惯例，我们计划为博客系统创建以下 API。

*   `GET /posts`获取所有帖子。
*   `GET /posts/{id}`通过 ID 获取单个帖子，如果没有找到，返回状态 404
*   `POST /posts`从请求体创建一个新帖子，将新帖子 URI 添加到响应头`Location`，并返回状态 201
*   `DELETE /posts/{id}`按 ID 删除单个帖子，返回状态 204。如果没有找到帖子，则返回状态 404。
*   …

运行以下命令创建控制器骨架。按照交互式指南创建一个名为`PostController`的控制器。

```
# php bin/console make:constroller
```

在 IDE 中打开*src/Controller/post Controller . PHP*。

在类级别上添加`Route`属性和两个函数:一个用于获取所有帖子，另一个用于通过 ID 获取单个帖子。

```
#[Route(path: "/posts", name: "posts_")]
class PostController extends AbstractController
{
    public function __construct(private PostRepository      $posts)
    {
    } #[Route(path: "", name: "all", methods: ["GET"])]
    function all(): Response
    {
        $data = $this->posts->findAll();
        return $this->json($data);
    }

}
```

启动应用程序，尝试访问[*http://localhost:8000/posts*](http://localhost:8000/posts)，直接在 JSON 视图中渲染模型时会抛出循环依赖异常。有一些解决方案可以避免这种情况，最简单的是在呈现 JSON 视图之前打破双向关系。在`Comment.post`和`Tag.posts`上添加一个`Ignore`属性。

```
//src/Entity/Comment.php
class Comment
{
    #[Ignore]
    private Post $post;
}//src/Entity/Tag.php
class Tag
{
    #[Ignore]
    private Collection $posts;
}
```

# 测试控制器

如前几节所述，为了测试控制器/API，创建一个测试类来扩展`WebTestCase`，它提供了大量处理请求和断言响应的工具。

运行以下命令创建一个测试框架。

```
# php bin/console make:test
```

按照交互步骤在`WebTestCase`上创建一个测试库。

```
class PostControllerTest extends WebTestCase
{
    public function testGetAllPosts(): void
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/posts'); $this->assertResponseIsSuccessful(); //
        $response = $client->getResponse();
        $data = $response->getContent();
        //dump($data);
        $this->assertStringContainsString("Symfony and PHP", $data);
    }}
```

如果您尝试运行测试，它将会失败。目前，没有任何数据可供测试。

# 为测试目的准备数据

`doctrine/doctrine-fixtures-bundle`用于为测试目的填充样本数据，而`dama/doctrine-test-bundle`确保在每次测试运行前恢复数据。

安装`doctrine/doctrine-fixtures-bundle`和`dama/doctrine-test-bundle`。

```
composer require --dev doctrine/doctrine-fixtures-bundle dama/doctrine-test-bundle
```

创造一个新的`Fixture`。

```
# php bin/console make:fixtures
```

在`load`功能中，保存一些测试数据。

```
class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        $data = PostFactory::create("Building Restful APIs with Symfony and PHP 8", "test content");
        $data->addTag(Tag::of( "Symfony"))
            ->addTag( Tag::of("PHP 8"))
            ->addComment(Comment::of("test comment 1"))
            ->addComment(Comment::of("test comment 2")); $manager->persist($data);
        $manager->flush();
    }
}
```

运行命令将示例数据手动加载到数据库中。

```
# php bin/console doctrine:fixtures:load
```

将下面的扩展配置添加到`phpunit.xml.dist`中，这样每次测试运行时，数据都将被清除并重新创建。

```
<extensions>
    <extension class="DAMA\DoctrineTestBundle\PHPUnit\PHPUnitExtension"/>
</extensions>
```

运行以下命令执行`PostControllerTest.php`。

```
# php .\vendor\bin\phpunit .\tests\Controller\PostControllerTest.php
```

# 分页结果

有许多 web 应用程序提供了一个输入字段，用于键入关键字和对搜索结果进行分页。假设请求提供了一个*关键字*来匹配帖子*标题*或*内容*字段，一个*偏移*来设置分页的偏移位置，一个*限制*来设置每页元素的限制大小。在`PostRepository`中创建一个函数，接受一个*关键字*、*偏移*和*限制*作为参数。

```
public function findByKeyword(string $q, int $offset = 0, int $limit = 20): Page
{
    $query = $this->createQueryBuilder("p")
        ->andWhere("p.title like :q or p.content like :q")
        ->setParameter('q', "%" . $q . "%")
        ->orderBy('p.createdAt', 'DESC')
        ->setMaxResults($limit)
        ->setFirstResult($offset)
        ->getQuery(); $paginator = new Paginator($query, $fetchJoinCollection = false);
    $c = count($paginator);
    $content = new ArrayCollection();
    foreach ($paginator as $post) {
        $content->add(PostSummaryDto::of($post->getId(), $post->getTitle()));
    }
    return Page::of ($content, $c, $offset, $limit);
}
```

首先，使用`createQueryBuilder`创建一个动态查询，然后创建一个教条`Paginator`实例来执行查询。`Paginator`实现`Countable`接口，使用`count`获取元素总数。最后，我们使用一个定制的`Page`对象来包装结果。

```
class Page
{
    private Collection $content;
    private int $totalElements;
    private int $offset;
    private int $limit; #[Pure] public function __construct()
    {
        $this->content = new ArrayCollection();
    } public static function of(Collection $content, int $totalElements, int $offset = 0, int $limit = 20): Page
    {
        $page = new Page();
        $page->setContent($content)
            ->setTotalElements($totalElements)
            ->setOffset($offset)
            ->setLimit($limit); return $page;
    }

    //
    //getters}
```

# 自定义 ArgumentResolver

在`PostController`中，让我们改进服务于路线`/posts`的函数，使其接受类似 */posts？q = Symfony&offset = 0&limit = 10*，确保参数可选。

```
#[Route(path: "", name: "all", methods: ["GET"])]
    function all(Request $req): Response
    {
        $keyword = $req->query->get('q')??'';
        $offset = $req->query->get('offset')??0;
        $limit = $req->query->get('limit')??10;

        $data = $this->posts->findByKeyword($keyword, $offset, $limit);
        return $this->json($data);
    }
```

它可以工作，但是查询参数处理看起来有点难看。如果它们可以作为路由路径参数来处理，那就太好了。

我们可以创建一个定制的`ArgumentResolver`来解析绑定的查询参数。

首先创建一个注释/属性类来标识需要由这个`ArgumentResolver`解析的查询参数。

```
#[Attribute(Attribute::TARGET_PARAMETER)]
final class QueryParam
{
    private null|string $name;
    private bool $required; /**
     * @param string|null $name
     * @param bool $required
     */
    public function __construct(?string $name = null, bool $required = false)
    {
        $this->name = $name;
        $this->required = $required;
    }

    //getters and setters

}
```

创建自定义`ArgumentResolver`实现内置`ArgugmentResolverInterface`。

```
class QueryParamValueResolver implements ArgumentValueResolverInterface, LoggerAwareInterface
{
    public function __construct()
    {
    } private LoggerInterface $logger; /**
     * @inheritDoc
     */
    public function resolve(Request $request, ArgumentMetadata $argument)
    {
        $argumentName = $argument->getName();
        $this->logger->info("Found [QueryParam] annotation/attribute on argument '" . $argumentName . "', applying [QueryParamValueResolver]");
        $type = $argument->getType();
        $nullable = $argument->isNullable();
        $this->logger->debug("The method argument type: '" . $type . "' and nullable: '" . $nullable . "'"); //read name property from QueryParam
        $attr = $argument->getAttributes(QueryParam::class)[0];// `QueryParam` is not repeatable
        $this->logger->debug("QueryParam:" . $attr);
        //if name property is not set in `QueryParam`, use the argument name instead.
        $name = $attr->getName() ?? $argumentName;
        $required = $attr->isRequired() ?? false;
        $this->logger->debug("Polished QueryParam values: name='" . $name . "', required='" . $required . "'"); //fetch query name from request
        $value = $request->query->get($name);
        $this->logger->debug("The request query parameter value: '" . $value . "'"); //if default value is set and query param value is not set, use default value instead.
        if (!$value && $argument->hasDefaultValue()) {
            $value = $argument->getDefaultValue();
            $this->logger->debug("After set default value: '" . $value . "'");
        } if ($required && !$value) {
            throw new \InvalidArgumentException("Request query parameter '" . $name . "' is required, but not set.");
        } $this->logger->debug("final resolved value: '" . $value . "'");

        //must return  a `yield` clause
        yield match ($type) {
            'int' => $value ? (int)$value : 0,
            'float' => $value ? (float)$value : .0,
            'bool' => (bool)$value,
            'string' => $value ? (string)$value : ($nullable ? null : ''),
            null => null
        };
    } public function supports(Request $request, ArgumentMetadata $argument): bool
    {
        $attrs = $argument->getAttributes(QueryParam::class);
        return count($attrs) > 0;
    } public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
}
```

运行时，调用`supports`函数检查当前请求是否满足要求，如果满足，则调用`resovle`函数。

在`supports`函数中，我们检查参数是否用`QueryParam`标注，如果存在，则从请求查询字符串中解析参数。

现在将服务于*/post*端点的函数改为如下。

```
#[Route(path: "", name: "all", methods: ["GET"])]
function all(#[QueryParam] $keyword,
    #[QueryParam] int $offset = 0,
    #[QueryParam] int $limit = 20): Response
    {
        $data = $this->posts->findByKeyword($keyword || '', $offset, $limit);
        return $this->json($data);
    }
```

运行应用程序并使用`curl`测试*/post*。

```
# curl [http://localhost:8000/posts](http://localhost:8000/posts)
{
    "content":[
    	{
            "id":"1ec3e1e0-17b3-6ed2-a01c-edecc112b436",
            "title":"Building Restful APIs with Symfony and PHP 8"
        }
    ],
    "totalElements":1,
    "offset":0,
    "limit":20
}
```

# 按 ID 获取帖子

按照上一节的设计，给`PostController`增加另一个功能来绘制路线`/posts/{id}`。

```
class PostController extends AbstractController
{
	//other functions... #[Route(path: "/{id}", name: "byId", methods: ["GET"])]
    function getById(Uuid $id): Response
    {
        $data = $this->posts->findOneBy(["id" => $id]);
        if ($data) {
            return $this->json($data);
        } else {
            return $this->json(["error" => "Post was not found by id:" . $id], 404);
        }
    }
}
```

运行应用程序，尝试访问[*http://localhost:8000/posts/{ id }*](http://localhost:8000/posts/%7Bid%7D)，会抛出这样的异常。

```
App\Controller\PostController::getById(): Argument #1 ($id) must be of type Symfony\Component\Uid\Uuid, string given, cal
led in D:\hantsylabs\symfony5-sample\rest-sample\vendor\symfony\http-kernel\HttpKernel.php on line 156
```

URI 中的`id`是一个字符串，不能直接作为`Uuid`使用。

Symfony 提供了`ParamConverter`来将请求属性转换成目标类型。我们可以创建一个自定义的`ParamConverter`来达到存档的目的。

# 自定义 ParamConverter

在 *src/Request/* 文件夹下创建一个新的类`UuidParamCovnerter`。

```
class UuidParamConverter implements ParamConverterInterface
{
    public function __construct(private LoggerInterface $logger)
    {
    }
 /**
     * @inheritDoc
     */
    public function apply(Request $request, ParamConverter $configuration): bool
    { $param = $configuration->getName(); if (!$request->attributes->has($param)) {
            return false;
        } $value = $request->attributes->get($param);
        $this->logger->info("parameter value:" . $value);
        if (!$value && $configuration->isOptional()) {
            $request->attributes->set($param, null); return true;
        } $data = Uuid::fromString($value);
        $request->attributes->set($param, $data); return true;
    } /**
     * @inheritDoc
     */
    public function supports(ParamConverter $configuration): bool
    {
        $className = $configuration->getClass();
        $this->logger->info("converting to UUID :{c}", ["c" => $className]);
        return $className && $className == Uuid::class;
    }
}
```

在上述代码中，

*   `supports`功能检查执行环境是否符合要求
*   执行转换的`apply`功能。如果`supports`返回 false，将跳过该转换步骤。

# 创建帖子

遵循 REST 约定，定义以下规则来服务一个端点来处理请求。

*   请求匹配 Http 动词/HTTP 方法:`POST`
*   请求匹配路由端点:*/发布*
*   将请求头`Content-Type`值设置为*应用程序/json* ，并使用请求体以 json 格式保存请求数据
*   如果成功，返回一个`CREATED` (201) Http 状态码，并将响应头*位置*值设置为新创建的帖子的 URI。

```
#[Route(path: "", name: "create", methods: ["POST"])]
public function create(Request $request): Response
{
    $data = $this->serializer->deserialize($request->getContent(), CreatePostDto::class, 'json');
    $entity = PostFactory::create($data->getTitle(), $data->getContent());
    $this->posts->getEntityManager()->persist($entity); return $this->json([], 201, ["Location" => "/posts/" . $entity->getId()]);
}
```

`posts->getEntityManager()`覆盖父方法以从父类获得一个`EntityManager`，你也可以直接在`PostController`中注入`ObjectManager`或`EntityManagerInterface`来完成持久化工作。教条`Repository`主要用于建立查询标准和执行定制查询。

在`PostControllerTest`文件中创建一个测试函数进行验证。

```
public function testCreatePost(): void
{
    $client = static::createClient();
    $data = CreatePostDto::of("test title", "test content");
    $crawler = $client->request(
        'POST',
        '/posts',
        [],
        [],
        [],
        $this->getContainer()->get('serializer')->serialize($data, 'json')
    ); $this->assertResponseIsSuccessful(); $response = $client->getResponse();
    $url = $response->headers->get('Location');
    //dump($data);
    $this->assertNotNull($url);
    $this->assertStringStartsWith("/posts/", $url);
}
```

# 转换请求正文

我们还可以使用注释/属性，通过引入自定义的`ArgumentResolver`来删除处理`Request`对象的原始代码。

创建一个`Body` *属性*。

```
#[Attribute(Attribute::TARGET_PARAMETER)]
final class Body
{
}
```

然后创建一个`BodyValueResolver`。

```
class BodyValueResolver implements ArgumentValueResolverInterface, LoggerAwareInterface
{
    public function __construct(private SerializerInterface $serializer)
    {
    } private LoggerInterface $logger; /**
     * @inheritDoc
     */
    public function resolve(Request $request, ArgumentMetadata $argument)
    {
        $type = $argument->getType();
        $this->logger->debug("The argument type:'" . $type . "'");
        $format = $request->getContentType() ?? 'json';
        $this->logger->debug("The request format:'" . $format . "'"); //read request body
        $content = $request->getContent();
        $data = $this->serializer->deserialize($content, $type, $format);
       // $this->logger->debug("deserialized data:{0}", [$data]);
        yield $data;
    } /**
     * @inheritDoc
     */
    public function supports(Request $request, ArgumentMetadata $argument): bool
    {
        $attrs = $argument->getAttributes(Body::class);
        return count($attrs) > 0;
    } public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
```

在`supports`方法中，它只是检测方法参数是否用`Body`属性进行了注释，然后应用`resolve`方法将请求体内容反序列化为类型化对象。

运行应用程序并通过*/post*测试端点。

```
curl -v [http://localhost:8000/posts](http://localhost:8000/posts) -H "Content-Type:application/json" -d "{\"title\":\"test title\",\"content\":\"test content\"}"
> POST /posts HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/7.55.1
> Accept: */*
> Content-Type:application/json
> Content-Length: 47
>
< HTTP/1.1 201 Created
< Cache-Control: no-cache, private
< Content-Type: application/json
< Date: Sun, 21 Nov 2021 08:42:49 GMT
< Location: /posts/1ec4aa70-1b21-6bce-93f8-b39330fe328e
< X-Powered-By: PHP/8.0.10
< X-Robots-Tag: noindex
< Content-Length: 2
<
[]
```

# 异常处理

Symfony 内核提供了一个事件机制来在`Controller`类中引发一个`Exception`，并在你的自定义`EventListener`或`EventSubscriber`中处理它们。

例如，创建一个`PostNotFoundException`。

```
class PostNotFoundException extends \RuntimeException
{ public function __construct(Uuid $uuid)
    {
        parent::__construct("Post #" . $uuid . " was not found");
    }}
```

创建一个 EventListener 来捕获该异常，并按预期处理该异常。

```
class ExceptionListener implements LoggerAwareInterface
{
    private LoggerInterface $logger; public function __construct()
    {
    } public function onKernelException(ExceptionEvent $event)
    {
        // You get the exception object from the received event
        $exception = $event->getThrowable();
        $data = ["error" => $exception->getMessage()]; // Customize your response object to display the exception details
        $response = new JsonResponse($data); // HttpExceptionInterface is a special type of exception that
        // holds status code and header details if ($exception instanceof PostNotFoundException) {
            $response->setStatusCode(Response::HTTP_NOT_FOUND);
        } else if ($exception instanceof HttpExceptionInterface) {
            $response->setStatusCode($exception->getStatusCode());
            $response->headers->replace($exception->getHeaders());
        } else {
            $response->setStatusCode(Response::HTTP_INTERNAL_SERVER_ERROR);
        } // sends the modified response object to the event
        $event->setResponse($response);
    } public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
}
```

将此`ExceptionListener`注册到 *config/service.yml* 文件中。

```
App\EventListener\ExceptionListener:
    tags:
      - { name: kernel.event_listener, event: kernel.exception, priority: 50 }
```

表示将`event.exception`事件绑定到`ExceptionListener`，并设置`priority`来设置执行时的顺序。

运行以下命令显示事件*内核上所有注册的`EventListener` / `EventSubscriber`。*

```
php bin/console debug:event-subscriber kernel.exception
```

将`getById`功能更改如下。

```
#[Route(path: "/{id}", name: "byId", methods: ["GET"])]
function getById(Uuid $id): Response
{
    $data = $this->posts->findOneBy(["id" => $id]);
    if ($data) {
   		return $this->json($data);
    } else {
    	throw new PostNotFoundException($id);
    }
}
```

添加一个测试来验证是否没有找到 post 并获得 404 状态代码。

```
public function testGetANoneExistingPost(): void
{
    $client = static::createClient();
    $id = Uuid::v4();
    $crawler = $client->request('GET', '/posts/' . $id); //
    $response = $client->getResponse();
    $this->assertResponseStatusCodeSame(404);
    $data = $response->getContent();
    $this->assertStringContainsString("Post #" . $id . " was not found", $data);
}
```

再次运行应用程序，并尝试通过一个不存在的 id 访问一篇文章。

```
curl [http://localhost:8000/posts/1ec3e1e0-17b3-6ed2-a01c-edecc112b438](http://localhost:8000/posts/1ec3e1e0-17b3-6ed2-a01c-edecc112b438) -H "Accept: application/json" -v
> GET /posts/1ec3e1e0-17b3-6ed2-a01c-edecc112b438 HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/7.55.1
> Accept: application/json
>
< HTTP/1.1 404 Not Found
< Cache-Control: no-cache, private
< Content-Type: application/json
< Date: Mon, 22 Nov 2021 03:57:51 GMT
< X-Powered-By: PHP/8.0.10
< X-Robots-Tag: noindex
< Content-Length: 69
<
{"error":"Post #1ec3e1e0-17b3-6ed2-a01c-edecc112b438 was not found."}
```

## 从我的 Github 获取[完整的源代码](https://github.com/hantsy/symfony5-sample/tree/master/rest-sample)。