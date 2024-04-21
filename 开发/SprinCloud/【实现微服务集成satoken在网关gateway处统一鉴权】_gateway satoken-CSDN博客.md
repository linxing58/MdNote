# 【实现微服务集成satoken在网关gateway处统一鉴权】_gateway satoken-CSDN博客
1\. 内容说明
--------

本文旨在使用开源轻量级 Java 权限认证框架[sa-token](https://so.csdn.net/so/search?q=sa-token&spm=1001.2101.3001.7020)+springcloud-gateway实现微服务在网关处统一鉴权。sa-token参考地址：https://sa-token.cc/doc.html#/  
项目按照业务分为三个板块，如图：

1.  api(也就是微服务中各种api接口，不涉及任何权限相关代码，只提供服务)
2.  auth(认证中心，实现登陆逻辑)
3.  gateway(网关，实现转发等，主要是统一鉴权)  
    ![](https://img-blog.csdnimg.cn/001cfc57474740819eb2f6b5ad712abf.png)
    

2\. 项目实现
--------

首先在idea创建一个项目mirco，然后在项目下依次创建三个module，这就不展开说了。接下来各模块填充内容。

### 2.1. auth模块

#### 2.1.1. pom.xml依赖

主要是web、satoken、satoken整合redis：

```
`<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.3.5.RELEASE</version>
    </dependency>
    <!-- Sa-Token 权限认证，在线文档：https://sa-token.cc -->
    <dependency>
        <groupId>cn.dev33</groupId>
        <artifactId>sa-token-spring-boot-starter</artifactId>
        <version>1.34.0</version>
    </dependency>
    <!-- Sa-Token 整合 Redis （使用 jackson 序列化方式） -->
    <dependency>
        <groupId>cn.dev33</groupId>
        <artifactId>sa-token-dao-redis-jackson</artifactId>
        <version>1.34.0</version>
    </dependency>
    <!-- 提供Redis连接池 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
</dependencies>` 

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


```

#### 2.1.2. controller类

提供登陆接口，实现登陆并返回登陆成功的token给前端：

```
`import cn.dev33.satoken.stp.SaTokenInfo;
import cn.dev33.satoken.stp.StpUtil;
import cn.dev33.satoken.util.SaResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AuthController {

    @GetMapping("/auth/doLogin")
    public SaResult doLogin(String username, String password) {
        
        if ("admin".equals(username) && "123456".equals(password)) {
            StpUtil.login(10001);
        } else if ("super".equals(username) && "123456".equals(password)) {
            StpUtil.login(10002);
        }else {
            return SaResult.error("登录失败");
        }
        
        SaTokenInfo tokenInfo = StpUtil.getTokenInfo();
        return SaResult.ok("登录成功").setData(tokenInfo);
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


```

#### 2.1.3. 启动类

```
`import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9


```

#### 2.1.4. application.yml配置文件

```
`server:
  port: 8082
spring:
  # redis配置
  redis:
    # Redis数据库索引（默认为0）
    database: 1
    # Redis服务器地址
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    # password:
    # 连接超时时间
    timeout: 10s
    lettuce:
      pool:
        # 连接池最大连接数
        max-active: 200
        # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1ms
        # 连接池中的最大空闲连接
        max-idle: 10
        # 连接池中的最小空闲连接
        min-idle: 0` 

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


```

### 2.2. gateway模块

#### 2.2.1. pom.xml依赖

主要是gateway、satoken、satoken整合redis：

```
`<dependencies>
    <!-- springCloud gateway -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
        <version>2.2.10.RELEASE</version>
    </dependency>
    <!-- httpClient依赖,缺少此依赖api网关转发请求时可能发生503错误 -->
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>4.5.13</version>
    </dependency>
    <dependency>
        <groupId>cn.dev33</groupId>
        <artifactId>sa-token-reactor-spring-boot-starter</artifactId>
        <version>1.34.0</version>
    </dependency>
    <!-- Sa-Token 整合 Redis （使用 jackson 序列化方式） -->
    <dependency>
        <groupId>cn.dev33</groupId>
        <artifactId>sa-token-dao-redis-jackson</artifactId>
        <version>1.34.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
</dependencies>` 

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


```

#### 2.2.2. 全局过滤器

主要是配置路由进行拦截[鉴权](https://so.csdn.net/so/search?q=%E9%89%B4%E6%9D%83&spm=1001.2101.3001.7020)，这里是整个鉴权的核心，具体的路由配置规则参考sa-token官网：

```
`import cn.dev33.satoken.reactor.filter.SaReactorFilter;
import cn.dev33.satoken.router.SaRouter;
import cn.dev33.satoken.stp.StpUtil;
import cn.dev33.satoken.util.SaResult;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SaTokenConfigure {
    
    @Bean
    public SaReactorFilter getSaReactorFilter() {
        return new SaReactorFilter()
                
                .addInclude("/**")    
                
                .addExclude("/favicon.ico")
                
                .setAuth(obj -> {
                    
                    SaRouter.match("/**", "/auth/doLogin", r -> StpUtil.checkLogin());

                    
                    SaRouter.match("/api/test1", r -> StpUtil.checkPermission("api.test1"));
                    SaRouter.match("/api/test2", r -> StpUtil.checkPermission("api.test2"));
                    SaRouter.match("/api/test3", r -> StpUtil.checkRoleOr("admin", "super"));

                    
                })
                
                .setError(e -> {
                    return SaResult.error(e.getMessage());
                })
                ;
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


```

#### 2.2.3. 自定义权限接口

主要是定义用户拥有的权限和角色，我这里直接写死的，实际项目中通常应该是auth认证模块登陆后从数据库查出用户的角色权限缓存到redis,然后这里再从redis取出来。

```
`import cn.dev33.satoken.stp.StpInterface;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

@Component
public class StpInterfaceImpl implements StpInterface {

    
    @Override
    public List<String> getPermissionList(Object loginId, String loginType) {
        
        List<String> list = new ArrayList<>();
        if (Integer.parseInt(loginId.toString()) == 10001) {
            list.add("api.test1");
        } else if (Integer.parseInt(loginId.toString()) == 10002) {
            list.add("api.test2");
        }
        return list;
    }

    
    @Override
    public List<String> getRoleList(Object loginId, String loginType) {
        
        List<String> list = new ArrayList<>();
        if (Integer.parseInt(loginId.toString()) == 10001) {
            list.add("admin");
        } else if (Integer.parseInt(loginId.toString()) == 10002) {
            list.add("super");
        }
        return list;
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


```

#### 2.2.4. 启动类

```
`import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9


```

#### 2.2.5. application.yml配置文件

```
`server:
  port: 8083
spring:
  application:
    name: gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      routes:
        - id: api
          uri: http://127.0.0.1:8081
          predicates:
            - Path=/api`

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


```

### 2.3. auth模块

#### 2.3.1. pom.xml依赖

主要是web：

```
`<dependencies>
    <!-- SpringBoot Web模块 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7


```

#### 2.3.2. controller类

提供一些接口，用来验证satoken权限：

```
`import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
    @GetMapping("/api/test1")
    public String test1() {
        return "访问成功--只有《api.test1》权限的才能访问";
    }

    @GetMapping("/api/test2")
    public String test2() {
        return "访问成功--只有《api.test2》权限的才能访问";
    }

    @GetMapping("/api/test3")
    public String test3() {
        return "访问成功--拥有《admin或者super角色》可以访问";
    }

    @GetMapping("/api/test4")
    public String test4() {
        return "访问成功--无需权限，登陆即可访问";
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


```

#### 2.3.3. 启动类

```
`import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiApplication.class, args);
    }
}` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9


```

#### 2.3.4. application.yml配置文件

```
`server:
  port: 8081` 

*   1
*   2


```

3\. 项目测试
--------

1.  首先通过网关登陆：http://localhost:8083/auth/doLogin?username=admin&password=123456  
    ![](https://img-blog.csdnimg.cn/e859f816310544b285b78d19502b2f48.png)
      
    如图正确的账户密码登陆成功返回token，这里登陆的admin用户。
    
2.  不带token访问，访问失败：http://localhost:8083/api/test1  
    ![](https://img-blog.csdnimg.cn/1ce006c29b094457803f3cd77a42fae2.png)
    
3.  携带token访问，访问成功：http://localhost:8083/api/test1  
    ![](https://img-blog.csdnimg.cn/078b2b7e960c4ed39492b7a287fac589.png)
    
4.  访问无权限的接口失败：http://localhost:8083/api/test2
    

![](https://img-blog.csdnimg.cn/08d9baf45e7940ab8badb3c593b51751.png)

5.  访问有角色权限的接口成功：http://localhost:8083/api/test3  
    ![](https://img-blog.csdnimg.cn/75953c8708f94ebd81c18d40ac46e109.png)
    

4\. 项目总结
--------

至此，我们完成了gateway处统一使用sa-token鉴权。以上只是最基础的网关统一鉴权的使用，更深层的比如内外网隔离等请自行参考sa-token官网。个人觉得sa-token相当牛逼，十分容易上手，不管你是单体、前后端分离还是微服务项目，都能轻松集成。相比于springsecurity学习成本降低很多。  
另外本博客涉及的代码已全部包含在文章里了，就不另外贴了。