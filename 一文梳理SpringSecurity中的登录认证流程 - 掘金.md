# 一文梳理SpringSecurity中的登录认证流程 - 掘金
## 前言

`SpringSecurity`作为一个出自 Spring 家族很强大的安全框架时长被引用到`SpringBoot`项目中用作登录认证和授权模块使用，但是对于大部分使用者来说都只停留在实现使用用户名和密码的方式登录。而对于企业的项目需求大多要实现多种登录认证方式，例如一个的登录功能往往需要支持下面几种登录模式：

-   用户名和密码模式
-   手机号和短信验证码模式
-   邮箱地址和邮件验证码模式
-   微信、QQ、微博、知乎、钉钉、支付宝等第三方授权登录模式
-   微信或 QQ 扫码登录模式

可以说一个看似简单的登录认证功能要同时实现方便用户选择的多种登录认证功能还是比较复杂的，尤其在项目集成了`SpringSecurity`框架作为登录认证模块的场景下。这个时候我们就不得不去通过阅读源码的方式弄清楚`SpringSecurity`中实现登录认证的具体流程是怎样的，在这个基础上实现框架的扩展功能。以笔者研究源码的经验告诉大家，如果从头开始研究源码其实是一件非常耗时耗力的活，如果有牛人把自己研究过的源码整理出一篇不错的文章出来的话，那当然拿来用即可，如果读了别人研究源码的文章之后依然无法解决你的困惑或者问题，那么这个时候建议你亲自去研究一番源码，从而找到别的作者研究源码的时候没有解决的扩展问题。

本文的目的就通过梳理`SpringSecurity`框架登录认证部分源码的方式，带你搞清楚以`SpringSecurity`实现登录认证的详细流程。为不久后在集成`SpringSecurity`作为登录认证模块的`SpringBoot`项目中，实现添加手机号 + 短信验证码或者邮箱地址 + 邮箱验证码模式的登录认证功或者实现其他认证方式能作好实现思路准备。

## 认识 SpringSecurity 中的过滤器链

我们知道`SpringSecurity`框架实现登录认证的底层原理是基于一系列的过滤器对请求进行拦截实现的，而且它有一个过滤器链，当一个过滤器对请求进行拦截认证通过之后会进入到下一个过滤器，知道所有的过滤器都通过认证才会到达请求要访问的端点。

在 `Spring Security` 中，其核心流程的执行也是依赖于一组过滤器，这些过滤器在框架启动后会自动进行初始化，如图所示：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1de017aa1fa4b948fe0c92ab4cf4809~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

图片来源：拉勾教育

在上图中，我们看到了几个常见的 Filter，比如 `BasicAuthenticationFilter`、`UsernamePasswordAuthenticationFilter` 等，这些类都直接或间接实现了 `Servlet` 中的 `Filter` 接口，并完成某一项具体的认证机制。例如，上图中的 ```BasicAuthenticationFilter`` 用来验证用户的身份凭证；而 UsernamePasswordAuthenticationFilter``` 会检查输入的用户名和密码，并根据认证结果决定是否将这一结果传递给下一个过滤器。

请注意，整个 Spring Security 过滤器链的末端是一个 `FilterSecurityInterceptor`，它本质上也是一个 `Filter`。但与其他用于完成认证操作的 `Filter` 不同，它的核心功能是实现**权限控制**，也就是用来判定该请求是否能够访问目标 HTTP 端点。`FilterSecurityInterceptor` 对于权限控制的粒度可以到方法级别，能够满足精细化访问控制。

## SpringSecurity 中与用户名密码登录认证相关的几个重要类

### Authentication 接口类

认证凭据类需要实现的接口，它的源码如下：

```java
public interface Authentication extends Principal, Serializable {
    
    Collection<? extends GrantedAuthority> getAuthorities();
    
    Object getCredentials();
    
    Object getDetails();
    
    Object getPrincipal();
    
    boolean isAuthenticated();
    
    void setAuthenticated(boolean authenticated) throws IllegalArgumentException;
}
```

与用户名和密码登录相关的`Authentication`接口实现类为`UsernamePasswordAuthenticationToken`

该类继承自`AbstractAuthenticationToken`抽象类, 它实现了`Authentication`接口中的大部分方法

`AbstractAuthenticationToken`抽象类的继承和实现关系图如下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b084a62fd7c24174a9b922c84186bab5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

`UsernamePasswordAuthenticationToken`类提供了两个构造方法，用于构造实例

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a558cabbb13641aaa621ff9e0557f4c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### AuthenticationProvider 接口类

这是一个认证器根接口类，几乎所有的认证认证逻辑类都要实现这个接口，它里面主要有两个方法

```java
public interface AuthenticationProvider {
    
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
    
    boolean supports(Class<?> authentication);
}

```

### AbstractUserDetailsAuthenticationProvider 抽象类

该抽象类实现了`AuthenticationProvider`接口中的所有抽象方法，但是新增了一个`additionalAuthenticationChecks`抽象方法并定义了一个检验`Authentication`是否合法的，用于在它的实现类中实现验证登录密码是否正确的逻辑；同时定义了一个私有类`DefaultPreAuthenticationChecks`用于检验`authentication`方法传递过来的`AuthenticationToken`参数是否合法。

该类的类继承与实现关系结构图如如下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffa06673ab6244979a8894c4fd66d445~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### DaoAuthenticationProvider 实现类

该类继承自上面的抽象类`AbstractUserDetailsAuthenticationProvider`, 实现了从数据库查询用户和校验密码的功能

用户名密码登录认证用到的`AuthenticationProvider`就是这个类

### AuthenticationManager 接口类

这个类可以叫做认证管理器类，在过滤器中的认证逻辑中正是通过该类的实现类去调用认证逻辑的，它只有一个认证方法

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

### ProviderManager 实现类

该类实现了`AuthenticationManager`接口类，同时还实现了`MessageSourceAware`和`InitializingBean`两个接口，该类有两个构造方法用来传递需要认证的`AuthenticationProvider`集合以及父`AuthenticationManager`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ee5c1955b7c4a52825857c1d62eaa69~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

它的认证方法源码逻辑如下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd6d079d847f4836865f6204293cf6c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### AuthenticationManagerBuilder 类

该类是`WebSecurityConfigurerAdapter#configure`方法中的参数类，可用于设置各种用户想用的认证方式，设置用户认证数据库查询服务`UserDetailsService`类以及添加自定义`AuthenticationProvider`类实例等

它的几个重要的方法如下：

1） 设置`parentAuthenticationManager`

```java
AuthenticationManagerBuilder parentAuthenticationManager(AuthenticationManager authenticationManager)
```

2）设置内存认证

```java
public InMemoryUserDetailsManagerConfigurer<AuthenticationManagerBuilder> inMemoryAuthentication() throws Exception {
        return (InMemoryUserDetailsManagerConfigurer)this.apply(new InMemoryUserDetailsManagerConfigurer());
    }
```

3）设置基于`jdbc`数据库认证的认证

```java
 public JdbcUserDetailsManagerConfigurer<AuthenticationManagerBuilder> jdbcAuthentication() throws Exception {
        return (JdbcUserDetailsManagerConfigurer)this.apply(new JdbcUserDetailsManagerConfigurer());
    }
```

4）设置自定义的`UserDetailsService`认证

```java
public <T extends UserDetailsService> DaoAuthenticationConfigurer<AuthenticationManagerBuilder, T> userDetailsService(T userDetailsService) throws Exception {
        this.defaultUserDetailsService = userDetailsService;
        return (DaoAuthenticationConfigurer)this.apply(new DaoAuthenticationConfigurer(userDetailsService));
    }
```

5）设置基于 Ldap 的认证方式

```java
public LdapAuthenticationProviderConfigurer<AuthenticationManagerBuilder> ldapAuthentication() throws Exception {
        return (LdapAuthenticationProviderConfigurer)this.apply(new LdapAuthenticationProviderConfigurer());
    }
```

6）添加自定义`AuthenticationProvider`类

```java
public AuthenticationManagerBuilder authenticationProvider(AuthenticationProvider authenticationProvider) {
        this.authenticationProviders.add(authenticationProvider);
        return this;
    }
```

后面我们自定义的`AuthenticationProvider`实现类就通过这个方法加入到认证器列表中

`AuthenticationManagerBuilder`类在`spring-security-config-5.2.4-RELEAS.jar`中的`org.springframework.security.config.annotation.authentication.configuration`包中的配置类

`AuthenticationConfiguration`中对`AuthenticationManagerBuilder`类中定义了 bean

`AuthenticationConfiguration.class`

```java
 @Bean
    public AuthenticationManagerBuilder authenticationManagerBuilder(ObjectPostProcessor<Object> objectPostProcessor, ApplicationContext context) {
        AuthenticationConfiguration.LazyPasswordEncoder defaultPasswordEncoder = new AuthenticationConfiguration.LazyPasswordEncoder(context);
        AuthenticationEventPublisher authenticationEventPublisher = (AuthenticationEventPublisher)getBeanOrNull(context, AuthenticationEventPublisher.class);
        AuthenticationConfiguration.DefaultPasswordEncoderAuthenticationManagerBuilder result = new AuthenticationConfiguration.DefaultPasswordEncoderAuthenticationManagerBuilder(objectPostProcessor, defaultPasswordEncoder);
        if (authenticationEventPublisher != null) {
            result.authenticationEventPublisher(authenticationEventPublisher);
        }

        return result;
    }
```

所以该类可以直接出现在自定义配置类`WebSecurityConfigurerAdapter#configure`方法的参数中

### HttpSecurity 类

该类是`WebSecurityConfigurerAdapter#configure(HttpSecurity http)`方法中的参数类型

通过这个类我们可以自定义各种与 web 相关的配置器和添加过滤器，其中的`formLogin`方法就是设置了一个基于用户名和密码登录认证的配置

常用的配置`Configurer`方法

```java
 
public FormLoginConfigurer<HttpSecurity> formLogin() throws Exception {
        return (FormLoginConfigurer)this.getOrApply(new FormLoginConfigurer());
    }
 
 public HttpSecurity formLogin(Customizer<FormLoginConfigurer<HttpSecurity>> formLoginCustomizer) throws Exception {
        formLoginCustomizer.customize(this.getOrApply(new FormLoginConfigurer()));
        return this;
    }

public CorsConfigurer<HttpSecurity> cors() throws Exception {
        return (CorsConfigurer)this.getOrApply(new CorsConfigurer());
    }

public HttpSecurity cors(Customizer<CorsConfigurer<HttpSecurity>> corsCustomizer) throws Exception {
        corsCustomizer.customize(this.getOrApply(new CorsConfigurer()));
        return this;
    }

 public AnonymousConfigurer<HttpSecurity> anonymous() throws Exception {
        return (AnonymousConfigurer)this.getOrApply(new AnonymousConfigurer());
    }

public HttpSecurity anonymous(Customizer<AnonymousConfigurer<HttpSecurity>> anonymousCustomizer) throws Exception {
        anonymousCustomizer.customize(this.getOrApply(new AnonymousConfigurer()));
        return this;
    }


public HttpBasicConfigurer<HttpSecurity> httpBasic() throws Exception {
        return (HttpBasicConfigurer)this.getOrApply(new HttpBasicConfigurer());
    }

    public HttpSecurity httpBasic(Customizer<HttpBasicConfigurer<HttpSecurity>> httpBasicCustomizer) throws Exception {
        httpBasicCustomizer.customize(this.getOrApply(new HttpBasicConfigurer()));
        return this;
    }

 public SessionManagementConfigurer<HttpSecurity> sessionManagement() throws Exception {
        return (SessionManagementConfigurer)this.getOrApply(new SessionManagementConfigurer());
    }

    public HttpSecurity sessionManagement(Customizer<SessionManagementConfigurer<HttpSecurity>> sessionManagementCustomizer) throws Exception {
        sessionManagementCustomizer.customize(this.getOrApply(new SessionManagementConfigurer()));
        return this;
    }
```

可以看到以上方法都是成对出现，一个不带参数的方法和一个带`Customizer<XXConfigure>`类型参数的方法。不带参数的方法返回一个`XXConfigure`类型实例，而带参数的方法返回一个`HttpSecurity`实例对象，也是 Configure 方法中的

调用以上的各种登录认证方法都在实例化`XXConfigure`类时自动实例化一个对应的过滤器并添加到过滤器链中

以下是添加注册过滤器方法

```java

public HttpSecurity addFilterAfter(Filter filter, Class<? extends Filter> afterFilter) {
        this.comparator.registerAfter(filter.getClass(), afterFilter);
        return this.addFilter(filter);
    }

 public HttpSecurity addFilterBefore(Filter filter, Class<? extends Filter> beforeFilter) {
        this.comparator.registerBefore(filter.getClass(), beforeFilter);
        return this.addFilter(filter);
    }
 
 public HttpSecurity addFilter(Filter filter) {
        Class<? extends Filter> filterClass = filter.getClass();
        if (!this.comparator.isRegistered(filterClass)) {
            throw new IllegalArgumentException("The Filter class " + filterClass.getName() + " does not have a registered order and cannot be added without a specified order. Consider using addFilterBefore or addFilterAfter instead.");
        } else {
            this.filters.add(filter);
            return this;
        }
    }
```

其他方法这里就不赘述了，读者可以在 IDEA 中打开`HttpSecurity`类的源码进行查看

### AbstractAuthenticationProcessingFilter 抽象类

该类为抽象认证处理过滤器，`SpringSecutity`中的自定义过滤器只需要继承这个过滤器即可

该类的几个重要方法如下：

```java

protected AbstractAuthenticationProcessingFilter(String defaultFilterProcessesUrl) {
        this.setFilterProcessesUrl(defaultFilterProcessesUrl);
    }

protected AbstractAuthenticationProcessingFilter(RequestMatcher requiresAuthenticationRequestMatcher) {
        Assert.notNull(requiresAuthenticationRequestMatcher, "requiresAuthenticationRequestMatcher cannot be null");
        this.requiresAuthenticationRequestMatcher = requiresAuthenticationRequestMatcher;
    }


public abstract Authentication attemptAuthentication(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws AuthenticationException, IOException, ServletException;


public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) {
    HttpServletRequest request = (HttpServletRequest)req;
        HttpServletResponse response = (HttpServletResponse)res;
        if (!this.requiresAuthentication(request, response)) {
            chain.doFilter(request, response);
        } else {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Request is to process authentication");
            }

            Authentication authResult;
            try {
                
                authResult = this.attemptAuthentication(request, response);
                if (authResult == null) {
                    return;
                }
                
                this.sessionStrategy.onAuthentication(authResult, request, response);
            } catch (InternalAuthenticationServiceException var8) {
                this.logger.error("An internal error occurred while trying to authenticate the user.", var8);
                
                this.unsuccessfulAuthentication(request, response, var8);
                return;
            } catch (AuthenticationException var9) {
                this.unsuccessfulAuthentication(request, response, var9);
                return;
            }

            if (this.continueChainBeforeSuccessfulAuthentication) {
                chain.doFilter(request, response);
            }
            
            this.successfulAuthentication(request, response, chain, authResult);
        }
}


protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Authentication success. Updating SecurityContextHolder to contain: " + authResult);
        }
        
        SecurityContextHolder.getContext().setAuthentication(authResult);
        
        this.rememberMeServices.loginSuccess(request, response, authResult);
        if (this.eventPublisher != null) {
            this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
        }
        
        this.successHandler.onAuthenticationSuccess(request, response, authResult);
    }



public void setFilterProcessesUrl(String filterProcessesUrl) {
        this.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher(filterProcessesUrl));
    }

protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        
        SecurityContextHolder.clearContext();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Authentication request failed: " + failed.toString(), failed);
            this.logger.debug("Updated SecurityContextHolder to contain null Authentication");
            this.logger.debug("Delegating to authentication failure handler " + this.failureHandler);
        }
        
        this.rememberMeServices.loginFail(request, response);
        
        this.failureHandler.onAuthenticationFailure(request, response, failed);
    }


public void setAuthenticationManager(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }


public void setAuthenticationSuccessHandler(AuthenticationSuccessHandler successHandler) {
        Assert.notNull(successHandler, "successHandler cannot be null");
        this.successHandler = successHandler;
}


public void setAuthenticationFailureHandler(AuthenticationFailureHandler failureHandler) {
        Assert.notNull(failureHandler, "failureHandler cannot be null");
        this.failureHandler = failureHandler;
}
```

可以看到在实例化时，实例化了一个`UsernamePasswordAuthenticationFilter`过滤器作为参数调用了其父类的构造方法，再查看其父类的构造方法：

`AbstractAuthenticationFilterConfigurer.class`

```java
protected AbstractAuthenticationFilterConfigurer(F authenticationFilter, String defaultLoginProcessingUrl) {
        this();
        this.authFilter = authenticationFilter;
        if (defaultLoginProcessingUrl != null) {
            this.loginProcessingUrl(defaultLoginProcessingUrl);
        }

    }
```

可以看到其把传入的过滤其作为自己的认证过滤器

然后我们进入`UsernamePasswordAuthenticationFilter`类中查看其构造方法和`attemptAuthentication`方法源码

```java

public UsernamePasswordAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login", "POST"));
    }

public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        } else {
            String username = this.obtainUsername(request);
            String password = this.obtainPassword(request);
            if (username == null) {
                username = "";
            }

            if (password == null) {
                password = "";
            }

            username = username.trim();
            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
            this.setDetails(request, authRequest);
            
            return this.getAuthenticationManager().authenticate(authRequest);
        }
    }
```

## 写在最后

一图胜千言，下图是笔者基于用户名 + 密码登录流程画的一个登录认证时序图，如有不准确的地方还请读者在评论区不吝指出，谢谢！

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c80c66df56e44128a57638a6d477cd8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

下一篇文章笔者将使用自定义的 `MobilePhoneAuthenticationProvider`认证器和`MobilePhoeAuthenticationFilter`过滤器，在集成`spring-security`的`SpringBoot`项目中实现

手机号码 + 短信验证码登录。功能在笔者的本地开发环境已实现，只等着明天输出文章即可，敬请期待！

## 参考文章

【1】[认证体系：如何深入理解 Spring Security 用户认证机制？](https://link.juejin.cn/?target=https%3A%2F%2Fkaiwu.lagou.com%2Fcourse%2FcourseInfo.htm%3FcourseId%3D960%23%2Fdetail%2Fpc%3Fid%3D7697 "https&#x3A;//kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7697")

【2】 [松哥手把手带你捋一遍 Spring Security 登录流程](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fz6GeR5O-vBzY3SHehmccVA "https&#x3A;//mp.weixin.qq.com/s/z6GeR5O-vBzY3SHehmccVA")

本位首发个人微信公众号【**阿福谈 Web 编程**】，觉得我的文章对你有帮助的读者朋友欢迎给我的这篇文章点赞、评论和转发，让我们一同在技术精进的路上一同成长，谢谢！ 
 [https://juejin.cn/post/7077209080267276318](https://juejin.cn/post/7077209080267276318)
