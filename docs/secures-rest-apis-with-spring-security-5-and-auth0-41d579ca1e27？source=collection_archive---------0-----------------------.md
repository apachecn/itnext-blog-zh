# 用 Spring Security 5 和 Auth0 保护 REST APIs

> 原文：<https://itnext.io/secures-rest-apis-with-spring-security-5-and-auth0-41d579ca1e27?source=collection_archive---------0----------------------->

Spring Security 5 弃用了最初的 Spring Security OAuth 模块，并重写了 OAuth2/Oidc 实现，使其成为核心模块的一部分。

![](img/7d6f7bd6189c4719d7ee5ed2bd4376b4.png)

[图片来自 https://unsplash.com/photos/SwbsW4uf1Qs](https://unsplash.com/photos/SwbsW4uf1Qs)

OAuth 协议定义了 4 个角色，包括*资源所有者*、*客户端*、*授权服务器*和*资源服务器*。假设您正在手机中使用一个应用程序，您(*资源所有者*)授权该应用程序(本身呈现一个*客户端*角色)执行一个授权流程，以从授权服务器获取访问令牌，然后利用该令牌与受保护的资源(来自*资源服务器*)进行交互。

Oidc 是对 OAuth2 协议的补充。在 Spring Security 5 中，它没有在 API 级别和配置中明确区分它们。并且现在大多数 OAuth2 兼容提供商支持 Oidc 协议。

> *如果你是 OAuth2 和 OIDC 的新手，请查看官方*[*OAuth 2 docs*](https://oauth.net/2/)*并探索*[*OpenId Connect*](https://openid.net/connect/)*相关规范。对于没有耐心的用户来说，来自 DigitalOcean 的* [*这篇入门文章*](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2) *是一个很好的起点。*

使用 Spring Security 5，很容易在应用程序中设置 OAuth2 *客户端*和*资源服务器*角色。还有[的另一个孵化器项目](https://github.com/spring-projects-experimental/spring-authorization-server)，它允许你服务一个定制的*授权服务器*。在实际应用中，我认为选择成熟的 IDP 服务来简化多种认证服务和高级功能的集成工作是一个明智的选择，例如，您希望集成 Google 帐户、脸书、Twitter 和自定义用户名/密码认证，并在您的应用中启用两阶段认证。Auth0 和 [Okta](https://www.okta.com) 是该领域领先的 IDP 服务，它们为您的应用程序中的所有用例提供出色的管理 UI 和丰富的 SDK。在开源世界中， [Keycloak](https://www.keycloak.org/) 被广泛用于托管定制认证服务器。

> *从 Okta 官方博客来看，Auth0 已经加入了 Okta 大家庭。我有点担心 Auth0 会像 Stormpath 一样消失。但是 Okta 的人宣称 Auth0 将作为一个独立的品牌运营。*

在本文中，我们将使用 Auth0 作为身份验证服务器角色来简化我们的开发过程。

为了演示 OAuth2 流程，首先我们将创建一个 RESTful API *应用程序*，它充当*资源服务器*。在以后的文章中，我们将创建一个 Angular SPA 客户端应用程序来与这个 API 后端进行交互。

官方的 [auth0 docs 页面](https://auth0.com/docs/)上提供了大量的教程。通常你可以在这些文档中找到与你的应用相匹配的用例，去*后端 API* 文档浏览教程，特别是。有一个[春季安全 5](https://auth0.com/docs/quickstart/backend/java-spring-security5) 的大教程。

如果您以前没有，请先注册一个 Auth0 帐户。或者直接使用社交网络账户登录。登录后，您会发现 Auth0 已经为您创建了一个开发目的域，它为您的新应用程序分类所有资源。

在仪表板 UI 中，展开左侧窗格中的*应用程序/API*，让我们创建一个新的 API 应用程序(OAuth2 协议中的*资源服务器*角色)。

*   点击*创建 API* 按钮开始进度。
*   填写必要字段，*受众*字段用于标识该 API，建议填写类似 URL 的值。比如说。这里我创建了一个名为 *Backend API* 的新 API 应用程序，并将受众设置为[*https://hantsy.github.io/api*](https://hantsy.github.io/api)。

Auth0 还提供了一个 *Auth0 管理 API* 供您访问 Auth0 APIs，并且出于测试目的，Auth0 还自动创建了一个*机器对机器*应用程序，允许您与自己的 API 进行交互。

接下来让我们创建我们的后端 API 应用程序。进入[https://start . spring . io](https://start.spring.io)，填写必填字段，选择 *Maven* 作为构建工具，选择 *Java* 作为语言，然后添加以下依赖项，

*   Web Starter
*   确认
*   数据 JPA
*   春天安全
*   龙目岛

点击 Generate 按钮将项目框架生成到一个档案中，下载它并将文件解压到你的磁盘中。将项目导入到您的 IDEA 中，打开 *pom.xml* 文件，添加 spring security oauth2 相关的依赖项。

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-jose</artifactId>
</dependency>
```

在这篇文章中，我们将把 Auth0 与 Spring Security 集成起来，以保护 RESTful APIs，并且我们将关注 API 应用程序的安全配置。为了简化开发工作，我从[hantsy/spring-web MVC-functional-sample](https://github.com/hantsy/spring-webmvc-functional-sample)复制了现有代码，用于为一个简单的博客应用程序生成 RESTful APIs。

> *关于 Spring WebMvc 函数式编程的介绍，查看* [*我在 Medium 上发表的帖子*](https://hantsy.medium.com/functional-programming-in-spring-mvc-20957653da1) *。*

声明一个`SecurityFilterChain` bean 来配置 Spring 安全性的细节。

```
@Bean
SecurityFilterChain springWebFilterChain(HttpSecurity http) throws Exception {
    return http
        .httpBasic(AbstractHttpConfigurer::disable)
        .csrf(AbstractHttpConfigurer::disable)
        .sessionManagement(c -> c.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeRequests(c -> c
                           .antMatchers("/", "/info").permitAll()
                           .antMatchers(HttpMethod.GET, "/posts/**").permitAll()//.hasAuthority("SCOPE_read:posts")
                           .antMatchers(HttpMethod.POST, "/posts/**").hasAuthority("SCOPE_write:posts")
                           .antMatchers(HttpMethod.PUT, "/posts/**").hasAuthority("SCOPE_write:posts")
                           .antMatchers(HttpMethod.DELETE, "/posts/**").hasAuthority("SCOPE_delete:posts")
                           .anyRequest().authenticated()
                          )
        .oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt)
        .build();
}
```

1.  `oauth2ResourceServer`表示这个应用程序将作为*资源服务器*的角色，它使用一个`JWT`令牌来验证传入的请求。
2.  `authorizeRequests`部分定义了针对 HTTP URI 和动词的安全规则，例如，创建新帖子时需要一个`write:posts`权限。

与*不透明的*令牌不同，JWT 令牌可以简单地通过授权服务器提供的本地公钥或 JWK 集合 uri 进行验证。

在 Spring Security 5 中，如果授权提供者实现了 OpenId 配置规范，并公开了整个 OAuth2/Oidc 配置元数据，如令牌 url、授权服务器 url、jwk 集合 uri、用户信息 uri 等，则可以从已配置的*issuer _ uri*中解码出 JWT 令牌。通过一个网址`.well-known/openid-configuration`。

在 *application.properties* 中添加以下配置。

```
spring:
  ...
  security:
    oauth2:
      resourceserver:
        jwt:
          # Replace with the domain of your Auth0 tenant.
          # Note the trailing slash is important!
          issuer-uri: [https://dev-ese8241b.us.auth0.com/](https://dev-ese8241b.us.auth0.com/)
```

若要检查访问群体是否已设置并包含在访问令牌中，请添加一个自定义的访问群体验证程序。

```
@RequiredArgsConstructor(staticName = "of")
public class AudienceValidator implements OAuth2TokenValidator<Jwt> {
    private final String audience;

    public OAuth2TokenValidatorResult validate(Jwt jwt) {
        OAuth2Error error = new OAuth2Error("invalid_token", "The required audience is missing", null);

        if (jwt.getAudience().contains(audience)) {
            return OAuth2TokenValidatorResult.success();
        }

        return OAuth2TokenValidatorResult.failure(error);
    }
}
```

并将这个验证器添加到`JwtDecoder` bean 中。

```
@Bean
JwtDecoder jwtDecoder(OAuth2ResourceServerProperties properties, @Value("${auth0.audience}") String audience) {
    /*
        By default, Spring Security does not validate the "aud" claim of the token, to ensure that this token is
        indeed intended for our app. Adding our own validator is easy to do:
        */ String issuerUri = properties.getJwt().getIssuerUri();
    NimbusJwtDecoder jwtDecoder = (NimbusJwtDecoder) JwtDecoders.fromOidcIssuerLocation(issuerUri); OAuth2TokenValidator<Jwt> audienceValidator = AudienceValidator.of(audience);
    OAuth2TokenValidator<Jwt> withIssuer = JwtValidators.createDefaultWithIssuer(issuerUri);
    OAuth2TokenValidator<Jwt> withAudience = new DelegatingOAuth2TokenValidator<>(withIssuer, audienceValidator); jwtDecoder.setJwtValidator(withAudience); return jwtDecoder;
}
```

接下来，让我们在 Auth0 管理仪表板中添加一些权限配置。

进入*后端 API* 的*设置*页面，点击*权限*选项卡，添加以下权限。

*   *阅读:帖子* —阅读帖子
*   *写:帖子* —创建和更新帖子
*   *删除:帖子* —删除帖子

切换到*机器对机器应用*选项卡，确保*测试应用*已启用，并展开该应用的配置，确保所有权限已勾选，点击**更新**按钮以应用更改。

现在切换到*测试*标签，有一个使用令牌测试 API 应用程序的指南，它在不同的脚本中可用。只需复制 **cURL** 命令，打开一个 Unix/bash 之类的终端，粘贴执行即可。

```
$ curl --request POST \
>   --url [https://dev-ese8241b.us.auth0.com/oauth/token](https://dev-ese8241b.us.auth0.com/oauth/token) \
>   --header 'content-type: application/json' \
>   --data '{"client_id":"IEXV2MbAiuEkV0J7ufH0Bqq2a2YIF3h1","client_secret":"OknFJUkFpBe_tpFPAP2hL10NliMbCGxMRiFH_DkTxylBnZ5WjP9aq6zQOtL2L7ea","audience":"https://hantsy.github.io/api","grant_type":"client_credentials"}'
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IlYzM1lvNzk5cC1XeFI2NHpJZ29QMyJ9.eyJpc3MiOiJodHRwczovL2Rldi1lc2U4MjQxYi51cy5hdXRoMC5jb20vIiwic3ViIjoiSUVYVjJNYkFpdUVrVjBKN3VmSDBCcXEyYTJZSUYzaDFAY2xpZW50cyIsImF1ZCI6Imh0dHBzOi8vaGFudHN5LmdpdGh1Yi5pby9hcGkiLCJpYXQiOjE2MjA1NDgzMjAsImV4cCI6MTYyMDYzNDcyMCwiYXpwIjoiSUVYVjJNYkFpdUVrVjBKN3VmSDBCcXEyYTJZSUYzaDEiLCJzY29wZSI6InJlYWQ6cG9zdHMgd3JpdGU6cG9zdHMgZGVsZXRlOnBvc3RzIiwiZ3R5IjoiY2xpZW50LWNyZWRlbnRpYWxzIn0.UCdmcIb8xREC_rQp3EVBx8EALzhPcpJL1Mh1n6UX-Sf2cHkSsiUFISkHHM-qa5qZ34aNxJwKfvKPnZsfcIH9xwaGMeZjHub22PwAyVscQpZBq2mPRe_28GiwMP9jtFmSPhuGQec6-K7f0410tZTEKggnX3qFDdsJBWEhnu4zeFSwn865--Lbgk52owdncMd8jp2O5olKdnkeieChGQaYsLQ_vM8JTDN-eu5hm1DEL8WcfCd_foIG6klcm_gzgHpAru6zJQb48OmWIxlL6VGQl7RqIQIfds2NBBG7xYVr_38M2cltx8EUyIcXedR-d4YG_Dfp-NJbFkpZcSs9PPTagw","scope":"read:posts write:posts delete:posts","expires_in":86400,"token_type":"Bearer"}
```

复制到 access_token，打开 [https://jwt.io](https://jwt.io) 页面，将 token 粘贴到*编码*框中，可以看到解码结果。

![](img/9f2a7e850b508148e72fb03c295ed3be.png)

现在让我们使用这个令牌来访问我们的*后端 API* 应用程序中受保护的 API。

首先启动一个 Postgres 服务器。只需运行下面的 docker 命令。

```
$ docker compose up postgres
```

然后运行应用程序。

```
$ mvn clean spring-boot:run
// or 
$ mvn clean package
$ java -jar ./target/xxx.jar
```

尝试创建一个新帖子。

```
$ curl --request POST \
  --url [http://localhost:8080/posts](http://localhost:8080/posts) \
  --header 'authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IlYzM1lvNzk5cC1XeFI2NHpJZ29QMyJ9.eyJpc3MiOiJodHRwczovL2Rldi1lc2U4MjQxYi51cy5hdXRoMC5jb20vIiwic3ViIjoiSUVYVjJNYkFpdUVrVjBKN3VmSDBCcXEyYTJZSUYzaDFAY2xpZW50cyIsImF1ZCI6Imh0dHBzOi8vaGFudHN5LmdpdGh1Yi5pby9hcGkiLCJpYXQiOjE2MjA1NDgwNzEsImV4cCI6MTYyMDYzNDQ3MSwiYXpwIjoiSUVYVjJNYkFpdUVrVjBKN3VmSDBCcXEyYTJZSUYzaDEiLCJzY29wZSI6InJlYWQ6cG9zdHMgd3JpdGU6cG9zdHMgZGVsZXRlOnBvc3RzIiwiZ3R5IjoiY2xpZW50LWNyZWRlbnRpYWxzIn0.Rg6UqIb2kjrPg39QRLOLsvI54TUsQM4B7j_6Eb4hlxFkzHWglD-V0b5kXfgSMIcBQ-fq5oLt7Fj8etgqMMpstyx-2Ob20rbCBd-msZ1o-xGQidkQsyEM6lL6GMX7d79Vw0H9eZN8D8j8_mVoqd-wENhWsBiSp3xq8Jz6KSdslrxGnMaQKbQ2IL7dgw8Y1kJBz8FgBcUUF5k-dhMB-DDLkaLsurwcb380Qk3gMD2JW97fxQWM_mi38ufcoopSWxznDE118ng7hQXT18vFB9N_RCNaPHslsKZsE6ytiwx6kiSYvGFN5RMWIa3Dyp6Rt4UjfTLx1WO9JDtxNc4NMSPQ-g' \
  --header 'Content-Type: application/json' \
  --data '{"title": "New Post about Spring Secrity and Auth0", "content": "Content of this post, TBD"}'
```

然后列出所有帖子，检查帖子是否按预期创建。

```
$ curl [http://localhost:8080/posts](http://localhost:8080/posts)
[ {
  "id" : 1,
  "version" : 0,
  "createdDate" : "2021-05-09T16:37:11.984273",
  "lastModifiedDate" : "2021-05-09T16:37:11.984273",
  "title" : "Post one",
  "content" : "content of Post one"
}, {
  "id" : 2,
  "version" : 0,
  "createdDate" : "2021-05-09T16:37:12.068004",
  "lastModifiedDate" : "2021-05-09T16:37:12.068004",
  "title" : "Post two",
  "content" : "content of Post two"
}, {
  "id" : 3,
  "version" : 0,
  "createdDate" : "2021-05-09T16:38:27.881024",
  "lastModifiedDate" : "2021-05-09T16:38:27.881024",
  "createdBy" : {
    "username" : "IEXV2MbAiuEkV0J7ufH0Bqq2a2YIF3h1@clients"
  },
  "lastModifiedBy" : {
    "username" : "IEXV2MbAiuEkV0J7ufH0Bqq2a2YIF3h1@clients"
  },
  "title" : "New Post about Spring Secrity and Auth0",
  "content" : "Content of this post, TBD"
} ]
```

*后端 API* 已准备好供客户端应用程序访问。现在让我们探索一下 Spring 安全测试模块中提供的测试工具，以测试受 JWT 令牌保护的 API。

Spring Security 提供了一个简单的`SecurityMockMvcRequestPostProcessors.jwt`来模仿 JWT 处理，并在 Spring MockMvc 环境中绕过 JWT 解码。

```
@Test
public void testSave() throws Exception { given(this.posts.save(any(Post.class)))
        .willReturn(Post.builder().title("test").content("test content").build()); this.mockMvc
        .perform(
        post("/posts")
        .with(jwt().jwt(jwtBuilder().claim("scope", "write:posts").build()))
        .content(this.objectMapper.writeValueAsBytes(Post.builder().title("test").content("test content").build()))
        .contentType(MediaType.APPLICATION_JSON)
    )
        .andExpect(status().isCreated()); verify(this.posts, times(1)).save(any(Post.class));
    verifyNoMoreInteractions(this.posts);
}private Jwt.Builder jwtBuilder() {
    return Jwt.withTokenValue("token").header("alg", "none").audience(List.of(audience));
}
```

在集成测试中，您可以使用从*测试应用程序*生成的相同访问令牌来与*后端 API* 交互。将令牌添加到 HTTP 头中以访问 API，可以放心地编写下面的测试代码。

```
@SpringBootTest(webEnvironment = RANDOM_PORT)
@Slf4j
public class IntegrationTests { @LocalServerPort
    private int port; @Autowired
    ObjectMapper objectMapper; @Value("${auth0.audience}")
    private String audience; private String token = "...";

    @BeforeEach
    public void setup() {
        RestAssured.port = this.port;
    } @Test
    public void createPostWithoutToken() throws Exception {
        //@formatter:off
        given()
            .body(Post.builder().title("test").content("test content").build())
            .contentType(ContentType.JSON)
        .when()
            .post("/posts")
        .then()
            .assertThat()
            .statusCode(HttpStatus.SC_UNAUTHORIZED);
        //@formatter:on
    } @Test
    public void createPost() throws Exception {
        //@formatter:off
        given()
            .auth().oauth2(token)
            .body(Post.builder().title("test").content("test content").build())
            .contentType(ContentType.JSON)
        .when()
            .post("/posts")
        .then()
            .assertThat()
            .statusCode(HttpStatus.SC_CREATED)
            .header("Location", notNullValue());
        //@formatter:on
    }}
```

## **从** [**my Github**](https://github.com/hantsy/spring-security-auth0-sample/tree/master/api) **中抓取一份源代码。**