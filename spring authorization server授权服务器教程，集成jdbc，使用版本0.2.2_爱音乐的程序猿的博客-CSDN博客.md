# spring authorization server授权服务器教程，集成jdbc，使用版本0.2.2_爱音乐的程序猿的博客-CSDN博客
* * *

spring authorization server是spring团队最新的认证授权服务器，之前的oauth2后面会逐步弃用。不过目前项目还没有到可生产阶段。  
springsecurityoauth迁移到新的授权服务器指南 [https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide](https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide)  
spring authorization server官方demo [https://github.com/spring-projects/spring-authorization-server](https://github.com/spring-projects/spring-authorization-server)  
本文基于官方demo修改

创建springboot启动项目，版本2.6.3

1.引入库
-----

代码如下（示例）：

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        
        
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-authorization-server</artifactId>
            <version>0.2.2</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.8</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-rsa</artifactId>
            <version>1.0.10.RELEASE</version>
        </dependency>

```

2.创建相关数据表
---------

sql示例：

```sql
CREATE TABLE oauth2_registered_client (
    id varchar(100) NOT NULL,
    client_id varchar(100) NOT NULL,
    client_id_issued_at timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL,
    client_secret varchar(200) DEFAULT NULL,
    client_secret_expires_at timestamp DEFAULT NULL,
    client_name varchar(200) NOT NULL,
    client_authentication_methods varchar(1000) NOT NULL,
    authorization_grant_types varchar(1000) NOT NULL,
    redirect_uris varchar(1000) DEFAULT NULL,
    scopes varchar(1000) NOT NULL,
    client_settings varchar(2000) NOT NULL,
    token_settings varchar(2000) NOT NULL,
    PRIMARY KEY (id)
);
CREATE TABLE oauth2_authorization_consent (
    registered_client_id varchar(100) NOT NULL,
    principal_name varchar(200) NOT NULL,
    authorities varchar(1000) NOT NULL,
    PRIMARY KEY (registered_client_id, principal_name)
);

CREATE TABLE oauth2_authorization (
    id varchar(100) NOT NULL,
    registered_client_id varchar(100) NOT NULL,
    principal_name varchar(200) NOT NULL,
    authorization_grant_type varchar(100) NOT NULL,
    attributes blob DEFAULT NULL,
    state varchar(500) DEFAULT NULL,
    authorization_code_value blob DEFAULT NULL,
    authorization_code_issued_at timestamp DEFAULT NULL,
    authorization_code_expires_at timestamp DEFAULT NULL,
    authorization_code_metadata blob DEFAULT NULL,
    access_token_value blob DEFAULT NULL,
    access_token_issued_at timestamp DEFAULT NULL,
    access_token_expires_at timestamp DEFAULT NULL,
    access_token_metadata blob DEFAULT NULL,
    access_token_type varchar(100) DEFAULT NULL,
    access_token_scopes varchar(1000) DEFAULT NULL,
    oidc_id_token_value blob DEFAULT NULL,
    oidc_id_token_issued_at timestamp DEFAULT NULL,
    oidc_id_token_expires_at timestamp DEFAULT NULL,
    oidc_id_token_metadata blob DEFAULT NULL,
    refresh_token_value blob DEFAULT NULL,
    refresh_token_issued_at timestamp DEFAULT NULL,
    refresh_token_expires_at timestamp DEFAULT NULL,
    refresh_token_metadata blob DEFAULT NULL,
    PRIMARY KEY (id)
);


```

3.配置文件
------

```yaml
server:
  port: 9500
spring:
  security:
    oauth2:
      resourceserver:
        jwt: 
          issuer-uri: http://127.0.0.1:9500  
  application:
    name: oauth2-auth
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.3.150:31736/ry-cloud?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
    username: root
    password: root
    druid:
      stat-view-servlet:
        enabled: true
        loginUsername: admin
        loginPassword: 123456
      initial-size: 5
      min-idle: 5
      maxActive: 20
      maxWait: 60000
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true
      maxPoolPreparedStatementPerConnectionSize: 20
      filters: stat,slf4j
      connectionProperties: druid.stat.mergeSql\=true;druid.stat.slowSqlMillis\=5000

```

4.放入官方认证html页面
--------------

![](https://img-blog.csdnimg.cn/6df69800cc2f408190a09046e6c19e9f.png)
  
在这个目录下，代码如下

```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"
          integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
    <title>Custom consent page - Consent required</title>
    <style>
        body {
            background-color: aliceblue;
        }
    </style>
	<script>
		function cancelConsent() {
			document.consent_form.reset();
			document.consent_form.submit();
		}
	</script>
</head>
<body>
<div class="container">
    <div class="py-5">
        <h1 class="text-center text-primary">App permissions</h1>
    </div>
    <div class="row">
        <div class="col text-center">
            <p>
                The application
                <span class="font-weight-bold text-primary" th:text="${clientId}"></span>
                wants to access your account
                <span class="font-weight-bold" th:text="${principalName}"></span>
            </p>
        </div>
    </div>
    <div class="row pb-3">
        <div class="col text-center"><p>The following permissions are requested by the above app.<br/>Please review
            these and consent if you approve.</p></div>
    </div>
    <div class="row">
        <div class="col text-center">
            <form name="consent_form" method="post" action="/oauth2/authorize">
                <input type="hidden" name="client_id" th:value="${clientId}">
                <input type="hidden" name="state" th:value="${state}">

                <div th:each="scope: ${scopes}" class="form-group form-check py-1">
                    <input class="form-check-input"
                           type="checkbox"
                           name="scope"
                           th:value="${scope.scope}"
                           th:id="${scope.scope}">
                    <label class="form-check-label font-weight-bold" th:for="${scope.scope}" th:text="${scope.scope}"></label>
                    <p class="text-primary" th:text="${scope.description}"></p>
                </div>

                <p th:if="${not #lists.isEmpty(previouslyApprovedScopes)}">You have already granted the following permissions to the above app:</p>
                <div th:each="scope: ${previouslyApprovedScopes}" class="form-group form-check py-1">
                    <input class="form-check-input"
                           type="checkbox"
                           th:id="${scope.scope}"
                           disabled
                           checked>
                    <label class="form-check-label font-weight-bold" th:for="${scope.scope}" th:text="${scope.scope}"></label>
                    <p class="text-primary" th:text="${scope.description}"></p>
                </div>

                <div class="form-group pt-3">
                    <button class="btn btn-primary btn-lg" type="submit" id="submit-consent">
                        Submit Consent
                    </button>
                </div>
                <div class="form-group">
                    <button class="btn btn-link regular" type="button" id="cancel-consent" onclick="cancelConsent();">
                        Cancel
                    </button>
                </div>
            </form>
        </div>
    </div>
    <div class="row pt-4">
        <div class="col text-center">
            <p>
                <small>
                    Your consent to provide access is required.
                    <br/>If you do not approve, click Cancel, in which case no information will be shared with the app.
                </small>
            </p>
        </div>
    </div>
</div>
</body>
</html>


```

5.生成jks文件
---------

windows下CMD命令窗口输入

```bash
keytool -genkeypair -alias shy_debug.jks -keyalg RSA -validity 7 -keystore shy_debug.jks

```

alias别名  
然后根据提示输入相关信息，记好密码和别名，后面要用到  
把生成的jks文件放到这里  
![](https://img-blog.csdnimg.cn/5321f8de2191415082e48114d6aa0bf1.png)

6.配置KeyPair
-----------

```java
@Configuration
public class KeyPairConfig {

    @Bean
    public KeyPair keyPair() throws Exception {
        ClassPathResource ksFile = new ClassPathResource("shy_debug.jks");
        KeyStoreKeyFactory ksFactory = new KeyStoreKeyFactory(ksFile, "haiwei".toCharArray());  
        return ksFactory.getKeyPair("shy_debug.jks");
    }
}

```

7.配置AuthorizationServerConfig授权服务器配置
------------------------------------

```java
@Configuration(proxyBeanMethods = false)
public class AuthorizationServerConfig {
	private static final String CUSTOM_CONSENT_PAGE_URI = "/oauth2/consent";
	@Autowired
	private KeyPair keyPair;
	@Bean
	@Order(Ordered.HIGHEST_PRECEDENCE)
	public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
		OAuth2AuthorizationServerConfigurer<HttpSecurity> authorizationServerConfigurer =
				new OAuth2AuthorizationServerConfigurer<>();
		authorizationServerConfigurer
				.authorizationEndpoint(authorizationEndpoint ->
						authorizationEndpoint.consentPage(CUSTOM_CONSENT_PAGE_URI));

		RequestMatcher endpointsMatcher = authorizationServerConfigurer
				.getEndpointsMatcher();

		http
			.requestMatcher(endpointsMatcher)
			.authorizeRequests(authorizeRequests -> authorizeRequests
						.anyRequest().authenticated()
			)
			.csrf(csrf -> csrf.ignoringRequestMatchers(endpointsMatcher))
			.apply(authorizationServerConfigurer);
		return http.formLogin(Customizer.withDefaults()).build();
	}

	
	@Bean
	public RegisteredClientRepository registeredClientRepository(JdbcTemplate jdbcTemplate) {
		
		
		RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
				.clientId("messaging-client")
				.clientSecret("{noop}secret")
				.clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
				.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
				.authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
				.authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
				.redirectUri("http://127.0.0.1:8080/login/oauth2/code/messaging-client-oidc")
				.redirectUri("http://www.baidu.com")
				.scope(OidcScopes.OPENID)
				.scope("message.read")
				.scope("message.write")
				.clientSettings(ClientSettings.builder().requireAuthorizationConsent(true).build())
				.build();

		
		JdbcRegisteredClientRepository registeredClientRepository = new JdbcRegisteredClientRepository(jdbcTemplate);
		registeredClientRepository.save(registeredClient);
		return registeredClientRepository;
	}
	

	@Bean
	public OAuth2AuthorizationService authorizationService(JdbcTemplate jdbcTemplate, RegisteredClientRepository registeredClientRepository) {
		return new JdbcOAuth2AuthorizationService(jdbcTemplate, registeredClientRepository);
	}

	@Bean
	public OAuth2AuthorizationConsentService authorizationConsentService(JdbcTemplate jdbcTemplate, RegisteredClientRepository registeredClientRepository) {
		return new JdbcOAuth2AuthorizationConsentService(jdbcTemplate, registeredClientRepository);
	}

	@Bean
	public JWKSource<SecurityContext> jwkSource() {
		RSAKey rsaKey = generateRsa();
		JWKSet jwkSet = new JWKSet(rsaKey);
		return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
	}
	public RSAKey generateRsa() {
		RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
		RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
		
		return new RSAKey.Builder(publicKey)
				.privateKey(privateKey)
				.keyID(UUID.randomUUID().toString())
				.build();
		
	}
	@Bean
	public ProviderSettings providerSettings() {
		return ProviderSettings.builder().issuer("http://127.0.0.1:9500").build();
	}

}


```

8.配置WebSecurityConfig基础security配置
---------------------------------

```
`@Configuration
@EnableWebSecurity(debug = true)
public class WebSecurityConfig {
    @Bean
    public SecurityFilterChain httpSecurityFilterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity
                .authorizeRequests(authorizeRequests ->
                        authorizeRequests.anyRequest().authenticated()
                )
                .formLogin(withDefaults());
        return httpSecurity.build();
    }
    
    
    @Bean
    UserDetailsService users() {
        UserDetails user = User.withDefaultPasswordEncoder()
                .username("admin")
                .password("111111")
                .roles("USER").authorities("test")
                .build();
        return new InMemoryUserDetailsManager(user);
    }
    
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37


```

```
`@Controller
public class AuthorizationConsentController {
	private final RegisteredClientRepository registeredClientRepository;
	private final OAuth2AuthorizationConsentService authorizationConsentService;

	public AuthorizationConsentController(RegisteredClientRepository registeredClientRepository,
                                          OAuth2AuthorizationConsentService authorizationConsentService) {
		this.registeredClientRepository = registeredClientRepository;
		this.authorizationConsentService = authorizationConsentService;
	}

	@GetMapping(value = "/oauth2/consent")
	public String consent(Principal principal, Model model,
			@RequestParam(OAuth2ParameterNames.CLIENT_ID) String clientId,
			@RequestParam(OAuth2ParameterNames.SCOPE) String scope,
			@RequestParam(OAuth2ParameterNames.STATE) String state) {

		
		Set<String> scopesToApprove = new HashSet<>();
		Set<String> previouslyApprovedScopes = new HashSet<>();
		RegisteredClient registeredClient = this.registeredClientRepository.findByClientId(clientId);
		OAuth2AuthorizationConsent currentAuthorizationConsent =
				this.authorizationConsentService.findById(registeredClient.getId(), principal.getName());
		Set<String> authorizedScopes;
		if (currentAuthorizationConsent != null) {
			authorizedScopes = currentAuthorizationConsent.getScopes();
		} else {
			authorizedScopes = Collections.emptySet();
		}
		for (String requestedScope : StringUtils.delimitedListToStringArray(scope, " ")) {
			if (authorizedScopes.contains(requestedScope)) {
				previouslyApprovedScopes.add(requestedScope);
			} else {
				scopesToApprove.add(requestedScope);
			}
		}

		model.addAttribute("clientId", clientId);
		model.addAttribute("state", state);
		model.addAttribute("scopes", withDescription(scopesToApprove));
		model.addAttribute("previouslyApprovedScopes", withDescription(previouslyApprovedScopes));
		model.addAttribute("principalName", principal.getName());

		return "consent";
	}

	private static Set<ScopeWithDescription> withDescription(Set<String> scopes) {
		Set<ScopeWithDescription> scopeWithDescriptions = new HashSet<>();
		for (String scope : scopes) {
			scopeWithDescriptions.add(new ScopeWithDescription(scope));

		}
		return scopeWithDescriptions;
	}

	public static class ScopeWithDescription {
		private static final String DEFAULT_DESCRIPTION = "UNKNOWN SCOPE - We cannot provide information about this permission, use caution when granting this.";
		private static final Map<String, String> scopeDescriptions = new HashMap<>();
		static {
			scopeDescriptions.put(
					"message.read",
					"This application will be able to read your message."
			);
			scopeDescriptions.put(
					"message.write",
					"This application will be able to add new messages. It will also be able to edit and delete existing messages."
			);
			scopeDescriptions.put(
					"other.scope",
					"This is another scope example of a scope description."
			);
		}

		public final String scope;
		public final String description;

		ScopeWithDescription(String scope) {
			this.scope = scope;
			this.description = scopeDescriptions.getOrDefault(scope, DEFAULT_DESCRIPTION);
		}
	}
}` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62
*   63
*   64
*   65
*   66
*   67
*   68
*   69
*   70
*   71
*   72
*   73
*   74
*   75
*   76
*   77
*   78
*   79
*   80
*   81
*   82


```

至此配置完毕

浏览器输入 [http://127.0.0.1:9500/oauth2/authorize?client\_id=messaging-client&response\_type=code&scope=message.read&redirect\_uri=http://www.baidu.com](http://127.0.0.1:9500/oauth2/authorize?client_id=messaging-client&response_type=code&scope=message.read&redirect_uri=http://www.baidu.com)  
会转到登陆页面  
![](https://img-blog.csdnimg.cn/ee50f13ea5cb4f06ad47f194067bbb63.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix6Z-z5LmQ55qE56iL5bqP54y_,size_15,color_FFFFFF,t_70,g_se,x_16)
  
输入账号密码会跳转到认证授权页面  
![](https://img-blog.csdnimg.cn/334169c147ff49ca87c635ddac23bd40.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix6Z-z5LmQ55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)
  
勾选上scope，点认证  
![](https://img-blog.csdnimg.cn/2cd471d0989c4a8fab06ac19cb4053e3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix6Z-z5LmQ55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)
  
这样就获取到code了  
然后用postman请求  
![](https://img-blog.csdnimg.cn/f3f5704595cf4b9b93b7f0764a9a88f5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix6Z-z5LmQ55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/817e63f2f3af48d49a111c39a17d6f8a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix6Z-z5LmQ55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)
  
这样就获取到token了

spring authorization server搭建完成，后面一篇文章我会说明资源服务器端如何接入认证服务器，资源服务器文章链接  
spring authorization server授权服务器教程，资源服务器搭建接入认证服务器[https://blog.csdn.net/qq\_35270805/article/details/123146959](https://blog.csdn.net/qq_35270805/article/details/123146959)