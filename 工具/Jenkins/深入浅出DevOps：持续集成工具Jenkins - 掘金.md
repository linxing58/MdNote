# 深入浅出DevOps：持续集成工具Jenkins - 掘金
携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第6天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

> 💯 作者： **俗世游子**【谢先生】。 8年开发3年架构。专注于Java、云原生、大数据等领域技术。  
> 💥 成就： 从CRUD入行，负责过亿级流量架构的设计和落地，解决了千万级数据治理问题。  
> 📖 同名社区： ​[​掘金​](https://juejin.cn/user/3359725700263694 "https://juejin.cn/user/3359725700263694")​、​[​51CTO​](https://link.juejin.cn/?target=https%3A%2F%2Fblog.51cto.com%2Fu_14948012 "https://blog.51cto.com/u_14948012")​、 ​[​gitee​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq "https://gitee.com/mr_sanq")​。  
> 📂 清单： ​[​goku-framework​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq%2Fgoku-framework "https://gitee.com/mr_sanq/goku-framework")​、​[​【更新中】享阅读II​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq%2Fenjoy-read-ii "https://gitee.com/mr_sanq/enjoy-read-ii")​

前言
--

本章上重头戏，介绍关于Jenkins的相关内容。

先来简单看一看Jenkins是干什么的吧！

Jenkins
-------

### 简介

Jenkins 是一个独立的开源自动化服务器，可用于自动化与构建、测试、交付或部署软件相关的各种任务，占据市场绝大份额，应用广泛。且Jenkins是基于Java开发的一种持续集成工具。

Jenkins最主要的工作就是将GitLab上可以构建的工程代码拉取并且进行构建，再根据流程可以选择发布到测试环境或是生产环境。并且Jenkins官方提供了大量的插件库，最大程度的简化了来自CI/CD过程中的各种琐碎功能。

> 表面上是小老头，实际上是变形金刚。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1a44c9829c14c4e83ecfc5badf42bfb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 什么是CI/CD/CM

那说到这里就不得不提一下CI/CD的概念了，毕竟这个东西贯穿了整个DevOps的生命周期。

#### 持续集成：CI

现代应用开发的目标是让多位开发人员同时处理同一应用的不同功能。但是，如果安排在一天内将所有分支源代码合并在一起，最终可能造成工作繁琐、耗时，且由于每一位开发人员对应用进行更改时，有可能会与其他开发人员同时进行的更改发生冲突，还需要对冲突文件进行更改，此时的工作会变得更加雪上加霜。

**持续集成【CI】**  可以帮助开发人员更加频繁地（有时甚至每天）将代码更改合并到共享分支或"主干"中。一旦开发人员对应用所做的更改被合并，系统就会通过自动构建应用并运行不同级别的自动化测试（单元测试和集成测试）来验证这些更改，确保这些更改没有对应用造成破坏。如果自动化测试发现新代码和现有代码之间存在冲突，**CI**可以更加轻松地快速修复这些错误。

#### 持续交付、部署：CD

这里分为交付和部署两个部分：

*   持续交付

完成 CI 中构建及单元测试和集成测试的自动化流程后，持续交付可自动将已验证的代码发布到存储库。而持续交付的目标是为了能够随时拥有一个可随时部署到生产环境的代码库。

在持续交付中，每个阶段（从代码更改的合并，到生产就绪型构建版本的交付）都涉及测试自动化和代码发布自动化。在流程结束时，运维团队可以快速、轻松地将应用部署到生产环境中。

*   持续部署

持续部署是成熟的 CI/CD的最后一环，且属于持续交付的延伸。持续部署可以自动将应用发布到生产环境，生产环境不是随随便便就可以部署的，所以持续部署在很大程度上都得依赖精心设计的测试自动化。

> 而编写自动化测试以适应 CI/CD 管道中的各种测试和发布阶段，是整个DevOps过程中最大的投资
> 
> 但是过了初始阶段，是非常能够享受DevOps的便利的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/396fedd22cbb47b18288c4d875f823ce~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 持续监控：CM

持续监控指进行持续性观察，一旦发现异常，立即发出警报。持续监控能力指持续监控系统的运行状态，分析监控数据，得出当前状态与期望状态之间的偏差，提供态势感知有关的决策支持。

监控系统一般情况下采用Zabbix或者Prometheus去实现，很多情况下我们通过这些专业的监控平台能够对生产环境出现的问题快速拿出处理办法。

\\

前置工作
----

### JDK

介绍了这么多，接下来我们就开始准备Jenkins的安装吧。Jenkins是基于Java开发的系统，那么肯定是需要JDK的支持的

*   可以直接下载rpm的文件格式，然后一下命令进行执行

> 阿里云盘下载：​[​jdk-8u221-linux-x64.rpm​](https://link.juejin.cn/?target=https%3A%2F%2Fwww.aliyundrive.com%2Fs%2FQHGiM99jDP4 "https://www.aliyundrive.com/s/QHGiM99jDP4")​

```null
rpm -ivh jdk.rpm

```

*   如果下载的是tar的文件格式，那么将其进行解压，然后添加到环境变量即可

```bash
tar xf jdk.tar.gz -C /usr/local
vim /etc/profile
   export JAVA_HOME=/usr/local/jdk
   export PATH=$PATH:$JAVA_HOME/bin
   
source /etc/profile

```

### maven

maven并不是必须安装的，在Jenkins中也内置了maven工具，我们还是用自己的，并且在构建项目的时候要用到，所以这里就提前准备下，安装过程和jdk的方式差不多，我就直接上手了

```bash
wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
tar xf apache-maven-3.8.5-bin.tar.gz -C /usr/local && mv apache-maven-3.8.5 maven


vim /etc/profile
    export M2_HOME=/usr/local/maven
    export PATH=$PATH:$M2_HOME/bin
source /etc/profile

```

安装完成之后，我们还需要对其进行配置，相信大家已经不陌生了，也就修改​`​settings.xml​`​​下的​`​localRepository​`​​和​`​mirrors​`​两个地方。

> 直接修改即可

```xml
<mirror>
    <id>aliyun-public</id>
    <mirrorOf>public</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
<mirror>
    <id>aliyun-central</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/central</url>
</mirror>
<mirror>
    <id>aliyun-jcenter</id>
    <mirrorOf>jcenter</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
<mirror>
    <id>aliyun-google</id>
    <mirrorOf>google</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/google</url>
</mirror>
<mirror>
    <id>aliyun-gradle-plugin</id>
    <mirrorOf>gradle-plugin</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/gradle-plugin</url>
</mirror>
<mirror>
    <id>aliyun-spring</id>
    <mirrorOf>spring</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/spring</url>
</mirror>
<mirror>
    <id>aliyun-spring-plugin</id>
    <mirrorOf>spring-plugin</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/spring-plugin</url>
</mirror>
<mirror>
    <id>aliyun-grails-core</id>
    <mirrorOf>grails-core</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/grails-core</url>
</mirror>
<mirror>
    <id>aliyun-apache-snapshots</id>
    <mirrorOf>apache snapshots</mirrorOf>
    <name>Nexus aliyun</name>
    <url>https://maven.aliyun.com/repository/apache-snapshots</url>
</mirror>

```

安装Jenkins
---------

Jenkins提供了多种安装方式，包括：war包，docker部署，安装程序等，按照自己的需要选择安装

> 这里通过Docker进行安装

1.  先来创建外部挂载目录，并且授予权限信息

> 防止在启动过程中由于权限不足而导致Jenkins启动失败

```bash
mkdir /data/jenkins && chmod a+x /data/jenkins

```

2.  docker启动

执行如下命令即可将启动一个docker容器，随即Jenkins就会进行启动

```ruby
docker run -d -p \
  --privileged=true \
  -p 8080:8080 -p 50000:50000 \
  --restart=always \ 
  -v /data/jenkins/:/var/jenkins_home/ \
  --name jenkins \
  jenkins/jenkins:2.332.1-lts

```

而想要验证Jenkins的启动是否成功，通过以下方式都能进行查看

```null
docker ps
docker logs -f jenkins

```

> 这里再提供关于docker-compose.yml的启动方式

```yaml
version: "3" 
services: 
    jenkins: 
        image: jenkins/jenkins:2.332.1-lts 
        container_name: jenkins 
        restart: always
        privileged: true
        ports: 
            - 8080:8080 
            - 50000:50000 
        volumes: 
            - /data/jenkins/:/var/jenkins_home/

```

随后执行如下命令即可完成启动

```null
docker-compose up -d

```

稍等一会通过​[​http://ip:8080​](https://link.juejin.cn/?target=http%3A%2F%2Fip%3A8080 "http://ip:8080")​就可以进入到如下界面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3f809a313224621a939ee85dd38df9b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

\\

### 插件加速

> 这里并不是必须操作

这里我们先不忙着进入到Web页面。由于网络影响，在安装插件的时候会出现耗时长，插件安装失败的问题，这里我们先进行一些插件加速的配置

*   **hudson.model.UpdateCenter.xml**

进入到主目录下，修改​`​hudson.model.UpdateCenter.xml​`​文件，将url修改为国内源

```ruby
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e5f8dac414c4d80afc8219df762f7c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

> 当然了，如果自己的网速可以，可以选择不设置的。

随后重启Jenkins服务，接下来我们就可以一步步的进行初始化配置了

初始化配置
-----

### 登录页面

接下来我们就正式进入到Jenkins内部，我们需要登录才能进行下一步，那么在页面上Jenkins已经为我们说明了默认密码所在的位置，我们直接查看就能得到密码

由于我们是通过Docker安装的Jenkins，所以我们需要执行这样的操作

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword

```

或者可以通过查看启动日志，在日志的最后也会输出相关的密码信息

```null
docker logs -f jenkins

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab8dbcb0bc32407295c96841288e8e08~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 必备插件

随后进入到插件安装的界面，随便怎么选都行。随后进入到Jenkins内部之后也有统一管理插件的地方

这里推荐几个需要安装的插件

#### Git Parameter

基于Git的参数化构建插件，在开始构建的时候我们可以对Git的分支，版本号，标签等参数进行自由选择

#### Publish Over SSH

通过SSH的方式连接到我们配置的远程服务器上，进行文件传输，shell脚本运行等操作

#### Gitlab Plugin

基于Gitlab的WebHooks操作，可以通过该插件构建触发器， 当GitLab中发生推送代码或创建合并请求时，触发 Jenkins 来执行构建任务

#### Chinese

有些Jenkins安装下来，系统是英文的，如果大家看的不习惯的话，这个插件可以将Jenkins汉化

#### DingTalk

整合钉钉通知，当构建成功之后，会下发通知给指定的钉钉，在最后我会配置让大家看看效果

> 如果在线安装插件还是比较慢的话，那么推荐采用离线安装的方式进行处理，​[​点击进入​](https://link.juejin.cn/?target=https%3A%2F%2Frepo.huaweicloud.com%2Fjenkins%2F "https://repo.huaweicloud.com/jenkins/")​到当前页面，这里会有关于Jenkins相关的安装包

点击安装之后会进入到当前页面，稍等片刻等待安装完成

> 可能会出现下载失败的插件，这里并不需要担心。进入到Jenkins之后在**Manage Plugins**处进行插件的安装，卸载等操作

#### Role-based Authorization Strategy

通过添加一个新的基于角色的机制来管理用户的权限，支持创建全局角色，项目角色等，方便成员间的权限控制。

支持采用正则表达式匹配项目名

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de38abdc3dd146b2b76e26546718cdf6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 3\. 创建管理员

进入到当前页面可以创建管理员账号，当前也可以不创建直接通过admin账号继续操作，这里随意

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5c5a5f137604aff9db68b7029df98c1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 4\. 实例配置

默认保持不动，直接点击 **保存并完成**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d3a6e8931c14806aff8cb35834dbc78~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

当看到这个页面的时候说明我们以及配置完成，接下来就是其他的操作了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c66b6adcd3e7460185a97bc858dd9e6b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

> 界面美观了不少，应该是改版过的  
> 建议进来之后先修改一下admin的密码，不然每次登录都需要到日志中查看

最后
--

本章就先到这里，大家最好能够根据本章内容实际安装好，接下来我们的主要目标就是驾驭它，让它为我们服务