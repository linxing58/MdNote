# sentinel 入门教程
| SpringCloud 微服务 精彩博文                                                                       |                                                                                    |
| ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| [**nacos 实战**（史上最全）](https://www.cnblogs.com/crazymakercircle/p/14231815.html)             | [sentinel （史上最全 + 入门教程）](https://www.cnblogs.com/crazymakercircle/p/14285001.html) |
| [**SpringCloud gateway** （史上最全）](https://www.cnblogs.com/crazymakercircle/p/11704077.html) |                                                                                    |

* * *

# sentinel 基本概念

> 开发的原因，需要对吞吐量（TPS）、QPS、并发数、响应时间（RT）几个概念做下了解，查自百度百科，记录如下：
>
> 1.  响应时间 (RT)  
>     　　响应时间是指系统对请求作出响应的时间。直观上看，这个指标与人对软件性能的主观感受是非常一致的，因为它完整地记录了整个计算机系统处理请求的时间。由于一个系统通常会提供许多功能，而不同功能的处理逻辑也千差万别，因而不同功能的响应时间也不尽相同，甚至同一功能在不同输入数据的情况下响应时间也不相同。所以，在讨论一个系统的响应时间时，人们通常是指该系统所有功能的平均时间或者所有功能的最大响应时间。当然，往往也需要对每个或每组功能讨论其平均响应时间和最大响应时间。  
>     　　对于单机的没有并发操作的应用系统而言，人们普遍认为响应时间是一个合理且准确的性能指标。需要指出的是，响应时间的绝对值并不能直接反映软件的性能的高低，软件性能的高低实际上取决于用户对该响应时间的接受程度。对于一个游戏软件来说，响应时间小于 100 毫秒应该是不错的，响应时间在 1 秒左右可能属于勉强可以接受，如果响应时间达到 3 秒就完全难以接受了。而对于编译系统来说，完整编译一个较大规模软件的源代码可能需要几十分钟甚至更长时间，但这些响应时间对于用户来说都是可以接受的。
> 2.  吞吐量 (Throughput)  
>     吞吐量是指系统在单位时间内处理请求的数量。对于无并发的应用系统而言，吞吐量与响应时间成严格的反比关系，实际上此时吞吐量就是响应时间的倒数。前面已经说过，对于单用户的系统，响应时间（或者系统响应时间和应用延迟时间）可以很好地度量系统的性能，但对于并发系统，通常需要用吞吐量作为性能指标。  
>     　　对于一个多用户的系统，如果只有一个用户使用时系统的平均响应时间是 t，当有你 n 个用户使用时，每个用户看到的响应时间通常并不是 n×t，而往往比 n×t 小很多（当然，在某些特殊情况下也可能比 n×t 大，甚至大很多）。这是因为处理每个请求需要用到很多资源，由于每个请求的处理过程中有许多不走难以并发执行，这导致在具体的一个时间点，所占资源往往并不多。也就是说在处理单个请求时，在每个时间点都可能有许多资源被闲置，当处理多个请求时，如果资源配置合理，每个用户看到的平均响应时间并不随用户数的增加而线性增加。实际上，不同系统的平均响应时间随用户数增加而增长的速度也不大相同，这也是采用吞吐量来度量并发系统的性能的主要原因。一般而言，吞吐量是一个比较通用的指标，两个具有不同用户数和用户使用模式的系统，如果其最大吞吐量基本一致，则可以判断两个系统的处理能力基本一致。
> 3.  并发用户数  
>     并发用户数是指系统可以同时承载的正常使用系统功能的用户的数量。与吞吐量相比，并发用户数是一个更直观但也更笼统的性能指标。实际上，并发用户数是一个非常不准确的指标，因为用户不同的使用模式会导致不同用户在单位时间发出不同数量的请求。一网站系统为例，假设用户只有注册后才能使用，但注册用户并不是每时每刻都在使用该网站，因此具体一个时刻只有部分注册用户同时在线，在线用户就在浏览网站时会花很多时间阅读网站上的信息，因而具体一个时刻只有部分在线用户同时向系统发出请求。这样，对于网站系统我们会有三个关于用户数的统计数字：注册用户数、在线用户数和同时发请求用户数。由于注册用户可能长时间不登陆网站，使用注册用户数作为性能指标会造成很大的误差。而在线用户数和同事发请求用户数都可以作为性能指标。相比而言，以在线用户作为性能指标更直观些，而以同时发请求用户数作为性能指标更准确些。
> 4.  QPS 每秒查询率 (Query Per Second)  
>     每秒查询率 QPS 是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准，在因特网上，作为域名系统服务器的机器的性能经常用每秒查询率来衡量。对应 fetches/sec，即每秒的响应请求数，也即是最大吞吐能力。 （看来是类似于 TPS，只是应用于特定场景的吞吐量）

# 1、什么是 Sentinel:

Sentinel 是阿里开源的项目，提供了流量控制、熔断降级、系统负载保护等多个维度来保障服务之间的稳定性。  
官网：[https://github.com/alibaba/Sentinel/wiki](https://github.com/alibaba/Sentinel/wiki)

2012 年，Sentinel 诞生于阿里巴巴，其主要目标是流量控制。2013-2017 年，Sentinel 迅速发展，并成为阿里巴巴所有微服务的基本组成部分。 它已在 6000 多个应用程序中使用，涵盖了几乎所有核心电子商务场景。2018 年，Sentinel 演变为一个开源项目。2020 年，Sentinel Golang 发布。

## **Sentinel 具有以下特征:**

**丰富的应用场景** ：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即  
突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。  
**完备的实时监控** ：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机  
器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。  
**广泛的开源生态** ：Sentinel 提供开箱即用的与其它开源框架 / 库的整合模块，例如与 Spring  
Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。

**完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快  
速地定制逻辑。例如定制规则管理、适配动态数据源等。

**Sentinel 的生态圈**

![](https://pics1.baidu.com/feed/500fd9f9d72a6059c55d49a66cf6bb9d023bba02.png?token=14715c498bc885222d716066ff4a15c0)

## Sentinel 主要特性：

![](https://img-blog.csdnimg.cn/20200529140109886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjQyNDM1OQ==,size_16,color_FFFFFF,t_70)

关于 Sentinel 与 Hystrix 的区别见：[https://yq.aliyun.com/articles/633786/](https://yq.aliyun.com/articles/633786/)

到这已经学习 Sentinel 的基本的使用，在很多的特性和 Hystrix 有很多类似的功能。以下是 Sentinel 和 Hystrix 的对比。

## Sentinel 的使用

**Sentinel 的使用可以分为两个部分:**

-   控制台（Dashboard）：控制台主要负责管理推送规则、监控、集群限流分配管理、机器发现等。
-   核心库（Java 客户端）：不依赖任何框架 / 库，能够运行于 Java 7 及以上的版本的运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。

在这里我们看下控制台的使用

### **Sentinel 中的管理控制台**

### 2.1 获取 Sentinel 控制台

您可以从 [release 页面](https://github.com/alibaba/Sentinel/releases) 下载最新版本的控制台 jar 包。

您也可以从最新版本的源码自行构建 Sentinel 控制台：

-   下载 [控制台](https://github.com/alibaba/Sentinel/tree/master/sentinel-dashboard) 工程
-   使用以下命令将代码打包成一个 fat jar: `mvn clean package`

### 2.2 sentinel 服务启动

```null
java  -server -Xms64m -Xmx256m  -Dserver.port=8849 -Dcsp.sentinel.dashboard.server=localhost:8849 -Dproject.name=sentinel-dashboard -jar /work/sentinel-dashboard-1.7.1.jar 


```

开机启动：启动命令可以加入到启动的 rc.local 配置文件， 之后做到开机启动

# 启动 sentinel

/usr/bin/su - root -c "nohup java -server -Xms64m -Xmx256m -Dserver.port=8849 -Dcsp.sentinel.dashboard.server=localhost:8849 -Dproject.name=sentinel-dashboard -jar /work/sentinel-dashboard-1.7.1.jar 2>&1 &"

除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。

由于调用关系的复杂性，如果调用链路中的某个资源不稳定，最终会导致请求发生堆积。Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

关于熔断降级的介绍见：Sentinel 熔断降级。

下面就使用基于注解的方式实现 Sentinel 的熔断降级的 demo。

> **注意**：启动 Sentinel 控制台需要 JDK 版本为 1.8 及以上版本。

使用如下命令启动控制台：

```null
nohup java  -server -Xms64m -Xmx256m  -Dserver.port=8849 -Dcsp.sentinel.dashboard.server=localhost:8849 -Dproject.name=sentinel-dashboard -jar /work/sentinel-dashboard-1.7.1.jar &
```

其中 -Dserver.port=8849 用于指定 Sentinel 控制台端口为 8849 ， 这个端口可以按需指定。

从 Sentinel 1.6.0 起，Sentinel 控制台引入基本的**登录**功能，默认用户名和密码都是 `sentinel`。可以参考 [鉴权模块文档](https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0#%E9%89%B4%E6%9D%83) 配置用户名和密码。

> 注：若您的应用为 Spring Boot 或 Spring Cloud 应用，您可以通过 Spring 配置文件来指定配置，详情请参考 [Spring Cloud Alibaba Sentinel 文档](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/Sentinel)。（1）获取 Sentinel 控制台  
> 您可以从官方 网站中 下载最新版本的控制台 jar 包，下载地址如下：

```null
https://github.com/alibaba/Sentinel/releases/download/1.6.3/sentinel-dashboard-1.7.1.jar
```

（2）启动  
使用如下命令启动控制台：  
其中 - Dserver.port=8888 用于指定 Sentinel 控制台端口为 8888 。  
从 Sentinel 1.6.0 起，Sentinel 控制台引入基本的登录功能，默认用户名和密码都是 sentinel 。可以参考 鉴权模块文档 配置用户名和密码。

```null
[root@192 ~]# java -Dserver.port=8888 -Dcsp.sentinel.dashboard.server=localhost:8888 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.6.3.jar
INFO: log base dir is: /root/logs/csp/
INFO: log name use pid is: false

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.5.RELEASE)

2020-02-08 13:07:29.316  INFO 114031 --- [           main] c.a.c.s.dashboard.DashboardApplication   : Starting DashboardApplication on 192.168.180.137 with PID 114031 (/root/sentinel-dashboard-1.6.3.jar started by root in /root)
2020-02-08 13:07:29.319  INFO 114031 --- [           main] c.a.c.s.dashboard.DashboardApplication   : No active profile set, falling back to default profiles: default
2020-02-08 13:07:29.456  INFO 114031 --- [           main] ConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@59690aa4: startup date [Sat Feb 08 13:07:29 CST 2020]; root of context hierarchy
2020-02-08 13:07:33.783  INFO 114031 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8888 (http)
```

启动 Sentinel 控制台需要 JDK 版本为 1.8 及以上版本。

[![](https://img2018.cnblogs.com/i-beta/1829785/202002/1829785-20200208130957043-1991794415.png)
](https://img2018.cnblogs.com/i-beta/1829785/202002/1829785-20200208130957043-1991794415.png)

**查看机器列表以及健康情况**  
默认情况下 Sentinel 会在客户端首次调用的时候进行初始化，开始向控制台发送心跳包。也可以配置  
sentinel.eager=true , 取消 Sentinel 控制台懒加载。  
打开浏览器即可展示 Sentinel 的管理控制台

[![](https://img2018.cnblogs.com/i-beta/1829785/202002/1829785-20200208133653172-1748300698.png)
](https://img2018.cnblogs.com/i-beta/1829785/202002/1829785-20200208133653172-1748300698.png)

### **客户端能接入控制台**

控制台启动后，客户端需要按照以下步骤接入到控制台。

父工程引入 alibaba 实现的 SpringCloud

```null
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

子工程中引入 sentinel

```null
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

（ 2）配置启动参数  
在工程的 application.yml 中添加 Sentinel 控制台配置信息

```null
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 192.168.180.137:8888   #sentinel控制台的请求地址
```

这里的 spring.cloud.sentinel.transport.dashboard 配置控制台的请求路径。

## Sentinel 与 Hystrix 的区别

![](https://yqfile.alicdn.com/eb621bf3d522b53ed4704290db32e254130ba9a2.png)

**迁移方案**  
Sentinel 官方提供了详细的由 Hystrix 迁移到 Sentinel 的方法

[![](https://img2018.cnblogs.com/i-beta/1829785/202002/1829785-20200208125155175-979540425.png)
](https://img2018.cnblogs.com/i-beta/1829785/202002/1829785-20200208125155175-979540425.png)

# 2 使用 Sentinel 来进行熔断与限流

Sentinel 可以简单的分为 Sentinel 核心库和 Dashboard。核心库不依赖 Dashboard，但是结合  
Dashboard 可以取得最好的效果。  
使用 Sentinel 来进行熔断保护，主要分为几个步骤:

1.  定义资源

    > 资源：可以是任何东西，一个服务，服务里的方法，甚至是一段代码。
2.  定义规则

    > 规则：Sentinel 支持以下几种规则：流量控制规则、熔断降级规则、系统保护规则、来源访问控制规则  
    > 和 热点参数规则。
3.  检验规则是否生效

Sentinel 的所有规则都可以在内存态中动态地查询及修改，修改之后立即生效. 先把可能需要保护的资源定义好，之后再配置规则。

也可以理解为，只要有了资源，我们就可以在任何时候灵活地定义各种流量控制规则。在编码的时候，只需要考虑这个代码是否需要保护，如果需要保  
护，就将之定义为一个资源。

## **1. 定义资源**

**资源**是 Sentinel 的关键概念。它可以是 Java 应用程序中的任何内容，例如，由应用程序提供的服务，或由应用程序调用的其它应用提供的服务，RPC 接口方法，甚至可以是一段代码。

只要通过 Sentinel API 定义的代码，就是资源，能够被 Sentinel 保护起来。大部分情况下，可以使用方法签名，URL，甚至服务名称作为资源名来标示资源。

把需要控制流量的代码用 Sentinel 的关键代码 SphU.entry("资源名") 和 entry.exit() 包围起来即可。

实例代码：

```null
    Entry entry = null;
    try {
        // 定义一个sentinel保护的资源，名称为test-sentinel-api
        entry = SphU.entry(resourceName);
        // 模拟执行被保护的业务逻辑耗时
        Thread.sleep(100);
        return a;
    } catch (BlockException e) {
        // 如果被保护的资源被限流或者降级了，就会抛出BlockException
        log.warn("资源被限流或降级了", e);
        return "资源被限流或降级了";
    } catch (InterruptedException e) {
        return "发生InterruptedException";
    } finally {
        if (entry != null) {
            entry.exit();
        }
 
        ContextUtil.exit();
    }
}
```

在下面的例子中， 用 try-with-resources 来定义资源。参考代码如下:

```null
public static void main(String[] args) {
    // 配置规则.
    initFlowRules();

    while (true) {
        // 1.5.0 版本开始可以直接利用 try-with-resources 特性
        try (Entry entry = SphU.entry("HelloWorld")) {
            // 被保护的逻辑
            System.out.println("hello world");
	} catch (BlockException ex) {
            // 处理被流控的逻辑
	    System.out.println("blocked!");
	}
    }
}
```

### 资源注解 @SentinelResource

也可以使用 Sentinel 提供的注解 @SentinelResource 来定义资源，实例如下:

```null
@SentinelResource("HelloWorld")
public void helloWorld() {
    // 资源中的逻辑
    System.out.println("hello world");
}
```

### @SentinelResource 注解

> 注意：注解方式埋点不支持 private 方法。

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

-   value：资源名称，必需项（不能为空）
-   entryType：entry 类型，可选项（默认为 EntryType.OUT）
-   blockHandler / blockHandlerClass:

blockHandler 对应处理 BlockException 的函数名称，可选项。blockHandler 函数访问范围需要是 public，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 blockHandlerClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

-   fallback /fallbackClass

    ：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。
-   defaultFallback

    （since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。

#### fallback 函数签名和位置要求：

-   返回值类型必须与原函数返回值类型一致；
-   方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
-   fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

#### defaultFallback 函数签名要求：

-   返回值类型必须与原函数返回值类型一致；
-   方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
-   defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。
-   exceptionsToIgnore（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

## **2. 定义规则**

规则主要有流控规则、 熔断降级规则、系统规则、权限规则、热点参数规则等：

一段硬编码的方式定义流量控制规则如下：

```null
private void initSystemRule() {
    List<SystemRule> rules = new ArrayList<>();
    SystemRule rule = new SystemRule();
    rule.setHighestSystemLoad(10);
    rules.add(rule);
    SystemRuleManager.loadRules(rules);
}
```

加载规则：

```null
FlowRuleManager.loadRules(List<FlowRule> rules); // 修改流控规则
DegradeRuleManager.loadRules(List<DegradeRule> rules); // 修改降级规则
SystemRuleManager.loadRules(List<SystemRule> rules); // 修改系统规则
AuthorityRuleManager.loadRules(List<AuthorityRule> rules); // 修改授权规则
```

# 3 sentinel 熔断降级

### 1 什么是熔断降级

熔断降级对调用链路中不稳定的资源进行熔断降级是保障高可用的重要措施之一。

由于调用关系的复杂性，如果调用链路中的某个资源不稳定，最终会导致请求发生堆积。Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）

### 2. 熔断降级规则

熔断降级规则包含下面几个重要的属性：

| Field | 说明 | 默认值 |
| resource | 资源名，即规则的作用对象 |  |
| grade | 熔断策略，支持慢调用比例 / 异常比例 / 异常数策略 | 慢调用比例 |
| count | 慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例 / 异常数模式下为对应的阈值 |  |
| timeWindow | 熔断时长，单位为 s |  |
| minRequestAmount | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入） | 5 |
| statIntervalMs | 统计时长（单位为 ms），如 60\*1000 代表分钟级（1.8.0 引入） | 1000 ms |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入） |  |

### 3 几种降级策略

我们通常用以下几种降级策略：

-   平均响应时间 (DEGRADE_GRADE_RT)：

    当资源的平均响应时间超过阈值（DegradeRule 中的 count，以 ms 为单位）之后，资源进入准降级状态。如果接下来 1s 内持续进入 5 个请求（即 QPS >= 5），它们的 RT 都持续超过这个阈值，那么在接下的时间窗口（DegradeRule 中的 timeWindow，以 s 为单位）之内，对这个方法的调用都会自动地熔断（抛出 DegradeException）。

    > 注意 Sentinel 默认统计的 RT 上限是 4900 ms，超出此阈值的都会算作 4900 ms，若需要变更此上限可以通过启动配置项 -Dcsp.sentinel.statistic.max.rt=xxx 来配置。
-   异常比例 (DEGRADE_GRADE_EXCEPTION_RATIO)：

    当资源的每秒异常总数占通过量的比值超过阈值（DegradeRule 中的 count）之后，资源进入降级状态，即在接下的时间窗口（DegradeRule 中的 timeWindow，以 s 为单位）之内，对这个方法的调用都会自动地返回。

    > 异常比率的阈值范围是 \[0.0, 1.0]，代表 0% - 100%。
-   异常数 (DEGRADE_GRADE_EXCEPTION_COUNT)：

    当资源近 1 分钟的异常数目超过阈值之后会进行熔断。

    > 注意由于统计时间窗口是分钟级别的，若 timeWindow 小于 60s，则结束熔断状态后仍可能再进入熔断状态。

### 4 熔断降级代码实现

可以通过调用 DegradeRuleManager.loadRules() 方法来用硬编码的方式定义流量控制规则。

```null
    @PostConstruct
    public void initSentinelRule()
    {
        //熔断规则： 5s内调用接口出现异常次数超过5的时候, 进行熔断
        List<DegradeRule> degradeRules = new ArrayList<>();
        DegradeRule rule = new DegradeRule();
        rule.setResource("queryGoodsInfo");
        rule.setCount(5);

        rule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT);//熔断规则
        rule.setTimeWindow(5);
        degradeRules.add(rule);
        DegradeRuleManager.loadRules(degradeRules);
    }
```

> 具体源码，请参见疯狂创客圈 crazy-springcloud 源码工程

### 5 控制台降级规则

配置

![](https://img-blog.csdnimg.cn/20210116194700580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

参数

| Field      | 说明                      | 默认值 |
| ---------- | ----------------------- | --- |
| resource   | 资源名，即限流规则的作用对象          |     |
| count      | 阈值                      |     |
| grade      | 降级模式，根据 RT 降级还是根据异常比例降级 | RT  |
| timeWindow | 降级的时间，单位为 s             |     |

### 6 与 Hystrix 的熔断对比：

Hystrix 常用的线程池隔离会造成线程上下切换的 overhead 比较大；Hystrix 使用的信号量隔离对某个资源调用的并发数进行控制，效果不错，但是无法对慢调用进行自动降级；

Sentinel 通过并发线程数的流量控制提供信号量隔离的功能；此外，Sentinel 支持的熔断降级维度更多，可对多种指标进行流控、熔断，且提供了实时监控和控制面板，功能更为强大。

# 4 Sentinel 流控（限流）

流量控制 (Flow Control)，原理是监控应用流量的 QPS 或并发线程数等指标，当达到指定阈值时对流量进行控制，避免系统被瞬时的流量高峰冲垮，保障应用高可用性。

通过流控规则来指定允许该资源通过的请求次数，例如下面的代码定义了资源 HelloWorld 每秒最多只能通过 20 个请求。 参考的规则定义如下：

```null
private static void initFlowRules(){
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    rule.setResource("HelloWorld");
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // Set limit QPS to 20.
    rule.setCount(20);
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

一条限流规则主要由下面几个因素组成，我们可以组合这些元素来实现不同的限流效果：

-   `resource`：资源名，即限流规则的作用对象
-   `count`: 限流阈值
-   `grade`: 限流阈值类型（QPS 或并发线程数）
-   `limitApp`: 流控针对的调用来源，若为 `default` 则不区分调用来源
-   `strategy`: 调用关系限流策略
-   `controlBehavior`: 流量控制效果（直接拒绝、Warm Up、匀速排队）

### 基本的参数

**资源名**：唯一名称，默认请求路径

**针对来源**：Sentinel 可以针对调用者进行限流，填写微服务名，默认为 default(不区分来源)

**阈值类型 / 单机阈值：** 

1.QPS：每秒请求数，当前调用该 api 的 QPS 到达阈值的时候进行限流

2\. 线程数：当调用该 api 的线程数到达阈值的时候，进行限流

**是否集群**：是否为集群

### **流控的几种`strategy`**：

1\. 直接：当 api 大达到限流条件时，直接限流

2\. 关联：当关联的资源到达阈值，就限流自己

3\. 链路：只记录指定路上的流量，指定资源从入口资源进来的流量，如果达到阈值，就进行限流，api 级别的限流

## **4.1 直接失败模式**

### 使用 API 进行资源定义

```null

    /**
     * 限流实现方式一: 抛出异常的方式定义资源
     *
     * @param orderId
     * @return
     */
    @ApiOperation(value = "纯代码限流")
    @GetMapping("/getOrder")
    @ResponseBody
    public String getOrder(@RequestParam(value = "orderId", required = false)String orderId)
    {

        Entry entry = null;
        // 资源名
        String resourceName = "getOrder";
        try
        {
            // entry可以理解成入口登记
            entry = SphU.entry(resourceName);
            // 被保护的逻辑, 这里为订单查询接口
            return "正常的业务逻辑 OrderInfo :" + orderId;
        } catch (BlockException blockException)
        {
            // 接口被限流的时候, 会进入到这里
            log.warn("---getOrder1接口被限流了---, exception: ", blockException);
            return "接口限流, 返回空";
        } finally
        {
            // SphU.entry(xxx) 需要与 entry.exit() 成对出现,否则会导致调用链记录异常
            if (entry != null)
            {
                entry.exit();
            }
        }

    }
```

### 代码限流规则

```null
//限流规则 QPS mode,
        List<FlowRule> rules = new ArrayList<FlowRule>();
        FlowRule rule1 = new FlowRule();
        rule1.setResource("getOrder");
        // QPS控制在2以内
        rule1.setCount(2);
        // QPS限流
        rule1.setGrade(RuleConstant.FLOW_GRADE_QPS);
        rule1.setLimitApp("default");
        rules.add(rule1);
        FlowRuleManager.loadRules(rules);

```

### 网页限流规则配置

选择 QPS，直接，快速失败，单机阈值为 2。

配置

!\[流控规则](![](https://img-blog.csdnimg.cn/20210116194515175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

参数

| Field           | 说明                                             | 默认值                 |
| --------------- | ---------------------------------------------- | ------------------- |
| resource        | 资源名，资源名是限流规则的作用对象                              |                     |
| count           | 限流阈值                                           |                     |
| grade           | 限流阈值类型，QPS 或线程数模式                              | QPS 模式              |
| limitApp        | 流控针对的调用来源                                      | `default`，代表不区分调用来源 |
| strategy        | 判断的根据是资源自身，还是根据其它关联资源 (`refResource`)，还是根据链路入口 | 根据资源本身              |
| controlBehavior | 流控效果（直接拒绝 / 排队等待 / 慢启动模式）                      | 直接拒绝                |

### 测试

频繁刷新请求，1 秒访问 2 次请求，正常，超过设置的阈值，将报默认的错误。

![](https://pics4.baidu.com/feed/d6ca7bcb0a46f21f736fcc3eb7e6e4660d33ae49.png?token=21c375e0121e5fac15a82a07284c66aa)

再次的 1 秒访问 2 次请求，访问正常。超过 2 次，访问异常

## 4.2 关联模式

调用关系包括调用方、被调用方；一个方法又可能会调用其它方法，形成一个调用链路的层次关系。Sentinel 通过 `NodeSelectorSlot` 建立不同资源间的调用的关系，并且通过 `ClusterBuilderSlot` 记录每个资源的实时统计信息。

当两个资源之间具有资源争抢或者依赖关系的时候，这两个资源便具有了关联。

比如对数据库同一个字段的读操作和写操作存在争抢，读的速度过高会影响写得速度，写的速度过高会影响读的速度。如果放任读写操作争抢资源，则争抢本身带来的开销会降低整体的吞吐量。可使用关联限流来避免具有关联关系的资源之间过度的争抢.

举例来说，`read_db` 和 `write_db` 这两个资源分别代表数据库读写，我们可以给 `read_db` 设置限流规则来达到写优先的目的。具体的方法：

```null
设置 `strategy` 为 `RuleConstant.STRATEGY_RELATE` 
设置 `refResource` 为 `write_db`。
这样当写库操作过于频繁时，读数据的请求会被限流。
```

还有一个例子，电商的 下订单 和 支付两个操作，需要优先保障 支付， 可以根据 支付接口的 流量阈值，来对订单接口进行限制，从而保护支付的目的。

### 使用注解进行资源定义

添加 2 个请求

```null
   @SentinelResource(value = "test1", blockHandler = "exceptionHandler")
    @GetMapping("/test1")
    public String test1()
    {
        log.info(Thread.currentThread().getName() + "\t" + "...test1");
        return "-------hello baby，i am test1";
    }


    // Block 异常处理函数，参数最后多一个 BlockException，其余与原函数一致.
    public String exceptionHandler(BlockException ex)
    {
        // Do some log here.
        ex.printStackTrace();
        log.info(Thread.currentThread().getName() + "\t" + "...exceptionHandler");
        return String.format("error: test1  is not OK");
    }

    @SentinelResource(value = "test1_ref")
    @GetMapping("/test1_ref")
    public String test1_ref()
    {
        log.info(Thread.currentThread().getName() + "\t" + "...test1_related");
        return "-------hello baby，i am test1_ref";
    }

```

### 代码配置关联限流规则

```null

        // 关联模式流控  QPS控制在1以内
        String refResource = "test1_ref";
        FlowRule rRule = new FlowRule("test1")
                .setCount(1)  // QPS控制在1以内
                .setStrategy(RuleConstant.STRATEGY_RELATE)
                .setRefResource(refResource);

        rules.add(rRule);
        FlowRuleManager.loadRules(rules);

```

### 网页限流规则配置

![](https://img-blog.csdnimg.cn/20210115180339225.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

### 测试

选择 QPS，单机阈值为 1，选择关联，关联资源为 / test_ref，这里用 Jmeter 模拟高并发，请求 / test_ref。

![](https://pics5.baidu.com/feed/a6efce1b9d16fdfa1299f057f44d035296ee7bde.png?token=ced899c66307cdd18da13c06403df487)

在大批量线程高并发访问 / test_ref，导致 / test 失效了

![](https://pics4.baidu.com/feed/d6ca7bcb0a46f21f736fcc3eb7e6e4660d33ae49.png?token=21c375e0121e5fac15a82a07284c66aa)

链路类型的关联也类似，就不再演示了。多个请求调用同一微服务。

## 4.3 Warm up（预热）模式

当流量突然增大的时候，我们常常会希望系统从空闲状态到繁忙状态的切换的时间长一些。即如果系统在此之前长期处于空闲的状态，我们希望处理请求的数量是缓步的增多，经过预期的时间以后，到达系统处理请求个数的最大值。Warm Up（冷启动，预热）模式就是为了实现这个目的的。

默认 coldFactor 为 3，即请求 QPS 从 threshold / 3 开始，经预热时长逐渐升至设定的 QPS 阈值。

### 使用注解定义资源

```null
    @SentinelResource(value = "testWarmUP", blockHandler = "exceptionHandlerOfWarmUp")
    @GetMapping("/testWarmUP")
    public String testWarmUP()
    {
        log.info(Thread.currentThread().getName() + "\t" + "...test1");
        return "-------hello baby，i am testWarmUP";
    }

```

### 代码限流规则

```null

        FlowRule warmUPRule = new FlowRule();
        warmUPRule.setResource("testWarmUP");
        warmUPRule.setCount(20);
        warmUPRule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        warmUPRule.setLimitApp("default");
        warmUPRule.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_WARM_UP);
        warmUPRule.setWarmUpPeriodSec(10);

```

### 网页限流规则配置

![](https://img-blog.csdnimg.cn/20210115201946419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

先在单机阈值 10/3，3 的时候，预热 10 秒后，慢慢将阈值升至 20。刚开始刷 / testWarmUP，会出现默认错误，预热时间到了后，阈值增加，没超过阈值刷新，请求正常。

通常冷启动的过程系统允许通过的 QPS 曲线如下图所示：

![](https://pics7.baidu.com/feed/9922720e0cf3d7ca95ce8b6db1dd310f6b63a93e.png?token=9a8b021a70b874e2d690ef0a1ab04ab0)

如秒杀系统在开启瞬间，会有很多流量上来，很可能把系统打死，预热方式就是为了保护系统，可慢慢的把流量放进来，慢慢的把阈值增长到设置的阈值。

### 通过 jmeter 进行测试

## 4.4 排队等待模式

匀速排队（RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。阈值必须设置为 QPS。

![](https://pics2.baidu.com/feed/42166d224f4a20a4de73da50d0901724730ed005.png?token=d2a53aac9415b83e46126f0781ae8e4c)

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

某瞬时来了大流量的请求, 而如果此时要处理所有请求，很可能会导致系统负载过高，影响稳定性。但其实可能后面几秒之内都没有消息投递，若直接把多余的消息丢掉则没有充分利用系统处理消息的能力。Sentinel 的 Rate Limiter 模式能在某一段时间间隔内以匀速方式处理这样的请求, 充分利用系统的处理能力, 也就是削峰填谷, 保证资源的稳定性.

Sentinel 会以固定的间隔时间让请求通过, 访问资源。当请求到来的时候，如果当前请求距离上个通过的请求通过的时间间隔不小于预设值，则让当前请求通过；否则，计算当前请求的预期通过时间，如果该请求的预期通过时间小于规则预设的 timeout 时间，则该请求会等待直到预设时间到来通过；反之，则马上抛出阻塞异常。

使用 Sentinel 的这种策略, 简单点说, 就是使用一个时间段 (比如 20s 的时间) 处理某一瞬时产生的大量请求, 起到一个削峰填谷的作用, 从而充分利用系统的处理能力, 下图能很形象的展示这种场景: X 轴代表时间, Y 轴代表系统处理的请求.

![](https://img-blog.csdnimg.cn/20190219142010891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpb25neGlhbnpl,size_16,color_FFFFFF,t_70)

### 示例

模拟 2 个用户同时并发的访问资源，发出 100 个请求,

如果设置 QPS 阈值为 1, 拒绝策略修改为 Rate Limiter 匀速 RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER 方式, 还需要设置 setMaxQueueingTimeMs(20 \* 1000) 表示每一请求最长等待时间, 这里等待时间大一点, 以保证让所有请求都能正常通过;

假设这里设置的排队等待时间过小的话, 导致排队等待的请求超时而抛出异常 BlockException, 最终结果可能是这 100 个并发请求中只有一个请求或几个才能正常通过, 所以使用这种模式得根据访问资源的耗时时间决定排队等待时间. 按照目前这种设置, QPS 阈值为 10 的话, 每一个请求相当于是以匀速 100ms 左右通过.

### 使用注解定义资源

```null
  
    
    @SentinelResource(value = "testLineUp",
    					blockHandler = "exceptionHandlerOftestLineUp")
    @GetMapping("/testLineUp")
        public String testLineUp()
    {
        log.info(Thread.currentThread().getName() + "\t" + "...test1");
        return "-------hello baby，i am testLineUp";
    }
```

### 代码限流规则

```null
    
        FlowRule lineUpRule = new FlowRule();
        lineUpRule.setResource("testLineUp");
        lineUpRule.setCount(10);
        lineUpRule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        lineUpRule.setLimitApp("default");
        lineUpRule.setMaxQueueingTimeMs(20 * 1000);
        // CONTROL_BEHAVIOR_DEFAULT means requests more than threshold will be rejected immediately.
        // CONTROL_BEHAVIOR_DEFAULT将超过阈值的流量立即拒绝掉.
        lineUpRule.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER);
        rules.add(lineUpRule);
```

### 网页限流规则配置

![](https://img-blog.csdnimg.cn/20210115201054724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

### 通过 jmeter 进行测试

![](https://img-blog.csdnimg.cn/20210115201802998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

## 4.5 热点规则 (ParamFlowRule)

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

-   商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
-   用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制 热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。 使用该规则需要引入依赖：

热点参数规则（`ParamFlowRule`）类似于流量控制规则（`FlowRule`）：

| 属性                | 说明                                                            | 默认值     |
| ----------------- | ------------------------------------------------------------- | ------- |
| resource          | 资源名，必填                                                        |         |
| count             | 限流阈值，必填                                                       |         |
| grade             | 限流模式                                                          | QPS 模式  |
| durationInSec     | 统计窗口时间长度（单位为秒），1.6.0 版本开始支持                                   | 1s      |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持                              | 快速失败    |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持                             | 0ms     |
| paramIdx          | 热点参数的索引，必填，对应 `SphU.entry(xxx, args)` 中的参数索引位置                |         |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型和字符串类型** |         |
| clusterMode       | 是否是集群参数流控规则                                                   | `false` |
| clusterConfig     | 集群流控相关配置                                                      |         |

### 自定义资源

```null

    @GetMapping("/byHotKey")
    @SentinelResource(value = "byHotKey",
            blockHandler = "userAccessError")
    public String test4(@RequestParam(value = "userId", required = false) String userId,
                        @RequestParam(value = "goodId", required = false) int goodId)
    {
        log.info(Thread.currentThread().getName() + "\t" + "...byHotKey");
        return "-----------by HotKey： UserId";
    }

```

### 限流规则代码：

可以通过 ParamFlowRuleManager 的 loadRules 方法更新热点参数规则，下面是官方实例：

```null
ParamFlowRule rule = new ParamFlowRule(resourceName)
    .setParamIdx(0)
    .setCount(5);
// 针对 int 类型的参数 PARAM_B，单独设置限流 QPS 阈值为 10，而不是全局的阈值 5.
ParamFlowItem item = new ParamFlowItem().setObject(String.valueOf(PARAM_B))
    .setClassType(int.class.getName())
    .setCount(10);
rule.setParamFlowItemList(Collections.singletonList(item));

ParamFlowRuleManager.loadRules(Collections.singletonList(rule));
```

具体的限流代码如下：

```null
        ParamFlowRule pRule = new ParamFlowRule("byHotKey")
                .setParamIdx(1)
                .setCount(1);
// 针对 参数值1000，单独设置限流 QPS 阈值为 5，而不是全局的阈值 1.
        ParamFlowItem item = new ParamFlowItem().setObject(String.valueOf(1000))
                .setClassType(int.class.getName())
                .setCount(5);
        pRule.setParamFlowItemList(Collections.singletonList(item));

        ParamFlowRuleManager.loadRules(Collections.singletonList(pRule));

```

### 网页限流规则配置

![](https://img-blog.csdnimg.cn/20210116194057665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

# 5. Sentinel 系统保护

### 系统保护的目的

在开始之前，我们先了解一下系统保护的目的：

-   保证系统不被拖垮
-   在系统稳定的前提下，保持系统的吞吐量

长期以来，系统保护的思路是根据硬指标，即系统的负载 (load1) 来做系统过载保护。当系统负载高于某个阈值，就禁止或者减少流量的进入；当 load 开始好转，则恢复流量的进入。这个思路给我们带来了不可避免的两个问题：

-   load 是一个 “结果”，如果根据 load 的情况来调节流量的通过率，那么就始终有延迟性。也就意味着通过率的任何调整，都会过一段时间才能看到效果。当前通过率是使 load 恶化的一个动作，那么也至少要过 1 秒之后才能观测到；同理，如果当前通过率调整是让 load 好转的一个动作，也需要 1 秒之后才能继续调整，这样就浪费了系统的处理能力。所以我们看到的曲线，总是会有抖动。
-   恢复慢。想象一下这样的一个场景（真实），出现了这样一个问题，下游应用不可靠，导致应用 RT 很高，从而 load 到了一个很高的点。过了一段时间之后下游应用恢复了，应用 RT 也相应减少。这个时候，其实应该大幅度增大流量的通过率；但是由于这个时候 load 仍然很高，通过率的恢复仍然不高。

系统保护的目标是 **在系统不被拖垮的情况下，提高系统的吞吐率，而不是 load 一定要到低于某个阈值**。如果我们还是按照固有的思维，超过特定的 load 就禁止流量进入，系统 load 恢复就放开流量，这样做的结果是无论我们怎么调参数，调比例，都是按照果来调节因，都无法取得良好的效果。

Sentinel 在系统自适应保护的做法是，用 load1 作为启动自适应保护的因子，而允许通过的流量由处理请求的能力，即请求的响应时间以及当前系统正在处理的请求速率来决定。

### 系统保护规则的应用

系统规则支持以下的模式：

-   **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
-   **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
-   **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
-   **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
-   **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则的参数说明:

-   highestSystemLoad 最大的 load1，参考值 -1 (不生效)
-   avgRt 所有入口流量的平均响应时间 -1 (不生效)
-   maxThread 入口流量的最大并发数 -1 (不生效)
-   qps 所有入口资源的 QPS -1 (不生效)

硬编码的方式定义流量控制规则如下：

```null
       List<SystemRule> srules = new ArrayList<>();
        SystemRule srule = new SystemRule();
        srule.setAvgRt(3000);
        srules.add(srule);
        SystemRuleManager.loadRules(srules);
```

### 网页限流规则配置

![](https://img-blog.csdnimg.cn/20210116194809712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NyYXp5bWFrZXJjaXJjbGU=,size_16,color_FFFFFF,t_70)

# 6 黑白名单规则

很多时候，我们需要根据调用方来限制资源是否通过，这时候可以使用 Sentinel 的访问控制（黑白名单）的功能。黑白名单根据资源的请求来源（origin）限制资源是否通过，若配置白名单则只有请求来源位于白名单内时才可通过；若配置黑名单则请求来源位于黑名单时不通过，其余的请求通过。

> 调用方信息通过 ContextUtil.enter(resourceName, origin) 方法中的 origin 参数传入。

## 访问控制规则 (AuthorityRule)

授权规则，即黑白名单规则（AuthorityRule）非常简单，主要有以下配置项：

-   resource：资源名，即限流规则的作用对象
-   limitApp：对应的黑名单 / 白名单，不同 origin 用 , 分隔，如 appA,appB
-   strategy：限制模式，AUTHORITY_WHITE 为白名单模式，AUTHORITY_BLACK 为黑名单模式，默认为白名单模式 比如我们希望控制对资源 test 的访问设置白名单，只有来源为 appA 和 appB 的请求才可通过，则可以配置如下白名单规则：

```null
AuthorityRule rule = new AuthorityRule();
rule.setResource("test");
rule.setStrategy(RuleConstant.AUTHORITY_WHITE);
rule.setLimitApp("appA,appB");
AuthorityRuleManager.loadRules(Collections.singletonList(rule));
```

# 7 核心组件

### Resource

resource 是 sentinel 中最重要的一个概念，sentinel 通过资源来保护具体的业务代码或其他后方服务。sentinel 把复杂的逻辑给屏蔽掉了，用户只需要为受保护的代码或服务定义一个资源，然后定义规则就可以了，剩下的通通交给 sentinel 来处理了。并且资源和规则是解耦的，规则甚至可以在运行时动态修改。定义完资源后，就可以通过在程序中埋点来保护你自己的服务了，埋点的方式有两种：

-   try-catch 方式（`通过 SphU.entry(...)`），当 catch 到 BlockException 时执行异常处理 (或 fallback)
-   if-else 方式（`通过 SphO.entry(...)`），当返回 false 时执行异常处理 (或 fallback)

以上这两种方式都是通过硬编码的形式定义资源然后进行资源埋点的，对业务代码的侵入太大，从 0.1.1 版本开始，sentinel 加入了注解的支持，可以通过注解来定义资源，具体的注解为：SentinelResource 。通过注解除了可以定义资源外，还可以指定 blockHandler 和 fallback 方法。

在 sentinel 中具体表示资源的类是：ResourceWrapper ，他是一个抽象的包装类，包装了资源的 Name 和 EntryType。他有两个实现类，分别是：StringResourceWrapper 和 MethodResourceWrapper。顾名思义，StringResourceWrapper 是通过对一串字符串进行包装，是一个通用的资源包装类，MethodResourceWrapper 是对方法调用的包装。

### Context

Context 是对资源操作时的上下文环境，每个资源操作 (`针对 Resource 进行的 entry/exit`) 必须属于一个 Context，如果程序中未指定 Context，会创建 name 为 "sentinel_default_context" 的默认 Context。一个 Context 生命周期内可能有多个资源操作，Context 生命周期内的最后一个资源 exit 时会清理该 Context，这也预示这整个 Context 生命周期的结束。Context 主要属性如下：

```null
public class Context {
   // context名字，默认名字 "sentinel_default_context"
   private final String name;
   // context入口节点，每个context必须有一个entranceNode
   private DefaultNode entranceNode;
   // context当前entry，Context生命周期中可能有多个Entry，所有curEntry会有变化
   private Entry curEntry;
   // The origin of this context (usually indicate different invokers, e.g. service consumer name or origin IP).
   private String origin = "";
   private final boolean async;
}
```

注意：一个 Context 生命期内 Context 只能初始化一次，因为是存到 ThreadLocal 中，并且只有在非 null 时才会进行初始化。

如果想在调用 SphU.entry() 或 SphO.entry() 前，自定义一个 context，则通过 ContextUtil.enter() 方法来创建。context 是保存在 ThreadLocal 中的，每次执行的时候会优先到 ThreadLocal 中获取，为 null 时会调用 `MyContextUtil.myEnter(Constants.CONTEXT_DEFAULT_NAME, "", resourceWrapper.getType())`创建一个 context。当 Entry 执行 exit 方法时，如果 entry 的 parent 节点为 null，表示是当前 Context 中最外层的 Entry 了，此时将 ThreadLocal 中的 context 清空。

#### Context 的创建与销毁

首先我们要清楚的一点就是，每次执行 entry() 方法，试图冲破一个资源时，都会生成一个上下文。这个上下文中会保存着调用链的根节点和当前的入口。

Context 是通过 ContextUtil 创建的，具体的方法是 trueEntry，代码如下：

```null
protected static Context trueEnter(String name, String origin) {
    // 先从ThreadLocal中获取
    Context context = contextHolder.get();
    if (context == null) {
        // 如果ThreadLocal中获取不到Context
        // 则根据name从map中获取根节点，只要是相同的资源名，就能直接从map中获取到node
        Map<String, DefaultNode> localCacheNameMap = contextNameNodeMap;
        DefaultNode node = localCacheNameMap.get(name);
        if (node == null) {
            // 省略部分代码
            try {
                LOCK.lock();
                node = contextNameNodeMap.get(name);
                if (node == null) {
                    // 省略部分代码
                    // 创建一个新的入口节点
                    node = new EntranceNode(new StringResourceWrapper(name, EntryType.IN), null);
                    Constants.ROOT.addChild(node);
                    // 省略部分代码
                }
            } finally {
                LOCK.unlock();
            }
        }
        // 创建一个新的Context，并设置Context的根节点，即设置EntranceNode
        context = new Context(node, name);
        context.setOrigin(origin);
        // 将该Context保存到ThreadLocal中去
        contextHolder.set(context);
    }
    return context;
}
```

上面的代码中我省略了部分代码，只保留了核心的部分。从源码中还是可以比较清晰的看出生成 Context 的过程：

-   1\. 先从 ThreadLocal 中获取，如果能获取到直接返回，如果获取不到则继续第 2 步
-   2\. 从一个 static 的 map 中根据上下文的名称获取，如果能获取到则直接返回，否则继续第 3 步
-   3\. 加锁后进行一次 double check，如果还是没能从 map 中获取到，则创建一个 EntranceNode，并把该 EntranceNode 添加到一个全局的 ROOT 节点中去，然后将该节点添加到 map 中去 (这部分代码在上述代码中省略了)
-   4\. 根据 EntranceNode 创建一个上下文，并将该上下文保存到 ThreadLocal 中去，下一个请求可以直接获取

那保存在 ThreadLocal 中的上下文什么时候会清除呢？从代码中可以看到具体的清除工作在 ContextUtil 的 exit 方法中，当执行该方法时，会将保存在 ThreadLocal 中的 context 对象清除，具体的代码非常简单，这里就不贴代码了。

那 ContextUtil.exit 方法什么时候会被调用呢？有两种情况：一是主动调用 ContextUtil.exit 的时候，二是当一个入口 Entry 要退出，执行该 Entry 的 trueExit 方法的时候，此时会触发 ContextUtil.exit 的方法。但是有一个前提，就是当前 Entry 的父 Entry 为 null 时，此时说明该 Entry 已经是最顶层的根节点了，可以清除 context。

### Entry

刚才在 Context 身影中也看到了 Entry 的出现，现在就谈谈 Entry。每次执行 SphU.entry() 或 SphO.entry() 都会返回一个 Entry，Entry 表示一次资源操作，内部会保存当前 invocation 信息。在一个 Context 生命周期中多次资源操作，也就是对应多个 Entry，这些 Entry 形成 parent/child 结构保存在 Entry 实例中，entry 类 CtEntry 结构如下：

```null
class CtEntry extends Entry {
   protected Entry parent = null;
   protected Entry child = null;

   protected ProcessorSlot<Object> chain;
   protected Context context;
}
public abstract class Entry implements AutoCloseable {
   private long createTime;
   private Node curNode;
   /**
    * {@link Node} of the specific origin, Usually the origin is the Service Consumer.
    */
   private Node originNode;
   private Throwable error; // 是否出现异常
   protected ResourceWrapper resourceWrapper; // 资源信息
}
```

Entry 实例代码中出现了 Node，这个又是什么东东呢 😦，咱们接着往下看：

### DefaultNode

Node（_关于 StatisticNode 的讨论放到下一小节_）默认实现类 DefaultNode，该类还有一个子类 EntranceNode；context 有一个 entranceNode 属性，Entry 中有一个 curNode 属性。

-   **EntranceNode**：该类的创建是在初始化 Context 时完成的（ContextUtil.trueEnter 方法），注意该类是针对 Context 维度的，也就是一个 context 有且仅有一个 EntranceNode。
-   **DefaultNode**：该类的创建是在 NodeSelectorSlot.entry 完成的，当不存在 context.name 对应的 DefaultNode 时会新建（new DefaultNode(resourceWrapper, null)，对应 resouce）并保存到本地缓存（NodeSelectorSlot 中 private volatile Map&lt;String, DefaultNode> map）；获取到 context.name 对应的 DefaultNode 后会将该 DefaultNode 设置到当前 context 的 curEntry.curNode 属性，也就是说，在 NodeSelectorSlot 中是一个 context 有且仅有一个 DefaultNode。

看到这里，你是不是有疑问？为什么一个 context 有且仅有一个 DefaultNode，我们的 resouece 跑哪去了呢，其实，这里的一个 context 有且仅有一个 DefaultNode 是在 NodeSelectorSlot 范围内，NodeSelectorSlot 是 ProcessorSlotChain 中的一环，获取 ProcessorSlotChain 是根据 Resource 维度来的。总结为一句话就是：**针对同一个 Resource，多个 context 对应多个 DefaultNode；针对不同 Resource，(不管是否是同一个 context) 对应多个不同 DefaultNode**。这还没看明白 : (，好吧，我不 bb 了，上图吧：

public class DefaultNode extends StatisticNode {  
private ResourceWrapper id;  
/\*\*  
\* The list of all child nodes.  
\* 子节点集合  
_/  
private volatile Set childList = new HashSet&lt;>();  
/_\*  
\* Associated cluster node.  
\*/  
private ClusterNode clusterNode;  
}

一个 Resouce 只有一个 clusterNode，多个 defaultNode 对应一个 clusterNode，如果 defaultNode.clusterNode 为 null，则在 ClusterBuilderSlot.entry 中会进行初始化。

同一个 Resource，对应同一个 ProcessorSlotChain，这块处理逻辑在 lookProcessChain 方法中，如下：

ProcessorSlot lookProcessChain(ResourceWrapper resourceWrapper) {  
ProcessorSlotChain chain = chainMap.get(resourceWrapper);  
if (chain == null) {  
synchronized (LOCK) {  
chain = chainMap.get(resourceWrapper);  
if (chain == null) {  
// Entry size limit.  
if (chainMap.size() >= Constants.MAX_SLOT_CHAIN_SIZE) {  
return null;  
}

```null
           chain = SlotChainProvider.newSlotChain();
           Map<ResourceWrapper, ProcessorSlotChain> newMap = newHashMap<ResourceWrapper, ProcessorSlotChain>(
               chainMap.size() + 1);
           newMap.putAll(chainMap);
           newMap.put(resourceWrapper, chain);
           chainMap = newMap;
      }
  }
```

}  
return chain;  
}

### StatisticNode

```null
StatisticNode中保存了资源的实时统计数据（基于滑动时间窗口机制），通过这些统计数据，sentinel才能进行限流、降级等一系列操作。StatisticNode属性如下：

public class StatisticNode implements Node {
   /**
    * 秒级的滑动时间窗口（时间窗口单位500ms）
    */
   private transient volatile Metric rollingCounterInSecond = newArrayMetric(SampleCountProperty.SAMPLE_COUNT,
       IntervalProperty.INTERVAL);
   /**
    * 分钟级的滑动时间窗口（时间窗口单位1s）
    */
   private transient Metric rollingCounterInMinute = new ArrayMetric(60, 60 * 1000,false);
   /**
    * The counter for thread count.

 线程个数用户触发线程数流控
*/
private LongAdder curThreadNum = new LongAdder();
}
public class ArrayMetric implements Metric {
      private final LeapArray<MetricBucket> data;
}
public class MetricBucket {
// 保存统计值
private final LongAdder[] counters;
// 最小rt
private volatile long minRt;
}
```

其中 MetricBucket.counters 数组大小为 MetricEvent 枚举值的个数，每个枚举对应一个统计项，比如 PASS 表示通过个数，限流可根据通过的个数和设置的限流规则配置 count 大小比较，得出是否触发限流操作，所有枚举值如下：

```null
public enum MetricEvent {
   PASS, // Normal pass.
   BLOCK, // Normal block.
   EXCEPTION,
   SUCCESS,
   RT,
   OCCUPIED_PASS
}
```

# 8 插槽 Slot

slot 是另一个 sentinel 中非常重要的概念，sentinel 的工作流程就是围绕着一个个插槽所组成的插槽链来展开的。需要注意的是每个插槽都有自己的职责，他们各司其职完好的配合，通过一定的编排顺序，来达到最终的限流降级的目的。默认的各个插槽之间的顺序是固定的，因为有的插槽需要依赖其他的插槽计算出来的结果才能进行工作。

但是这并不意味着我们只能按照框架的定义来，sentinel 通过 SlotChainBuilder 作为 SPI 接口，使得 Slot Chain 具备了扩展的能力。我们可以通过实现 SlotsChainBuilder 接口加入自定义的 slot 并自定义编排各个 slot 之间的顺序，从而可以给 sentinel 添加自定义的功能。

那 SlotChain 是在哪创建的呢？是在 CtSph.lookProcessChain() 方法中创建的，并且该方法会根据当前请求的资源先去一个静态的 HashMap 中获取，如果获取不到才会创建，创建后会保存到 HashMap 中。这就意味着，同一个资源会全局共享一个 SlotChain。默认生成 ProcessorSlotChain 为：

```null
// DefaultSlotChainBuilder
public ProcessorSlotChain build() {
   ProcessorSlotChain chain = new DefaultProcessorSlotChain();
   chain.addLast(new NodeSelectorSlot());
   chain.addLast(new ClusterBuilderSlot());
   chain.addLast(new LogSlot());
   chain.addLast(new StatisticSlot());
   chain.addLast(new SystemSlot());
   chain.addLast(new AuthoritySlot());
   chain.addLast(new FlowSlot());
   chain.addLast(new DegradeSlot());

   return chain;
```

这里大概的介绍下每种 Slot 的功能职责：

-   `NodeSelectorSlot` 负责收集资源的路径，并将这些资源的调用路径，以树状结构存储起来，用于根据调用路径来限流降级；
-   `ClusterBuilderSlot` 则用于存储资源的统计信息以及调用者信息，例如该资源的 RT, QPS, thread count 等等，这些信息将用作为多维度限流，降级的依据；
-   `StatisticsSlot` 则用于记录，统计不同维度的 runtime 信息；
-   `SystemSlot` 则通过系统的状态，例如 load1 等，来控制总的入口流量；
-   `AuthoritySlot` 则根据黑白名单，来做黑白名单控制；
-   `FlowSlot` 则用于根据预设的限流规则，以及前面 slot 统计的状态，来进行限流；
-   `DegradeSlot` 则通过统计信息，以及预设的规则，来做熔断降级；

每个 Slot 执行完业务逻辑处理后，会调用 fireEntry() 方法，该方法将会触发下一个节点的 entry 方法，下一个节点又会调用他的 fireEntry，以此类推直到最后一个 Slot，由此就形成了 sentinel 的责任链。

下面我们就来详细研究下这些 Slot 的原理。

## NodeSelectorSlot

`NodeSelectorSlot` 是用来构造调用链的，具体的是将资源的调用路径，封装成一个一个的节点，再组成一个树状的结构来形成一个完整的调用链，`NodeSelectorSlot`是所有 Slot 中最关键也是最复杂的一个 Slot，这里涉及到以下几个核心的概念：

-   Resource

资源是 Sentinel 的关键概念。它可以是 Java 应用程序中的任何内容，例如，由应用程序提供的服务，或由应用程序调用的其它服务，甚至可以是一段代码。

只要通过 Sentinel API 定义的代码，就是资源，能够被 Sentinel 保护起来。大部分情况下，可以使用方法签名，URL，甚至服务名称作为资源名来标示资源。

简单来说，资源就是 Sentinel 用来保护系统的一个媒介。源码中用来包装资源的类是：`com.alibaba.csp.sentinel.slotchain.ResourceWrapper`，他有两个子类：`StringResourceWrapper` 和 `MethodResourceWrapper`，通过名字就知道可以将一段字符串或一个方法包装为一个资源。

打个比方，我有一个服务 A，请求非常多，经常会被陡增的流量冲垮，为了防止这种情况，简单的做法，我们可以定义一个 Sentinel 的资源，通过该资源来对请求进行调整，使得允许通过的请求不会把服务 A 搞崩溃。

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuLzMzZTY5MjUzMTgzNGFmYjFkYWNmMTRmYTYyNTAyNzFmLmpwZw?x-oss-process=image/format,png)

每个资源的状态也是不同的，这取决于资源后端的服务，有的资源可能比较稳定，有的资源可能不太稳定。那么在整个调用链中，Sentinel 需要对不稳定资源进行控制。当调用链路中某个资源出现不稳定，例如，表现为 timeout，或者异常比例升高的时候，则对这个资源的调用进行限制，并让请求快速失败，避免影响到其它的资源，最终导致雪崩的后果。

-   Context

上下文是一个用来保存调用链当前状态的元数据的类，每次进入一个资源时，就会创建一个上下文。**相同的资源名可能会创建多个上下文。** 一个 Context 中包含了三个核心的对象：

1）当前调用链的根节点：EntranceNode

2）当前的入口：Entry

3）当前入口所关联的节点：Node

上下文中只会保存一个当前正在处理的入口 Entry，另外还会保存调用链的根节点。**需要注意的是，每次进入一个新的资源时，都会创建一个新的上下文。** 

-   Entry

每次调用 `SphU#entry()` 都会生成一个 Entry 入口，该入口中会保存了以下数据：入口的创建时间，当前入口所关联的节点，当前入口所关联的调用源对应的节点。Entry 是一个抽象类，他只有一个实现类，在 CtSph 中的一个静态类：CtEntry

-   Node

节点是用来保存某个资源的各种实时统计信息的，他是一个接口，通过访问节点，就可以获取到对应资源的实时状态，以此为依据进行限流和降级操作。

可能看到这里，大家还是比较懵，这么多类到底有什么用，接下来就让我们更进一步，挖掘一下这些类的作用，在这之前，我先给大家展示一下他们之间的关系，如下图所示：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuL2ZlNTVhOTFmYTI1NzNhZmU0ZDRiOTM2ZTRiMmQyYjEyLmpwZw?x-oss-process=image/format,png)

这里把几种 Node 的作用先大概介绍下：

| 节点            | 作用                                                                                                  |
| ------------- | --------------------------------------------------------------------------------------------------- |
| StatisticNode | 执行具体的资源统计操作                                                                                         |
| DefaultNode   | 该节点持有指定上下文中指定资源的统计信息，当在同一个上下文中多次调用 entry 方法时，该节点可能下会创建有一系列的子节点。 另外每个 DefaultNode 中会关联一个 ClusterNode |
| ClusterNode   | 该节点中保存了资源的总体的运行时统计信息，包括 rt，线程数，qps 等等，相同的资源会全局共享同一个 ClusterNode，不管他属于哪个上下文                          |
| EntranceNode  | 该节点表示一棵调用链树的入口节点，通过他可以获取调用链树中所有的子节点                                                                 |

## 调用链树

当在一个上下文中多次调用了 SphU#entry() 方法时，就会创建一棵调用链树。具体的代码在 entry 方法中创建 CtEntry 对象时：

```null
CtEntry(ResourceWrapper resourceWrapper, ProcessorSlot<Object> chain, Context context) {
    super(resourceWrapper);
    this.chain = chain;
    this.context = context;
    // 获取「上下文」中上一次的入口
    parent = context.getCurEntry();
    if (parent != null) {
        // 然后将当前入口设置为上一次入口的子节点
        ((CtEntry)parent).child = this;
    }
    // 设置「上下文」的当前入口为该类本身
    context.setCurEntry(this);
}
```

这里可能看代码没有那么直观，可以用一些图形来描述一下这个过程。

### 构造树干

#### 创建 context

context 的创建在上面已经分析过了，初始化的时候，context 中的 curEntry 属性是没有值的，如下图所示：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuLzllZjlmNThlNjkzMjE3ZTc1Y2JjMjEzNjM3ZTc2Y2E5LmpwZw?x-oss-process=image/format,png)

#### 创建 Entry

每创建一个新的 Entry 对象时，都会重新设置 context 的 curEntry，并将 context 原来的 curEntry 设置为该新 Entry 对象的父节点，如下图所示：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuL2UyMTMzNjYwZTc5OGVkMTU5MTY5YmFmMDM4N2I4NzA1LmpwZw?x-oss-process=image/format,png)

#### 退出 Entry

某个 Entry 退出时，将会重新设置 context 的 curEntry，当该 Entry 是最顶层的一个入口时，将会把 ThreadLocal 中保存的 context 也清除掉，如下图所示：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuLzlkYzQzNjY1MjliMTIwN2Y2NjA4OGExMDE2NGU0Y2ZlLmpwZw?x-oss-process=image/format,png)

### 构造叶子节点

上面的过程是构造了一棵调用链的树，但是这棵树只有树干，没有叶子，那叶子节点是在什么时候创建的呢？DefaultNode 就是叶子节点，在叶子节点中保存着目标资源在当前状态下的统计信息。通过分析，我们知道了叶子节点是在 NodeSelectorSlot 的 entry 方法中创建的。具体的代码如下：

```null
@Override
public void entry(Context context, ResourceWrapper resourceWrapper, Object obj, int count, Object... args) throws Throwable {
    // 根据「上下文」的名称获取DefaultNode
    // 多线程环境下，每个线程都会创建一个context，
    // 只要资源名相同，则context的名称也相同，那么获取到的节点就相同
    DefaultNode node = map.get(context.getName());
    if (node == null) {
        synchronized (this) {
            node = map.get(context.getName());
            if (node == null) {
                // 如果当前「上下文」中没有该节点，则创建一个DefaultNode节点
                node = Env.nodeBuilder.buildTreeNode(resourceWrapper, null);
                // 省略部分代码
            }
            // 将当前node作为「上下文」的最后一个节点的子节点添加进去
            // 如果context的curEntry.parent.curNode为null，则添加到entranceNode中去
            // 否则添加到context的curEntry.parent.curNode中去
            ((DefaultNode)context.getLastNode()).addChild(node);
        }
    }
    // 将该节点设置为「上下文」中的当前节点
    // 实际是将当前节点赋值给context中curEntry的curNode
    // 在Context的getLastNode中会用到在此处设置的curNode
    context.setCurNode(node);
    fireEntry(context, resourceWrapper, node, count, args);
}
```

上面的代码可以分解成下面这些步骤：  
1）获取当前上下文对应的 DefaultNode，如果没有的话会为当前的调用新生成一个 DefaultNode 节点，它的作用是对资源进行各种统计度量以便进行流控；  
2）将新创建的 DefaultNode 节点，添加到 context 中，作为「entranceNode」或者「curEntry.parent.curNode」的子节点；  
3）将 DefaultNode 节点，添加到 context 中，作为「curEntry」的 curNode。

上面的第 2 步，不是每次都会执行。我们先看第 3 步，把当前 DefaultNode 设置为 context 的 curNode，实际上是把当前节点赋值给 context 中 curEntry 的 curNode，用图形表示就是这样：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuL2ZmYjgwODNlYWY1MDZlOTQ0ZGYxNjI3ZTdjZjg5NTEzLmpwZw?x-oss-process=image/format,png)

多次创建不同的 Entry，并且执行 NodeSelectorSlot 的 entry 方法后，就会变成这样一棵调用链树：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuL2Y1YmM5YjRmNTgzZDYzZGM0ZGU1ZThiYWU0MWU4YzIwLmpwZw?x-oss-process=image/format,png)

**PS：这里图中的 node0，node1，node2 可能是相同的 node，因为在同一个 context 中从 map 中获取的 node 是同一个，这里只是为了表述的更清楚所以用了不同的节点名。** 

#### 保存子节点

上面已经分析了叶子节点的构造过程，叶子节点是保存在各个 Entry 的 curNode 属性中的。

我们知道 context 中只保存了入口节点和当前 Entry，那子节点是什么时候保存的呢，其实子节点就是上面代码中的第 2 步中保存的。

下面我们来分析上面的第 2 步的情况：

第一次调用 NodeSelectorSlot 的 entry 方法时，map 中肯定是没有 DefaultNode 的，那就会进入第 2 步中，创建一个 node，创建完成后会把该节点加入到 context 的 lastNode 的子节点中去。我们先看一下 context 的 getLastNode 方法：

```null
public Node getLastNode() {
    // 如果curEntry不存在时，返回entranceNode
    // 否则返回curEntry的lastNode，
    // 需要注意的是curEntry的lastNode是获取的parent的curNode，
    // 如果每次进入的资源不同，就会每次都创建一个CtEntry，则parent为null，
    // 所以curEntry.getLastNode()也为null
    if (curEntry != null && curEntry.getLastNode() != null) {
        return curEntry.getLastNode();
    } else {
        return entranceNode;
    }
}
```

代码中我们可以知道，lastNode 的值可能是 context 中的 entranceNode 也可能是 curEntry.parent.curNode，但是他们都是「DefaultNode」类型的节点，DefaultNode 的所有子节点是保存在一个 HashSet 中的。

第一次调用 getLastNode 方法时，context 中 curEntry 是 null，因为 curEntry 是在第 3 步中才赋值的。所以，lastNode 最初的值就是 context 的 entranceNode。那么将 node 添加到 entranceNode 的子节点中去之后就变成了下面这样：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuLzU3NWIyN2NkODE2NzEzOGY1NmFlODI4MDhkYWM3OTkwLmpwZw?x-oss-process=image/format,png)

紧接着再进入一次，资源名不同，会再次生成一个新的 Entry，上面的图形就变成下图这样：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuLzRjNDRmMmJkMGIyY2M1MjIzODY4Zjc1ZTI4ZDk4OTFhLmpwZw?x-oss-process=image/format,png)

此时再次调用 context 的 getLastNode 方法，因为此时 curEntry 的 parent 不再是 null 了，所以获取到的 lastNode 是 curEntry.parent.curNode，在上图中可以很方便的看出，这个节点就是**node0**。那么把当前节点 node1 添加到 lastNode 的子节点中去，上面的图形就变成下图这样：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuLzE1YzExYzM0NjhhMjExYjYwODNlYTRlNmI5MjJjMDQ1LmpwZw?x-oss-process=image/format,png)

然后将当前 node 设置给 context 的 curNode，上面的图形就变成下图这样：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuL2YwYTMxNWM5MmRlZDhlNzMxYTI0NTZiZjg1MTc0MjIzLmpwZw?x-oss-process=image/format,png)

假如再创建一个 Entry，然后再进入一次不同的资源名，上面的图就变成下面这样：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3N0YXRpYy5pb2NvZGVyLmNuL2U0NjE4YmJiNzgwMzc3NGM4ZDFmODYyMjkwOGVmNmQ1LmpwZw?x-oss-process=image/format,png)

至此 NodeSelectorSlot 的基本功能已经大致分析清楚了。

**PS：以上的分析是基于每次执行 SphU.entry(name) 时，资源名都是不一样的前提下。如果资源名都一样的话，那么生成的 node 都相同，则只会再第一次把 node 加入到 entranceNode 的子节点中去，其他的时候，只会创建一个新的 Entry，然后替换 context 中的 curEntry 的值。** 

## ClusterBuilderSlot

NodeSelectorSlot 的 entry 方法执行完之后，会调用 fireEntry 方法，此时会触发 ClusterBuilderSlot 的 entry 方法。

ClusterBuilderSlot 的 entry 方法比较简单，具体代码如下：

```null
@Override
public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count, Object... args) throws Throwable {
    if (clusterNode == null) {
        synchronized (lock) {
            if (clusterNode == null) {
                // Create the cluster node.
                clusterNode = Env.nodeBuilder.buildClusterNode();
                // 将clusterNode保存到全局的map中去
                HashMap<ResourceWrapper, ClusterNode> newMap = new HashMap<ResourceWrapper, ClusterNode>(16);
                newMap.putAll(clusterNodeMap);
                newMap.put(node.getId(), clusterNode);
 
                clusterNodeMap = newMap;
            }
        }
    }
    // 将clusterNode塞到DefaultNode中去
    node.setClusterNode(clusterNode);
 
    // 省略部分代码
 
    fireEntry(context, resourceWrapper, node, count, args);
}
```

NodeSelectorSlot 的职责比较简单，主要做了两件事：

一、为每个资源创建一个 clusterNode，然后把 clusterNode 塞到 DefaultNode 中去

二、将 clusterNode 保持到全局的 map 中去，用资源作为 map 的 key

**PS：一个资源只有一个 ClusterNode，但是可以有多个 DefaultNode**

## StatistcSlot

StatisticSlot 负责来统计资源的实时状态，具体的代码如下：

```null
@Override
public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count, Object... args) throws Throwable {
    try {
        // 触发下一个Slot的entry方法
        fireEntry(context, resourceWrapper, node, count, args);
        // 如果能通过SlotChain中后面的Slot的entry方法，说明没有被限流或降级
        // 统计信息
        node.increaseThreadNum();
        node.addPassRequest();
        // 省略部分代码
    } catch (BlockException e) {
        context.getCurEntry().setError(e);
        // Add block count.
        node.increaseBlockedQps();
        // 省略部分代码
        throw e;
    } catch (Throwable e) {
        context.getCurEntry().setError(e);
        // Should not happen
        node.increaseExceptionQps();
        // 省略部分代码
        throw e;
    }
}

@Override
public void exit(Context context, ResourceWrapper resourceWrapper, int count, Object... args) {
    DefaultNode node = (DefaultNode)context.getCurNode();
    if (context.getCurEntry().getError() == null) {
        long rt = TimeUtil.currentTimeMillis() - context.getCurEntry().getCreateTime();
        if (rt > Constants.TIME_DROP_VALVE) {
            rt = Constants.TIME_DROP_VALVE;
        }
        node.rt(rt);
        // 省略部分代码
        node.decreaseThreadNum();
        // 省略部分代码
    } 
    fireExit(context, resourceWrapper, count);
}

```

代码分成了两部分，第一部分是 entry 方法，该方法首先会触发后续 slot 的 entry 方法，即 SystemSlot、FlowSlot、DegradeSlot 等的规则，如果规则不通过，就会抛出 BlockException，则会在 node 中统计被 block 的数量。反之会在 node 中统计通过的请求数和线程数等信息。第二部分是在 exit 方法中，当退出该 Entry 入口时，会统计 rt 的时间，并减少线程数。

这些统计的实时数据会被后续的校验规则所使用，具体的统计方式是通过 `滑动窗口` 来实现的。

## SystemSlot

SystemSlot 就是根据总的请求统计信息，来做流控，主要是防止系统被搞垮，具体的代码如下：

```null
@Override
public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count,
                  boolean prioritized, Object... args) throws Throwable {
    SystemRuleManager.checkSystem(resourceWrapper);
    fireEntry(context, resourceWrapper, node, count, prioritized, args);
}

```

```null
public static void checkSystem(ResourceWrapper resourceWrapper) throws BlockException {
    if (resourceWrapper == null) {
        return;
    }
    // Ensure the checking switch is on.
    if (!checkSystemStatus.get()) {
        return;
    }

    // for inbound traffic only
    if (resourceWrapper.getEntryType() != EntryType.IN) {
        return;
    }

    // total qps
    double currentQps = Constants.ENTRY_NODE == null ? 0.0 : Constants.ENTRY_NODE.successQps();
    if (currentQps > qps) {
        throw new SystemBlockException(resourceWrapper.getName(), "qps");
    }

    // total thread
    int currentThread = Constants.ENTRY_NODE == null ? 0 : Constants.ENTRY_NODE.curThreadNum();
    if (currentThread > maxThread) {
        throw new SystemBlockException(resourceWrapper.getName(), "thread");
    }

    double rt = Constants.ENTRY_NODE == null ? 0 : Constants.ENTRY_NODE.avgRt();
    if (rt > maxRt) {
        throw new SystemBlockException(resourceWrapper.getName(), "rt");
    }

    // BBR算法
    // load. BBR algorithm.
    if (highestSystemLoadIsSet && getCurrentSystemAvgLoad() > highestSystemLoad) {
        if (!checkBbr(currentThread)) {
            throw new SystemBlockException(resourceWrapper.getName(), "load");
        }
    }

    // cpu usage
    if (highestCpuUsageIsSet && getCurrentCpuUsage() > highestCpuUsage) {
        throw new SystemBlockException(resourceWrapper.getName(), "cpu");
    }
}

```

其中的 Constants.ENTRY_NODE 是一个全局的 ClusterNode，该节点的值是在 StatisticsSlot 中进行统计的。

当前的统计值和系统配置的进行比较，各个维度超过范围抛 BlockException

## AuthoritySlot

AuthoritySlot 做的事也比较简单，主要是根据黑白名单进行过滤，只要有一条规则校验不通过，就抛出异常。

```null
@Override
public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count, boolean prioritized, Object... args)
    throws Throwable {
    checkBlackWhiteAuthority(resourceWrapper, context);
    fireEntry(context, resourceWrapper, node, count, prioritized, args);
}

void checkBlackWhiteAuthority(ResourceWrapper resource, Context context) throws AuthorityException {
    // 通过监听来的规则集
    Map<String, Set<AuthorityRule>> authorityRules = AuthorityRuleManager.getAuthorityRules();

    if (authorityRules == null) {
        return;
    }


    // 根据资源名称获取相应的规则
    Set<AuthorityRule> rules = authorityRules.get(resource.getName());
    if (rules == null) {
        return;
    }

    for (AuthorityRule rule : rules) {
        // 黑名单白名单验证
        // 只要有一条规则校验不通过，就抛出AuthorityException
        if (!AuthorityRuleChecker.passCheck(rule, context)) {
            throw new AuthorityException(context.getOrigin(), rule);
        }
    }
}

```

## FlowSlot

FlowSlot 主要是根据前面统计好的信息，与设置的限流规则进行匹配校验，如果规则校验不通过则进行限流，具体的代码如下：

```null
@Override
public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count,
                  boolean prioritized, Object... args) throws Throwable {
    checkFlow(resourceWrapper, context, node, count, prioritized);

    fireEntry(context, resourceWrapper, node, count, prioritized, args);
}

void checkFlow(ResourceWrapper resource, Context context, DefaultNode node, int count, boolean prioritized)
    throws BlockException {
    checker.checkFlow(ruleProvider, resource, context, node, count, prioritized);
}

```

## DegradeSlot

DegradeSlot 主要是根据前面统计好的信息，与设置的降级规则进行匹配校验，如果规则校验不通过则进行降级，具体的代码如下：

```null
@Override
public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count,
                  boolean prioritized, Object... args) throws Throwable {
    performChecking(context, resourceWrapper);

    fireEntry(context, resourceWrapper, node, count, prioritized, args);
}

void performChecking(Context context, ResourceWrapper r) throws BlockException {
    List<CircuitBreaker> circuitBreakers = 		
        DegradeRuleManager.getCircuitBreakers(r.getName());
    
    if (circuitBreakers == null || circuitBreakers.isEmpty()) {
        return;
    }
    for (CircuitBreaker cb : circuitBreakers) {
        if (!cb.tryPass(context)) {
            throw new DegradeException(cb.getRule().getLimitApp(), cb.getRule());
        }
    }
}

```

## DefaultProcessorSlotChain

Chain 是链条的意思，从 build 的方法可看出，ProcessorSlotChain 是一个链表，里面添加了很多个 Slot。都是 ProcessorSlot 的子类。具体的实现需要到 DefaultProcessorSlotChain 中去看。

```null
public class DefaultProcessorSlotChain extends ProcessorSlotChain {

    AbstractLinkedProcessorSlot<?> first = new AbstractLinkedProcessorSlot<Object>() {

        @Override
        public void entry(Context context, ResourceWrapper resourceWrapper, Object t, int count, boolean prioritized, Object... args)
            throws Throwable {
            super.fireEntry(context, resourceWrapper, t, count, prioritized, args);
        }

        @Override
        public void exit(Context context, ResourceWrapper resourceWrapper, int count, Object... args) {
            super.fireExit(context, resourceWrapper, count, args);
        }

    };
    AbstractLinkedProcessorSlot<?> end = first;

    @Override
    public void addFirst(AbstractLinkedProcessorSlot<?> protocolProcessor) {
        protocolProcessor.setNext(first.getNext());
        first.setNext(protocolProcessor);
        if (end == first) {
            end = protocolProcessor;
        }
    }

    @Override
    public void addLast(AbstractLinkedProcessorSlot<?> protocolProcessor) {
        end.setNext(protocolProcessor);
        end = protocolProcessor;
    }

    /**
     * Same as {@link #addLast(AbstractLinkedProcessorSlot)}.
     *
     * @param next processor to be added.
     */
    @Override
    public void setNext(AbstractLinkedProcessorSlot<?> next) {
        addLast(next);
    }

    @Override
    public AbstractLinkedProcessorSlot<?> getNext() {
        return first.getNext();
    }

    @Override
    public void entry(Context context, ResourceWrapper resourceWrapper, Object t, int count, boolean prioritized, Object... args)
        throws Throwable {
        first.transformEntry(context, resourceWrapper, t, count, prioritized, args);
    }

    @Override
    public void exit(Context context, ResourceWrapper resourceWrapper, int count, Object... args) {
        first.exit(context, resourceWrapper, count, args);
    }

}

```

DefaultProcessorSlotChain 中有两个 AbstractLinkedProcessorSlot 类型的变量：first 和 end，这就是链表的头结点和尾节点。

创建 DefaultProcessorSlotChain 对象时，首先创建了首节点，然后把首节点赋值给了尾节点，可以用下图表示：  
![](https://img-blog.csdnimg.cn/20200610170547287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h4eWFzY3g=,size_16,color_FFFFFF,t_70)

将第一个节点添加到链表中后，整个链表的结构变成了如下图这样：

![](https://img-blog.csdnimg.cn/20200525165953933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h4eWFzY3g=,size_16,color_FFFFFF,t_70)

将所有的节点都加入到链表中后，整个链表的结构变成了如下图所示：

![](https://img-blog.csdnimg.cn/20200525170026637.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h4eWFzY3g=,size_16,color_FFFFFF,t_70)

这样就将所有的 Slot 对象添加到了链表中去了，每一个 Slot 都是继承自 AbstractLinkedProcessorSlot。而 AbstractLinkedProcessorSlot 是一种责任链的设计，每个对象中都有一个 next 属性，指向的是另一个 AbstractLinkedProcessorSlot 对象。其实责任链模式在很多框架中都有，比如 Netty 中是通过 pipeline 来实现的。

知道了 SlotChain 是如何创建的了，那接下来就要看下是如何执行 Slot 的 entry 方法的了

从这里可以看到，从 fireEntry 方法中就开始传递执行 entry 了，这里会执行当前节点的下一个节点 transformEntry 方法，上面已经分析过了，transformEntry 方法会触发当前节点的 entry，也就是说 fireEntry 方法实际是触发了下一个节点的 entry 方法。  
![](https://img-blog.csdnimg.cn/20200525170301722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h4eWFzY3g=,size_16,color_FFFFFF,t_70)

从最初的调用 Chain 的 entry() 方法，转变成了调用 SlotChain 中 Slot 的 entry() 方法。从 `@SpiOrder(-10000)` 知道，SlotChain 中的第一个 Slot 节点是 NodeSelectorSlot。

## slot 总结

sentinel 的限流降级等功能，主要是通过一个 SlotChain 实现的。在链式插槽中，有 7 个核心的 Slot，这些 Slot 各司其职，可以分为以下几种类型：

一、进行资源调用路径构造的 NodeSelectorSlot 和 ClusterBuilderSlot

二、进行资源的实时状态统计的 StatisticsSlot

三、进行系统保护，限流，降级等规则校验的 SystemSlot、AuthoritySlot、FlowSlot、DegradeSlot

后面几个 Slot 依赖于前面几个 Slot 统计的结果。至此，每种 Slot 的功能已经基本分析清楚了。

# 9 .sentinel 滑动窗口实现原理

## 基本原理

滑动窗口可以先拆为滑动跟窗口两个词，先介绍下`窗口`，你可以这么理解，一段是时间就是窗口，比如说我们可以把这个 1s 认为是 1 个`窗口`  
这个样子我们就能将 1 分钟就可以划分成 60 个`窗口`了，这个没毛病吧。如下图我们就分成了 60 个`窗口`（这个多了我们就画 5 个表示一下）  
![](https://img-blog.csdnimg.cn/20201227233202648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_1,color_FFFFFF,t_70)

比如现在处于第 1 秒上，那 1s 那个窗口就是`当前窗口`，就如下图中红框表示。  
![](https://img-blog.csdnimg.cn/20201227233403528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_16,color_FFFFFF,t_70)

好了，窗口就介绍完了，现在在来看下`滑动`，滑动很简单，比如说现在时间由第 1 秒变成了第 2 秒，就是从当前这个窗口 ----> 下一个窗口就可以了，这个时候下一个窗口就变成了当前窗口，之前那个当前窗口就变成了上一个窗口，这个过程其实就是滑动。  
![](https://img-blog.csdnimg.cn/20201227233659906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_16,color_FFFFFF,t_70)

好了，介绍完了滑动窗口，我们再来介绍下这个 sentinel 的滑动窗口的实现原理。  
其实你要是理解了上面这个滑动窗口的意思，sentinel 实现原理就简单了。  
先是介绍下窗口中里面都存储些啥。也就是上面这个小框框都有啥。

1.  它得有个开始时间吧，不然你怎么知道这个窗口是什么时候开始的
2.  还得有个窗口的长度吧，不然你咋知道窗口啥时候结束，通过这个开始时间 + 窗口长度 = 窗口结束时间，就比如说上面的 1s，间隔 1s
3.  最后就是要在这个窗口里面统计的东西，你总不能白搞些窗口，搞些滑动吧。所以这里就存储了一堆要统计的指标（qps，rt 等等）

说完了这一个小窗口里面的东西，就得来说说是怎么划分这个小窗口，怎么管理这些小窗口的了，也就是我们的视野得往上提高一下了，不能总聚在这个小窗口上。

1.  要知道有多少个小窗口，在 sentinel 中也就是 sampleCount，比如说我们有 60 个窗口。
2.  还有就是 intervalInMs，这个 intervalInMs 是用来计算这个窗口长度的，intervalInMs / 窗口数量 = 窗口长度。也就是我给你 1 分钟，你给我分成 60 个窗口，这个时候窗口长度就是 1s 了，那如果我给你 1s，你给我分 2 个窗口，这个时候窗口长度就是 500 毫秒了，这个 1 分钟，就是 intervalInMs。
3.  再就是存储这个窗口的容器（这里是数组），毕竟那么多窗口，还得提供计算当前时间窗口的方法等等

最后我们就来看看这个当前时间窗口是怎么计算的。  
咱们就拿 60 个窗口，这个 60 个窗口放在数组中，窗口长度是 1s 来计算，看看当前时间戳的一个时间窗口是是在数组中哪个位置。

比如说当前时间戳是 1609085401454 ms，算出秒 = 1609085401454 /1000（窗口长度）

```null
在数组的位置 = 算出秒 %数组长度
```

我们再来计算下 某个时间戳对应窗口的起始时间，还是以 1609085401454 来计算

```null
窗口startTime = 1609085401454 - 1609085401454%1000（窗口长度）=454
```

这里 1609085401454%1000（窗口长度） 能算出来它的毫秒值，也就是 454 ， 减去这个后就变成了 1609085401000

好了，sentinel 滑动窗口原理就介绍完成了。

## 2.sentinel 使用滑动窗口都统计啥

我们来介绍下使用这个滑动窗口都来统计啥

```null
public enum MetricEvent {
    /**
     * Normal pass.
     */
    PASS,// 通过
    /**
     * Normal block.
     */
    BLOCK,// 拒绝的
    EXCEPTION,// 异常
    SUCCESS,//成功
    RT,// 耗时
    /**
     * Passed in future quota (pre-occupied, since 1.5.0).
     */
    OCCUPIED_PASS
}
1234567891011121314151617
```

这是最基本的指标，然后通过这些指标，又可以计算出来比如说最大，最小，平均等等的一些指标。

## 3. 滑动窗口源码实现

我们先来看下这个窗口里面的统计指标的实现 MetricBucket

#### 3.1 MetricBucket

这个 MetricBucket 是由 LongAdder 数组组成的，一个 LongAdder 就是一个 MetricEvent ，也就是第二小节里面的 PASS ，BLOCK 等等。  
我们稍微看下就可以了  
![](https://img-blog.csdnimg.cn/202012280030215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_16,color_FFFFFF,t_70)

可以看到它在实例化的时候创建一个 LongAdder 数据，个数就是那堆 event 的数量。这个 LongAdder 是 jdk8 里面的原子操作类，你可以把它简单认为 AtomicLong。然后下面就是一堆 get 跟 add 的方法了，这里我们就不看了。  
接下来再来看看那这个窗口的实现 WindowWrap 类

#### 3.2 WindowWrap

先来看下这个成员  
![](https://img-blog.csdnimg.cn/20201228003408745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_16,color_FFFFFF,t_70)

窗口长度，窗口 startTime ，指标统计的 都有了，下面的就没啥好看的了，我们再来看下的一个方法吧  
![](https://img-blog.csdnimg.cn/20201228003539534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_16,color_FFFFFF,t_70)

就是判断某个时间戳是不是在这个窗口中  
时间戳要大于等于窗口开始时间 && 小于这个结束时间。

接下来再来看下这个管理窗口的类 LeapArray

#### 3.3 LeapArray

![](https://img-blog.csdnimg.cn/20201228003735306.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_16,color_FFFFFF,t_70)

看下它的成员， 窗口长度， 样本数 sampleCount 也就是窗口个数， intervalInMs ，再就是窗口数组  
看看它的构造方法  
![](https://img-blog.csdnimg.cn/20201228003857976.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1YW5zaGFuZ3NoZW5naHVv,size_16,color_FFFFFF,t_70)

这个构造方法其实就是计算出来这个窗口长度，创建了窗口数组。  
再来看下一个很重要的方法 
 [https://www.cnblogs.com/crazymakercircle/p/14285001.html](https://www.cnblogs.com/crazymakercircle/p/14285001.html)
