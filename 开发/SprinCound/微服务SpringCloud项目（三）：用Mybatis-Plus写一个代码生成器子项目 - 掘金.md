# 微服务SpringCloud项目（三）：用Mybatis-Plus写一个代码生成器子项目 - 掘金
**小知识，大挑战！本文正在参与 “[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https&#x3A;//juejin.cn/post/7008476801634680869")” 创作活动。** 

## 📖前言

    心态好了，就没那么累了。心情好了，所见皆是明媚风景。
    复制代码

> **`“一时解决不了的问题，那就利用这个契机，看清自己的局限性，对自己进行一场拨乱反正。” 正如老话所说，一念放下，万般自在。如果你正被烦心事扰乱心神，不妨学会断舍离。断掉胡思乱想，社区垃圾情绪，离开负面能量。心态好了，就没那么累了。心情好了，所见皆是明媚风景。`**

**`PS：先出个 Gradle 版本的生成器，后面我会把他集成到 springcloud 项目中去，欢迎关注哦!`**

### 🚀先上一下 `git` 地址

**[传送门](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2FyongjiachenAdvance%2Fmybatis-plus-code-gen-demo.git "https&#x3A;//gitee.com/yongjiachenAdvance/mybatis-plus-code-gen-demo.git")**

### 🚓进入正题

#### 1. 创建项目我就不讲了

#### 2. 引入依赖，如下所示 -- `Maven` 版本的后面也出一个

    plugins {
        id 'java-library'
        id 'org.springframework.boot' version '2.2.1.RELEASE'
        id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    }

    group 'com.cyj.codegen'
    version '1.0.0'

    apply plugin: 'idea'
    apply plugin: 'java-library'
    apply plugin: 'io.spring.dependency-management'
    apply plugin: 'org.springframework.boot'

    // 指定JDK版本
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
    // 指定编码格式
    [compileJava,compileTestJava,javadoc]*.options*.encoding = 'UTF-8'

    // 源文件配置
    // sourceSets.main.resources.srcDirs = ["src/main/java","src/main/resources"]

    // 源文件配置
    sourceSets {
        main {
            java.srcDir('src/main/java')
            resources.srcDir('src/main/resources')
        }
        test {
            java.srcDir('src/test/java')
            resources.srcDir('src/test/resources')
        }
    }


    sourceCompatibility = 1.8
    targetCompatibility = 1.8

    repositories {
        mavenLocal()
        maven {
            url 'https://maven.aliyun.com/repository/public/'
        }
        maven {
            url 'https://oss.sonatype.org/content/repositories/snapshots/'
        }
        mavenCentral()
    }

    // 依赖的版本号进行统一控制
    ext {
        springBootVar = '2.2.1.RELEASE'
        springBootMybatis = '2.1.3'
        httpClient = '4.5.2'
        mysql = '5.1.21'
        junit = '4.12'
        fastJson = '1.2.75'
        guava = '27.0-jre'
        hutoolVersion = '5.6.6'
        druidVer = '1.1.21'
        swagger2 = '2.9.2'
        swagger2UI = '2.9.2'
        swaggerBootstrapUI = '1.9.6'
        knife4j = '2.0.8'
    }

    dependencies {
        compile('org.apache.commons:commons-lang3:3.11')
        compile('org.apache.commons:commons-collections4:4.4')
        // 引入 mybatis-plus 依赖包
        compile('com.baomidou:mybatis-plus-boot-starter:3.3.2')
        // 引入 SpringBootWeb 依赖包
        compile("org.springframework.boot:spring-boot-starter-web:${springBootVar}")
        // SpringBoot 核心依赖引入
        compile("org.springframework.boot:spring-boot-starter:${springBootVar}") {
            // 排除tomcat
            exclude module: "tomcat-embed-el"
        }
        // SpringBootMybatis 依赖引入
        compile("org.mybatis.spring.boot:mybatis-spring-boot-starter:${springBootMybatis}")
        //httpclient 相关依赖
        compile("org.apache.httpcomponents:httpclient:${httpClient}")
        // 引入数据库链接驱动 mysql-connector-java 依赖
        compile("mysql:mysql-connector-java")
        // 引入 SpringDataJpa 依赖
        compile("org.springframework.boot:spring-boot-starter-data-jpa:${springBootVar}")
        // 引入 Druid 依赖
        compile("com.alibaba:druid-spring-boot-starter:${druidVer}")
        // 引入 pool 依赖
        compile("org.apache.commons:commons-pool2")
        // 引入 swagger2 依赖
        compile("io.springfox:springfox-swagger2:${swagger2}")
        // 引入 swagger2UI 依赖
        compile("io.springfox:springfox-swagger-ui:${swagger2UI}")
        // 引入 优化的 swagger2UI 依赖 https://xiaoym.gitee.io/knife4j/documentation
        compile("com.github.xiaoymin:knife4j-spring-boot-starter:${knife4j}")
        // 引入下多数据源 dynamic
        compile('com.baomidou:dynamic-datasource-spring-boot-starter:3.3.2')
        // 引入 jasypt 依赖包
        // compile('org.jasypt:jasypt:1.9.2')
        implementation 'commons-configuration:commons-configuration:1.10'
        implementation 'org.apache.velocity:velocity:1.7'
        // json 相关依赖
        compile("com.alibaba:fastjson:${fastJson}")
        implementation "com.google.guava:guava:${guava}"
        implementation "cn.hutool:hutool-all:${hutoolVersion}"
        compileOnly 'javax.servlet:javax.servlet-api:4.0.1'
        implementation 'org.springframework.boot:spring-boot-configuration-processor'
        // 数据库密码加密
        implementation 'com.github.ulisesbocchio:jasypt-spring-boot-starter:2.1.0'
        implementation 'org.springframework.boot:spring-boot-starter-actuator'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
    }

    //修改gradle 构建编码为UTF-8
    tasks.withType(JavaCompile) {
        options.encoding = "UTF-8"
    }

    test {
        useJUnitPlatform()
    }
    复制代码

#### 3. 修改启动类 -- 这顶扫描的 mapper 存放位置

```java
package com.cyj.codegen;

import cn.hutool.core.date.DateUtil;
import com.cyj.codegen.common.Constants;
import lombok.extern.slf4j.Slf4j;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@Slf4j
@MapperScan("com.cyj.codegen.mapper")
@SpringBootApplication(scanBasePackages = {Constants.SCAN_BASE_PACKAGES})
public class CodeGenApplication {

    
    public static void main(String[] args) {
        log.info("Mybatis-plus代码生成器项目开始启动ing！======>{}", DateUtil.now());
        SpringApplication.run(CodeGenApplication.class, args);
        log.info("Mybatis-plus代码生成器项目启动成功ing.......！======>{}", DateUtil.now());
    }

}
```

#### 4. 创建数据库 -- 执行静态资源下的 `pig_codegen.sql` 文件

#### 5. 添加一个骚气的 `banner.txt` 你是一个好孩子

```java
   _  _                                                              __ _                      _             _                _  _
  | || |   ___    _  _      o O O  __ _      _ _    ___      o O O  / _` |   ___     ___    __| |     o O O | |__     ___    | || |
   _, |  / _ \  | +| |    o      / _` |    | '_|  / -_)    o       __, |  / _ \   / _ \  / _` |    o      | '_ \   / _ \    _, |
  _|__/   ___/   _,_|   TS__[O] __,_|   _|_|_   ___|   TS__[O]  |___/   ___/   ___/  __,_|   TS__[O] |_.__/   ___/   _|__/
_| """"|_|"""""|_|"""""| {======|_|"""""|_|"""""|_|"""""| {======|_|"""""|_|"""""|_|"""""|_|"""""| {======|_|"""""|_|"""""|_| """"|
"`-0-0-'"`-0-0-'"`-0-0-'./o--000'"`-0-0-'"`-0-0-'"`-0-0-'./o--000'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'./o--000'"`-0-0-'"`-0-0-'"`-0-0-'
```

#### 6. 创建数据库全局常量类 `DataSourceConstants` 和 `Constants`

> **`注意修改 DS_DRIVER 参数，区分 5.7.x 和 8.0.x 版本，修改 SCAN_BASE_PACKAGES 参数用以扫描你的包路径`**

```java
package com.cyj.codegen.common;


public interface DataSourceConstants {

    
    String DS_NAME = "name";

    
    String DS_DRIVER = "com.mysql.cj.jdbc.Driver";

    
    String DS_MASTER = "master";

    
    String DS_JDBC_URL = "url";

    
    String DS_USER_NAME = "username";

    
    String DS_USER_PWD = "password";

}
```

#### 7. 校验异常类 `CheckedException`

```java
@NoArgsConstructor
public class CheckedException extends RuntimeException {

    private static final long serialVersionUID = 1L;

    public CheckedException(String message) {
        super(message);
    }

    public CheckedException(Throwable cause) {
        super(cause);
    }

    public CheckedException(String message, Throwable cause) {
        super(message, cause);
    }

    public CheckedException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }

}
```

#### 8. 配置 `Swagger` 我就不讲了

#### 9. 在 `model` 下创建实体类

> **`列实体对象 --ColumnEntity，生成器配置实体对象 --GenConfig，多数据源 --GenDatasourceConf，生成记录表 --GenFormConf，表属性 --TableEntity`**

PS: 组合形成基础的生产器基础结构。

#### 10. `service` 层用于实现这些类的增删改查等基础操作

#### 11. 代码生成器帮助类 --`CodeGenUtils` 重点

> `getTemplates` 方法用于获取配置 -- 也就是我们写得 `.vm` 生成模板 `generatorCode` 方法用于生成代码 `renderData` 方法用于渲染数据 `columnToJava` 方法用于表名转换成 Java 类名 `getConfig` 方法用于获取我们的配置信息 `getFileName` 方法用于获取文件名

#### 12. 关于配置资源下的 `template`

> **里面的 `.vm` 文件中通过 `velocity` 来获取变量名称，更多的用法可以自行百度查询使用。** 

#### 13. 当然请注意修改 `generator.properties` 文件里的包路径和数据类型哦

#### 14. 演示下生成代码

> **因为篇幅原因，这里只演示代码生成，其他接口也都给出了可以自行测试查看效果**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58c38691433f44cc9cbe9e260e6d6e14~tplv-k3u1fbpfcp-watermark.awebp?)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45c2a7331740490aba0342490bbed413~tplv-k3u1fbpfcp-watermark.awebp?)

**最后感谢大家耐心观看完毕，原创不易，留个点赞收藏是您对我最大的鼓励！**

* * *

### 🎉总结：

-   **更多参考精彩博文请看这里：[《陈永佳的博客》](https://juejin.cn/user/862483929905639/posts "https&#x3A;//juejin.cn/user/862483929905639/posts")**
-   **希望这些代码可以帮助小伙伴们减轻一些 `crud` 负担，以及一些灵感思路。** 
-   **喜欢博主的小伙伴可以加个关注、点个赞哦，持续更新嘿嘿！** 
    [https://juejin.cn/post/7011722064168452110#heading-16](https://juejin.cn/post/7011722064168452110#heading-16)
