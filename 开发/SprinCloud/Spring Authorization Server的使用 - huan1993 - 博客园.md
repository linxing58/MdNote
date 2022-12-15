# Spring Authorization Server的使用 - huan1993 - 博客园
在 `Spring Security 5`中，现在已经不提供了 `授权服务器` 的配置，但是 **授权服务器** 在我们平时的开发过程中用的还是比较多的。不过 Spring 官方提供了一个 由Spring官方主导，社区驱动的授权服务 `spring-authorization-server`，目前已经到了 0.1.2 的版本，不过该项目还是一个实验性的项目，不可在生产环境中使用，此处来使用项目搭建一个简单的授权服务器。

1、了解 oauth2 协议、流程。[可以参考阮一峰的这篇文章](https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)  
2、JWT、JWS、JWK的概念

`JWT：`指的是 JSON Web Token，由 header.payload.signture 组成。不存在签名的JWT是不安全的，存在签名的JWT是不可窜改的。  
`JWS：`指的是签过名的JWT，即拥有签名的JWT。  
`JWK：`既然涉及到签名，就涉及到签名算法，对称加密还是非对称加密，那么就需要加密的 密钥或者公私钥对。此处我们将 JWT的密钥或者公私钥对统一称为 JSON WEB KEY，即 JWK。

1、 完成授权码(`authorization-code`)流程。

> 最安全的流程，需要用户的参与。

2、 完成客户端(`client credentials`)流程。

> 没有用户的参与，一般可以用于内部系统之间的访问，或者系统间不需要用户的参与。

3、简化模式在新的 spring-authorization-server 项目中已经被弃用了。  
4、刷新令牌。  
5、撤销令牌。  
6、查看颁发的某个token信息。  
7、查看JWK信息。  
8、个性化JWT token,即给JWT token中增加额外信息。

**完成案例：**   
`张三`通过`QQ登录`的方式来登录`CSDN`网站。  
登录后，CSDN就可以获取到QQ颁发的`token`，CSDN网站拿着token就可以获取张三在QQ资源服务器上的 `个人信息` 了。

**角色分析**  
`张三：` 用户即资源拥有者  
`CSDN：`客户端  
`QQ：`授权服务器  
`个人信息：` 即用户的资源，保存在资源服务器中

1、引入授权服务器依赖
-----------

```null
<dependency>
    <groupId>org.springframework.security.experimental</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>0.1.2</version>
</dependency>

```

2、创建授权服务器用户
-----------

`张三`通过`QQ登录`的方式来登录`CSDN`网站。

此处完成用户`张三`的创建，这个张三是授权服务器的用户，此处即QQ服务器的用户。

```null
@EnableWebSecurity
public class DefaultSecurityConfig {

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeRequests(authorizeRequests ->
                        authorizeRequests.anyRequest().authenticated()
                )
                .formLogin();
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    
    @Bean
    UserDetailsService users() {
        UserDetails user = User.builder()
                .username("zhangsan")
                .password(passwordEncoder().encode("zhangsan123"))
                .roles("USER")
                .build();
        return new InMemoryUserDetailsManager(user);
    }
}

```

3、创建授权服务器和客户端
-------------

`张三`通过`QQ登录`的方式来登录`CSDN`网站。

此处完成QQ授权服务器和客户端CSDN的创建。

```null
package com.huan.study.authorization.config;

import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jose.jwk.source.JWKSource;
import com.nimbusds.jose.proc.SecurityContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.OAuth2AuthorizationServerConfiguration;
import org.springframework.security.config.annotation.web.configurers.oauth2.server.authorization.OAuth2AuthorizationServerConfigurer;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.core.AuthorizationGrantType;
import org.springframework.security.oauth2.core.ClientAuthenticationMethod;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.server.authorization.JdbcOAuth2AuthorizationConsentService;
import org.springframework.security.oauth2.server.authorization.JdbcOAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.OAuth2AuthorizationConsentService;
import org.springframework.security.oauth2.server.authorization.OAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.client.JdbcRegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClient;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.config.ProviderSettings;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.util.matcher.RequestMatcher;

import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.time.Duration;
import java.util.UUID;


@Configuration
public class AuthorizationConfig {

    @Autowired
    private PasswordEncoder passwordEncoder;

	
    class CustomOAuth2TokenCustomizer implements OAuth2TokenCustomizer<JwtEncodingContext> {

        @Override
        public void customize(JwtEncodingContext context) {
            
            context.getHeaders().header("client-id", context.getRegisteredClient().getClientId());
        }
    }

    
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
        
        http.setSharedObject(OAuth2TokenCustomizer.class, new CustomOAuth2TokenCustomizer());
        
        OAuth2AuthorizationServerConfigurer<HttpSecurity> authorizationServerConfigurer =
                new OAuth2AuthorizationServerConfigurer<>();
        RequestMatcher endpointsMatcher = authorizationServerConfigurer.getEndpointsMatcher();

        return http
                .requestMatcher(endpointsMatcher)
                .authorizeRequests(authorizeRequests -> authorizeRequests.anyRequest().authenticated())
                .csrf(csrf -> csrf.ignoringRequestMatchers(endpointsMatcher))
                .apply(authorizationServerConfigurer)
                .and()
                .formLogin()
                .and()
                .build();
    }

    
    @Bean
    public RegisteredClientRepository registeredClientRepository(JdbcTemplate jdbcTemplate) {
        RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
                
                .clientId("csdn")
                
                .clientSecret(passwordEncoder.encode("csdn123"))
                
                .clientAuthenticationMethod(ClientAuthenticationMethod.BASIC)
                
                .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                
                .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
                
                .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                
                .authorizationGrantType(AuthorizationGrantType.PASSWORD)
                
                .authorizationGrantType(AuthorizationGrantType.IMPLICIT)
                
                .redirectUri("https://www.baidu.com")
                
                .scope("user.userInfo")
                .scope("user.photos")
                .clientSettings(clientSettings -> {
                    
                    
                    clientSettings.requireUserConsent(true);
                })
                .tokenSettings(tokenSettings -> {
                    
                    tokenSettings.accessTokenTimeToLive(Duration.ofHours(1));
                    
                    tokenSettings.refreshTokenTimeToLive(Duration.ofDays(3));
                    
                    tokenSettings.reuseRefreshTokens(true);
                })
                .build();

        JdbcRegisteredClientRepository jdbcRegisteredClientRepository = new JdbcRegisteredClientRepository(jdbcTemplate);
        if (null == jdbcRegisteredClientRepository.findByClientId("csdn")) {
            jdbcRegisteredClientRepository.save(registeredClient);
        }

        return jdbcRegisteredClientRepository;
    }

    
    @Bean
    public OAuth2AuthorizationService authorizationService(JdbcTemplate jdbcTemplate, RegisteredClientRepository registeredClientRepository) {
        JdbcOAuth2AuthorizationService authorizationService = new JdbcOAuth2AuthorizationService(jdbcTemplate, registeredClientRepository);

        class CustomOAuth2AuthorizationRowMapper extends JdbcOAuth2AuthorizationService.OAuth2AuthorizationRowMapper {
            public CustomOAuth2AuthorizationRowMapper(RegisteredClientRepository registeredClientRepository) {
                super(registeredClientRepository);
                getObjectMapper().configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
                this.setLobHandler(new DefaultLobHandler());
            }
        }

        CustomOAuth2AuthorizationRowMapper oAuth2AuthorizationRowMapper =
                new CustomOAuth2AuthorizationRowMapper(registeredClientRepository);

        authorizationService.setAuthorizationRowMapper(oAuth2AuthorizationRowMapper);
        return authorizationService;
    }

    
    @Bean
    public OAuth2AuthorizationConsentService authorizationConsentService(JdbcTemplate jdbcTemplate, RegisteredClientRepository registeredClientRepository) {
        return new JdbcOAuth2AuthorizationConsentService(jdbcTemplate, registeredClientRepository);
    }

    
    @Bean
    public JWKSource<SecurityContext> jwkSource() throws NoSuchAlgorithmException {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
        keyPairGenerator.initialize(2048);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        RSAKey rsaKey = new RSAKey.Builder(publicKey)
                .privateKey(privateKey)
                .keyID(UUID.randomUUID().toString())
                .build();
        JWKSet jwkSet = new JWKSet(rsaKey);
        return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
    }

    
    @Bean
    public JwtDecoder jwtDecoder(JWKSource<SecurityContext> jwkSource) {
        return OAuth2AuthorizationServerConfiguration.jwtDecoder(jwkSource);
    }

    
    @Bean
    public ProviderSettings providerSettings() {
        return new ProviderSettings()
                
                .tokenEndpoint("/oauth2/token")
                
                
                .issuer("http://qq.com:8080");
    }
}


```

**注意⚠️：**   
1、需要将 qq.com 在系统的 host 文件中与 127.0.0.1 映射起来。  
2、因为客户端信息、授权信息(token信息等)保存到数据库，因此需要将表建好。  
![](https://img-blog.csdnimg.cn/20210716144226276.png)
  
3、详细信息看上方代码的注释

从上方的代码中可知：

`资源所有者：`张三 用户名和密码为：zhangsan/zhangsan123  
`客户端信息：`CSDN clientId和clientSecret：csdn/csdn123  
`授权服务器地址：` qq.com  
`clientSecret` 的值不可泄漏给客户端，必须保存在服务器端。

1、授权码流程
-------

### 1、获取授权码

> http://qq.com:8080/oauth2/authorize?client\_id=csdn&response\_type=code&redirect\_uri=https://www.baidu.com&scope=user.userInfo user.userInfo

client\_id=csdn：表示客户端是谁  
response\_type=code：表示返回授权码  
scope=user.userInfo user.userInfo：获取多个权限以空格分开  
redirect\_uri=https://www.baidu.com：跳转请求，用户同意或拒绝后

### 2、根据授权码获取token

```null
 curl -i -X POST \
   -H "Authorization:Basic Y3Nkbjpjc2RuMTIz" \
 'http://qq.com:8080/oauth2/token?grant_type=authorization_code&code=tDrZ-LcQDG0julJBcGY5mjtXpE04mpmXjWr9vr0-rQFP7UuNFIP6kFArcYwYo4U-iZXFiDcK4p0wihS_iUv4CBnlYRt79QDoBBXMmQBBBm9jCblEJFHZS-WalCoob6aQ&redirect_uri=https%3A%2F%2Fwww.baidu.com'

```

Authorization： 携带具体的 clientId 和 clientSecret 的base64的值  
grant\_type=authorization\_code 表示采用的方式是授权码  
code=xxx：上一步获取到的授权码

### 3、流程演示

![](https://img-blog.csdnimg.cn/20210716144230580.png)

2、根据刷新令牌获取token
---------------

```null
curl -i -X POST \
   -H "Authorization:Basic Y3Nkbjpjc2RuMTIz" \
 'http://qq.com:8080/oauth2/token?grant_type=refresh_token&refresh_token=Wpu3ruj8FhI-T1pFmnRKfadOrhsHiH1JLkVg2CCFFYd7bYPN-jICwNtPgZIXi3jcWqR6FOOBYWo56W44B5vm374nvM8FcMzTZaywu-pz3EcHvFdFmLJrqAixtTQZvMzx'

```

![](https://img-blog.csdnimg.cn/20210716144232843.png)

3、客户端模式
-------

此模式下，没有用户的参与，只有客户端和授权服务器之间的参与。

```null
curl -i -X POST \
   -H "Authorization:Basic Y3Nkbjpjc2RuMTIz" \
 'http://qq.com:8080/oauth2/token?grant_type=client_credentials'

```

![](https://img-blog.csdnimg.cn/20210716144234346.png)

4、撤销令牌
------

```null
curl -i -X POST \
 'http://qq.com:8080/oauth2/revoke?token=令牌'

```

5、查看token 的信息
-------------

```null
curl -i -X POST \
   -H "Authorization:Basic Y3Nkbjpjc2RuMTIz" \
 'http://qq.com:8080/oauth2/introspect?token=XXX'

```

![](https://img-blog.csdnimg.cn/20210716144234609.png)

6、查看JWK信息
---------

```null
curl -i -X GET \
 'http://qq.com:8080/oauth2/jwks'

```

![](https://img-blog.csdnimg.cn/20210716144234762.png)

[https://gitee.com/huan1993/spring-cloud-parent/tree/master/security/authorization-server](https://gitee.com/huan1993/spring-cloud-parent/tree/master/security/authorization-server)

1、[https://github.com/spring-projects-experimental/spring-authorization-server](https://github.com/spring-projects-experimental/spring-authorization-server)