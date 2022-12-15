# 深入浅出DevOps：SonarQube提升代码质量 - 掘金
我报名参加金石计划1期挑战——瓜分10万奖池，这是我的第2篇文章，[点击查看活动详情](https://s.juejin.cn/ds/jooSN7t "https://s.juejin.cn/ds/jooSN7t")

> 💯 作者： **俗世游子**【谢先生】。 8年开发3年架构。专注于Java、云原生、大数据等领域技术。  
> 💥 成就： 从CRUD入行，负责过亿级流量架构的设计和落地，解决了千万级数据治理问题。  
> 📖 同名社区： ​[​掘金​](https://juejin.cn/user/3359725700263694 "https://juejin.cn/user/3359725700263694")​、​[​51CTO​](https://link.juejin.cn/?target=https%3A%2F%2Fblog.51cto.com%2Fu_14948012 "https://blog.51cto.com/u_14948012")​、 ​[​gitee​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq "https://gitee.com/mr_sanq")​。  
> 📂 清单： ​[​goku-framework​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq%2Fgoku-framework "https://gitee.com/mr_sanq/goku-framework")​、​[​【更新中】享阅读II​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq%2Fenjoy-read-ii "https://gitee.com/mr_sanq/enjoy-read-ii")​

[深入浅出DevOps：DevOps核心思想](https://juejin.cn/post/7134696448616038414 "https://juejin.cn/post/7134696448616038414")

[深入浅出DevOps：版本控制Git&Gitlab](https://juejin.cn/post/7135949075387514894 "https://juejin.cn/post/7135949075387514894")

[深入浅出DevOps：持续集成工具Jenkins](https://juejin.cn/post/7137300017152262151 "https://juejin.cn/post/7137300017152262151")

[深入浅出DevOps：简易Docker入门](https://juejin.cn/post/7137666687872008205 "https://juejin.cn/post/7137666687872008205")

[深入浅出DevOps：Jenkins实战之CI](https://juejin.cn/post/7139406386911248414 "https://juejin.cn/post/7139406386911248414")

为什么要提升代码质量
----------

本人所在公司业务方面已经开发的差不多了，现在质量部门开始找事情了：开始对代码编写规范，编写质量，数据建模规范有强制要求。最近一直就在搞这个事情。

但其实我们想一想，不管是项目初期或者其他时间，我们为什么都在强调代码规范问题，这会为我们产生什么样的影响呢？

首先，在企业中开发的项目大多都是多人协作开发，每个人的技术水平和编码习惯很难保证统一，如果没有一个良好的规范和约束，那么其他人在接手项目时他们的内心一定是这样的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3244199aedd439b88448648875e7686~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

其次，提升自己的代码质量也能对自己产生比较好的效果，比如：

*   心情愉悦，不至于让自己陷入到“祖传代码”中
*   降低因为踩坑而造成的工作时长，省下来的时间摸鱼不香么

上面两点都不重要，而最重要的一点是：

*   提高自身的工程和架构能力。提高代码质量是一个持续性的工作，如何指定相应的规范、如何将重构进行延续这是一门学问。需要我们好好的探讨。

> 可以从 “工程创建”- “技术选型”- “Git规范”- “前后端协同”- “代码编写”等方面好好的对自己的项目进行检查

SonarQube
---------

单有了规范还不行，还需要有检查落地情况的工具。而SonarQube就是其中的代表产品。

### 简介

**SonarQube**是自动代码审查工具，用于检测代码中的错误、漏洞和代码异味。它可以与您现有的工作流程集成，以实现跨项目分支和拉取请求的持续代码检查。拥有如下特性：

*   代码覆盖：通过单元测试，将会显示哪行代码被选中
*   改善编码规则：通过特定的规则对整个项目中的代码情况进行检查，分析其中的错误、漏洞和代码异味并进行统计，生成响应的报告。并且根据Git提交记录分配到响应的提交者
*   对比数据：比较同一张表中的任何测量的趋势

**SonarQube**功能强大，适用于 29 种编程语言，包括Java，Scala，JavaScript等语言，而且能够集成在IDE、Jenkins、Git等服务中，方便随时查看代码质量分析报告。

### 小插曲

SonarQube提供了响应的IDEA的插件，在插件中找到**SonarLint**并进行安装

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c9fcbc0793c4c2cb2e5fe82bd9c5cb0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

当你在代码编写的过程中，SonarLint会自动对代码进行规范检查，在SonarLint的窗口中会显示出来，并且代码块会出现灰色的波浪线

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b09299632404410b6876b20942c7baa~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### SonarQue的安装

插件介绍完之后，我们就正式进入到SonarQube的安装阶段。首先先来看配置要求 ​[​最新版环境要求​](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.sonarqube.org%2F8.9%2Frequirements%2Frequirements%2F "https://docs.sonarqube.org/8.9/requirements/requirements/")​：

*   选择安装的是SonarQube比较新的版本：**8.9.3-community**
*   直接安装在物理机上的话需要 **安装JDK11的版本，** Java 11 以外的版本不受官方支持**​**
*   **​**服务器资源最少需要2G内存，磁盘的读写性能必须要好**​**

> SonarQube的检索采用的是Elasticsearch来处理，因而对服务器的要求相对比较高

*   而数据库SonarQube支持​[​Microsoft SQL Server​](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.sonarqube.org%2F8.9%2Fsetup%2Finstall-server%2F%23 "https://docs.sonarqube.org/8.9/setup/install-server/#")​​，​[​Oracle​](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.sonarqube.org%2F8.9%2Fsetup%2Finstall-server%2F%23 "https://docs.sonarqube.org/8.9/setup/install-server/#")​​，​[​PostgreSQL​](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.sonarqube.org%2F8.9%2Fsetup%2Finstall-server%2F%23 "https://docs.sonarqube.org/8.9/setup/install-server/#")​。并且在7.9版本后放弃了对MySQL的支持。

> 这里就采用PostgreSQL作为存储数据库

#### Docker安装

本人对PostgreSQL不熟，就直接使用Docker来安装了。点击​[​官网​](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.sonarqube.org%2F8.9%2Fsetup%2Finstall-server%2F%23 "https://docs.sonarqube.org/8.9/setup/install-server/#")​可以找到其他的安装方式

> 设置容器在同一个网络中，容器内部就可以直接通过容器名访问

```yaml
version: "3"
services:
    db:
        image: postgres
        container_name: db
        restart: always
        privileged: true
        ports:
            - 5432:5432
        networks:
            - sonarnet
        environment:
            POSTGRES_USER: sonar
            POSTGRES_PASSWORD: sonar
        volumes:
            - /opt/postgresql:/var/lib/postgresql
    sonarqube:
        image: sonarqube:8.9.3-community
        container_name: sonarqube
        restart: always
        privileged: true
        depends_on:
            - db
        ports:
            - "9000:9000"
        networks:
            - sonarnet
        environment:
            SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
            SONAR_JDBC_USERNAME: sonar
            SONAR_JDBC_PASSWORD: sonar
        volumes:
            - /opt/sonarqube/data:/opt/sonarqube/data
            - /opt/sonarqube/extensions:/opt/sonarqube/extensions
            - /opt/sonarqube/logs:/opt/sonarqube/logs
networks:
    sonarnet:
        driver: bridge

```

然后执行如下命令进行操作

```null
docker-compose up -d

```

镜像下载完成之后会自动启动SonarQube服务，但是首次是启动不成功的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe309ef7dcbd4a0183d4107c57fd8c13~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

由于在SonarQube中使用了Elasticsearch组件，所以在启动服务的时候需要设置文件的句柄数。相信使用过Elasticsearch的同学都了解这个情况。

那么我们就来设置一下，想要设置为永久有效，就需要对​`​/etc/​`​​`​sysctl.conf​`​文件进行编辑，根据上方提示修改指定配置

```ini
vm.max_map_count=262144

```

保存好之后执行​`​sysctl -p​`​ 进行刷新，然后重新启动就好

### 操作

接下来等待较长时间之后，我们就可以通过浏览器来访问​[​http://ip:9000​](https://link.juejin.cn/?target=http%3A%2F%2Fip%3A9000 "http://ip:9000")​进入到Web页面，其中默认账户和密码都为**admin**

第一次通过默认账户密码登录成功之后，会要求你修改密码，修改成功之后就能进入到首页

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b9f1a0ebb714bafa7ec8f62c6d270b7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 插件安装

SonarQube除了自身的功能之外，本身和Jenkins一样，也支持安装插件来扩展功能。在安装插件的过程中注意SonarQube的相关提示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/052d4d4b0f454d7d8c79a16aa2019435~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

插件统一管理在Administration>Marketplace位置下，然后根据自己的需要安装对应的插件即可

注意，插件安装完成之后需要重启服务，可以在Web上直接点击重启按钮或者通过Docker操作

##### Chinese Pack

默认SonarQube全部是英文，安装该插件之后可以转换为中文

### 创建项目

接下来我们回到项目页面，默认情况下这里什么都没有，接下来我们看看创建项目的N中方式

#### Maven提交

配合Maven是最简单的一种方式，我们只需要在本地的​`​settings.xml​`​中将SonarQube的基本信息配置上就可以

> ​[​点击​](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.sonarqube.org%2F8.9%2Fanalysis%2Fscan%2Fsonarscanner-for-maven%2F "https://docs.sonarqube.org/8.9/analysis/scan/sonarscanner-for-maven/")​查看更多相关信息

```xml
<profiles>
    <profile>
      <id>sonar</id>
      <activation>
          <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
           
          <sonar.login>admin</sonar.login>
          
          <sonar.password>admin123</sonar.password>
          
          <sonar.host.url>http://192.168.10.200:9000</sonar.host.url>
          
          <sonar.inclusions>**/*.java,**/*.xml</sonar.inclusions>
      </properties>   
    </profile>
</profiles>
<activeProfiles>
    <activeProfile>sonar</activeProfile>
</activeProfiles>

```

那么接下来在项目中执行​`​mvn sonar:sonar​`​就可以完成项目的创建了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eda116050d644c2b9e7ada5f13d5bd1c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

等待一段时间之后，我们可以通过点击提示的地址去查看相关信息，首先保证项目是正常提交成功的，我们再说其他信息

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91ba9ad0638c47b8aa8fb8b4d975ede5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/360fdb2e62164e06a8d6320ddc5a1e5f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

通过点击对应的信息我们都可以看到相关的说明，并且根据我们配置的质量规则会告诉你当前问题的原因和解决方式

#### SonarScanner

除了配合Maven之外，SonarScanner也是一种方式，SonarScanner是独立的工具，专业服务与SonarQube。

为了之后能够和Jenkins进行整合，接下来我选择在SonarQube所在的服务器上进行操作

*   **下载**

```bash
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.0.2311-linux.zip
unzip sonar-scanner-cli-4.6.0.2311-linux.zip

```

*   **配置文件配置**

通过编辑​`​$install_directory/conf/sonar-scanner.properties​`​，在此处配置有关SonarQube基础环境，例如SonarQube服务器连接详细信息

```ini

sonar.host.url=http://192.168.10.200:9000
sonar.login=admin
sonar.password=admin123

```

如果不想这样配置的话，那么在执行命令的过程中也可以通过​`​-D​`​的方式将参数配置到启动程序中，比如

*   \-Dsonar.login=xxx
*   \-Dsonar.password=xxx
*   ...

那配置完成之后，就需要配置项目信息了，这样才能一起玩起来，需要配置如下关键信息：

*   项目名称和本地source

```ini

sonar.projectName=

sonar.projectBaseDir=

sonar.sources=.

```

*   推送到SonarQube的唯一标识，也可以认定为是在SonarQube项目页的展示名

```null
sonar.projectKey=

```

*   项目编辑之后的生成目录

```null
sonar.java.binaries=

```

#### sonar-project.properties

那么，根据我们之前的DevOps\_App项目来介绍，这个配置文件应该是这样的

```ini
sonar.projectName=DevOps_App
sonar.projectKey=DevOps_App
sonar.sources=.
sonar.java.binaries=target/

```

通过执行​`​sonar-scanner​`​就可以完成推送到SonarQube的流程。

而属性 ​`​project.settings​`​可用于指定项目配置文件的路径，这样能够替代无法在项目根目录下创建 sonar-project.properties 文件的麻烦问题

```ini
sonar-scanner -Dproject.settings=xxx/sonar-project.properties

```

#### 命令行

如果不方便在项目下书写，那么还有其他的替代方案。

*   **命令行直接带参数**

通过执行​`​sonar-scanner​`​，在其后通过​`​-D​`​的方式跟上相对应的配置也可以完成推送到SonarQube的流程

```ini
./sonar-scanner -Dsonar.projectName=DevOps_App -Dsonar.projectKey=DevOps_App -Dsonar.sources=. -Dsonar.java.binaries=target/

```

两种方式都是能看到最终的结果，完美！！！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/390fc7d09fff4b30968cd7740891412b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

最后
--

好了，到这里关于SonarQube相关的内容就先介绍到这里，本节主要是一个抛砖引玉的过程，实际生产中还需要对SonarQube的规则和项目进行自定义，这需要大家在接下来好好的熟悉熟悉