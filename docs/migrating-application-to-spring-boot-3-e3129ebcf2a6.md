# 将应用程序迁移到 Spring Boot 3

> 原文：<https://itnext.io/migrating-application-to-spring-boot-3-e3129ebcf2a6?source=collection_archive---------0----------------------->

上周标志着 Spring Boot 新的主要版本的发布。

我通常倾向于在物理条件允许的情况下尽快将我的应用程序迁移到新版本，但总是害怕迁移。这从来都不是直截了当的，尤其是当涉及到开源库或框架的时候。

![](img/3f5419966df56898f160df0b29737a7b.png)

所以，我花了上个周末的时间试图将一个相对简单的 Spring Boot 2.x 网络服务迁移到 Spring Boot 3.x

所讨论的 web 服务是一个相当标准的应用程序，它使用 Spring JPA 与后端通信，JPA Security 授权请求，并使用 Rest 和 Websockets 与客户端通信。

# 什么进展顺利

官方的迁移指南在周末还没有发布，所以我主要使用[这篇博客文章](https://spring.io/blog/2022/05/24/preparing-for-spring-boot-3-0)作为指南以及 Spring 框架文档和谷歌搜索来尝试和解决任何迁移问题。

好消息是切换到新版本就像改变 org.springframework.boot gradle 插件的版本一样简单。

不出所料，由于切换到 Jakarta EE9，这立即导致了许多编译错误。

通过使用 IDE 中的全局搜索和替换功能，替换以下软件包，相对容易地修复了此问题:

*   javax.servlet -> jakarta.servlet
*   javax . persistence-> Jakarta . persistence
*   javax . annotations-> Jakarta . annotations
*   javax.transaction ->雅加达. transaction

我没有使用遗留属性文件，所以在这方面没有问题。

# 什么不太顺利

我遇到的第一个问题与 WebSecurityConfigurerAdapter 的删除有关。在 Spring Security 5 中，一个或多个安全配置类通常会继承 WebSecurityConfigurerAdapter 并覆盖 configure 方法。

在 Spring Security 6 中，它现在是一个以 HttpSecurity 对象作为参数并返回 SecurityFilterChain 对象的 Bean。

这是一个相对简单的改变，而且，它大大简化了事情:以前，如果我想为应用程序的不同部分使用多种安全机制，我必须为每个配置创建许多静态类。现在，它只是主 Web 安全配置类中的一些 beans。

一个变化需要修改源代码:在 Spring Security 5 中，我们使用了。authorizeRequest 对象的 antMatchers 方法。在 Spring Security 6 中，它被重命名为. requestMatchers。否则，一切都和以前一样，只是这个方法现在必须有一个调用，通过调用 http.build()从 http 参数创建 SecurityFilterChain 对象。

总而言之，我们修改了以下 web 安全配置

```
 @Configuration
 public static class WebSecurityConfig extends WebSecurityConfigurerAdapter {
  public WebSecurityConfig() {
   super(false);
  }

  @Override
  protected void configure(HttpSecurity http) throws Exception {
   logger.info("Configure application security******************************************************");
   http.cors()
     .and()
     .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
     .and()
     .antMatcher("/ws/**")
     .authorizeRequests(autorizeRequests -> autorizeRequests
       .antMatchers(HttpMethod.GET, "/ws/healthz", "/ws/ready", "/ws/version").permitAll()
       .antMatchers(HttpMethod.GET,
         "/ws/user/*",
         "/ws/user/avatar/*",
         "/ws/user/search").hasAnyAuthority("SCOPE_tmt:user")
       .antMatchers(HttpMethod.POST,
         "/ws/friend",
         "/ws/user/trip",
         "/ws/trip/*").hasAnyAuthority("SCOPE_tmt:user")
     ).oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt);
   http.csrf().disable();
   http.headers().frameOptions().disable();
  }
 }
```

变成了下面的一个:

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {
...
 @Bean
 @Order(1)
 public SecurityFilterChain auth0FilterChain(HttpSecurity http) throws Exception {
  logger.info("Configure application security******************************************************");
  http.cors()
    .and()
    .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
    .and()
    .securityMatcher("/ws/**")
    .authorizeHttpRequests(autorizeRequests -> autorizeRequests
      .requestMatchers(HttpMethod.GET, "/ws/healthz", "/ws/ready", "/ws/version").permitAll()
      .requestMatchers(HttpMethod.GET,
        "/ws/user/**",
        "/ws/user/avatar/*",
        "/ws/user/search").hasAnyAuthority("SCOPE_tmt:user")
      .requestMatchers(HttpMethod.POST,
        "/ws/friend",
        "/ws/user/trip",
        "/ws/trip/*").hasAnyAuthority("SCOPE_tmt:user")
    ).oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt);
  http.csrf().disable();
  http.headers().frameOptions().disable();

  return http.build();
 }
...
}
```

在处理网络安全变化时，最大的惊喜是它变得更加严格。因此，在以前的版本中，我可以定义以下规则:

```
.requestMatchers(HttpMethod.GET,
      "/ws/user/*",
```

它将处理/ws/user 下的所有请求，例如/ws/user/1 或/ws/user/1/details。

切换到 Spring Security 6 后，以前的配置不再有效。为了正确处理请求，它必须如下:

```
.requestMatchers(HttpMethod.GET,
      "/ws/user/**",
```

# 结论

我个人感到惊喜的是，这是一次相当容易的经历。显然，这是一个相对简单的项目，更复杂的项目的移植可能不太容易，但总的来说，我喜欢 Spring Framework 的主要版本几乎没有突破性的变化。