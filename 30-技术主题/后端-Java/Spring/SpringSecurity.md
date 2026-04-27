# SpringSecurity


这是一个Spring的重量级安全框架，主要有两个作用：



1. 用户认证：用户是否能够登录
2. 用户授权：用户是否拥有某些权限做一些事情



Apache下的Shiro也是一个安全框架，不过它是轻量级的。它不局限于web开发，在web开发中需要手动编写代码进行配置。



常见安全管理技术栈的组合：



+ SSM + Shiro
+ SpringBoot/SpringCloud + SpringSecurity



## SpringSecurity基本原理


SpringSecurity的本质是一个过滤器链



## 过滤器链


1. WebAsyncManagerIntegrationFilter：将Security上下文与Spring Web中用于处理异步请求映射的 WebAsyncManager 进行集成。
2. SecurityContextPersistenceFilter：在每次请求处理之前将该请求相关的安全上下文信息加载到SecurityContextHolder中，然后在该次请求处理完成之后，将SecurityContextHolder中关于这次请求的信息存储到一个“仓储”中，然后将SecurityContextHolder中的信息清除  
例如: 在Session中维护一个用户的安全信息就是这个过滤器处理的。
3. HeaderWriterFilter：用于将头信息加入响应中
4. CsrfFilter：用于处理跨站请求伪造
5. LogoutFilter：用于处理退出登录
6. UsernamePasswordAuthenticationFilter：用于处理基于表单的登录请求，从表单中获取用户名和密码。默认情况下处理来自“/login”的请求。从表单中获取用户名和密码时，默认使用的表单name值为“username”和“password”，这两个值可以通过设置这个过滤器的usernameParameter 和 passwordParameter 两个参数的值进行修改。
7. DefaultLoginPageGeneratingFilter：如果没有配置登录页面，那系统初始化时就会配置这个过滤器，并且用于在需要进行登录时生成一个登录表单页面。
8. BasicAuthenticationFilter：检测和处理http basic认证
9. RequestCacheAwareFilter：用来处理请求的缓存
10. SecurityContextHolderAwareRequestFilter：主要是包装请求对象request
11. AnonymousAuthenticationFilter：检测SecurityContextHolder中是否存在Authentication对象，如果不存在为其提供一个匿名Authentication
12. SessionManagementFilter：管理session的过滤器
13. ExceptionTranslationFilter：处理 AccessDeniedException 和 AuthenticationException 异常
14. FilterSecurityInterceptor：可以看做过滤器链的出口
15. RememberMeAuthenticationFilter：当用户没有登录而直接访问资源时, 从cookie里找出用户的信息, 如果Spring Security能够识别出用户提供的remember me cookie, 用户将不必填写用户名和密码, 而是直接登录进入系统，该过滤器默认不开启。



### 基本认证流程


1. 先是一个请求带着身份信息进来
2. 经过`AuthenticationManager`的认证，
3. 再通过`SecurityContextHolder`获取`SecurityContext`，
4. 最后将认证后的信息放入到`SecurityContext`。



## 1、简单介绍其中三个过滤器


1. FilterSecurityInterceptor：一个方法级的权限过滤器，位于过滤器链的末尾（最底层）。
2. ExceptionTranslationFilter：一个异常过滤器，用来处理在认证授权过程中抛出的异常。
3. UsernamePasswordAuthenticationFilter：对"/login"的POST请求进行拦截，校验表单中的用户名和密码。



## 2、过滤器加载过程






DelegatingFilterProxy是一个重要的类，从这个类开始配置了过滤器链的各个配置项



+ UsernamePasswordAuthenticationFilter：负责处理用户登录请求
+ ExceptionTranslationFilter：负责处理过滤器链中抛出的任何AcessDeniedException和AuthenticationException
+ FilterSecurityInterceptor：负责权限校验



### 认证流程


1. 登录表单提交的用户名和密码会被**UsernamePasswordAuthticationFilter**封装成**Authentication对象**
2. 中间会调用一系列的认证方法，最终会调用**UserDetailsService接口实现类的loadUserByUsername(username)**来获取一个**UserDetails对象**
3. 通过**PasswordEncoder**对比Authentication对象的密码和UserDetails对象的密码是否一致， 
    1. 一致：将UserDetails对象中的权限信息（authorities）设置到Authentication对象中，并将Authentication对象返回至UsernamePasswordAuthticationFilter。
    2. 不一致：没有上述操作，也不返回Authentication对象。
4. 当返回了Authentication对象时，就会使用**SecurityContextHondler.getContext().setAuthentication()**来储存该Authentication对象，其他过滤器可以使用SecurityContextHondler来获取该Authentication对象。



## 3、两个重要接口


### 3.1、UserDetailsService接口


+  当没有配置时，SpringBoot会使用”user“作为用户名和随机生成的密码。 
+  自定义配置 
    1. 通过实现**UserDetailsService接口**，可以自定义认证逻辑。编写查询数据库过程，返回Security框架提供的User对象。
    2. 继承**UsernamePasswordAuthenticationFilter类**，重写其中三个方法。



### 3.2、PasswordEncoder接口


数据加密接口，用于加密返回的User对象的密码，其常用子类：BCryptPasswordEncoder(hash加密)



## 4、Web开发中的权限管理


### 4.1、内置访问控制方法


1. **permitAll()**：允许所有人访问
2. denyAll()：禁止所有人访问
3. anonymous()：允许匿名访问
4. **authenticated()**：允许已验证的用户访问
5. fullyAuthenticated()：允许完全验证的用户访问
6. **rememberMe()**：记住我



### 4.2、认证（用户登录）


```java
//Security推荐使用的加密器
PasswordEncoder encoder = new BCryptPasswordEncoder();
String encode = encoder.encode("密码");
```



+ 从数据库中查询账号、密码：



```java
//1. Security配置类：指定使用的UserDetailsService
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    //自动注入UserDetailsService的实现类
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
                .passwordEncoder(password());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()    //登录表单
                .loginPage("/login.html")   //登陆页面
                .loginProcessingUrl("/login")   //登录访问的路径
                .defaultSuccessUrl("/success").permitAll()    //登录成功后访问的路径
                .and().authorizeRequests()
                .antMatchers("/", "/login", "/hello").permitAll() //某些路径可以直接访问
                .anyRequest().authenticated()	//拦截所有请求
                .and().csrf().disable();    //关闭csrf防火墙
    }

    @Bean
    PasswordEncoder password() {
        return new BCryptPasswordEncoder();
    }
}


//UserDetailsService接口实现类，编写登录逻辑，从数据库中查询用户信息，用于登录
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Users users = userMapper.queryUserByUserName(username);

        if (users == null) {
            throw new UsernameNotFoundException("账号不存在!!!");
        }

        //此用户所具有的权限列表
        List<GrantedAuthority> authorityList = 							AuthorityUtils.commaSeparatedStringToAuthorityList("role");

        String encode = new BCryptPasswordEncoder().encode(users.getPassword());

        return new User(users.getUserName(), encode, authorityList);
    }
}
```



### 4.3、授权（权限管理）


+  几个重要方法： 
    1. boolean hasAuthority()：判断当前主体是否具有某一项指定权限
    2. boolean hasAnyAuthority()：判断当前主体是否具有某几项指定权限中的任意一项
    3. boolean hasRole()：判断当前主体是否具有某一个指定角色
    4. boolean hasAnyRole()：判断当前主体是否具有某几个指定角色中的任意一个
+  权限添加： 
    1.  在配置类中设置当前访问路径需要具有什么权限 

```java
.antMatchers("/", "/login", "/hello").hasAuthority("权限等级");

.antMatchers("/", "/login", "/hello").hasAnyAuthority("权限等级","权限等级");

.antMatchers("/", "/login", "/hello").hasRole("角色");

.antMatchers("/", "/login", "/hello").hasAnyRole("角色","角色");
```

 

    2.  在UserDetailsService实现类中，为返回的User对象设置权限 

```java
List<GrantedAuthority> authorityList = 							AuthorityUtils.commaSeparatedStringToAuthorityList("权限等级");

//必须要有前缀：ROLE_
List<GrantedAuthority> authorityList = 							AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_角色");

//权限等级和角色可以混合书写
List<GrantedAuthority> authorityList = 							AuthorityUtils.commaSeparatedStringToAuthorityList("权限等级", "ROLE_角色");
```

 

+  依据IP地址进行授权 

```java
.antMatchers("/", "/login", "/hello").hasIpAddress("ip地址");
```

 

+  注解方式的权限添加： 
    1.  启动类开启注解：[@EnableGlobalMethodSecurity(securedEnabled ](/EnableGlobalMethodSecurity(securedEnabled ) = true)  
    2.  在**controller的方法**上使用注解，设置角色（相当于设置了路径的访问权限，角色名严格区分大小写） 
    3.  在UserDetailsService实现类中设置用户角色、权限 

```java
List<GrantedAuthority> authorityList = 							AuthorityUtils.commaSeparatedStringToAuthorityList("权限等级", "ROLE_角色");
```

 

    - 可供使用的注解： 
        1. @Secured：用户具有某项**角色**即可访问方法，需要**"ROLE_"**作为前缀，配置中设置角色不用加前缀
        2. @PreAuthoriz(hasAuthority()/hasRole())：在方法或类执行前，对用户进行是否具有某项权限的验证
        3. @PostAuthoriz(hasAuthority()/hasRole())：在方法或类执行后，对用户进行是否具有某项权限的验证
        4. @PostFilter()：对返回的数据进行过滤，在权限验证后
        5. @PreFilter：在进入控制器方法之前，对传入的参数进行过滤，
+  基于access表达式的权限控制  
访问控制的底层都是access表达式 

```java
.access("permitAll()");
.access("denyAll()");
.access("anonymousAll()");
.access("authenticated()");
.access("fullyAuthenticated()");
.access("remeberMe()");
.access("hasRole('角色')");
.access("hasAuthority('权限')");
```

 



## 5、Security的配置类中自定义无权限页面、注销跳转页面


```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //配置自定义无权限访问页面
        http.exceptionHandling().accessDeniedPage("/not_authorized");
        
        //配置自定义无权限访问的handler
         http.exceptionHandling().accessDeniedHandler(AccessDeniedHandler实现类对象);
        
        //注销登录并跳转页面
        http.logout().logoutUrl("/logout").logoutSuccessUrl("/login").permitAll();
    }
```



## 6、基于数据库的记住我


1. 在Security的配置类中进行配置：



```java
//注入数据源
@Autowired
private DataSource dataSource;

@Bean
PersistentTokenRepository persistentTokenRepository() {
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();

    jdbcTokenRepository.setDataSource(dataSource);
    //自动在数据库中创建一个专用于”RememberMe“的表
    jdbcTokenRepository.setCreateTableOnStartup(true);

    return jdbcTokenRepository;
}
```



2.  在Security的配置类中的**configure(HttpSecurity http)**中开启记住我功能 

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    //配置自定义无权限访问页面
    http.exceptionHandling().accessDeniedPage("/not_authorized");

    //登录表单提交
    http.formLogin()
        //登陆页面
        .loginPage("/loginPage")
        //登录表单的action
        .loginProcessingUrl("/login")
        //登录成功后的访问路径, 必须是POST请求
        .successForwardUrl("/toMain").permitAll()
        .failureForwardUrl("/error").permitAll();

    //用户授权登录
    http.authorizeRequests()
        //某些路径可以直接访问
        .antMatchers("/", "/login").permitAll()
        //所有请求都需要被认证
        .anyRequest().authenticated();
    //.anyRequest().access("@myServiceImpl.hasPermission(request, authentication)")

    //开启自动登录
    http.rememberMe()
        //持久层对象
        .tokenRepository(persistentTokenRepository())
        //设置记住我的有效时间，单位是秒
        .tokenValiditySeconds(60)
        //指定使用的用户配置
        .userDetailsService(userDetailsService);

    //注销登录并跳转页面
    http.logout()
        .logoutUrl("/logout")
        .logoutSuccessUrl("/loginPage").permitAll();

    //关闭csrf防火墙
    //http.csrf().disable();
}
```

 

    3.  在登陆页面中添加”RememberMe“选项 

```html
<!-- 注意 name属性的值必须为 remember-me -->
<input type="checkbox" name="remember-me">记住我
```

 



## 7、Thymeleaf支持Security框架


1.  引入依赖 

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```

 

2.  html页面中引入thymeleaf、security命名空间 

```html
<html lang="en"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```

 



在页面中就能进行一些权限的判断、一些数据的显示



## 8、CSRF


跨站请求伪造，Security默认开启CSRF防护



原理：客户端会携带一个token字符串（加密字符串），与session域中的token进行比较，判断两者是否一致



我们只需要添加一点代码即可：



```html
//采用了thymeleaf模板引擎，在loginform中添加隐藏域
<input type="hidden" th:name="${_csrf.parameterName}" th:value="_csrf.token">
```



## 9、前后端分离的自定义成功、失败页面


### 1、自定义成功页面


**AuthenticationSuccessHandler**接口实现类：



```java
/**
 * 自定义成功页面的跳转机制
 */
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    private final String url;

    public MyAuthticationSuccessHandler(String url) {
        this.url = url;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        //获取登录的用户
        User user = (User) authentication.getPrincipal();
        
        //请求重定向
        response.sendRedirect(url);
    }
}
```



### 2、自定义失败页面


**AuthenticationFailureHandler**接口实现类：



```java
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {
    private final String url;

    public MyAuthenticationFailureHandler(String url) {
        this.url = url;
    }

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {

        response.sendRedirect(url);
    }
}
```



### 3、Security配置类的**configure(HttpSecurity http)**中配置自定义成功、失败页面


```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    //登录表单提交
    http.formLogin()
        //登陆页面
        .loginPage("/loginPage")
        //登录表单的action
        .loginProcessingUrl("/login")
        //在外的成功页面的跳转，不能与successForwardUrl()共存
        .successHandler(new MyAuthenticationSuccessHandler(url))
        //在外的失败页面的跳转，不能与failureForwardUrl()共存
        .failureHandler(new MyAuthenticationFailureHandler(url));
}
```



### 


# Spring Security Oauth2


### Oauth2认证协议


Oauth2认证协议为用户资源的授权提供了一个安全、开放而又简易的标准；任何第三方都可以使用Oauth认证服务，任何的服务提供商都可以实现自身的Oauth认证服务



### 1、授权模式（Authorization Code）


1.  授权码模式（最复杂、最安全、最常用）  
**用户代理(User-Agent)**先向**授权服务器(Authorization-Server)**申请**授权码**，授权服务器发送授权码给用户代理，用户代理再将授权码发送给**Client**，Client带着授权码向授权服务器申请**授权令牌(Access Token)或刷新令牌(Refresh Token)**。 
2.  简化授权模式（Implicit） 
3.  密码模式(Resource Owner  PasswordCredentials)：将用户密码发送给授权服务器，授权服务器检查密码正确后，返回token。 
4.  客户端模式(Client Credentials)：客户端授权给授权服务器，授权服务器返回token 



### 2、刷新token


+ 还没有笔记



1. Authorize Endpoint：授权端点，进行授权
2. Token Endpoint：令牌端点，经过授权拿到对应的token
3. Introspection Endpoint：校验端点，校验token的合法性
4. Revocation Endpoint：撤销端点，撤销授权



引入Oauth依赖



```xml
spring-security-oauth2  -> 被废弃，建议不使用，否则后期无法维护
spring-cloud-starter-oauth2 -> 引用 spring-security-oauth2，但尚未标注被废弃
spring-security-oauth2-autoconfigure  -> 自动配置，没有用处
spring-boot-starter-oauth2-client -> 最新
spring-boot-starter-oauth2-resource-server  -> 最新

<!-- 授权服务器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>

<!-- 开发客户端 -->
```



## Spring Security Oauth2架构




## Redis存储token


1. 引入依赖



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```



2. 在主配置文件中添加配置



```properties
#Redis配置
spring.redis.host=ip地址
```



# 前后端分离版本


## SecurityConfig


```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    @Resource
    private AuthenticationEntryPoint authenticationEntryPoint;
    @Resource
    private AccessDeniedHandler accessDeniedHandler;

    @Bean
    public PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //允许跨域请求
        http.cors();

        //关闭跨域访问防护
        http.csrf().disable();

        //设置不创建HttpSession：基于token的认证，不需要从session中获取SecurityConfig，
        http.sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

        // 配置jwt登录认证过滤器
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        //配置自定义异常处理器
        http.exceptionHandling()
                //认证异常
                .authenticationEntryPoint(authenticationEntryPoint)
                //授权异常
                .accessDeniedHandler(accessDeniedHandler);

        //请求授权
        http.authorizeRequests()
                //允许请求匿名访问
                .antMatchers("/login").anonymous()
                //上面授权过的请求除外，其余请求全部需要被认证
                .anyRequest().authenticated();
    }
}
```



## 自定义登录逻辑


实现UserDetailsService接口，重写其方法，实现自定义登录逻辑



## 登录时的身份验证（Service）


需要用到**AuthenticationManager对象**，依靠这个对象进行身份验证



AuthenticationManager：身份验证管理器；  
拥有方法：authenticate，能够对自定义登录逻辑实现类返回的UserDetails进行验证



### 身份验证成功


1. 使用LoginUser的userId生成JWT(token)
2. 将LoginUser的完整信息存入redis中
3. 封装token和loginUserInfo返回给前端



### 身份验证失败


直接抛出一个登录失败异常即可，后续进行异常捕获，向前端返回一个登录失败信息



## Token(JWT )与Redis


### JwtUtils


```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.*;

/**
 * JWT工具类
 * JWT中需要保存一些数据：
 */
@Component
public class JwtUtils {

    //一般放在官方声明中
    // 密钥，后续会进行加密
    private final String JWT_SECRET = "BugOS_ly";
    // 签发时间
    private final String ISSUED_AT = "iat";
    // 过期时间
    private final long EXPIRATION_TIME = 1000 * 60 * 30;

    //一般放在自定义声明中
    // jwt主题
    private final String SUBJECT = "sub";
    // 当前登录用户
    public final String LOGIN_USER_ID = "loginUserId";
    // 用户权限
    private final String AUTHORITIES = "authorities";


    /**
     * 根据Authentication(验证)对象生成JWT
     */
    public String generateTokenByAuthentication(Authentication authentication) {
        //获取登录用户的角色权限
        StringBuffer authoritiesString = spliceAuthorities(authentication.getAuthorities());

        //自定义属性
        Map<String, Object> claims = new HashMap<>();
        //jwt主题
        claims.put(SUBJECT, authentication.getName());
        //jwt中自定义声明：用户权限
        claims.put(AUTHORITIES, authoritiesString);

        return generateTokenByClaims(claims);
    }

    /**
     * 根据UserDetails对象创建claims集合然后生成对应JWT
     */
    public String generateTokenByUserDetails(UserDetails userDetails) {
        //获取登录用户的角色权限
        StringBuffer stringBuffer = spliceAuthorities(userDetails.getAuthorities());

        //自定义声明（private claims）
        Map<String, Object> claims = new HashMap<>();

        //JWT主题
        claims.put(SUBJECT, userDetails.getUsername());
        //用户权限
        claims.put(AUTHORITIES, stringBuffer);

        return generateTokenByClaims(claims);
    }

    /**
     * 根据Claims（声明）生成JWT，一般用来服务本类中生成jwt的其他方法（生成基础jwt，通过传参claims来自定义jwt）
     */
    public String generateTokenByClaims(Map<String, Object> claims) {
        return Jwts.builder()
                //自定义属性
                .setClaims(claims)
                //JWT的签发日期
                .setIssuedAt(new Date())
                //过期时间
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
                //签名
                .signWith(SignatureAlgorithm.HS256, JWT_SECRET)
                .compact();
    }

    /**
     * 拼接用户权限，转为字符串
     */
    private StringBuffer spliceAuthorities(Collection<? extends GrantedAuthority> authorities) {
        StringBuffer permissionString = new StringBuffer();

        // 拼接用户权限字符串
        for (GrantedAuthority authority : authorities) {
            permissionString.append(authority.getAuthority()).append(",");
        }

        return permissionString;
    }

    /**
     * 加密JWT密钥并返回加密后的密钥
     */
    private SecretKey createSecret() {
        //加密（可解密的）
        byte[] decode = Base64.getDecoder().decode(JWT_SECRET);
        return new SecretKeySpec(decode, 0, decode.length, "AES");
    }

    /**
     * 解析JWT, 获得Claims（声明）,如果token过期，则返回null
     */
    public Claims parseToken(String token) throws ExpiredJwtException {
        Claims claims;
        try {
            claims = Jwts.parser()
                    .setSigningKey(JWT_SECRET)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (ExpiredJwtException e) {
            //如果过期,在异常中调用, 返回claims, 否则无法解析过期的token
            claims = null;
        }
        return claims;
    }

    /**
     * 从JWT中获得主题
     */
    public String getSubjectFromToken(String token) {
        try {
            return parseToken(token).getSubject();
        } catch (ExpiredJwtException e) {
            //如果过期, 需要在此处异常调用如下的方法, 否则拿不到用户名
            return e.getClaims().getSubject();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 从JWT中获取签发时间
     */
    public Date getIssuedAtFromToken(String token) {
        Date issuedAt;
        try {
            Claims claims = parseToken(token);
            issuedAt = claims.getIssuedAt();
        } catch (Exception e) {
            issuedAt = null;
        }
        return issuedAt;
    }

    /**
     * 从JWT中获取过期时间
     */
    public Date getExpirationFromToken(String token) {
        Date expiration;
        try {
            Claims claims = parseToken(token);
            expiration = claims.getExpiration();
        } catch (Exception e) {
            expiration = null;
        }
        return expiration;
    }

    /**
     * 判断JWT是否过期
     */
    private Boolean isTokenExpired(String token) {
        Date expiration = getExpirationFromToken(token);
        //判断过期时间是否在当前时间之前
        return expiration.before(new Date());
    }

    /**
     * JWT是否可以被刷新(过期就可以被刷新) isCanRefreshToken
     */
    public Boolean isRefreshed(String token) {
        return isTokenExpired(token);
    }

    /**
     * 刷新JWT
     */
    public String refreshToken(String token) {
        String refreshedToken;
        try {
            //获得Token的Claims, 由于在生成JWT的时候会根据当前时间更新过期时间, 我们只需要手动修改
            //放在自定义属性中的创建时间就可以了
            Claims claims = parseToken(token);
            claims.put(ISSUED_AT, new Date());
            //利用修改后的claims再次生成token, 就不需要我们每次都去查用户的信息和权限了
            refreshedToken = generateTokenByClaims(claims);
        } catch (Exception e) {
            refreshedToken = null;
        }
        return refreshedToken;
    }

    /**
     * 判断Token是否合法
     */
    public Boolean isValidate(String token, UserDetails userDetails) {
        String subject = getSubjectFromToken(token);
        return (
                //如果用户名与token一致且token没有过期, 则认为合法
                userDetails.getUsername().equals(subject) && !isTokenExpired(token)
        );
    }

}
```



### JWT验证令牌过滤器（身份认证过滤器）


```java
import com.alibaba.fastjson.JSON;
import com.bug.framework.domain.LoginUser;
import com.bug.framework.utils.JwtUtils;
import com.bug.framework.utils.RedisCache;
import com.bug.framework.utils.WebUtils;
import io.jsonwebtoken.Claims;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.authentication.DisabledException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;
import org.springframework.util.ObjectUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.annotation.Resource;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * JWT验证令牌过滤器（身份认证过滤器）
 */
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    /**
     * 身份认证入口，自己编写的AuthenticationEntryPoint接口实现类
     */
    @Resource
    private AuthenticationEntryPoint authenticationEntryPoint;
    /**
     * 拒绝访问处理程序，自己编写的accessDeniedHandler接口实现类
     */
    @Resource
    private AccessDeniedHandler accessDeniedHandler;
    @Resource
    private JwtUtils jwtUtils;
    @Resource
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        //从请求头中获token（token放在请求头中）
        String token = request.getHeader("token");

        if (ObjectUtils.isEmpty(token)) {//白名单请求中无token，
            filterChain.doFilter(request, response);
            return;
        }

        //解析token，获取用户ID
        Claims claims = jwtUtils.parseToken(token);

        if (ObjectUtils.isEmpty(claims)) {
            AccessDeniedException exception = new AccessDeniedException("身份验证已过期，请重新登录");
            accessDeniedHandler.handle(request, response, exception);
            return;
        }

        Object loginUserId = claims.get(jwtUtils.LOGIN_USER_ID);

        //获取Redis中对应的key，然后获取redis中的LoginUser对象信息
        String redisJwtKey = redisCache.getRedisJwtKey(Long.valueOf(String.valueOf(loginUserId)));
        LoginUser loginUser = JSON.parseObject(JSON.toJSONString(redisCache.get(redisJwtKey)), LoginUser.class);

        //判断loginUser是否为空
        if (ObjectUtils.isEmpty(loginUser)) {
            AuthenticationException exception = new DisabledException("用户未登录或登录已过期");
            authenticationEntryPoint.commence(request, response, exception);
            return;
        }

        //将此登录用户设置为已认证状态（当然，loginUser我们知道是已认证状态）
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken
                (loginUser, null, loginUser.getAuthorities());

        //将loginUser存入SecurityContext中
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);

        //放行请求
        filterChain.doFilter(request, response);
    }
}
```



### LoginService验证身份


```java
/**
 * 身份验证管理器，拥有方法：authenticate，能够对自定义登录逻辑实现类返回的UserDetails进行验证
 */
@Resource
private AuthenticationManager authenticationManager;

@Override
public ResponseResult<BlogLoginUserVo> login(User user) {

    UsernamePasswordAuthenticationToken authenticationToken =
            new UsernamePasswordAuthenticationToken(user.getUserName(), user.getPassword());

    // 调用UserDetailsService进行验证
    Authentication authentication = authenticationManager.authenticate(authenticationToken);

    if (ObjectUtils.isEmpty(authentication)) {
        throw new RuntimeException("登录失败");
    }

    // 若认证通过：使用LoginUser的userId生成JWT(token)
    LoginUser loginUser = (LoginUser) authentication.getPrincipal();
    HashMap<String, Object> claims = new HashMap<>();
    claims.put(jwtUtils.LOGIN_USER_ID, loginUser.getUserId());
    String token = jwtUtils.generateTokenByClaims(claims);

    // 将loginUser的完整信息存入redis中
    redisCache.set(redisCache.getRedisJwtKey(loginUser.getUser().getId()), loginUser, redisCache.getExpirationTime());

    // 封装token和loginUserInfo返回给前端
    LoginUserInfo loginUserInfo = BeanCopyUtils.copySingleBean(loginUser.getUser(), LoginUserInfo.class);

    return new ResponseResultBuilder().success(new BlogLoginUserVo(token, loginUserInfo));
}
```

