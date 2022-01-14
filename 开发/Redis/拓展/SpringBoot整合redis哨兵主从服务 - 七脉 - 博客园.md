# SpringBoot整合redis哨兵主从服务 - 七脉 - 博客园
### 前提环境：

　　主从配置　　[http://www.cnblogs.com/zwcry/p/9046207.html](http://www.cnblogs.com/zwcry/p/9046207.html)

　　哨兵配置　　[https://www.cnblogs.com/zwcry/p/9134721.html](https://www.cnblogs.com/zwcry/p/9134721.html)

### 1. 配置 pom.xml

![](https://common.cnblogs.com/images/copycode.gif)

&lt;project xmlns\\="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0)"xmlns:xsi\\="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"xsi:schemaLocation\\="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0) [http://maven.apache.org/xsd/maven-4.0.0.xsd"\\>](http://maven.apache.org/xsd/maven-4.0.0.xsd"\>)
      &lt;modelVersion>4.0.0&lt;/modelVersion>
      &lt;groupId>szw&lt;/groupId>
      &lt;artifactId>springboot_redis_sentinel&lt;/artifactId>
      &lt;version>0.0.1-SNAPSHOT&lt;/version>
      &lt;name>springboot_redis_sentinel&lt;/name>
      &lt;description>springboot 整合 redis 哨兵 &lt;/description>

      <properties\>
        <project.build.sourceEncoding\>UTF-8</project.build.sourceEncoding\>
        <project.reporting.outputEncoding\>UTF-8</project.reporting.outputEncoding\>
        <java.version\>1.8</java.version\>
        <start-class\>com.sze.redis.SzwRedisApplication</start-class\>
    </properties\>

      <parent\>
        <groupId\>org.springframework.boot</groupId\>
        <artifactId\>spring-boot-starter-parent</artifactId\>
        <version\>1.4.2.RELEASE</version\>
        <relativePath\></relativePath\>
    </parent\>

    <dependencies\>
        <!-- 使用web启动器 \-->
        <dependency\>
            <groupId\>org.springframework.boot</groupId\>
            <artifactId\>spring-boot-starter-web</artifactId\>
        </dependency\>
        <!-- 使用aop模板启动器 \-->
        <dependency\>
            <groupId\>org.springframework.boot</groupId\>
            <artifactId\>spring-boot-starter-aop</artifactId\>
        </dependency\>
        <!-- 测试 \-->
        <dependency\>
            <groupId\>org.springframework.boot</groupId\>
            <artifactId\>spring-boot-starter-test</artifactId\>
            <scope\>test</scope\>
        </dependency\>
        <dependency\>
            <groupId\>org.springframework.boot</groupId\>
            <artifactId\>spring-boot-starter-redis</artifactId\>
        </dependency\>
    </dependencies\>

    <!-- deploy \-->
    <distributionManagement\>
        <repository\>
            <id\>releases</id\>
            <name\>Releases</name\>
            <url\>http://192.168.3.71:8081/nexus/content/repositories/releases/</url\>
        </repository\>
        <snapshotRepository\>
            <id\>snapshots</id\>
            <name\>Snapshots</name\>
            <url\>http://192.168.3.71:8081/nexus/content/repositories/snapshots/</url\>
        </snapshotRepository\>
    </distributionManagement\>

    <!-- download \-->
    <repositories\>
        <repository\>
            <id\>sicdt</id\>
            <name\>Sicdt</name\>
            <url\>http://192.168.3.71:8081/nexus/content/groups/public</url\>
        </repository\>
        <repository\>
            <id\>spring-releases</id\>
            <url\>https://repo.spring.io/libs-release</url\>
        </repository\>
    </repositories\>
    <pluginRepositories\>
        <pluginRepository\>
            <id\>spring-releases</id\>
            <url\>https://repo.spring.io/libs-release</url\>
        </pluginRepository\>
    </pluginRepositories\>

    <build\>  
        <plugins\> 
            <!-- 要将源码放上去，需要加入这个插件 \-->  
            <plugin\>  
                <groupId\>org.apache.maven.plugins</groupId\>  
                <artifactId\>maven-source-plugin</artifactId\>  
                <configuration\>  
                    <attach\>true</attach\>  
                </configuration\>  
                <executions\>  
                    <execution\>  
                        <phase\>compile</phase\>  
                        <goals\>  
                            <goal\>jar</goal\>  
                        </goals\>  
                    </execution\>  
                </executions\>  
            </plugin\>
            <!-- 打包 \-->
            <plugin\>
                <groupId\>org.springframework.boot</groupId\>
                <artifactId\>spring-boot-maven-plugin</artifactId\>
                <configuration\>
                    <fork\>true</fork\>
                </configuration\>
            </plugin\>  
        </plugins\>  
    </build\>

&lt;/project>

![](https://common.cnblogs.com/images/copycode.gif)

### 2. 配置 application.properties

![](https://common.cnblogs.com/images/copycode.gif)

\## 单服务器
spring.redis.host=39.107.119.256
\## 单端口
spring.redis.port=6381
\## 连接池最大连接数（使用负值表示没有限制） 
spring.redis.pool.max-active=300
\## Redis 数据库索引 (默认为 0) 
spring.redis.database=0
\## 连接池最大阻塞等待时间（使用负值表示没有限制） 
spring.redis.pool.max-wait=-1
\## 连接池中的最大空闲连接 
spring.redis.pool.max-idle=100
\## 连接池中的最小空闲连接 
spring.redis.pool.min-idle=20
\## 连接超时时间（毫秒） 
spring.redis.timeout=60000

\#哨兵的配置列表  
spring.redis.sentinel.master=mymaster
spring.redis.sentinel.nodes=39.107.119.256:26379  
## 哨兵集群  
#spring.redis.sentinel.nodes=39.107.119.254:26379,39.107.119.254:26380  

![](https://common.cnblogs.com/images/copycode.gif)

### 3. 编写启动类

![](https://common.cnblogs.com/images/copycode.gif)

package com.szw.redis; import org.springframework.boot.SpringApplication; import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication public class SzwRedisApplication { public static void main(String\[] args) {System.setProperty("spring.devtools.restart.enabled", "false");
        SpringApplication.run(SzwRedisApplication.class, args);
    }
}

![](https://common.cnblogs.com/images/copycode.gif)

### 4. 单元测试

![](https://common.cnblogs.com/images/copycode.gif)

package com.sze.redis; import javax.annotation.PostConstruct; import org.junit.Test; import org.junit.runner.RunWith; import org.springframework.beans.factory.annotation.Autowired; import org.springframework.boot.test.context.SpringBootTest; import org.springframework.data.redis.core.StringRedisTemplate; import org.springframework.data.redis.core.ValueOperations; import org.springframework.test.annotation.DirtiesContext; import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest public class SentinelTest {

    @Autowired
    StringRedisTemplate redisTemplate;

    ValueOperations<String, String> stringRedis;

    @PostConstruct public void init(){
        stringRedis\=redisTemplate.opsForValue();
    }


    @Test public void testString (){
        stringRedis.set("name", "丁洁");
        System.out.println(stringRedis.get("name"));
    }

}

![](https://common.cnblogs.com/images/copycode.gif)

![](https://images2018.cnblogs.com/blog/1253415/201806/1253415-20180608161525446-952089512.png)

### 5. 多个哨兵配置

![](https://common.cnblogs.com/images/copycode.gif)

\## 单服务器
spring.redis.host=39.107.119.256
\## 单端口
spring.redis.port=6381
\## 连接池最大连接数（使用负值表示没有限制） 
spring.redis.pool.max-active=300
\## Redis 数据库索引 (默认为 0) 
spring.redis.database=0
\## 连接池最大阻塞等待时间（使用负值表示没有限制） 
spring.redis.pool.max-wait=-1
\## 连接池中的最大空闲连接 
spring.redis.pool.max-idle=100
\## 连接池中的最小空闲连接 
spring.redis.pool.min-idle=20
\## 连接超时时间（毫秒） 
spring.redis.timeout=60000

\#哨兵的配置列表  
spring.redis.sentinel.master=mymaster
spring.redis.sentinel.nodes=39.107.119.256:26379,39.107.119.256:26380

![](https://common.cnblogs.com/images/copycode.gif) 
 [https://www.cnblogs.com/zwcry/p/9156243.html](https://www.cnblogs.com/zwcry/p/9156243.html)
