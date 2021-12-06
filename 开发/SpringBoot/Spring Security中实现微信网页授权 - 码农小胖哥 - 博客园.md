# Spring Security中实现微信网页授权 - 码农小胖哥 - 博客园
微信公众号提供了微信支付、微信优惠券、微信 H5 红包、微信红包封面等等促销工具来帮助我们的应用拉新保活。但是这些福利要想正确地发放到用户的手里就必须拿到用户特定的（微信应用）微信标识`openid`甚至是用户的微信用户信息。如果用户在微信客户端中访问我们第三方网页，公众号可以通过微信网页授权机制，来获取用户基本信息，进而实现业务逻辑。今天就结合**Spring Security**来实现一下微信公众号网页授权。

## 环境准备

在开始之前我们需要准备好微信网页开发的环境。

### 微信公众号服务号

请注意，一定是**微信公众号服务号**，只有**服务号**才提供这样的能力。像胖哥的这样公众号虽然也是认证过的公众号，但是只能发发文章并不具备提供服务的能力。但是微信公众平台提供了沙盒功能来模拟服务号，可以降低开发难度，你可以到[微信公众号测试账号页面](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)申请，申请成功后别忘了关注测试公众号。

> 微信公众号服务号只有企事业单位、政府机关才能开通。

### 内网穿透

因为微信服务器需要回调开发者提供的回调接口，为了能够本地调试，内网穿透工具也是必须的。启动内网穿透后，需要把内网穿透工具提供的**虚拟域名**配置到微信测试帐号的回调配置中

![](https://img2020.cnblogs.com/other/1739473/202108/1739473-20210815143010980-778674957.png)

打开后**只需要填写域名，不要带协议头**。例如回调是`https://felord.cn/wechat/callback`，只能填写成这样：

![](https://img2020.cnblogs.com/other/1739473/202108/1739473-20210815143011147-37491922.png)

然后我们就可以开发了。

## OAuth2.0 客户端集成

> 基于 **Spring Security 5.x**

微信网页授权的文档在[网页授权](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html)，这里不再赘述。我们只聊聊如何结合 Spring Security 的事。微信网页授权是通过**OAuth2.0**机制实现的，在用户授权给公众号后，公众号可以获取到一个网页授权特有的接口调用凭证（网页授权`access_token`），通过网页授权获得的`access_token`可以进行授权后接口调用，如获取用户的基本信息。

我们需要引入 Spring Security 提供的 OAuth2.0 相关的模块：

```null
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-client</artifactId>
        </dependency>
```

> 由于我们需要获取用户的微信信息，所以要用到`OAuth2.0 Login`；如果你用不到用户信息可以选择`OAuth2.0 Client`。

## 微信网页授权流程

接着按照微信提供的流程来结合 Spring Security。

### 获取授权码 code

微信网页授权使用的是**OAuth2.0**的授权码模式。我们先来看如何获取授权码。

```null
https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE
```

这是微信获取`code`的**OAuth2.0**端点模板，这不是一个纯粹的**OAuth2.0**协议。微信做了一些参数上的变动。这里原生的`client_id`被替换成了`appid`，而且末尾还要加`#wechat_redirect` 。这无疑增加了集成的难度。

这里先放一放，我们目标转向**Spring Security**的`code`获取流程。

**Spring Security**会提供一个模版链接：

```null
{baseUrl}/oauth2/authorization/{registrationId}
```

当使用该链接请求**OAuth2.0**客户端时会被`OAuth2AuthorizationRequestRedirectFilter`拦截。机制这里不讲了，在我个人博客`felord.cn`中的`Spring Security 实战干货：客户端 OAuth2 授权请求的入口`一文中有详细阐述。

拦截之后会根据配置组装获取授权码的请求 URL，由于微信的不一样所以我们针对性的定制，也就是改造`OAuth2AuthorizationRequestRedirectFilter`中的`OAuth2AuthorizationRequestResolver`。

#### 自定义 URL

因为 Spring Security 会根据模板链接去组装一个链接而不是我们填参数就行了，所以需要我们对构建 URL 的处理器进行自定义。

```null

public class WechatOAuth2AuthRequestBuilderCustomizer {
   private static final String WECHAT_ID= "wechat";

    
    public static void customize(OAuth2AuthorizationRequest.Builder builder) {
       String regId = (String) builder.build()
               .getAttributes()
               .get(OAuth2ParameterNames.REGISTRATION_ID);
       if (WECHAT_ID.equals(regId)){
           builder.authorizationRequestUri(WechatOAuth2RequestUriBuilderCustomizer::customize);
       }
    }

    
    private static class WechatOAuth2RequestUriBuilderCustomizer {

        
        public static URI customize(UriBuilder builder) {
            String reqUri = builder.build().toString()
                    .replaceAll("client_id=", "appid=")
                    .concat("#wechat_redirect");
            return URI.create(reqUri);
        }
    }
}
```

#### 配置解析器

把上面个性化改造的逻辑配置到`OAuth2AuthorizationRequestResolver`:

```null

private OAuth2AuthorizationRequestResolver oAuth2AuthorizationRequestResolver(ClientRegistrationRepository clientRegistrationRepository) {
    DefaultOAuth2AuthorizationRequestResolver resolver = new DefaultOAuth2AuthorizationRequestResolver(clientRegistrationRepository,
            OAuth2AuthorizationRequestRedirectFilter.DEFAULT_AUTHORIZATION_REQUEST_BASE_URI);
    resolver.setAuthorizationRequestCustomizer(WechatOAuth2AuthRequestBuilderCustomizer::customize);
    return resolver;
}
```

#### 配置到 Spring Security

适配好的`OAuth2AuthorizationRequestResolver`配置到`HttpSecurity`, 伪代码:

```null
    httpSecurity.oauth2Login()
                
                .authorizationEndpoint().authorizationRequestResolver(authorizationRequestResolver)
```

### 通过 code 换取网页授权 access_token

接下来第二步是用`code`去换`token`。

#### 构建请求参数

这是微信网页授权获取`access_token`的模板：

```null
GET https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN
```

其中前半段`https://api.weixin.qq.com/sns/oauth2/refresh_token`可以通过配置 OAuth2.0 的`token-uri`来指定；后半段参数需要我们针对微信进行定制。**Spring Security**中定制`token-uri`的工具由`OAuth2AuthorizationCodeGrantRequestEntityConverter`这个转换器负责，这里需要来改造一下。

我们先拼接参数：

```null
    private MultiValueMap<String, String> buildWechatQueryParameters(OAuth2AuthorizationCodeGrantRequest authorizationCodeGrantRequest) {
        
        ClientRegistration clientRegistration = authorizationCodeGrantRequest.getClientRegistration();
        OAuth2AuthorizationExchange authorizationExchange = authorizationCodeGrantRequest.getAuthorizationExchange();
        MultiValueMap<String, String> formParameters = new LinkedMultiValueMap<>();
        
        formParameters.add(OAuth2ParameterNames.GRANT_TYPE, authorizationCodeGrantRequest.getGrantType().getValue());
        
        formParameters.add(OAuth2ParameterNames.CODE, authorizationExchange.getAuthorizationResponse().getCode());
        
        String redirectUri = authorizationExchange.getAuthorizationRequest().getRedirectUri();
        if (redirectUri != null) {
            formParameters.add(OAuth2ParameterNames.REDIRECT_URI, redirectUri);
        }
        
        formParameters.add("appid", clientRegistration.getClientId());
        
        formParameters.add("secret", clientRegistration.getClientSecret());
        return formParameters;
    }
```

然后生成`RestTemplate`的请求对象`RequestEntity`:

```null
    @Override
    public RequestEntity<?> convert(OAuth2AuthorizationCodeGrantRequest authorizationCodeGrantRequest) {
        ClientRegistration clientRegistration = authorizationCodeGrantRequest.getClientRegistration();
        HttpHeaders headers = getTokenRequestHeaders(clientRegistration);


        String tokenUri = clientRegistration.getProviderDetails().getTokenUri();
        
        if (WECHAT_ID.equals(clientRegistration.getRegistrationId())) {
            MultiValueMap<String, String> queryParameters = this.buildWechatQueryParameters(authorizationCodeGrantRequest);
            URI uri = UriComponentsBuilder.fromUriString(tokenUri).queryParams(queryParameters).build().toUri();
            return RequestEntity.get(uri).headers(headers).build();
        }
        
        MultiValueMap<String, String> formParameters = this.buildFormParameters(authorizationCodeGrantRequest);
        URI uri = UriComponentsBuilder.fromUriString(tokenUri).build()
                .toUri();
        return new RequestEntity<>(formParameters, headers, HttpMethod.POST, uri);
    }
```

这样兼容性就改造好了。

#### 兼容 token 返回解析

微信公众号授权`token-uri`的返回值虽然文档说是个`json`，可它喵的`Content-Type`是`text-plain`。如果是`application/json`，**Spring Security**就直接接收了。你说微信坑不坑？我们只能再写个适配来正确的反序列化微信接口的返回值。

Spring Security 中对`token-uri`的返回值的解析转换同样由`OAuth2AccessTokenResponseClient`中的`OAuth2AccessTokenResponseHttpMessageConverter`负责。

首先增加`Content-Type`为`text-plain`的适配；其次因为**Spring Security**接收`token`返回的对象要求必须显式声明`tokenType`，而微信返回的响应体中没有，我们一律指定为`OAuth2AccessToken.TokenType.BEARER`即可兼容。代码比较简单就不放了，有兴趣可以去看我给的 DEMO。

#### 配置到 Spring Security

先配置好我们上面两个步骤的请求客户端：

```null
    
    private OAuth2AccessTokenResponseClient<OAuth2AuthorizationCodeGrantRequest> accessTokenResponseClient() {
        DefaultAuthorizationCodeTokenResponseClient tokenResponseClient = new DefaultAuthorizationCodeTokenResponseClient();
        tokenResponseClient.setRequestEntityConverter(new WechatOAuth2AuthorizationCodeGrantRequestEntityConverter());

        OAuth2AccessTokenResponseHttpMessageConverter tokenResponseHttpMessageConverter = new OAuth2AccessTokenResponseHttpMessageConverter();
        
        tokenResponseHttpMessageConverter.setSupportedMediaTypes(Arrays.asList(MediaType.APPLICATION_JSON,
                MediaType.TEXT_PLAIN,
                new MediaType("application", "*+json")));
        
        tokenResponseHttpMessageConverter.setTokenResponseConverter(new WechatMapOAuth2AccessTokenResponseConverter());

        RestTemplate restTemplate = new RestTemplate(
                Arrays.asList(new FormHttpMessageConverter(),
                        tokenResponseHttpMessageConverter
                ));

        restTemplate.setErrorHandler(new OAuth2ErrorResponseErrorHandler());
        tokenResponseClient.setRestOperations(restTemplate);
        return tokenResponseClient;
    }
```

再把请求客户端配置到`HttpSecurity`：

```null
   
httpSecurity.oauth2Login()
   .tokenEndpoint().accessTokenResponseClient(accessTokenResponseClient)
```

### 根据 token 获取用户信息

> 微信公众号网页授权获取用户信息需要`scope`包含`snsapi_userinfo`。

Spring Security 中定义了一个 OAuth2.0 获取用户信息的抽象接口：

```null
@FunctionalInterface
public interface OAuth2UserService<R extends OAuth2UserRequest, U extends OAuth2User> {

	U loadUser(R userRequest) throws OAuth2AuthenticationException;

}
```

所以我们针对性的实现即可，需要实现三个相关概念。

#### OAuth2UserRequest

`OAuth2UserRequest`是请求`user-info-uri`的入参实体，包含了三大块属性：

-   `ClientRegistration` 微信**OAuth2.0**客户端配置
-   `OAuth2AccessToken` 从`token-uri`获取的`access_token`的抽象实体
-   `additionalParameters` 一些`token-uri`返回的额外参数，比如`openid`就可以从这里面取得

根据微信获取用户信息的端点 API 这个能满足需要，不过需要注意的是。如果使用的是 **OAuth2.0 Client** 就无法从`additionalParameters`获取`openid`等额外参数。

#### OAuth2User

这个用来封装微信用户信息，细节看下面的注释：

```null

@Data
public class WechatOAuth2User implements OAuth2User {
    private String openid;
    private String nickname;
    private Integer sex;
    private String province;
    private String city;
    private String country;
    private String headimgurl;
    private List<String> privilege;
    private String unionid;


    @Override
    public Map<String, Object> getAttributes() {
        
        return Collections.emptyMap();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        
        return null;
    }

    @Override
    public String getName() {
        
        return openid;
    }
}
```

> **注意：**  `getName()`一定不能返回`null`。

#### OAuth2UserService

参数`OAuth2UserRequest`和返回值`OAuth2User`都准备好了，就剩下去请求微信服务器了。借鉴请求`token-uri`的实现，还是一个`RestTemplate`调用，核心就这几行：

```null
LinkedMultiValueMap<String, String> queryParams = new LinkedMultiValueMap<>();

queryParams.add(OAuth2ParameterNames.ACCESS_TOKEN, userRequest.getAccessToken().getTokenValue());

queryParams.add(OPENID_KEY, String.valueOf(userRequest.getAdditionalParameters().get(OPENID_KEY)));

queryParams.add(LANG_KEY, DEFAULT_LANG);

URI userInfoEndpoint = UriComponentsBuilder.fromUriString(userInfoUri).queryParams(queryParams).build().toUri();

return this.restOperations.exchange(userInfoEndpoint, HttpMethod.GET, null, OAUTH2_USER_OBJECT);
```

#### 配置到 Spring Security

```null

httpSecurity.oauth2Login()
      .userInfoEndpoint().userService(oAuth2UserService);
```

这里补充一下，写一个授权成功后跳转的接口并配置为授权登录成功后的跳转的 url。

```null

httpSecurity.oauth2Login().defaultSuccessUrl("/weixin/h5/redirect")
```

在这个接口里可以通过`@RegisteredOAuth2AuthorizedClient`和`@AuthenticationPrincipal`分别拿到认证客户端的信息和用户信息。

```null
@GetMapping("/h5/redirect")
public void sendRedirect(HttpServletResponse response,
                         @RegisteredOAuth2AuthorizedClient("wechat") OAuth2AuthorizedClient authorizedClient,
                         @AuthenticationPrincipal WechatOAuth2User principal) throws IOException {
    

    
    OAuth2AccessToken accessToken = authorizedClient.getAccessToken();
    System.out.println("accessToken = " + accessToken);
    
    System.out.println("principal = " + principal);
    response.sendRedirect("https://felord.cn");
}
```

到此微信公众号授权就集成到**Spring Security**中了。

## 相关配置

`application.yaml`相关的配置：

```null
spring:
  security:
    oauth2:
      client:
        registration:
          wechat:
            
            
            client-id: wxdf9033184b2xxx38e7f
            
            client-secret: bf1306baaa0dxxxxxxb15eb02d68df5
            
            
            redirect-uri:  '{baseUrl}/login/oauth2/code/{registrationId}'
            authorization-grant-type: authorization_code
            scope: snsapi_userinfo
        provider:
          wechat:
            authorization-uri: https://open.weixin.qq.com/connect/oauth2/authorize
            token-uri: https://api.weixin.qq.com/sns/oauth2/access_token
            user-info-uri: https://api.weixin.qq.com/sns/userinfo
```

 [https://www.cnblogs.com/felordcn/p/15143384.html](https://www.cnblogs.com/felordcn/p/15143384.html)
