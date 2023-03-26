# 初始化安装后 Nacos 动态路由配置不生效
1、每次初始化安装整套项目，包括安装 Nacos 和其他服务还有mysql,redis等其他中间件，安装后 Nacos 获取不到 nacos 路由信息（包括后续新写入动态路由配置）！只有手动重启 Nacos 服务后，才会生效，后续更新的动态路由配置也会正常；

#### **Nacos： 2.1.0**

#### **spring-boot：2.6.14**

#### **spring-cloud：2021.0.1**

#### **spring-cloud-alibaba：2021.0.1.0**

### 1、需要引入的pom依赖

 ```null
		
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>

``` 

### 2、动态刷新路由service

```null
package com.cherf.flow.gateway.nacosconfig;

import java.util.ArrayList;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.Executor;
import javax.annotation.PostConstruct;

import com.alibaba.nacos.api.PropertyKeyConst;
import com.cherf.flow.gateway.config.RoutesConfig;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.gateway.event.RefreshRoutesEvent;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.route.RouteDefinitionWriter;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.stereotype.Component;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.nacos.api.NacosFactory;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.config.listener.Listener;
import com.alibaba.nacos.api.exception.NacosException;
import reactor.core.publisher.Mono;

@Component
public class NacosDynamicRouteService implements ApplicationEventPublisherAware {

    private static final Logger log = LoggerFactory.getLogger(NacosDynamicRouteService.class);


    public static final long DEFAULT_TIMEOUT = 30000;

    @Value("${spring.cloud.nacos.discovery.server-addr}")
    private String serverAddr;

    
    @Value("${flowGateway.dataId}")
    private String dataId;

    
    @Value("${flowGateway.group}")
    private String group;

    
    @Autowired
    private RoutesConfig routesConfig;

    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;

    private ApplicationEventPublisher applicationEventPublisher;

    private static final List<String> ROUTE_LIST = new ArrayList<>();

    
    @PostConstruct
    public void init() {
        log.info("gateway route init...");
        try {
            List<RouteDefinition> definitionList = new ArrayList<>();
            definitionList.addAll(routesConfig.getRoutes());
            Properties properties = new Properties();
            
            properties.put(PropertyKeyConst.SERVER_ADDR, serverAddr);
            ConfigService configService = NacosFactory.createConfigService(properties);
            if (configService != null) {
                String configInfo = configService.getConfig(dataId, group, DEFAULT_TIMEOUT);
                if (StringUtils.isNotBlank(configInfo)) {
                    definitionList.addAll(JSON.parseArray(configInfo, RouteDefinition.class));
                }
            }
            log.info("update route : {}", definitionList);
            for (RouteDefinition definition : definitionList) {
                addRoute(definition);
            }
        } catch (Exception e) {
            log.error("初始化网关路由时发生错误", e);
        }
    }

    @PostConstruct
    public void dynamicRouteByNacosListener() {
        try {
            Properties properties = new Properties();
            
            properties.put(PropertyKeyConst.SERVER_ADDR, serverAddr);
            ConfigService configService = NacosFactory.createConfigService(properties);
            if (configService != null) {
                configService.getConfig(dataId, group, 5000);
                configService.addListener(dataId, group, new Listener() {
                    @Override
                    public void receiveConfigInfo(String configInfo) {
                        if (StringUtils.isNotBlank(configInfo)) {
                            
                            clearRoute();
                            try {
                                List<RouteDefinition> gatewayRouteDefinitions = JSONObject.parseArray(configInfo, RouteDefinition.class);
                                gatewayRouteDefinitions.addAll(routesConfig.getRoutes());
                                for (RouteDefinition routeDefinition : gatewayRouteDefinitions) {
                                    addRoute(routeDefinition);
                                }
                                publish();
                            } catch (Exception e) {
                                log.error("加载网关路由时发生错误", e);
                            }
                        }
                    }

                    @Override
                    public Executor getExecutor() {
                        return null;
                    }
                });
            }
        } catch (NacosException e) {
            log.error("加载网关路由时发生错误", e);
        }
    }


    private void clearRoute() {
        for (String id : ROUTE_LIST) {
            this.routeDefinitionWriter.delete(Mono.just(id)).subscribe();
        }
        ROUTE_LIST.clear();
    }

    private void addRoute(RouteDefinition definition) {
        try {
            routeDefinitionWriter.save(Mono.just(definition)).subscribe();
            ROUTE_LIST.add(definition.getId());
        } catch (Exception e) {
            log.error("初始化网关路由时发生错误", e);
        }
    }

    private void publish() {
        this.applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this.routeDefinitionWriter));
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}

``` 

### 3、配置文件中的路由信息配置

```null
package com.cherf.flow.gateway.config;


import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.stereotype.Component;
import java.util.List;


@ConfigurationProperties(prefix = "spring.cloud.gateway")
@Data
@Component
public class RoutesConfig {

    
    private List<RouteDefinition> routes;
}


``` 

### 4、yml配置信息

```null
spring:
  cloud:
    nacos:
      discovery:
        
        
        server-addr: NACOS_HOST
      config:
        
        server-addr: NACOS_HOST
        
        file-extension: yml
        username:
        password:
    gateway:
      discovery:
        locator:
          enabled: true 
      routes:
        - id: i-console-api 
          uri: lb://i-console-api 
          predicates:
            
            - Path= /api/**,/oapi/**
          filters: 
            - name: RequestRateLimiter
              args:
                
                redis-rate-limiter.replenishRate: 100
                
                redis-rate-limiter.burstCapacity: 100

        - id: i-system-api 
          uri: lb://i-system-api 
          predicates:
            
            - Path= /sys/**,/user/**
          filters: 
            - name: RequestRateLimiter
              args:
                
                redis-rate-limiter.replenishRate: 100
                
                redis-rate-limiter.burstCapacity: 100


flowGateway:
  dataId: gateway-flow
  group: DEFAULT_GROUP

``` 

### 5、Nacos页面

**新增配置**  
[![](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321294-583578771.png)
](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321294-583578771.png)  
**路由信息**  
[![](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321312-1280271707.png)
](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321312-1280271707.png)

### 6、启动类修改

\*\* 在启动类中需要添加注解@EeableDiscoveryClient**

```null
package com.cherf;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication(exclude = {SecurityAutoConfiguration.class})
@EnableDiscoveryClient
public class FlowGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(FlowGatewayApplication.class, args);
    }
}


``` 

PS：[参考大佬实现代码，实现代码网上很多可自行百度！](https://www.bbsmax.com/A/x9J2YNKEz6/)

配置文件不生效的把名字改为`bootstrap.yml`  
仔细看 Data Id 和 Group 配置；  
yml 中配置是否开启了；  
[![](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321290-631681777.png)
](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321290-631681777.png)  
[还有很多可能的原因可以看官方issue去解决](https://github.com/alibaba/nacos/issues/1960)  
这些都不符合我的情况！

后续发现是安装后只有第一次自动更新配置有问题，重启Nacos后消失，不是一直有问题，所以就从nacos刷新配置的代码下手，**最终定位问题出在时区上**；  
使用 `show VARIABLES like '%time_zone%';` 查看数据库时区；  
[![](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321336-903856940.png)
](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321336-903856940.png)

#### 原因

`/nacos/conf/` 目录下 `application.properties` 中数据源 url 的时区为 UTC 和数据库不一致导致  
[![](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321318-1744218241.png)
](https://img2023.cnblogs.com/blog/2888631/202303/2888631-20230301175321318-1744218241.png)

#### 解决

将 nacos 配置文件中的 `serverTimezone=UTC` 改成 `serverTimezone=Asia/Shanghai` 后解决！

解决了我的问题，但是也可能不适用于其他场景！欢迎大家讨论！

\_\_EOF\_\_

[![](https://images.cnblogs.com/cnblogs_com/blogs/754269/galleries/2166487/t_220525085400_pinkish_sunset-wallpaper-2560x1440.jpg)
](https://images.cnblogs.com/cnblogs_com/blogs/754269/galleries/2166487/t_220525085400_pinkish_sunset-wallpaper-2560x1440.jpg)

*   **本文作者：**  [cherf](https://www.cnblogs.com/cherf)
*   **本文链接：**  [https://www.cnblogs.com/cherf/p/17169197.html](https://www.cnblogs.com/cherf/p/17169197.html)
*   **关于博主：**  评论和私信会在第一时间回复。或者[直接私信](https://msg.cnblogs.com/msg/send/cherf)我。
*   **版权声明：**  本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-nd/4.0/ "BY-NC-SA") 许可协议。转载请注明出处！
*   **声援博主：**  如果您觉得文章对您有帮助，可以点击文章右下角**【推荐】** 一下。