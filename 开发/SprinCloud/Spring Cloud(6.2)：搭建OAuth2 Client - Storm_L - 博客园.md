# Spring Cloud(6.2)：搭建OAuth2 Client - Storm_L - 博客园
**配置web.xml**

添加spring-cloud-starter-security，spring-security-oauth2-autoconfigure和spring-boot-starter-oauth2-client 3个依赖。

![](https://common.cnblogs.com/images/copycode.gif)

<!\-- Spring cloud starter: Security -->
<!\-- Include: web, actuator, security, zuul, etc. -->
<dependency\>
    <groupId\>org.springframework.cloud</groupId\>
    <artifactId\>spring\-cloud\-starter\-security</artifactId\>
</dependency\>
<!\-- Spring Security OAuth2 Autoconfigure (optional in spring-cloud-security after 2.1) -->
<dependency\>
    <groupId\>org.springframework.security.oauth.boot</groupId\>
    <artifactId\>spring\-security\-oauth2\-autoconfigure</artifactId\>
</dependency\>
<!\-- Spring Security OAuth2 Client (optional in spring-cloud-security after 2.1) -->
<dependency\>
    <groupId\>org.springframework.boot</groupId\>
    <artifactId\>spring\-boot\-starter\-oauth2\-client</artifactId\>
</dependency\>

![](https://common.cnblogs.com/images/copycode.gif)

此外，它还是一个Eureka Client和Config Client，如何配置Eureka Client和Config Client请看前面章节。

**配置Application**

添加@EnableOAuth2Client注解，声明为OAuth2 Client。

![](https://common.cnblogs.com/images/copycode.gif)

@SpringBootApplication
@EnableOAuth2Client public class AppSqlApplication { public static void main(String\[\] args) {
        SpringApplication.run(AppSqlApplication.class, args);
    }
}

![](https://common.cnblogs.com/images/copycode.gif)

**SSO单点登录的**配置****

**（1）配置Configer**

该服务作为一个OAuth2 Client，可以使用上一节的OAuth2 Server来登录。这其实就是SSO单点登录的例子。

![](https://common.cnblogs.com/images/copycode.gif)

package com.mycloud.demo.config; import java.util.ArrayList; import java.util.List; import org.springframework.context.annotation.Configuration; import org.springframework.security.config.annotation.web.builders.HttpSecurity; import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter; import org.springframework.security.core.GrantedAuthority; import org.springframework.security.core.authority.SimpleGrantedAuthority; import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService; import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest; import org.springframework.security.oauth2.client.userinfo.OAuth2UserService; import org.springframework.security.oauth2.core.user.DefaultOAuth2User; import org.springframework.security.oauth2.core.user.OAuth2User;

@Configuration public class AppSqlConfiger2 extends WebSecurityConfigurerAdapter { /\* (non-Javadoc)
    \* @see org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter#configure(org.springframework.security.config.annotation.web.builders.HttpSecurity) \*/ @Override protected void configure(HttpSecurity http) throws Exception { //@formatter:off
 http.authorizeRequests()
                .antMatchers("/", "/index\*\*", "/login\*\*", "/error\*\*").permitAll()
                .antMatchers("/sql-sp-search/\*\*").hasRole("SQL\_USER")
                .antMatchers("/sql-tr-search/\*\*").hasRole("SQL\_USER")
            .and().logout().logoutSuccessUrl("/")
            .and().oauth2Login().userInfoEndpoint().userService(userService()) ; //@formatter:on
 }

    @SuppressWarnings("unchecked") private OAuth2UserService<OAuth2UserRequest, OAuth2User> userService() { //@formatter:off // Why configure this? // The default user-authority is DefaultOAuth2UserService.java, OAuth2UserAuthority.java and DefaultOAuth2User.java. // But the authority is set to ROLE\_USER instead of authorities come from user-info endpoint on OAuth2UserAuthority. // So we need to reset the authority again after 'DefaultOAuth2UserService.loadUser()'. //@formatter:on
        final OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService(); return (userRequest) -> {
            OAuth2User oauthUser \= delegate.loadUser(userRequest);

            List<GrantedAuthority> authorities = new ArrayList<>();
            Object obj \= oauthUser.getAttributes().get("authorities"); if (obj != null) {
                List<String> authoritiesStrList = (List<String>) obj; for (String elem : authoritiesStrList) {
                    GrantedAuthority authority \= new SimpleGrantedAuthority(elem);
                    authorities.add(authority);
                }
            } return new DefaultOAuth2User(authorities, oauthUser.getAttributes(), "user");
        };
    }
}

![](https://common.cnblogs.com/images/copycode.gif)

**\[注1\]** 我们在这里使用了oauth2Login()及自定义了一个OAuth2UserService。OAuth2UserService实际上就是上一节中讲的调用Authorization Server的/user端点拿到的User信息。至于为什么自定义了一个OAuth2UserService，可以看代码中的注释。

**（2）配置参数**

![](https://common.cnblogs.com/images/copycode.gif)

\## Spring info
spring:
  \# OAuth2 Client info (see ClientRegistration.java) 
  security:
    oauth2:
      client:
        registration:
          server-auth:
            client-id: app-sql-client
            client-secret: '{cipher}e93cce4a8056a7359ded238e97a1c6d25142e6b688873a1e6181ac06753dd9ae'
            \# basic or post (default is basic)
            # client-authentication-method: basic
            # authorizationGrantType cannot be null, authorizationGrantType must be authorization\_code
            authorization-grant-type: authorization\_code
            \# redirectUriTemplate cannot be empty, default is '{baseUrl}/login/oauth2/code/{registrationId}'
            redirect-uri: http://localhost:10200/app-sql/login/oauth2/code/server-auth
            scope: all
            \# client-name: app-sql
        provider:
          server-auth:
 # It's very important to add 'scope=all' to the url. Auth Server didn't config the scope, so in client side,
            # we must config it, otherwise, 'Empty scope (either the client or the user is not allowed the requested scopes)'
            # error occurred.
            # use zuul to replace 'http://localhost:10030/server-auth/xxx'
            token-uri: http://localhost:10020/server-zuul/s3/server-auth/oauth/token?scope=all
            authorization-uri: http://localhost:10020/server-zuul/s3/server-auth/oauth/authorize
            user-info-uri: http://localhost:10020/server-zuul/s3/server-auth/user
 # header, form or query (default is header)
            # user-info-authentication-method: header
            # the key used to get the user's "name"
            user-name-attribute: user

![](https://common.cnblogs.com/images/copycode.gif)

**使用OAuth2RestTemplate在Client端调用被OAuth2保护的Resource**

上面几步实际上是搭建一个SSO登录系统，但是如果我们想要在OAuth2 Client端调用OAuth2 Resource时，就需要做一些额外配置了。OAuth2 Resource的配置会在下一节详细介绍，这里主要来讲在OAuth2 Client端如何配置。

在OAuth2 Client端配置的核心就是**OAuth2RestTemplate**。我们通过给OAuth2RestTemplate配置好所有访问OAuth2 Authorization Server的参数创建OAuth2RestTemplate，接下来对OAuth2 Resource的调用交给OAuth2RestTemplate即可。

这里使用password模式为例。

**（1）配置Configer**

![](https://common.cnblogs.com/images/copycode.gif)

@Configuration public class AppSqlConfiger {

    @Bean
    @ConfigurationProperties("app-sql.resources.app-db") protected OAuth2ProtectedResourceDetails resource() { return new ResourceOwnerPasswordResourceDetails();
    }

    @Bean public OAuth2RestTemplate restTemplate() {
        AccessTokenRequest atr \= new DefaultAccessTokenRequest(); return new OAuth2RestTemplate(resource(), new DefaultOAuth2ClientContext(atr));
    }
}

![](https://common.cnblogs.com/images/copycode.gif)

**（2）**配置参数****

![](https://common.cnblogs.com/images/copycode.gif)

\## Regard 'app-sql' as oauth2-client and 'app-db' as oauth2-resource.
## Create OAuth2RestTemplate in 'app-sql' and set parameters(see AppSqlConfiger.java) and call 'app-db'.
app-sql:
  resources:
    app-db:
      clientId: app-sql-client
      clientSecret: '{cipher}e93cce4a8056a7359ded238e97a1c6d25142e6b688873a1e6181ac06753dd9ae'
      grantType: password
      \# use zuul to replace 'http://localhost:10030/server-auth/xxx'
      accessTokenUri: http://localhost:10020/server-zuul/s3/server-auth/oauth/token
      scope: all
      username: '{cipher}18a05787c976397d4d7d090ad326cc7417122109f590d8a8ba80cfa35f7a15c3'
      password: '{cipher}ab3894c202393761b8f789dd9f047e116b2008ae98b5f8119030796f905471d8'

![](https://common.cnblogs.com/images/copycode.gif)

**再看Discovery Client**

在Eureka Client一节中，我们讲了使用discoveryClient和loadBalancer来发现服务。现在，我们可以通过zuul来调用其他服务，和loadBalancer类似：

ServiceInstance instance = loadBalancer.choose("server-zuul");
String path \= String.format("http://%s:%s/server-zuul/a2/app-db/structure-search/app/MORT/env/%s/db/%s/name/%s", instance.getHost(), instance.getPort(), env, db, name);
ResponseEntity<String> response = restTemplate.exchange(path, HttpMethod.GET, null, String.class);