# 微服务SpringCloud项目（六）：整合oauth并使用password密码认证模式 - 掘金
**小知识，大挑战！本文正在参与「[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https&#x3A;//juejin.cn/post/7008476801634680869")」创作活动**

**本文已参与 [「掘力星计划」](https://juejin.cn/post/7012210233804079141 "https&#x3A;//juejin.cn/post/7012210233804079141") ，赢取创作大礼包，挑战创作激励金。** 

## 📖前言

    心态好了，就没那么累了。心情好了，所见皆是明媚风景。
    复制代码

> **`“一时解决不了的问题，那就利用这个契机，看清自己的局限性，对自己进行一场拨乱反正。” 正如老话所说，一念放下，万般自在。如果你正被烦心事扰乱心神，不妨学会断舍离。断掉胡思乱想，社区垃圾情绪，离开负面能量。心态好了，就没那么累了。心情好了，所见皆是明媚风景。`**

### 🚓`引入依赖`

* * *

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>

    
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-client</artifactId>
        <version>${spring-boot-admin.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>

    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    
    <dependency>
        <groupId>com.github.ulisesbocchio</groupId>
        <artifactId>jasypt-spring-boot-starter</artifactId>
        <version>${jasypt.version}</version>
    </dependency>

    
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql-connector}</version>
        <scope>provided</scope>
    </dependency>

    
    <dependency>
        
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>${druid.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
        <version>${common-pool.version}</version>
    </dependency>

    

    
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>${mybatis-plus.version}</version>
    </dependency>

    
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
        <version>${mybatis-plus-dynamic.version}</version>
    </dependency>
    
</dependencies>
```

### 1. 启动类添加

* * *

```java
@EnableFeignClients

@EnableResourceServer
@EnableDiscoveryClient
```

### 2. 创建一个授权的配置文件 `AuthorizationServerConfig.java`

* * *

> **根据官方指示我们需要创建一个配置类去实现 `AuthorizationServerConfigurer` 所以我们，创建一个类去继承它的实现类`AuthorizationServerConfigurerAdapter`，具体代码如下：** 

```java
package com.cyj.dream.auth.config;

import com.cyj.dream.auth.entity.SysUser;
import com.cyj.dream.auth.persistence.service.impl.CustomUserServiceImpl;
import com.cyj.dream.core.constant.Constant;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.CompositeTokenGranter;
import org.springframework.security.oauth2.provider.TokenGranter;
import org.springframework.security.oauth2.provider.client.ClientCredentialsTokenGranter;
import org.springframework.security.oauth2.provider.client.JdbcClientDetailsService;
import org.springframework.security.oauth2.provider.code.AuthorizationCodeTokenGranter;
import org.springframework.security.oauth2.provider.implicit.ImplicitTokenGranter;
import org.springframework.security.oauth2.provider.password.ResourceOwnerPasswordTokenGranter;
import org.springframework.security.oauth2.provider.refresh.RefreshTokenGranter;
import org.springframework.security.oauth2.provider.token.DefaultTokenServices;
import org.springframework.security.oauth2.provider.token.TokenEnhancer;
import org.springframework.security.oauth2.provider.token.TokenEnhancerChain;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.redis.RedisTokenStore;

import javax.sql.DataSource;
import java.util.*;
import java.util.concurrent.TimeUnit;



@Configuration 
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private DataSource dataSource;

    @Autowired
    private CustomUserServiceImpl userDetailsService;

    
    private static final int ACCESS_TOKEN_VALIDITY_SECONDS = 7200 * 12 * 7;

    
    private static final int REFRESH_TOKEN_VALIDITY_SECONDS = 7200 * 12 * 7;

    
    @Bean
    public TokenStore redisTokenStore() {
        RedisTokenStore tokenStore = new RedisTokenStore(redisConnectionFactory);
        
        tokenStore.setPrefix(Constant.tokenPrefix + "_");
        return tokenStore;
    }

    
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory().withClient("dream").secret(passwordEncoder.encode("c83fb51ff6e807e8805c6dd9d5707365"))
                
                .redirectUris("http://www.baidu.com")
                .authorizedGrantTypes("authorization_code", "password", "refresh_token").scopes("all")
                .accessTokenValiditySeconds(ACCESS_TOKEN_VALIDITY_SECONDS)
                
                .refreshTokenValiditySeconds(REFRESH_TOKEN_VALIDITY_SECONDS);
        
    }

    
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        tokenEnhancerChain.setTokenEnhancers(Arrays.asList(tokenEnhancer()));
        endpoints.authenticationManager(authenticationManager).allowedTokenEndpointRequestMethods(HttpMethod.GET,
                HttpMethod.POST, HttpMethod.PUT,
                HttpMethod.DELETE)
                .tokenEnhancer(tokenEnhancerChain)
                
                .tokenStore(redisTokenStore()).userDetailsService(userDetailsService)
                
                .tokenGranter(tokenGranter(endpoints));
        
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        
        defaultTokenServices.setAccessTokenValiditySeconds((int) TimeUnit.HOURS.toSeconds(2));
        
        defaultTokenServices.setRefreshTokenValiditySeconds((int) TimeUnit.DAYS.toSeconds(30));
        
        defaultTokenServices.setReuseRefreshToken(true);
        defaultTokenServices.setSupportRefreshToken(true);
        defaultTokenServices.setTokenStore(endpoints.getTokenStore());
        defaultTokenServices.setClientDetailsService(endpoints.getClientDetailsService());
        defaultTokenServices.setTokenEnhancer(endpoints.getTokenEnhancer());
        endpoints.tokenServices(defaultTokenServices);
    }

    
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        
        security.allowFormAuthenticationForClients()
                .tokenKeyAccess("isAuthenticated()")
                .checkTokenAccess("permitAll()");
    }

    
    private TokenGranter tokenGranter(AuthorizationServerEndpointsConfigurer endpoints) {
        List<TokenGranter> list = new ArrayList<>();
        
        list.add(new RefreshTokenGranter(endpoints.getTokenServices(), endpoints.getClientDetailsService(), endpoints.getOAuth2RequestFactory()));
        
        list.add(new AuthorizationCodeTokenGranter(endpoints.getTokenServices(), endpoints.getAuthorizationCodeServices(), endpoints.getClientDetailsService(), endpoints.getOAuth2RequestFactory()));
        
        list.add(new ClientCredentialsTokenGranter(endpoints.getTokenServices(), endpoints.getClientDetailsService(), endpoints.getOAuth2RequestFactory()));
        
        list.add(new ResourceOwnerPasswordTokenGranter(authenticationManager, endpoints.getTokenServices(), endpoints.getClientDetailsService(), endpoints.getOAuth2RequestFactory()));
        
        list.add(new ImplicitTokenGranter(endpoints.getTokenServices(), endpoints.getClientDetailsService(), endpoints.getOAuth2RequestFactory()));
        return new CompositeTokenGranter(list);
    }

    
    @Bean
    public TokenEnhancer tokenEnhancer() {
        return (accessToken, authentication) -> {
            final Map<String, Object> additionalInfo = new HashMap<>(2);
            additionalInfo.put("license", "SunnyChen-DreamChardonnay");
            SysUser user = (SysUser) authentication.getUserAuthentication().getPrincipal();
            if (user != null) {
                additionalInfo.put("userId", user.getSysUserId());
                additionalInfo.put("userPhone", user.getSysUserPhone());
                additionalInfo.put("userDeptId", user.getSysUserInfoDepartmentId());
            }
            ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
            return accessToken;
        };
    }

    
    @Bean
    AuthenticationManager authenticationManager() {
        AuthenticationManager authenticationManager = new AuthenticationManager() {
            @Override
            public Authentication authenticate(Authentication authentication) throws AuthenticationException {
                return daoAuhthenticationProvider().authenticate(authentication);
            }
        };
        return authenticationManager;
    }

    
    @Bean
    public AuthenticationProvider daoAuhthenticationProvider() {
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
        daoAuthenticationProvider.setUserDetailsService(userDetailsService);
        daoAuthenticationProvider.setHideUserNotFoundExceptions(false);
        daoAuthenticationProvider.setPasswordEncoder(passwordEncoder);
        return daoAuthenticationProvider;
    }

}
```

### 3. 由于授权服务器本身也是资源服务器，所以也创建一个资源配置，代码如下

* * *

```java
package com.cyj.dream.auth.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;


@Configuration

@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http    
                .requestMatchers().
                antMatchers("/user", "/test/need_token", "/logout", "/remove", "/update", "/test/need_admin", "/test/scope")
                .and().authorizeRequests().anyRequest().authenticated();
    }

}
```

### 4. 创建 `webSecurity` 配置

* * *

```tex
/*
     * OK ，关于这个配置我要多说两句：
     *
     * 1.首先当我们要自定义Spring Security的时候我们需要继承自WebSecurityConfigurerAdapter来完成，相关配置重写对应
     * 方法即可。 2.我们在这里注册CustomUserService的Bean，然后通过重写configure方法添加我们自定义的认证方式。
     * 3.在configure(HttpSecurity http)方法中，我们设置了登录页面，而且登录页面任何人都可以访问，然后设置了登录失败地址，也设置了注销请求，
     * 注销请求也是任何人都可以访问的。
     * 4.permitAll表示该请求任何人都可以访问，.anyRequest().authenticated(),表示其他的请求都必须要有权限认证。
     * 5.这里我们可以通过匹配器来匹配路径，比如antMatchers方法，假设我要管理员才可以访问admin文件夹下的内容，我可以这样来写：.
     * antMatchers("/admin/**").hasRole("ROLE_ADMIN")，也可以设置admin文件夹下的文件可以有多个角色来访问，
     * 写法如下：.antMatchers("/admin/**").hasAnyRole("ROLE_ADMIN","ROLE_USER")
     * 6.可以通过hasIpAddress来指定某一个ip可以访问该资源,假设只允许访问ip为210.210.210.210的请求获取admin下的资源，
     * 写法如下.antMatchers("/admin/**").hasIpAddress("210.210.210.210")
     * 7.更多的权限控制方式参看源码：
     */
```

> **代码如下：** 

```java
package com.cyj.dream.auth.config;

import com.cyj.dream.auth.handler.*;
import com.cyj.dream.auth.persistence.service.impl.CustomUserServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;


@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomUserServiceImpl userDetailsService;

    
    @Autowired
    private MyAuthenticationSuccessHandler userLoginSuccessHandler;

    
    @Autowired
    private MyAuthenticationFailureHandler userLoginFailureHandler;

    
    @Autowired
    private UserLogoutSuccessHandler userLogoutSuccessHandler;

    
    @Autowired
    private UserAuthAccessDeniedHandler userAuthAccessDeniedHandler;

    
    @Autowired
    private UserAuthenticationEntryPointHandler userAuthenticationEntryPointHandler;

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    @Bean
    public UserDetailsService userDetailsService() {
        return new CustomUserServiceImpl();
    }

    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService())
                .passwordEncoder(passwordEncoder());
    }

    
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        
        http.authorizeRequests()
                .antMatchers("/webjars/**", "/js/**", "/css/**", "/images/*", "/fonts/**", "/**/*.png", "/**/*.jpg",
                        "/static/**")
                
                .permitAll()
                
                
                
                .antMatchers("/login", "/oauth/**")
                .permitAll()
                .antMatchers("/**")
                .fullyAuthenticated()
                .and()
                
                .httpBasic()
                .authenticationEntryPoint(userAuthenticationEntryPointHandler)
                .and()
                .csrf()
                .disable()
                
                .formLogin()
                
                
                
                .loginPage("/login")
                
                .successHandler(userLoginSuccessHandler)
                
                .failureHandler(userLoginFailureHandler)
                .permitAll()
                .and()
                
                .logout()
                
                .logoutUrl("/login/loginOut")
                
                .logoutSuccessHandler(userLogoutSuccessHandler)
                .and()
                
                .exceptionHandling()
                .accessDeniedHandler(userAuthAccessDeniedHandler)
                .and()
                
                .cors()
                .and()
                
                .csrf().disable();
                
        
        http.headers().frameOptions().sameOrigin();
        
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        
        http.headers().cacheControl();
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        
        web.ignoring().antMatchers(
                "/error",
                "/v2/api-docs/**",
                "/favicon.ico",
                "/css/**",
                "/js/**",
                "/images/*",
                "/fonts/**",
                "/**/*.png",
                "/**/*.jpg")
                
                .antMatchers("/templates/**")
                .antMatchers("/static/**")
                .antMatchers("/webjars/**")
                .antMatchers("/swagger-ui.html/**")
                .antMatchers("/v2/**")
                .antMatchers("/doc.html")
                .antMatchers("/api-docs-ext/**")
                .antMatchers("/swagger-resources/**");
        ;
    }

}
```

### 5. 创建一个 `controller` 进行访问测试

* * *

```java
package com.cyj.dream.auth.controller;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.cyj.dream.auth.entity.SysUser;
import com.cyj.dream.auth.persistence.service.ITbSysUserService;
import com.cyj.dream.core.aspect.annotation.ResponseResult;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.security.Principal;


@Slf4j
@ResponseResult
@RestController
@RequestMapping(value = "/authUser", name = "用户控制器")
@Api(value = "authUser", tags = "用户控制器")
public class UserController {

    @Autowired
    private ITbSysUserService iTbSysUserService;

    @ApiOperation("通过名称获取用户信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "userName", value = "用户名称", dataType = "String", required = true)
    })
    @RequestMapping(value = "getByName", method = RequestMethod.GET, name = "通过名称获取用户信息")
    public SysUser getByName(@RequestParam(value = "userName") String userName){
        LambdaQueryWrapper<SysUser> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(SysUser::getUsername, userName);
        return iTbSysUserService.getOne(wrapper);
    }

    @ApiOperation("获取授权的用户信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "principal", value = "当前用户", dataType = "Principal", required = true)
    })
    @RequestMapping(value = "current", method = RequestMethod.GET, name = "获取授权的用户信息")
    public Principal user(Principal principal){
        
        return principal;
    }

}
```

> 带 `token` 的请求

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf909c8fee3449a5a98e511d92bcd7bd~tplv-k3u1fbpfcp-watermark.awebp)

> **不带 `token` 的请求**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41cc134466fc4bacb75baef6196ac55e~tplv-k3u1fbpfcp-watermark.awebp)

> 带 `token`，但是没有`ROLE_ADMIN`权限

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0172be2cd73c4b33927b05fb41bd6447~tplv-k3u1fbpfcp-watermark.awebp)

### 6. 最后关于`Fegin` 调用服务 `Token` 丢失的问题

* * *

在微服务中我们经常会使用 RestTemplate 或 Fegin 来进行服务之间的调用，在这里就会出现一个问题，我们去调用别的服务的时候就会出现 token 丢失的情况，导致我们没有权限去访问。

所以我们需要加上一些拦截器将我们的 `token` 带走，针对`fegin`的配置代码如下：

```java
public class FeignRequestInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate requestTemplate) {
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        assert attributes != null;
        HttpServletRequest request = attributes.getRequest();    
    
    Enumeration<String> headerNames = request.getHeaderNames();
    if (headerNames != null) {
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            String value = request.getHeader(name);
            requestTemplate.header(name, value);
        }
    }

    
    Enumeration<String> parameterNames = request.getParameterNames();
    StringBuilder body = new StringBuilder();
    if (parameterNames != null) {
        while (parameterNames.hasMoreElements()) {
            String name = parameterNames.nextElement();
            String value = request.getParameter(name);

            
            if ("access_token".equals(name)) {
                requestTemplate.header("authorization", "Bearer " + value);
            }

            
            else {
                body.append(name).append("=").append(value).append("&");
            }
        }
    }

    
    if (body.length() > 0) {
        
        body.deleteCharAt(body.length() - 1);
        requestTemplate.body(body.toString());
    }
}
}
```

然后将这个拦截器加入到我们 `fegin` 请求拦截器中：

```java
@Configuration
public class FeignRequestConfiguration {
	@Bean
	public RequestInterceptor requestInterceptor() {
    	return new FeignRequestInterceptor();
	}
}
```

### 7. `Spring Security` 和 `Shiro`

* * *

#### 相同点：

1：认证功能

2：授权功能

3：加密功能

4：会话管理

5：缓存支持

6：rememberMe 功能.......

#### 不同点：

##### 优点：

1：Spring Security 基于 Spring 开发，项目中如果使用 Spring 作为基础，配合 Spring Security 做权限更加方便，而 Shiro 需要和 Spring 进行整合开发

2：Spring Security 功能比 Shiro 更加丰富些，例如安全防护

3：Spring Security 社区资源比 Shiro 丰富

##### 缺点：

1：Shiro 的配置和使用比较简单，Spring Security 上手复杂

2：Shiro 依赖性低，不需要任何框架和容器，可以独立运行，而 Spring Security 依赖于 Spring 容器

### 8. 数据库表

```sql

create table oauth_client_details (
  client_id VARCHAR(256) PRIMARY KEY,
  resource_ids VARCHAR(256),
  client_secret VARCHAR(256),
  scope VARCHAR(256),
  authorized_grant_types VARCHAR(256),
  web_server_redirect_uri VARCHAR(256),
  authorities VARCHAR(256),
  access_token_validity INTEGER,
  refresh_token_validity INTEGER,
  additional_information VARCHAR(4096),
  autoapprove VARCHAR(256)
);

create table oauth_client_token (
  token_id VARCHAR(256),
  token LONGVARBINARY,
  authentication_id VARCHAR(256) PRIMARY KEY,
  user_name VARCHAR(256),
  client_id VARCHAR(256)
);

create table oauth_access_token (
  token_id VARCHAR(256),
  token LONGVARBINARY,
  authentication_id VARCHAR(256) PRIMARY KEY,
  user_name VARCHAR(256),
  client_id VARCHAR(256),
  authentication LONGVARBINARY,
  refresh_token VARCHAR(256)
);

create table oauth_refresh_token (
  token_id VARCHAR(256),
  token LONGVARBINARY,
  authentication LONGVARBINARY
);

create table oauth_code (
  code VARCHAR(256), authentication LONGVARBINARY
);

create table oauth_approvals (
	userId VARCHAR(256),
	clientId VARCHAR(256),
	scope VARCHAR(256),
	status VARCHAR(10),
	expiresAt TIMESTAMP,
	lastModifiedAt TIMESTAMP
);


create table ClientDetails (
  appId VARCHAR(256) PRIMARY KEY,
  resourceIds VARCHAR(256),
  appSecret VARCHAR(256),
  scope VARCHAR(256),
  grantTypes VARCHAR(256),
  redirectUrl VARCHAR(256),
  authorities VARCHAR(256),
  access_token_validity INTEGER,
  refresh_token_validity INTEGER,
  additionalInformation VARCHAR(4096),
  autoApproveScopes VARCHAR(256)
);
```

* * *

**最后感谢大家耐心观看完毕，具体请求和一些结果返回可以参见上一章内容，留个点赞收藏是您对我最大的鼓励！**

* * *

### 🎉总结：

-   **更多参考精彩博文请看这里：[《陈永佳的博客》](https://juejin.cn/user/862483929905639/posts "https&#x3A;//juejin.cn/user/862483929905639/posts")**
-   **整合 `auth` 这里有一些类我没有放置，大家可以先阅读文章，后续我会把代码传到 `git` 上方便大家参考学习~**
-   **喜欢博主的小伙伴可以加个关注、点个赞哦，持续更新嘿嘿！** 
    [https://juejin.cn/post/7017611944546795533#heading-4](https://juejin.cn/post/7017611944546795533#heading-4)
