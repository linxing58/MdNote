# Spring Cloud(6.3)：搭建OAuth2 Resource Server - Storm_L - 博客园
**配置web.xml**

添加spring-cloud-starter-security，spring-security-oauth2-autoconfigure2个依赖。

![](https://common.cnblogs.com/images/copycode.gif)

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

**配置Application**

添加@EnableResourceServer注解，声明为OAuth2 Resource Server。

![](https://common.cnblogs.com/images/copycode.gif)

@SpringBootApplication
@EnableResourceServer // Enable OAuth2 Resource Server
public class ResourceServerApplication { public static void main(String\[\] args) {
        SpringApplication.run(ResourceServerApplication.class, args);
    }
}

![](https://common.cnblogs.com/images/copycode.gif)

**配置Configer及参数**

ResourceServerConfigurer.java

![](https://common.cnblogs.com/images/copycode.gif)

package com.mytools.config; import org.springframework.context.annotation.Configuration; import org.springframework.security.config.annotation.web.builders.HttpSecurity; import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;

@Configuration public class ResourceServerConfigurer extends ResourceServerConfigurerAdapter {

    @Override public void configure(HttpSecurity http) throws Exception { //@formatter:off
 http.authorizeRequests()
                .antMatchers("/structure-search/\*\*", "/data-search/\*\*").hasAnyRole("SQL\_USER")
                .anyRequest().authenticated(); //@formatter:on
 }
}

![](https://common.cnblogs.com/images/copycode.gif)

application.yml

![](https://common.cnblogs.com/images/copycode.gif)

\## Security info
security:
  oauth2:
    resource:
 # 定义一个回调URL调用Authorization Server来查看令牌是否有效
      # use zuul to replace 'http://server-auth/server-auth/user'
      userInfoUri: http://localhost:10020/server-zuul/s3/server-auth/user

![](https://common.cnblogs.com/images/copycode.gif)