---
title: Reference to Spring Security
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-02-07 12:39:19
thumbnail:
---



#### Http Basic 认证

http Basic 认证是最简单的一种认证模式。这种认证模式会将用户名密码用 base64 加密，放在请求头里，向这样：

![20210207124543](D:\Dev\pic\20210202\20210207124543.png)

因为 base64 加密是可逆的，所以可以解密出密码，所以这种认证是比较简陋的。

spring security 支持 http basic 认证，首先加入 spring security 依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

然后加上 spring security 的配置即可。

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.httpBasic()
                .and()
                .authorizeRequests().anyRequest().authenticated();
    }
}
```

#### 使用自定义密码

配置好 spring security 后启动项目，会在控制台打印出密码。

```
Using generated security password: 8362705a-cb9f-4257-8df5-6a1457b7e2bd
```

默认用户名为 user ，使用 user/8362705a-cb9f-4257-8df5-6a1457b7e2bd 即可登录。

如果想自定义用户名密码，可以使用配置：

```yml
spring:
  security:
    user:
      name: john
      password: 123
```

配置生效，控制台就不会在输出密码。

#### PasswordEncoder

PasswordEncoder 是 spring security 定义一个密码校验接口。

```java
public interface PasswordEncoder {
    String encode(CharSequence var1); // 加密

    boolean matches(CharSequence var1, String var2); // 判断密码和加密后的字符串是否匹配

    default boolean upgradeEncoding(String encodedPassword) {
    	// 密码是否需要更新
        return false;
    }
}
```

spring security 推荐使用 BCryptPasswordEncoder 。使用 BCryptPasswordEncoder 生成的加密字符串长度60 位，并且每次都不一样。

```properties
password=123
encodePassword=$2a$10$8tfTy2EHkmAewpuwCnVwxemZFvO0DdUMH1GWvjofhEOMJ.LptcSQG
encodePassword=$2a$10$wsmI8tfkvCc6EWbnexFj6OshzrM2DkZCv3OjalpGvyPgEefdmO9L6
```

这两个都是 123 加密后的字符串。BCryptPasswordEncoder 生成的加密字符串包含 4 部分：

```
$2a 								// BCrypt 算法版本 3位
$10 								// 算法强度 3位
$8tfTy2EHkmAewpuwCnVwxe 			// 随机盐 22位
mZFvO0DdUMH1GWvjofhEOMJ.LptcSQG 	// hash 31位
```

BCrypt 加密的过程是先获得一个随机盐，然后做 10 次加盐哈希，然后按照格式将版本、次数、随机盐和 hash 拼起来得到最终的字符串。由于 BCrypt 不可逆，所以无法 decode ，只能用加密串中的盐对原密码再进行 BCrypt 加密，比较加密后的字符串是否相等。

#### FormLogin 认证

和 http basic 的配置类似，formLogin 认证也需要配置 configure 方法。

```java
protected void configure(HttpSecurity http) throws Exception {
    http.csrf().disable()
    .formLogin()
    .loginPage("/login.html")
    .loginProcessingUrl("/login")
    .and()
    .authorizeRequests()
    .antMatchers("/", "/orders", "/products").hasAnyAuthority("ROLE_user", "ROLE_admin")
    .antMatchers("/sys/**").hasAnyRole("admin")
    .and()
    .authorizeRequests().anyRequest().authenticated();
}
```

这了我们加了角色和权限的控制，对应可以配合测试数据进行验证。

```java
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
            .withUser("John")
            .password(passwordEncoder().encode("123"))
            .roles("user")
            .and()
            .withUser("admin")
            .password(passwordEncoder().encode("123"))
            .roles("admin")
            .and()
            .passwordEncoder(passwordEncoder());
}

@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

#### 自定义登录成功/失败处理

spring security 提供了默认的登录成功/失败处理，我们可以自定义返回 json 数据。

自定义两个处理类实现 AuthenticationSuccessHandler 和 AuthenticationFailureHandler 接口。

```java
@Component
public class CustomAuthenticationSuccessHandler extends SavedRequestAwareAuthenticationSuccessHandler {

    private static final String LOGIN_RETURN_TYPE = "json";
    @Value("${spring.security.login-return-type}")
    private String returnType;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws ServletException, IOException {
        if (LOGIN_RETURN_TYPE.equals(returnType)) {
            response.setContentType(MediaType.APPLICATION_PROBLEM_JSON_VALUE);
            response.setCharacterEncoding("UTF-8");
            response.getWriter().write(new ObjectMapper().writeValueAsString(Response.success()));
        } else {
            super.onAuthenticationSuccess(request, response, authentication);
        }

    }
}
```

SavedRequestAwareAuthenticationSuccessHandler 是 AuthenticationSuccessHandler 的实现类，提供保存登录前页面的功能，我们可以直接继承。

```java
@Component
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    private static final String LOGIN_RETURN_TYPE = "json";
    @Value("${spring.security.login-return-type}")
    private String returnType;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        if (LOGIN_RETURN_TYPE.equals(returnType)) {
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.setCharacterEncoding("UTF-8");
            response.getWriter().write(new ObjectMapper().writeValueAsString(Response.failure(400, "用户名或密码错误")));
        }else {
            super.onAuthenticationFailure(request, response, exception);
        }
    }
}
```

失败处理也是一样，继承 SimpleUrlAuthenticationFailureHandler ，它提供了登录失败跳转到登录页面的功能。

第二步，在配置文件中添加 successHandler 和 failureHandler 的配置。

```java
http.formLogin()
    .successHandler(customAuthenticationSuccessHandler)
    .failureHandler(customAuthenticationFailureHandler)
```

#### 自定义 403

```java
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    private static final String LOGIN_RETURN_TYPE = "json";
    @Value("${spring.security.login-return-type}")
    private String returnType;

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        if (LOGIN_RETURN_TYPE.equals(returnType)) {
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.setCharacterEncoding("UTF-8");
            response.getWriter().write(new ObjectMapper().writeValueAsString(Response.failure(403, "权限不足")));
        }else{
            response.sendRedirect("/403.html");
        }
    }
}
```

在 configure 配置

```java
http.exceptionHandling()
    .accessDeniedHandler(customAccessDeniedHandler);
```



#### Session 管理

spring security 的 session 管理可以通过 sessionManagement() 进行配置。

```java
http.sessionManagement()
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
```

##### Session 创建策略 

有 4 种 session 创建策略：

1. always：没有 session 就创建 session 。
2. ifRequired：需要时创建 session 。
3. never： 不主动创建 session ，如果有 session 就使用 。
4. statless：不创建和使用任何 session 。

##### Session 超时配置

```properties
server.servlet.session.timeout=10m
```

##### Session 保护

```java
http.sessionManagement()
    .sessionFixation()
    .migrateSession();
```

migrateSession: 默认方式，对同一个 SESSIONID，每次登录都会创建一个新的 SESSIONID ，旧的 SESSIONID 失效。

none：原 SESSIONID 保留。

newSession：创建一个新的 Session 。

##### 防止脚本窃取 session

```properties
server.servlet.session.cookie.http-only=true
```

##### 限制登录用户数量

```
http.maximumSessions(1) // 最大1个session用户登录
    .maxSessionsPreventsLogin(false)// false：后登录的把先登录的踢掉，true：后登录不能登录
    .expiredSessionStrategy(new CustomExpiredSessionStrategy());
```

#### 动态认证和授权

认证和授权信息保存在数据库中，通过查询数据库获取用户的认证和授权信息。

##### 动态认证

UsernamePasswordAuthenticationFilter 先获取用户名密码，生成一个 UsernamePasswordAuthenticationToken 对象，交给 AuthenticationManager 进行认证，我们需要从数据库中获取认证信息，会交由 DaoAuthenticationProvider 处理。在 DaoAuthenticationProvider  中会调用UserDetailsService 的 loadUserByUsername() 方法获得 UserDetails 对象。认证通过后会创建一个 SuccessAuthentication 。按照这个处理过程我们就需要定义一个 UserDetailsService 的实现类，实现 loadUserByUsername() 方法，来查询我们的数据库，并最终返回一个 UserDetails 对象。

```java
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    SysUser sysUser = getUserByName(username);
    if (null == sysUser) {
        throw new UsernameNotFoundException(username);
    }
    List<GeneralGrantedAuthority> authorities = new ArrayList<>();
    for (SysRole role : sysUser.getRoles()) {
        authorities.add(new GeneralGrantedAuthority(role.getCode(), role.getPermissions()));
    }
    return new org.springframework.security.core.userdetails.User(sysUser.getUsername(),
            sysUser.getPassword(),
            sysUser.getState() == 1,
            sysUser.getAccountExpired() == 0,
            sysUser.getAccountExpired() == 0,
            sysUser.getAccountLocked() == 0,
            authorities);
}
```

定义了实现类，还需要将我们自己的实现类注册到 configure 。

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userService).passwordEncoder(passwordEncoder());
}
```

这样，就实现了使用数据库数据进行认证。

##### 动态授权

```java
public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
    Object principal = authentication.getPrincipal();
    boolean hasPermission = false;
    if (principal instanceof UserDetails) {
        for (GrantedAuthority p : ((UserDetails) principal).getAuthorities()) {
            List<SysPermission> permissions = ((GeneralGrantedAuthority) p).getPermissions();
            for (SysPermission permission : permissions) {
                if (antPathMatcher.match(permission.getUrl(), request.getRequestURI())) {
                    hasPermission = true;
                    break;
                }
            }
        }
    }
    return hasPermission;
}
```

然后在 configure 中加入处理规则。

```java
http.authorizeRequests()
	.anyRequest().access("@permissionServiceImpl.hasPermission(request, authentication)")
```

#### RememberMe

```java
http.rememberMe()
	.rememberMeParameter("remember-me") // form表单中的参数名称
	.rememberMeCookieName("remember-me-cookie") // 浏览器中的cookie name
	.tokenValiditySeconds(60 * 60 * 24);
```

这样 rememberMe 的 token 信息是保存在内存中，如果要持久化到数据库，需要加上 

```java
.tokenRepository(persistentTokenRepository());

@Bean
public PersistentTokenRepository persistentTokenRepository(){
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    return jdbcTokenRepository;
}
```

#### 登出

```java
http.logout()
    .logoutUrl("/logout")
    .logoutSuccessUrl("/login.html")
    .logoutSuccessHandler(customLogoutSuccessHandler);
```

logout 会清空 Session 信息、ContextHolder 中的信息、以及 rememberMe 的信息。

#### 集成 JWT

将 TokenStore 换成 JwtTokenStore 。

```java
@Bean
TokenStore tokenStore() {
    return new JwtTokenStore(jwtAccessTokenConverter());
}
```

将 spring security Token 转换成 JWT Token 。

```java
@Bean 
JwtAccessTokenConverter jwtAccessTokenConverter() {
    JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
    jwtAccessTokenConverter.setSigningKey(jwtSecret);
    return jwtAccessTokenConverter;
}
```



#### 跨域访问

常用方式：

1. 使用 html 特殊标签，如 script、link、img、iframe 。

2. 使用代理。

3. CORS ，跨站资源共享。

   Access-Control-Allow-Origin ，允许哪些 ip 或域名可以跨域访问

   Access-Control-Max-Age ，表示在多少秒内不需要重复校验请求的跨域访问权限

   Access-Control-Allow-Methods ，表示允许跨域请求的 HTTP 方法

   Access-Control-Allow-Headers ，表示访问请求中允许携带哪些 Header 信息，如 Accept、Content-Type 等

spring security 中 CORS 配置：

```java
http.cors();

@Bean
CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration corsConfiguration = new CorsConfiguration();
    corsConfiguration.setAllowedOrigins(Arrays.asList("http://localhost:8080"));
    corsConfiguration.setAllowedMethods(Arrays.asList("GET", "POST"));
    corsConfiguration.applyPermitDefaultValues();
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", corsConfiguration);
    return source;
}
```

#### CSRF

跨站请求伪造。只放 POST、PUT、DELETE 等请求，不防 GET 请求。spring security 会返回一个 CSRF token ，需要在请求的 header 中加上 X-XSRF-TOKEN ，才能正常响应。

```java
http.csrf()
	.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
	.ignoringAntMatchers("/login");
```



#### 认证流程

我们已经介绍了 Http Basic 认证和 FormLogin 认证，我们来看下 spring security 的认证的处理过程是怎样的。

spring security 的认证都是通过过滤器实现的。启动 spring security 项目之后，在控制台会输出过滤器链的信息：

```java
org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter
org.springframework.security.web.context.SecurityContextPersistenceFilter
org.springframework.security.web.header.HeaderWriterFilter
org.springframework.security.web.authentication.logout.LogoutFilter
org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter
org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter
org.springframework.security.web.savedrequest.RequestCacheAwareFilter
org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter
org.springframework.security.web.authentication.AnonymousAuthenticationFilter
org.springframework.security.web.session.SessionManagementFilter
org.springframework.security.web.access.ExceptionTranslationFilter
org.springframework.security.web.access.intercept.FilterSecurityInterceptor
```

