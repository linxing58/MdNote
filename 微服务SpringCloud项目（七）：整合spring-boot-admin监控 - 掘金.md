# 微服务SpringCloud项目（七）：整合spring-boot-admin监控 - 掘金
**小知识，大挑战！本文正在参与「[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https&#x3A;//juejin.cn/post/7008476801634680869")」创作活动**

**本文已参与 [「掘力星计划」](https://juejin.cn/post/7012210233804079141 "https&#x3A;//juejin.cn/post/7012210233804079141") ，赢取创作大礼包，挑战创作激励金。** 

## 📖前言

    心态好了，就没那么累了。心情好了，所见皆是明媚风景。
    复制代码

> **`“一时解决不了的问题，那就利用这个契机，看清自己的局限性，对自己进行一场拨乱反正。” 正如老话所说，一念放下，万般自在。如果你正被烦心事扰乱心神，不妨学会断舍离。断掉胡思乱想，社区垃圾情绪，离开负面能量。心态好了，就没那么累了。心情好了，所见皆是明媚风景。`**

### 🚓`将 Spring Boot Admin Server 启动器添加到您的依赖项中：`

* * *

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
            <version>2.3.1</version>
        </dependency>

        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-client</artifactId>
        </dependency>
```

### 1. 启动类添加注解

* * *

```java
@Slf4j
@EnableAdminServer
@EnableDiscoveryClient
@SpringBootApplication
```

> **PS：通过添加 `@EnableAdminServer` 到配置中来引入 `Spring Boot Admin Server` 配置**

### 2. `yml` 配置

```yaml
spring:
  application:
    
    name: dream-monitor
  
  main:
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        group: DEFAULT_GROUP
        namespace: 
        enabled: true
        encode: UTF-8
        username: nacos
        
        password: 
        max-retry: 32
        file-extension: yml
      discovery:
        server-addr: 
  
  profiles:
    active: dev
```

### 3. 启动

* * *

> **至此，`SpringBoot Admin Server` 已经配置完毕，启动该模块。** 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5ce3977bbb048afbf7f0a3d6663a77a~tplv-k3u1fbpfcp-watermark.awebp?)

### 4. `SPRING BOOT ADMIN CLIENT` 客户端接入

* * *

> **您可以使用任何 `Spring Cloud DiscoveryClient` 实现（`Eureka、Consul、Nacos`），这里以 `SpringCloud Alibaba Nacos` 为例。** 

#### 1. 将依赖添加进你的项目中：

```xml
<dependency>
     <groupId>de.codecentric</groupId>
     <artifactId>spring-boot-admin-starter-client</artifactId>
     <version>2.3.1</version>
</dependency>
```

#### 2. 通过添加 `@EnableDiscoveryClient` 到配置中来启用发现：

```java
@SpringBootApplication
@EnableDiscoveryClient
```

#### 3. 告诉 `Nacos` 客户端在哪里可以找到服务注册表：

> **`bootstrap.yml`**

```yaml
spring:
  application:
    name: gateway
  cloud:
    nacos:
      discovery:
        server-addr: nacos地址
  profiles:
    active: dev
server:
  port: 9999
```

> **`application.yml`**

```yaml
management:
    endpoints:
      web:
        exposure:
          include: "*"  
    endpoint:
      health:
        show-details: ALWAYS
```

> PS：与 `Spring Boot 2` 一样，默认情况下，大多数端点都不通过 `http` 公开，我们公开了所有端点。对于生产，您应该仔细选择要公开的端点。

#### 4. 启动 `Client` 即可查看效果

![](https://www.freesion.com/images/126/35247675a6c41b44d27d5986ad9263d6.png)

![](https://www.freesion.com/images/816/b4310fc421fed08b07324cf3df939cf0.png)

* * *

**最后感谢大家耐心观看完毕，下节讲一下如何添加 `security` 安全密码，留个点赞收藏是您对我最大的鼓励！**

* * *

### 🎉总结：

-   **更多参考精彩博文请看这里：[《陈永佳的博客》](https://juejin.cn/user/862483929905639/posts "https&#x3A;//juejin.cn/user/862483929905639/posts")**
-   **喜欢博主的小伙伴可以加个关注、点个赞哦，持续更新嘿嘿！** 
    [https://juejin.cn/post/7018039824414310430](https://juejin.cn/post/7018039824414310430)
