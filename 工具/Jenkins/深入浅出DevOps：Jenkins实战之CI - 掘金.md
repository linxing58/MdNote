# 深入浅出DevOps：Jenkins实战之CI - 掘金
我报名参加金石计划1期挑战——瓜分10万奖池，这是我的第1篇文章，[点击查看活动详情](https://s.juejin.cn/ds/jooSN7t "https://s.juejin.cn/ds/jooSN7t")

> 💯 作者： **俗世游子**【谢先生】。 8年开发3年架构。专注于Java、云原生、大数据等领域技术。  
> 💥 成就： 从CRUD入行，负责过亿级流量架构的设计和落地，解决了千万级数据治理问题。  
> 📖 同名社区： ​[​掘金​](https://juejin.cn/user/3359725700263694 "https://juejin.cn/user/3359725700263694")​、​[​51CTO​](https://link.juejin.cn/?target=https%3A%2F%2Fblog.51cto.com%2Fu_14948012 "https://blog.51cto.com/u_14948012")​、 ​[​gitee​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq "https://gitee.com/mr_sanq")​。  
> 📂 清单： ​[​goku-framework​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq%2Fgoku-framework "https://gitee.com/mr_sanq/goku-framework")​、​[​【更新中】享阅读II​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq%2Fenjoy-read-ii "https://gitee.com/mr_sanq/enjoy-read-ii")​

前言
--

我们先来罗列一下我们目前已经处理好的工具集

*   **Git&GitLab**
*   **Maven**
*   **Jenkins**
*   **Docker**

那好，本章就到了最最激动人心的时刻，就是通过这些来上手Jenkins，通过一个实际的小栗子将整个CI流程处理完成

环境规划
----

这里先来规定一下环境，一定要保证以下两点：

1.  保证GitLab，Jenkins是能够正常访问到的
2.  至少还需要准备一台2G内存的机器，机器上要安装了Docker环境（虚拟机就可以，土豪例外），用来处理SpringBoot程序的执行

| 机器IP | 端口 | 描述 |
| --- | --- | --- |
| 192.168.10.200 | 88 | GitLab |
| 192.168.10.200 | 8080 | Jenkins |
| 192.168.10.201 | 8080 | SpringBoot服务 |

基础配置
----

### 配置Jenkins中的Maven和JDK

#### 配置Maven

在关于Jenkins的基础介绍中，我们已经在宿主机配置过Maven，但是由于Jenkins是基于Docker方式部署的，容器内部无法直接使用宿主机上的Maven，所以我们需要进行一些额外操作

*   将Maven安装包整个复制到Jenkins的挂载目录中

```bash
cd /usr/local/
cp -rp maven/ /opt/jenkins

```

挂载目录和容器内部是共享的，所以在容器内部就已经有了maven这个目录

```bash

docker exec -it jenkins bash

cd /var/jenkins_home/

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/686ac63e5ad943748bb3f68faac95c82~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

那么我们进入到`系统管理>全局工具配置`，进行相关配置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5e5467d4fa54402add8320bae866126~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

在最下面能找到Maven的配置项，点击Maven安装能够出现配置窗口

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9231ab9af4b846c4a3813cb3b18e4d0b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这里注意**不要选择自动安装**，根据如下方式选择我们配置的Maven

> Q：为什么MAVEN\_HOME下的地址是 _/var/jenkins\_home_\*  
> \*A：这是因为Jenkins采用容器化方式搭建，配置的路径自然得是容器内部的地址，而容器内部的数据目录在`/var/jenkins_home`下，因而maven这里配置 `/var/jenkins_home/maven`下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f65584ea21e4d949f735b1464817152~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

保存之后，Maven就配置完成了。

#### 配置JDK

JDK的配置和上一步是一样的流程，这里就不多说了。本人项目采用的是Java 15开发的，[下载JDK15](https://link.juejin.cn/?target=https%3A%2F%2Frepo.huaweicloud.com%2Fopenjdk%2F15.0.2%2Fopenjdk-15.0.2_linux-x64_bin.tar.gz "https://repo.huaweicloud.com/openjdk/15.0.2/openjdk-15.0.2_linux-x64_bin.tar.gz")的版本，配置完成之后就是这个样子的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45aafed95a6e4344b4d25390b4e90bca~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

Jenkins任务
---------

那接下来我们就开始创建新的任务。点击菜单栏中新建任务出现如下图中的页面，这里用来配置任务名称和任务风格，我们就选择第一个自由风格就好

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd5c2b5c32414adaa297fc265dddcdd8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

接下来就要注意几个关键点：

### 源码管理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93232b08b5614c659b581957fbb0708c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

项目源码肯定是从搭建好的GitLab上获取到的，Jenkins本身有Git插件，可以通过项目地址和登录账户将项目从远程仓库克隆到本地，所以我们这里只要配置好远程仓库地址和登录账号就可以了

还有一点一定要注意，在GitLab上已经没有了master分支，统一全部为main分支。所以一定要记得修改**Branches to build**位置的分支

#### **Credentials**

默认情况下**Credentials**的位置是没有选项的，需要点击旁边的添加进行操作，点击会出现如下弹框，然后根据对应的信息输入相关内容即可

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0511dafb48a5424a90065f5e07e321e3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### **构建**

构建方式我们肯定选择的是Maven方式，点击**增加构建步骤**，在出现的选项中选择调用**顶部Maven目标**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed0f7a8b37c44b1783489d14abb8dc03~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

而我们在这里看到的maven就是上一步配置的全局工具，而目标那一栏就和常规的maven操作是一样的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b5da36d7df44206b1a5b82795346ec0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

接下来点击保存，我们就先验证一下这个操作操作

### 立即构建

点击菜单中**立即构建**，会在下方出现每次构建的历史，随后在构建历史中出现构建过程中的进度条，点击进度条会进入到**控制台输出**页面，这里会实时展示构建的日志，很是智能

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1318cd04d3284741bda71aa23ec4ce38~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 控制台日志

1.  首先是通过git将项目克隆到本地的日志

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/587916aceee042209d38428f0139c0b7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

1.  其次是整个项目构建的过程，这里由于本人事先实验过，所以没有出现下载依赖的过程，大家在首次构建的时候会出现下载依赖的过程，这属于正常现象

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35382631571a4de992285dba2ab8338c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 验证结果

随后我们进入到容器内部，进入到日志中的目录结构中查看是否存在构建后的jar包

```bash
docker exec -it jenkins bash 
cd /var/jenkins_home/workspace/Devops_App/target

```

或者也可以在挂载目录下验证下，都是同样的结果

```bash
cd /opt/jenkins/workspace/Devops_App/target

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2753d02b84bb4cbbb3ed003715d93d7f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这样说明我们上面的流程是正常的。

推送到目标服务器
--------

当我们的程序被打包成功之后，我们就要准备开始执行当前程序。我们已经准备了一台服务器，用来执行当前程序。

那么我们接下来的难点就是如何将Jenkins打包好的程序推送到目标服务器上并开始执行。

### **Publish over SSH**

还记得当时推荐的插件 `Publish over SSH` 吧，它就是做这个工作的。

现在我们从 `系统管理 > 全局配置`中找到配置目标服务器的位置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01dbb6312f7640ea8c98ee7592d5bd2f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

到这里我们就开始配置目标服务器

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee51612a910a4ce7a791ffae459e785b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

以下需要填写远程服务器的相关信息，这里需要非常注意的是

*   **Remote Directory下的目录必须在目标服务器上存在，否则测试链接无法成功**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62e04da503fc460889a5e7d28b0c7439~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

选中当前选项，可以根据需要选择**密码登录**或者秘钥登录

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/776c4bb90b344c6b8425e2587bb9a5f3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 推送到目标服务器

我们继续回到任务的配置页面，选择**构建后操作**，从中选中**Send build artifacts over SSH**，配置要推送过去的文件立即构建就能看到效果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d62d941c43c14e68a250faba235c857d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**Source files** 是支持推送多文件的，中间采用英文逗号分割

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2026e745bb054315ab3dd4ad560a06b1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

细心的大家肯定能发现在日志中会多出这么几条日志，这里就是推送文件的日志，接下来我们再去目标服务器上看看

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce7eea2d4da04942af99a83c3636a2f6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

能够看到，已经完美的成功了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84d9e4ec8f11435d8f8441a3accb21dc~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 构建镜像并执行

那接下来我们就要开始在目标服务器上执行程序了，我们基于Docker来运行服务，所以：

#### 第一步： 编写一个Dockerfile

```bash

FROM registry.cn-beijing.aliyuncs.com/mr_sanq/base_repo:15.0.2_7


ENV JAVA_OPS=""

WORKDIR /opt/apps


ADD target/*.jar $WORKDIR/app.jar

EXPOSE 8080


ENTRYPOINT ["sh","-c","java $JAVA_OPS -jar $WORKDIR/app.jar"]

```

#### 第二步：改造推送配置

多文件如果不在同一个目录下，那么是无法指定移除前缀的

这里重点来看**Exec command**，在这里我们可以执行相关命令或者执行shell脚本，就相当于我们通过远程连接工具进行的操作

```bash
cd /opt/app/${JOB_NAME}
docker build -t ${JOB_NAME}:v0.0.${BUILD_NUMBER} .
docker container stop ${JOB_NAME} && docker container rm ${JOB_NAME}
docker run -d -p 8080:8080 --name ${JOB_NAME} ${JOB_NAME}:v0.0.${BUILD_NUMBER}

```

> 更多关于Jenkins在构建的变量可以通过点击对应**Jenkins environment variables**的链接进行查看

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bbe54bede7c4c71a7d4a556e55cb823~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 第三步：验证结果

能够发现在目标服务器上已经成功运行

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8692f4c014a42e1a01278e9e5d8c47e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

最后
--

本章虽然介绍性的内容不多，但是内容全在一步一步的操作中，大家可以按照这个流程，赶快让自己的项目在Jenkins上执行起来吧！！

过往内容
----

[深入浅出DevOps：DevOps核心思想](https://juejin.cn/post/7134696448616038414 "https://juejin.cn/post/7134696448616038414")

[深入浅出DevOps：版本控制Git&Gitlab](https://juejin.cn/post/7135949075387514894 "https://juejin.cn/post/7135949075387514894")

[深入浅出DevOps：持续集成工具Jenkins](https://juejin.cn/post/7137300017152262151 "https://juejin.cn/post/7137300017152262151")

[深入浅出DevOps：简易Docker入门](https://juejin.cn/post/7137666687872008205 "https://juejin.cn/post/7137666687872008205")