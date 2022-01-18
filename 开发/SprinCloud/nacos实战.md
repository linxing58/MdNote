# nacos实战
```null
问题，既然有了Eureka ，为啥还要用Nacos？
```

而 Nacos 作为微服务核心的服务注册与发现中心，让大家在 Eureka 和 Consule 之外有了新的选择，开箱即用，上手简洁，暂时也没发现有太大的坑。

### 1.1 与 eureka 对比

> 1 eureka 2.0 闭源码了。
>
> 2 从官网来看 nacos 的注册的实例数是大于 eureka 的,
>
> 3 因为 nacos 使用的 raft 协议, nacos 集群的一致性要远大于 eureka 集群.

分布式一致性协议 Raft，自 2013 年论文发表，之后就受到了技术领域的热捧，与其他的分布式一致性算法比，Raft 相对比较简单并且易于实现，这也是 Raft 能异军突起的主要因素。

![](https://img-blog.csdnimg.cn/20191014133644698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYW9IYW5adW9GZW5nWmhvdQ==,size_16,color_FFFFFF,t_70)

> ## Raft 的数据一致性策略
>
> Raft 协议强依赖 Leader 节点来确保集群数据一致性。即 client 发送过来的数据均先到达 Leader 节点，Leader 接收到数据后，先将数据标记为 uncommitted 状态，随后 Leader 开始向所有 Follower 复制数据并等待响应，在获得集群中大于 N/2 个 Follower 的已成功接收数据完毕的响应后，Leader 将数据的状态标记为 committed，随后向 client 发送数据已接收确认，在向 client 发送出已数据接收后，再向所有 Follower 节点发送通知表明该数据状态为 committed。

### 1.2 与 springcloud config 对比

#### 三大优势：

-   springcloud config 大部分场景结合 git 使用, 动态变更还需要依赖 Spring Cloud Bus 消息总线来通过所有的客户端变化.
-   springcloud config 不提供可视化界面
-   nacos config 使用长连接更新配置, 一旦配置有变动后，通知 Provider 的过程非常的迅速, 从速度上秒杀 springcloud 原来的 config 几条街,

目前 Spring Cloud Alibaba 主要有三个组件：

-   Nacos：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
-   Sentinel：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
-   AliCloud OSS: 阿里云对象存储服务（Object Storage Service，简称  
    OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。

### Spring Cloud Alibaba 套件和 Spring Cloud Netflix 套件类比

仔细看看各组件的功能描述，Spring Cloud Alibaba 套件和 Spring Cloud Netflix 套件大致的对应关系：

-   Nacos = Eureka/Consule + Config + Admin
-   Sentinel = Hystrix + Dashboard + Turbine
-   Dubbo = Ribbon + Feign
-   RocketMQ = RabbitMQ
-   Schedulerx = Quartz
-   AliCloud OSS、AliCloud SLS 这三个应该是独有的  
    链路跟踪（Sleuth、Zipkin）不知道会不会在 Sentinel 里  
    以上只是猜测，待我从坑里爬出来之后再回来更新。也欢迎大家一起交流探讨~  
    这里我就先试试 Nacos。

## 3.1、Nacos 的架构

![](https://img-blog.csdnimg.cn/20190329154650649.png)

这是 Nacos 的架构图，可以看到它确实是融合了服务注册发现中心、配置中心、服务管理等功能，类似于 Eureka/Consule + Config + Admin 的合体。

另外通过官方文档发现，Nacos 除了可以和 Spring Cloud 集成，还可以和 Spring、SpringBoot 进行集成。

不过我们只关注于 Spring Cloud，别的就略过了，直接上手实战吧。

## 3.2、Nacos Server 的下载和安装

在使用 Nacos 之前，需要先下载 Nacos 并启动 Nacos Server。

> 安装的参考教程：
>
> [https://www.cnblogs.com/crazymakercircle/p/11992539.html](https://www.cnblogs.com/crazymakercircle/p/11992539.html)

## 4.1 两种模式

Nacos Server 有两种运行模式：

-   standalone
-   cluster

## 4.2、standalone 模式

此模式一般用于 demo 和测试，不用改任何配置，直接敲以下命令执行

```null
sh bin/startup.sh -m standalone
```

Windows 的话就是

```null
cmd bin/startup.cmd -m standalone
```

然后从 [http://cdh1:8848/nacos/index.html](http://cdh1:8848/nacos/index.html) 进入控制台就能看到如下界面了

![](https://img-blog.csdnimg.cn/20190329154915102.png)

默认账号和密码为：nacos nacos

## 4.3、cluster 模式

测试环境，可以先用 standalone 模式撸起来，享受 coding 的快感，但是，生产环境可以使用 cluster 模式。

### cluster 模式需要依赖 MySQL，然后改两个配置文件：

```null
conf/cluster.conf
conf/application.properties

```

大致如下：

1： cluster.conf，填入要运行 Nacos Server 机器的 ip

```null
192.168.100.155
192.168.100.156

```

2.  修改 NACOS_PATH/conf/application.properties，加入 MySQL 配置

```null
db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=root
```

创建一个名为 nacos_config 的 database，将 NACOS_PATH/conf/nacos-mysql.sql 中的表结构导入刚才创建的库中，这几张表的用途就自己研究吧

## 4.4 Nacos Server 的配置数据是存在哪里呢？

```null
问题来了： Nacos Server 的配置数据是存在哪里呢？
```

我们没有对 Nacos Server 做任何配置，那么数据只有两个位置可以存储：

-   内存
-   本地数据库

如果我们现在重启刚刚在运行的 Nacos Server，会发现刚才加的 nacos.properties 配置还在，说明不是内存存储的。

这时候我们打开 NACOS_PATH/data，会发现里边有个 derby-data 目录，我们的配置数据现在就存储在这个库中。

> Derby 是 Java 编写的数据库，属于 Apache 的一个开源项目

如果将数据源改为我们熟悉的 MySQL 呢？当然可以。

> 注意：不支持 MySQL 8.0 版本

这里有两个坑：

> Nacos Server 的数据源是用 Derby 还是 MySQL 完全是由其运行模式决定的：

```null
standalone 的话仅会使用 Derby，即使在 application.properties 里边配置 MySQL 也照样无视；
cluster 模式会自动使用 MySQL，这时候如果没有 MySQL 的配置，是会报错的。
```

官方提供的 cluster.conf 示例如下

```null


10.10.109.214
11.16.128.34
11.16.128.36

```

以上配置结束后，运行 Nacos Server 就能看到效果了。

### 实战的工程

实战的工程的目录结构如下：

![](https://img-blog.csdnimg.cn/20210104193150374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

## 5.1 如何使用 Nacos Client 组件

#### 首先引入 Spring Cloud Alibaba 的 BOM

```null
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
    <relativePath/>
</parent>
<properties>
    <spring-cloud.version>Finchley.SR2</spring-cloud.version>
    <spring-cloud-alibaba.version>0.2.0.RELEASE</spring-cloud-alibaba.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

这里版本号有坑，文档上说和 Spring Boot 2.0.x 版本兼容，但是实测 2.0.6.RELEASE 报错

```null
java.lang.NoClassDefFoundError: org/springframework/core/env/EnvironmentCapable

```

## 5.2、演示的模块结构

服务注册中心和服务发现的服务端都是由 Nacos Server 来提供的，我们只需要提供 Service 向其注册就好了。

这里模拟提供两个 service：provider 和 consumer

```null
alibaba
├── service-provider-demo
│   ├── pom.xml
│   └── src
└── sevice-consumer-demo
│   ├── pom.xml
│   └── src
└── pom.xml

```

## 5.3、provider 微服务

#### step1：在 provider 和 consumer 的 pom 添加以下依赖：

```null
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

```

#### step2：启动类

使用 Spring Cloud 的原生注解 @EnableDiscoveryClient 开启服务注册与发现

```null
package com.crazymaker.cloud.nacos.demo.starter;

import com.crazymaker.springcloud.standard.context.SpringContextUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.Environment;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.List;

@EnableSwagger2
@SpringBootApplication
@EnableDiscoveryClient
@Slf4j
public class ServiceProviderApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext =   SpringApplication.run(ServiceProviderApplication.class, args);

        Environment env = applicationContext.getEnvironment();
        String port = env.getProperty("server.port");
        String path = env.getProperty("server.servlet.context-path");
        System.out.println("\n--------------------------------------\n\t" +
                "Application is running! Access URLs:\n\t" +
                "Local: \t\thttp://localhost:" + port + path+ "/index.html\n\t" +
               "swagger-ui: \thttp://localhost:" + port + path + "/swagger-ui.html\n\t" +
                "----------------------------------------------------------");

    }
}
```

#### step3：服务提供者的 Rest 服务接口

service-provider-demo 提供 一个非常简单的 Rest 服务接口以供访问

```null
package com.crazymaker.cloud.nacos.demo.controller;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/echo")
public class EchoController {
    
    @RequestMapping(value = "/{string}", method = RequestMethod.GET)
    public String echo(@PathVariable String string) {
        return "echo: " + string;
    }
}

```

#### step4：配置文件

```null
spring:
  application:
    name: service-provider-demo
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_SERVER:cdh1:8848}
server:
  port: 18080
```

#### step5：启动之后，通过 swagger UI 访问：

![](https://img-blog.csdnimg.cn/20210104153259531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

## 5.4、Consumer 微服务演示 RPC 远程调用

在 NacosConsumerApplication 中集成 RestTemplate 和 Ribbon

```null
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
  return new RestTemplate();
}

```

#### 消费者的 controller 类

```null
package com.crazymaker.cloud.nacos.demo.consumer.controller;

import com.crazymaker.cloud.nacos.demo.consumer.client.EchoClient;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@RequestMapping("/echo")
@Api(tags = "服务- 消费者")
public class EchoConsumerController {


    
    @Resource
    EchoClient echoClient;


    
    @ApiOperation(value = "消费回显服务接口")
    @RequestMapping(value = "/{string}", method = RequestMethod.GET)
    public String echoRemoteEcho(@PathVariable String string) {
        return "provider echo is:" + echoClient.echo(string);
    }
}


```

#### 消费者配置文件

```null
spring:
  application:
    name: sevice-consumer-demo
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
server:
  port: 18081

```

#### 通过 swagger UI 访问消费者：

![](https://img-blog.csdnimg.cn/20210104155555838.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

访问远程的 echo API：

![](https://img-blog.csdnimg.cn/20210104163056237.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210104163124914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

## 5.5 涉及到的演示地址：

服务提供者 service-provider-demo：

[http://localhost:18080/provider/swagger-ui.html#/Echo\_%E6%BC%94%E7%A4%BA](http://localhost:18080/provider/swagger-ui.html#/Echo_%E6%BC%94%E7%A4%BA)

服务消费者：

\[[http://localhost:18081/consumer/swagger-ui.html#/ 服务 - 消费者 / echoRemoteEchoUsingGET](http://localhost:18081/consumer/swagger-ui.html#/%E6%9C%8D%E5%8A%A1-%20%E6%B6%88%E8%B4%B9%E8%80%85/echoRemoteEchoUsingGET)]([http://localhost:18081/consumer/swagger-ui.html#/ 服务 -](http://localhost:18081/consumer/swagger-ui.html#/%E6%9C%8D%E5%8A%A1-) 消费者 / echoRemoteEchoUsingGET)

注册中心 Nacos：

[http://cdh1:8848/nacos/index.html#/serviceManagement?dataId=&group=&appName=&namespace=](http://cdh1:8848/nacos/index.html#/serviceManagement?dataId=&group=&appName=&namespace=)

## 5.6、Nacos Console

这时候查看 Nacos Console 也能看到已注册的服务列表及其详情

![](https://img-blog.csdnimg.cn/20210104155420489.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

## 6.1、基本概念

#### 1. Profile

Java 项目一般都会有多个 Profile 配置，用于区分开发环境，测试环境，准生产环境，生成环境等，每个环境对应一个 properties 文件（或是 yml/yaml 文件），然后通过设置 spring.profiles.active 的值来决定使用哪个配置文件。

例子：

```null
spring:
  application:
    name: sharding-jdbc-provider
  jpa:
    hibernate:
      ddl-auto: none
      dialect: org.hibernate.dialect.MySQL5InnoDBDialect
      show-sql: true
  profiles:
     active: sharding-db-table    
    
    
```

Nacos Config 的作用就把这些文件的内容都移到一个统一的配置中心，即方便维护又支持实时修改后动态刷新应用。

#### 2. Data ID

当使用 Nacos Config 后，Profile 的配置就存储到 Data ID 下，即一个 Profile 对应一个 Data ID

Data ID 的拼接格式：${prefix} - ${spring.profiles.active} . ${file-extension}

-   prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix 来配置
-   spring.profiles.active 取 spring.profiles.active 的值，即为当前环境对应的 profile
-   file-extension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置

#### 3. Group

Group 默认为 DEFAULT_GROUP，可以通过 spring.cloud.nacos.config.group 来配置，当配置项太多或者有重名时，可以通过分组来方便管理

最后就和原来使用 springcloud 一样通过 @RefreshScope 和 @Value 注解即可

## 6.2 通过 Nacos 的 console 去增加配置

这回首先要在 nacos 中配置相关的配置，打开 Nacos 配置界面，依次创建 2 个 Data ID

-   nacos-config-demo-dev.yaml 开发环境的配置
-   nacos-config-demo-test.yaml 测试环境的配置

#### 1、nacos-config-demo-dev.yaml

内容如下图：

![](https://img-blog.csdnimg.cn/20210104174312428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

#### 2、nacos-config-demo-sit.yaml

> 内容如下图：

![](https://img-blog.csdnimg.cn/20210104174236932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

## 6.3 使用 Nacos Config Client 组件

问题 2：微服务 Provider 实例上，如何使用 Nacos Config Client 组件的有哪些步骤？

#### 1 加载 nacos config 的客户端依赖：

```null
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>${nacos.version}</version>
</dependency>
```

#### 启动类

```null
package com.crazymaker.cloud.nacos.demo.consumer.starter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration;
import org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration;
import org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration;
import org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.Environment;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@EnableSwagger2
@EnableDiscoveryClient
@Slf4j
@SpringBootApplication(
        scanBasePackages =
                {
                        "com.crazymaker.cloud.nacos.demo",
                        "com.crazymaker.springcloud.standard"
                },
        exclude = {SecurityAutoConfiguration.class,
                
                DataSourceAutoConfiguration.class,
                DataSourceTransactionManagerAutoConfiguration.class,
                HibernateJpaAutoConfiguration.class,
                
                RedisAutoConfiguration.class,
                RedisRepositoriesAutoConfiguration.class})

@EnableFeignClients(basePackages =
        {"com.crazymaker.cloud.nacos.demo.consumer.client"})
public class ConfigDomeProviderApplication
{
    public static void main(String[] args)
    {
        ConfigurableApplicationContext applicationContext = null;
        try
        {
            applicationContext = SpringApplication.run(ConfigDomeProviderApplication.class, args);
            System.out.println("Server startup done.");
        } catch (Exception e)
        {
            log.error("服务启动报错", e);
            return;
        }

        Environment env = applicationContext.getEnvironment();
        String port = env.getProperty("server.port");
        String path = env.getProperty("server.servlet.context-path");
        System.out.println("\n----------------------------------------------------------\n\t" +
                "Application is running! Access URLs:\n\t" +
                "Local: \t\thttp://localhost:" + port + path + "/index.html\n\t" +
                "swagger-ui: \thttp://localhost:" + port + path + "/swagger-ui.html\n\t" +
                "----------------------------------------------------------");

    }
}
```

#### 控制类：

```null
package com.crazymaker.cloud.nacos.demo.config.controller;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@RequestMapping("/config")
@Api(tags = "Nacos 配置中心演示")
public class ConfigGetController {


    @Value("${foo.bar:empty}")
    private String bar;


    @Value("${spring.datasource.username:empty}")
    private String dbusername;

    
    @ApiOperation(value = "获取配置的内容")
    @RequestMapping(value = "/bar", method = RequestMethod.GET)
    public String getBar() {
        return "bar is :"+bar;
    }
    
    @ApiOperation(value = "获取配置的db username")
    @RequestMapping(value = "/dbusername", method = RequestMethod.GET)
    public String getDbusername() {
        return "db username is :"+bar;
    }
}

```

#### 2 bootstrap 配置文件

然后是在配置文件 (bootstrap.yml) 中加入以下的内容：

```null
spring:
  application:
    name: nacos-config-demo-provider
  profiles:
    active:  dev
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_SERVER:cdh1:8848}
      config:
        server-addr: ${NACOS_SERVER:cdh1:8848}
        prefix: nacos-config-demo
        group: DEFAULT_GROUP
        file-extension: yaml
server:
  port: 18083
  servlet:
    context-path: /config

```

### 6.4 测试结果

启动程序，通过 swagger ui 访问：

> [http://localhost:18083/config/swagger-ui.html#](http://localhost:18083/config/swagger-ui.html#)

![](https://img-blog.csdnimg.cn/20210104180120300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

执行结果如下：

![](https://img-blog.csdnimg.cn/20210104180306705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

## 6.4 可以端如何与服务端的配置文件相互对应

-   config.prefix 来对应主配置文件
-   使用 spring.cloud.nacos.config.ext-config 选项来对应更多的文件

    eg：

```null
spring:
  application:
    name: nacos-config-demo-provider
  profiles:
    active: dev
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_SERVER:cdh1:8848}
      config:
        server-addr: ${NACOS_SERVER:cdh1:8848}
        prefix: nacos-config-demo
        group: DEFAULT_GROUP
        file-extension: yaml
        ext-config:
          - data-id: crazymaker-db-dev.yml
            group: DEFAULT_GROUP
            refresh: true
          - data-id: crazymaker-redis-dev.yml
            group: DEFAULT_GROUP
            refresh: true
          - data-id: crazymaker-common-dev.yml
            group: DEFAULT_GROUP
            refresh: true
          - data-id: some.properties
            group: DEFAULT_GROUP
            refresh: true

```

启动程序，发现可以获取到其他 data-id 的配置 , 大家可以自行配置。

## 7、配置的隔离

在实际的应用中，存在着以下几种环境隔离的要求：

1、开发环境、测试环境、准生产环境和生产环境需要隔离

2、不同项目需要隔离

3、同一项目，不同的模块需要隔离

可以通过三种方式来进行配置隔离：Nacos 的服务器、namespace 命名空间、group 分组，在 bootstrap.yml 文件中可以通过配置 Nacos 的 server-addr、namespace 和 group 来区分不同的配置信息。

-   Nacos 的服务器 spring.cloud.nacos.config.server-addr
-   Nacos 的命名空间 spring.cloud.nacos.config.namespace，注意，这里使用命名空间的 ID 不是名称
-   Nacos 的分组 spring.cloud.nacos.config.group

## 8 nacos 集群搭建

如果我们要搭建集群的话，那么肯定是不能用内嵌的数据库，不然数据无法共享。所以，集群搭建的时候我们需要将 Nacos 对接 Mysql 进行数据存储。

集群模式跟我们平时进行扩容是一样的，可以通过 Nginx 转发到多个节点，最前面挂一个域名即可，如下图：![](https://img-blog.csdnimg.cn/20190807114946379.png)

### IP 规划

通常如果我们只是为了体验的话，直接在本地起动 3 个实例就可以了，没必要真的去搞三台服务器，下面我们就以在本地的方式来搭建集群。 将 Nacos 的解压包复制分成 3 份，分别是:

nacos  
nacos1  
nacos2

进入 nacos 的 conf 目录，编辑 application.properties 文件，增加数据库配置

```null

spring.datasource.platform=mysql


db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```

复制代码同样的步骤进入 nacos1 和 nacos2 操作一遍，唯一需要修改的就是 application.properties 文件中的 server.port，默认 nacos 的 server.port=8848，

我们在本地启动三个实例，那么端口肯定会冲突，所以其他 2 个实例的端口我们需要进行修改，比如 nacos1 修改成 8847，nacos2 修改成 8846。  
数据库配置信息好了后，我们需要将对应的数据库和表进行初始化，数据库脚本在 conf 目录下的 nacos-mysql.sql 中，执行即可。

最后一步需要配置一份集群节点信息，配置文件在 conf 目录下的 cluster.conf.example 文件，我们进行重命名成 cluster.conf。 然后编辑 cluster.conf 文件，增加 3 个节点的信息，格式为 IP:PORT，三个目录都一致即可。

```null
127.0.0.1:8848
127.0.0.1:8847
127.0.0.1:8846
```

启动的话直接到 bin 目录下，执行./startup.sh 就可以了，默认就是集群模式，不需要加任何参数。

### 集群的使用

上面的集群，虽然可用， 但仍不是真正的集群， 我们一般不会这么用。nacos 集群的使用一般有 4 种方式：

-   [http://ip1](http://ip1/):port/openAPI 直连 ip 模式，不同的节点，则需要修改 ip 才可以使用。
-   [http://VIP](http://vip/):port/openAPI VIP 模式高可用，客户端 vip 即可，VIP 下面挂 server 真实 ip，部署比较麻烦，需要部署 vip（keepalive）。
-   [http://nacos.com](http://nacos.com/):port/openAPI 域名模式，可读性好，而且换 ip 方便，在 host 文件配置本地域名即可。
-   [http:// 反向代理](http://xn--mnqq3j3fv28i/):port/openAPI 反向代理模式

> 这里介绍一下反向代理模式。

关于 Nginx 的安装和配置，本文就不进行讲解了，不会的可以自己去尝试下，反向代理模式 核心配置如下：

```null
upstream nacos_server {
  server 127.0.0.1:8848;
  server 127.0.0.1:8847;
  server 127.0.0.1:8846;
}

server {
listen 8648;
   server_name localhost;
  
  
  location / {
     proxy_pass http://nacos_server;
     index index.html index.htm;
   }
}

 
```

整体来说，nacos 的集群搭建方式还是挺简单的，没什么特别要注意的，最好是能通过域名的方式来进行访问，另外数据库这块如果上生产环境，也需要考虑高可用问题，至少也得有个主从。

8648 的 nginx 提供的 nacos 服务接口，可以自定义。 我们访问

[http://localhost:8648/nacos/#/clusterManagement?dataId=&group=&appName=&namespace=&serverId=](http://localhost:8648/nacos/#/clusterManagement?dataId=&group=&appName=&namespace=&serverId=)

，就可以看到：

![](https://img2018.cnblogs.com/blog/493842/201907/493842-20190717151826015-620927747.png)

我们可以简单测试一下，杀掉 一个的 nacos ，看服务是否正常。 后面，我们对微服务提供 nacos 服务的时候，只要配置这个 nginx 端口就好了！！ 
 [https://www.cnblogs.com/crazymakercircle/p/14231815.html](https://www.cnblogs.com/crazymakercircle/p/14231815.html)
