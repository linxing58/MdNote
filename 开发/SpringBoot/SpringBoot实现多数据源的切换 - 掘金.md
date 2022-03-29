# SpringBoot实现多数据源的切换 - 掘金
## 码农在囧途

> 每当晴空万里的时候，天空依然是湛蓝的，夜里我还是会仰望这难得的星空俯瞰诺大的城市，万家灯火中却没有一盏灯是为我亮着的。

## 前言

我们在进行软件开发的过程中，刚开始的时候因为无法估量系统后期的访问量和并发量，所以一开始会采用单体架构，后期如果网站流量变大， 并发量变大，那么就可能会将架构扩展为微服务架构，各个微服务对应一个数据库，不过这样的成本就有点大了，可能只是有些模块用的人比较多， 有些模块没什么人用，如果都进行服务拆分，其实也没那个必要，如果有些模块用的人比较多，那么我们可以采用读写分离来减轻压力，这样的话， 可以在一定程度上提升系统的用户体验，不过这只是在数据库的 I/O 上面做方案，如果系统的压力很大，那么肯定要做负载均衡，我们今天就先说 实现数据库的读写分离。我们要在代码层面实现数据库的读写分离，那么核心就是数据源的切换，本文基于 AOP 来实现数据源的切换。

## 工程结构

```yml
└─com
    └─steak
        └─transaction
            │  TransactionDemoApplication.java
            │  
            ├─datasource
            │  │  DatasourceChooser.java
            │  │  DatasourceConfiguration.java
            │  │  DatasourceContext.java
            │  │  DatasourceScope.java
            │  │  DynamicDatasourceAspect.java
            │  │  
            │  └─properties
            │          DynamicDatasourceProperties.java
            │          MainDatasourceProperties.java
            │          
            ├─execute
            │      PlaceOrderExecute.java
            │      
            ├─rest
            │      PlaceOrderApi.java
            │      
            ├─result
            │      R.java
            │      
            └─service
                    IntegralService.java
                    OrderService.java
                    StockService.java
```

在下面的实现中，我们一个有三个数据源，其中有一个是默认的，如果不指定具体的数据源，那么就使用默认的，我们是基于申明式的方式来切换 数据源，只需要在具体的接口上加上注解便能实现数据源的切换。

## 编码实现

### yml 文件

主数据源直接使用 spring 的配置，其他数据源采用自定义的方式，这里采用一个 map 结构来定义，方便进行解析，可以在 yml 文件里进行添加多个，在代码 逻辑层面不用改动。

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      username: root
      password: 123456
      url: jdbc:mysql://127.0.0.1:3306/db?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC
      driver-class-name: com.mysql.cj.jdbc.Driver

dynamic:
  datasource: {
    slave1: {
      username: 'root',
      password: '123456',
      url: ' url: jdbc:mysql://127.0.0.1:3306/db?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC',
      driver-class-name: 'com.mysql.cj.jdbc.Driver'
    },
    slave2: {
      username: 'root',
      password: '123456',
      url: ' url: jdbc:mysql://127.0.0.1:3306/db?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC',
      driver-class-name: 'com.mysql.cj.jdbc.Driver'
    }
  }
```

### 主数据源 MainDatasourceProperties

对于主数据源，我们单独拿出来放在一个类里面，不过其实也可以放到 dynamic 里面，只是需要做一定的处理，我们就简单的放几个连接属性。

```java

@Component
@ConfigurationProperties(prefix = "spring.datasource.druid")
@Data
public class MainDatasourceProperties {
    private String username;
    private String password;
    private String url;
    private String driverClassName;
}
```

### 其他数据源 DynamicDatasourceProperties

其他数据源使用一个 Map 来接受 yml 文件中的数据源配置

```java

@Component
@ConfigurationProperties(prefix = "dynamic")
@Data
public class DynamicDatasourceProperties {
    private Map<String,Map<String,String>> datasource;
}
```

### 数据源配置类 DatasourceConfiguration

配置类中主要对 DataSource 进行配置，主数据源我们按照正常的 bean 来定义连接属性，而其他数据数据源则使用反射的方式来进行连接属性的 配置，因为主数据源一般是不会变动的，但是其他数据源可能会发生变动，可能会添加，这时候如果通过硬编码取配置，那么每增加一个数据源，就需要 增加一个配置，显然不太行，所以就使用反射来进行赋值。

```java

@Configuration
@AllArgsConstructor
public class DatasourceConfiguration {

    final MainDatasourceProperties mainDatasourceProperties;
    final DynamicDatasourceProperties dynamicDatasourceProperties;

    @Bean
    public DataSource datasource(){
        Map<Object,Object> datasourceMap = new HashMap<>();
        DatasourceChooser datasourceChooser = new DatasourceChooser();
        
        DruidDataSource mainDataSource = new DruidDataSource();
        mainDataSource.setUsername(mainDatasourceProperties.getUsername());
        mainDataSource.setPassword(mainDatasourceProperties.getPassword());
        mainDataSource.setUrl(mainDatasourceProperties.getUrl());
        mainDataSource.setDriverClassName(mainDatasourceProperties.getDriverClassName());
        datasourceMap.put("main",mainDataSource);
        
        Map<String, Map<String, String>> sourceMap = dynamicDatasourceProperties.getDatasource();
        sourceMap.forEach((datasourceName,datasourceMaps) -> {
            DruidDataSource dataSource = new DruidDataSource();
            datasourceMaps.forEach((K,V) -> {
                String setField = "set" + K.substring(0, 1).toUpperCase() + K.substring(1);
                
                String[] strings = setField.split("");
                StringBuilder newStr = new StringBuilder();
                for (int i = 0; i < strings.length; i++) {
                    if (strings[i].equals("-")) strings[i + 1] = strings[i + 1].toUpperCase();
                    if (!strings[i].equals("-")) newStr.append(strings[i]);
                }
                try {
                    DruidDataSource.class.getMethod(newStr.toString(),String.class).invoke(dataSource,V);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
            datasourceMap.put(datasourceName,dataSource);
        });
        
        datasourceChooser.setTargetDataSources(datasourceMap);
        
        datasourceChooser.setDefaultTargetDataSource(mainDataSource);
        return datasourceChooser;
    }
}
```

上面使用数据源配置类中使用反射对其他数据源进行连接属性的设置，然后设置目标数据源和默认数据源，里面有一个`DatasourceChooser`。

### DatasourceChooser

`DatasourceChooser`继承自`AbstractRoutingDataSource`，`AbstractRoutingDataSource`可以实现数据源的切换，它里面的 `determineCurrentLookupKey()`方法需要我们返回一个数据源的名称，它会自动给我们匹配上数据源。

```java

public class DatasourceChooser extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return DatasourceContext.getDatasource();
    }
}
```

如下是`AbstractRoutingDataSource`的部分源码，我们可以看出数据源是一个 Map 结构，可以通过数据源名称查找到对应的数据源。

```java
package org.springframework.jdbc.datasource.lookup;

public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean {
    @Nullable
    private Map<Object, DataSource> resolvedDataSources;
    public AbstractRoutingDataSource() {
    }
    protected DataSource determineTargetDataSource() {
        Object lookupKey = this.determineCurrentLookupKey();
        DataSource dataSource = (DataSource)this.resolvedDataSources.get(lookupKey);
    }

    @Nullable
    protected abstract Object determineCurrentLookupKey();
}
```

### DatasourceContext

`DatasourceContext`内部是一个`ThreadLocal`，主要是用来存储每一个线程的数据源名称和获取数据源名称，而数据源的名称我们用过 AOP 切面 来获取。

```java

public class DatasourceContext {
    private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();
    public static void setDatasource(String key){
        threadLocal.set(key);
    }
    public static String getDatasource(){
        return threadLocal.get();
    }
}
```

### 数据源注解 DatasourceScope

`DatasourceScope`标准在方法上面，通过`scope`来指定数据源，不指定默认为主数据源`main`。

```java

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DatasourceScope {
    String scope() default "main";
}
```

### 数据源切面 DynamicDatasourceAspect

我们在访问每一个带有`DatasourceScope`注解的方法时，都会经过数据源切面`DynamicDatasourceAspect`，获取到注解上面的 `scope`的值后，通过`DatasourceContext`设置数据源名称，便可实现对数据源的切换。

```java

@Aspect
@Component
public class DynamicDatasourceAspect {

    @Pointcut("@annotation(dataSourceScope)")
    public void dynamicPointcut(DatasourceScope dataSourceScope){}

    @Around(value = "dynamicPointcut(dataSourceScope)", argNames = "joinPoint,dataSourceScope")
    public Object dynamicAround(ProceedingJoinPoint joinPoint , DatasourceScope dataSourceScope) throws Throwable {
        String scope = dataSourceScope.scope();
        DatasourceContext.setDatasource(scope);
        return joinPoint.proceed();
    }
}
```

## 使用

只需要在具体的方法上面标注数据源注解`@DatasourceScope`，并指定`scope`的值，便可实现切换，如果不指定，那么就使用主数据源。

```java

@Service
@AllArgsConstructor
public class OrderService {
    
    private JdbcTemplate jdbcTemplate;
    
    @DatasourceScope(scope = "slave1")
    public R saveOrder(Integer userId , Integer commodityId){
        String sql = "INSERT INTO `order`(user_id,commodity_id) VALUES("+userId+","+commodityId+")";
        jdbcTemplate.execute(sql);
        return R.builder().code(200).msg("save order success").build();
    }
}
```

> 关于多数据源的切换，就分享到这里，感谢你的观看，下期见 
>  [https://juejin.cn/post/7070701713145102343](https://juejin.cn/post/7070701713145102343)
