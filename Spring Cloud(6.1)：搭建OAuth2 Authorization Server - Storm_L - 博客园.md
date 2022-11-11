# Spring Cloud(6.1)：搭建OAuth2 Authorization Server - Storm_L - 博客园
**配置web.xml**

添加spring-cloud-starter-security和spring-security-oauth2-autoconfigure两个依赖。

![](https://common.cnblogs.com/images/copycode.gif)

</dependency\>
<!-- Spring cloud starter: Security \-->
<!-- Include: web, actuator, security, zuul, etc. \-->
<dependency\>
    <groupId\>org.springframework.cloud</groupId\>
    <artifactId\>spring-cloud-starter-security</artifactId\>
</dependency\>
<!-- Spring Security OAuth2 Autoconfigure (optional in spring-cloud-security after 2.1) \-->
<dependency\>
    <groupId\>org.springframework.security.oauth.boot</groupId\>
    <artifactId\>spring-security-oauth2-autoconfigure</artifactId\>
</dependency\>

![](https://common.cnblogs.com/images/copycode.gif)

此外，它还是一个Eureka Client和Config Client，如何配置Eureka Client和Config Client请看前面章节。

**配置WebSecurity**

![](https://common.cnblogs.com/images/copycode.gif)

package com.mytools.config; import org.springframework.beans.factory.annotation.Autowired; import org.springframework.context.annotation.Bean; import org.springframework.context.annotation.Configuration; import org.springframework.security.authentication.AuthenticationManager; import org.springframework.security.config.annotation.web.builders.HttpSecurity; import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter; import org.springframework.security.core.userdetails.UserDetailsService; import org.springframework.security.crypto.factory.PasswordEncoderFactories; import org.springframework.security.crypto.password.PasswordEncoder; /\*\* \* Spring Security Configuration. \*/ @Configuration public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired private UserDetailsService userDetailsService; /\*\* \* password encodeer \*/ @Bean public PasswordEncoder passwordEncoder() { return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Override
    @Bean public AuthenticationManager authenticationManagerBean() throws Exception { return super.authenticationManagerBean();
    } /\* (non-Javadoc)
     \* @see org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter#configure(org.springframework.security.config.annotation.web.builders.HttpSecurity) \*/ @Override protected void configure(HttpSecurity http) throws Exception { //@formatter:off
        http.authorizeRequests() // configure authorize request rule
                .antMatchers("/index").permitAll() // .antMatchers("/url/\*\*").hasRole("ADMIN") // some urls have access ADMIN // .anyRequest().authenticated() // any other request need to authenticate
 .and()
            .formLogin() // login as form
                .loginPage("/login") // login url (default is login page with framework) // .defaultSuccessUrl("/index") // login success url (default is index)
                .failureUrl("/login-error") // login fail url
 .and() // .logout() // logout config // .logoutUrl("/logout") // logout url (default is logout) // .logoutSuccessUrl("/index") // logout success url (default is login)
            .rememberMe() // Remember me
                .key("uniqueAndSecret") // generate the contents of the token
                .tokenValiditySeconds(60 \* 60 \* 24 \* 30) // 30 days
                .userDetailsService(userDetailsService) // register UserDetailsService for remember me functionality // .and() //.httpBasic() // use HTTP Basic authentication(in header) for an application
 ; //@formatter:on
 }
}

![](https://common.cnblogs.com/images/copycode.gif)

**说明：** 

（1）UserDetailsService的配置：有两种方式，一种是实现UserDetails/UserDetailsService接口，从DB中获取User和Role。另一种是使用InMemoryUserDetailsManagerConfigurer在内存中创建user和Role。这里使用了第一种，这一部分是Spring Security的范畴，是这里不再贴代码。如果对这部分不熟悉，可以参考：

[Spring Security(1)：认证和授权的核心组件介绍及源码分析](https://www.cnblogs.com/storml/p/10930121.html)

[Spring Security Reference](https://docs.spring.io/spring-security/site/docs/5.1.5.RELEASE/reference/htmlsingle/)

（2）PasswordEncoder的配置：使用PasswordEncoderFactories可以通过不同前缀来识别和创建各种不同的PasswordEncoder。在当前的Spring Security版本中，password必须加密，不加密会报错。

（3）AuthenticationManager的配置：AuthenticationManager虽然在Spring Security自动配置中已经创建，但是并没有暴露为一个Spring Bean（exposed as a Bean）。我们在这里覆盖它并声明它为一个Bean，目的是在配置Authorization Server时配置AuthorizationServerEndpoints时使用（for the password grant）。

（4）HttpSecurity的配置：这一部分是Spring Security的范畴，这里不再赘述。这里主要自定义了User&Role&Path的mapping关系，及login, index, logout,remember-me等逻辑和页面等。

**配置Authorization Server**

![](https://common.cnblogs.com/images/copycode.gif)

package com.mytools.config; import org.springframework.beans.factory.annotation.Autowired; import org.springframework.context.annotation.Configuration; import org.springframework.http.HttpMethod; import org.springframework.security.authentication.AuthenticationManager; import org.springframework.security.core.userdetails.UserDetailsService; import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer; import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter; import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer; import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer; import org.springframework.security.oauth2.provider.ClientDetailsService; /\*\* \* OAuth2 Authorization Server Configuration. \*/ @Configuration public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired private AuthenticationManager authenticationManager;

    @Autowired private UserDetailsService userDetailsService;

    @Autowired private ClientDetailsService clientDetailsService; /\*\* \* 用来配置令牌端点(Token Endpoint)的安全约束<br>
     \* 
     \* @see org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter#configure(org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer) \*/ @Override public void configure(AuthorizationServerSecurityConfigurer security) throws Exception { // 如果配置这个的话，且url中有client\_id和client\_secret的会走ClientCredentialsTokenEndpointFilter来保护 // 如果没有配置这个，或者配置了这个但是url中没有client\_id和client\_secret的，走basic认证保护 // \[IMPORTANT\] 这里如果不设置，client app里的clientAuthenticationScheme应该设置为header，反之设置为form // security.allowFormAuthenticationForClients();
 } /\*\* \* 用来配置客户端详情服务（ClientDetailsService），客户端详情信息在这里进行初始化<br>
     \* 
     \* @see org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter#configure(org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer) \*/ @Override public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.withClientDetails(clientDetailsService);
    } /\* 定义授权和令牌端点以及令牌服务<br>
     \* @see org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter#configure(org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer) \*/ @Override public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager) // 使用Spring提供的AuthenticationManager开启密码授权
                .userDetailsService(userDetailsService) // 注入一个 UserDetailsService，那么刷新令牌授权将包含对用户详细信息的检查，以确保该帐户仍然是活动的
                .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST); // 默认只支持POST
 }
}

![](https://common.cnblogs.com/images/copycode.gif)

**说明：** 

（1）注入AuthenticationManager及UserDetailsService：在配置AuthorizationServerEndpointsConfigurer时使用。

（2）ClientDetailsService配置：与UserDetailsService的配置类似，同样有两种方式，一种是实现ClientDetails/ClientDetailsService接口，从DB中获取Client。另一种是使用InMemoryClientDetailsServiceBuilder在内存中创建Client。这里使用了第一种，也不再贴代码。

（3）AuthorizationServerSecurityConfigurer配置：用来配置Authorization Server令牌端点(Token Endpoint)的安全约束，一般不用配置。

（4）ClientDetailsServiceConfigurer配置：配置ClientDetails。

（5）AuthorizationServerEndpointsConfigurer配置：配置authenticationManager和userDetailsService是告诉Authorization Server使用Spring Security提供的验证管理器及用户详细信息服务。

**配置ResourceServer**

![](https://common.cnblogs.com/images/copycode.gif)

package com.mytools.config; import org.springframework.context.annotation.Configuration; import org.springframework.security.config.annotation.web.builders.HttpSecurity; import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter; import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer; /\*\* \* OAuth2 Resource Server Configuration. \*/ @Configuration public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter { /\*\* \* ResourceId: We can give each of the ResourceServer instance to a resourceId.<br>
     \* Here is the resourceId validation on OAuth2AuthenticationManager#authenticate:<br>
     \* 
     \* <pre>
     \* Collection<String> resourceIds = auth.getOAuth2Request().getResourceIds();
     \* if (resourceId != null && resourceIds != null && !resourceIds.isEmpty() && !resourceIds.contains(resourceId)) {
     \*     throw new OAuth2AccessDeniedException("Invalid token does not contain resource id (" + resourceId + ")");
     \* }
     \* </pre>
     \* 
     \* @throws Exception \*/ @Override public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("server-auth-resource"); super.configure(resources);
    }

    @Override public void configure(HttpSecurity http) throws Exception { // \[IMPORTANT\] 为什么要提前加antMatcher? 可以看一下antMatcher()的注释: // Allows configuring the HttpSecurity to only be invoked when matching the provided ant pattern.
        http.antMatcher("/user").authorizeRequests() // .antMatchers("xxx", "xxx").permitAll()
 .anyRequest().authenticated();
    }
}

![](https://common.cnblogs.com/images/copycode.gif)

主要定义了resource-id及受保护的资源的path及Role&Path的mapping关系

**配置ServerAuthApplication**

![](https://common.cnblogs.com/images/copycode.gif)

package com.mytools; import java.util.HashMap; import java.util.Map; import org.springframework.boot.SpringApplication; import org.springframework.boot.autoconfigure.SpringBootApplication; import org.springframework.security.core.authority.AuthorityUtils; import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer; import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer; import org.springframework.security.oauth2.provider.OAuth2Authentication; import org.springframework.web.bind.annotation.RequestMapping; import org.springframework.web.bind.annotation.RequestParam; import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
@EnableAuthorizationServer
@EnableResourceServer public class ServerAuthApplication2 { public static void main(String\[\] args) {
        SpringApplication.run(ServerAuthApplication2.class, args);
    } /\*\* \* 映射到/user, Resource Server会调用该端点<br>
     \* Resource Server中的@EnableResourceServer会强制执行一个过滤器，<br>
     \* 该拦截器会用传入的token回调\[security.oauth2.resource.userInfoUri\]中定义的URI来查看令牌是否有效。<br>
     \* 此外，该URI还会从Authorization Server传回一个Map，包含Principal and GrantedAuthority信息。<br>
     \* 这个信息是必须的。详细请看：UserInfoTokenServices.loadAuthentication<br>
     \* 
     \* @param user
     \* @return
     \*/ @RequestMapping(value \= "/user", produces = "application/json") public Map<String, Object> user(OAuth2Authentication user, @RequestParam(required = false) String client) {
        Map<String, Object> userInfo = new HashMap<>();
        userInfo.put("user", user.getUserAuthentication().getName());
        userInfo.put("authorities", AuthorityUtils.authorityListToSet(user.getUserAuthentication().getAuthorities())); return userInfo;
    }
}

![](https://common.cnblogs.com/images/copycode.gif)

**说明：** 

（1）使用@EnableAuthorizationServer声明为一个OAuth2 Authorization Server。

（2）使用@EnableResourceServer声明为一个OAuth2 Resource Server。这里声明其为一个Resource Server只是为了保护/user这一个端点。下面/user端点的定义会具体说明原因。

（3）/user Endpoint：Resource Server中的@EnableResourceServer会强制执行一个拦截器，该拦截器会用传入的token回调Resource Server配置文件中定义的security.oauth2.resource.userInfoUri来查看令牌是否有效。这个userInfoUri就映射到了Authorization Server中/user Endpoint。在/user Endpoint中返回了一个含有包含Principal和GrantedAuthority信息的Map。最终Resource Server拿到这个Map，进而判断这个User有没有这个权限来访问Resource Server中的某个path。相关源码可以看UserInfoTokenServices的loadAuthentication方法。

**验证**

**准备数据**

![](https://common.cnblogs.com/images/copycode.gif)

\-- Create user
INSERT INTO user (username, password, email, enabled, create\_user, create\_date\_time, update\_user, update\_date\_time) VALUES ('admin', '{bcrypt}$2a$10$bmixgIna/bd5gU5ORrWng.xUs2sGBh3BRqj927ChKkAvJA8CVGZmm', 'admin@email.com', true, 'admin', '2019-01-01 10:00:00.000', 'admin', '2019-01-01 10:00:00.000'); \-- Create client
INSERT INTO client (client\_id, client\_secret, authorized\_grant\_types\_str, registered\_redirect\_uris\_str, enabled, create\_user, create\_date\_time, update\_user, update\_date\_time) VALUES ('dummy-client', '{bcrypt}$2a$10$nbLJ9DdK/HLlKc.Gm/5S4utfxht9D3mj5M7cm9peFDbBGgTLPEh0u', 'authorization\_code,password', 'https://www.google.com', true, 'admin', '2019-01-01 10:00:00.000', 'admin', '2019-01-01 10:00:00.000'); \-- Create role
INSERT INTO role (rolename, role\_level, enabled, create\_user, create\_date\_time, update\_user, update\_date\_time) VALUES ('ADMIN', 1, true, 'admin', '2019-01-01 10:00:00.000', 'admin', '2019-01-01 10:00:00.000'); \-- Create user\_role\_map
INSERT INTO user\_role\_map (username, rolename) VALUES ('admin', 'ADMIN');

![](https://common.cnblogs.com/images/copycode.gif)

**Authorization Code（授权码模式）**

（1）调用 http://localhost:10030/server-auth/oauth/authorize?response\_type=code&client\_id=dummy-client&state=test-state&redirect\_uri=https://www.google.com&scope=all

如果使用了Zuul，则可以调用 http://localhost:10020/server-zuul/s3/server-auth/oauth/authorize?response\_type=code&client\_id=dummy-client&state=test-state&redirect\_uri=https://www.google.com&scope=all  

如果需要登录，则需要填写登录账号admin 密码admin。

（2）选择scope: all

（3）Get Authorization Code from redirect url: https://www.google.com/?code=ST77hh&state=test-state

（4）Call url with Authorization Code: http://localhost:10030/server-auth/oauth/token?client\_id=dummy-client&grant\_type=authorization\_code&code=ST77hh&redirect\_uri=https://www.google.com&scope=all

如果使用了Zuul，则可以调用 http://localhost:10030/server-auth/oauth/token?client\_id=dummy-client&grant\_type=authorization\_code&code=ST77hh&redirect\_uri=https://www.google.com&scope=all

（5）Http Basic Authorization: username is dummy-client, password is dummy-client

（6）Get json with token: {"access\_token":"8b867ab3-d900-4f1c-947a-b33dc20a91c1","token\_type":"bearer","expires\_in":43085,"scope":"all"}

****Resource Owner Password（密码模式）****

（1）调用 http://localhost:10030/server-auth/oauth/token?client\_id=dummy-client&grant\_type=password&username=admin&password=admin&scope=all

如果使用了Zuul，则可以调用 http://localhost:10020/server-zuul/s3/server-auth/oauth/token?client\_id=dummy-client&grant\_type=password&username=admin&password=admin&scope=all

（2）Http Basic Authorization: username is dummy-client, password is dummy-client

（3）Get json with token: {"access\_token":"8b867ab3-d900-4f1c-947a-b33dc20a91c1","token\_type":"bearer","expires\_in":43085,"scope":"all"}

**\[注1\]** 如果使用Zuul调用，则需要配置以下内容：

（1）server-auth中的application.yml

![](https://common.cnblogs.com/images/copycode.gif)

\## Eureka info
eureka:
  \# 如果设置了true并且也设置了eureka.instance.ip-address那么就将此ip地址注册到Eureka中，那么调用的时候，发送的请求目的地就是此Ip地址
  instance:
    preferIpAddress: true
    ipAddress: localhost

![](https://common.cnblogs.com/images/copycode.gif)

（2）server-zuul中的application.yml

\## Zuul info
zuul:
  \# Zuul不会将敏感HTTP首部（如Cookie,Set-Cookie,Authorization）转发到下游服务。它是一个黑名单。这里排除了Cookie,Set-Cookie,Authorization为后面的OAuth2服务
  sensitiveHeaders:

**其他参数说明**

**\[scope\]**

如果请求中含有scope

 - Client没有配置scopes，可以认为是All Scopes，则可以使用（approve）请求的scope

 - Client配置了一组scopes，如果包含请求的scope，则可以使用（approve）请求的scope；如果不包含，则报错（scope didn't match）

如果请求中没有scope

 - Client没有配置scopes，且请求中也没有，报错（empty scope is not allowed）

 - Client配置了一组scopes，则可以使用（approve）Client的所有scopes

**\[注\]** 注意上面有一个"approve"关键字。如果Client为scopes配置了AutoApprove=true，则会跳过approve这一步。

**\[client\_id\]**

如果在Authorization Server中配置的一个ClientDetails中没有配置resourceId，则这个Client有访问所有resource的权限。

如果在Resource Server没有配置resourceId，则这个resource可以被所有Client有访问。

如果两端都配置了，且Client的resourceIds包含Resource Server的resourceId，这个resource才可以被这个Client访问。

**\[注\]** 代码来源：OAuth2AuthenticationManager#authenticate

**\[client\_secret\]**

在OAuth2标准中，client\_secret并不是required field。但是，在Spring Security中，client\_id/client\_secret被当作了UserDetails，同样会调用AuthenticationProvider.authenticate()方法，最终调用DaoAuthenticationProvider.additionalAuthenticationChecks()，再调用PasswordEncoder的match()方法。PasswordEncoder的实现类（比如DelegatingPasswordEncoder，BCryptPasswordEncoder）在验证空password时都不会通过。

**解决方法：** 可以重写PasswordEncoder实现类的match()方法，也可以设置client\_secret为required field。这里使用后一种。