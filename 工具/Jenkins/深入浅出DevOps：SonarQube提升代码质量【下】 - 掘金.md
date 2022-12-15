# 深入浅出DevOps：SonarQube提升代码质量【下】 - 掘金
我报名参加金石计划1期挑战——瓜分10万奖池，这是我的第3篇文章，[点击查看活动详情](https://s.juejin.cn/ds/jooSN7t "https://s.juejin.cn/ds/jooSN7t")

> 💯 作者： **俗世游子**【谢先生】。 8年开发3年架构。专注于Java、云原生、大数据等领域技术。  
> 💥 成就： 从CRUD入行，负责过亿级流量架构的设计和落地，解决了千万级数据治理问题。  
> 📖 同名社区：[掘金](https://juejin.cn/user/3359725700263694 "https://juejin.cn/user/3359725700263694")​、​[​github​](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxiezhyan "https://github.com/xiezhyan")​​、​[​51CTO​](https://link.juejin.cn/?target=https%3A%2F%2Fblog.51cto.com%2Fu_14948012 "https://blog.51cto.com/u_14948012")​、​[​gitee​](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fmr_sanq "https://gitee.com/mr_sanq")​​  
> 📂 清单： ​​[​goku-framework​](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxiezhyan%2Fgoku-framework "https://github.com/xiezhyan/goku-framework")​​、​[​【更新中】享阅读II​](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fxiezhyan%2Fenjoy-read-ii "https://github.com/xiezhyan/enjoy-read-ii")​

[深入浅出DevOps：DevOps核心思想](https://juejin.cn/post/7134696448616038414 "https://juejin.cn/post/7134696448616038414")

[深入浅出DevOps：版本控制Git&Gitlab](https://juejin.cn/post/7135949075387514894 "https://juejin.cn/post/7135949075387514894")

[深入浅出DevOps：持续集成工具Jenkins](https://juejin.cn/post/7137300017152262151 "https://juejin.cn/post/7137300017152262151")

[深入浅出DevOps：简易Docker入门](https://juejin.cn/post/7137666687872008205 "https://juejin.cn/post/7137666687872008205")

[深入浅出DevOps：Jenkins实战之CI](https://juejin.cn/post/7139406386911248414 "https://juejin.cn/post/7139406386911248414")

[深入浅出DevOps：SonarQube提升代码质量](https://juejin.cn/post/7140608660006240270 "https://juejin.cn/post/7140608660006240270")

前言
--

既然我们已经了解了关于SonarQube的相关内容，那么接下来就到了本章的重头戏：

*   Jenkins横行的世界里，哪里还需要手动将项目推送进行检测，自然是让Jenkins帮助我们完成这一切。那么接下来我们就看看如何在Jenkins中对SonarQube进行操作吧

集成SonarQube
-----------

### 主要流程

将SonarQube整合到Jenkins上其实也不难，只需要完成三步：

1.  在Jenkins中下载SonarScanner的插件
2.  在配置中心配置SonarScanner的完整路径
3.  准备就绪之后，开始项目配置，执行SonarScanner的脚本

那么，接下来就完成这第一步

### 下载插件

进入到Jenkins的插件管理页面，在其中搜索集成SonarQube Scanner，然后点击安装

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f78491177b764bda88cce588c490564a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这样等待片刻安装成功之后，在​`​全局工具配置​`​中能够看到相关SonarQube Scanner的配置信息就可以

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67ddfe00881d47e190db8bc7a6fcf9a8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 配置SonarQube Scanner

然后接下来就开始进行配置吧，上一节相信大家都下载过了，如果没有下载大家通过下面的方式就可以直接下载到服务器上

```bash
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.0.2311-linux.zip

```

这里我们自己将sonar-scanner下载，将其放到Jenkins数据卷挂载的本地目录中。这样sonar-scanner也就实际存在于​`​/var/jenkins_home​`​

*   **可执行程序下载**

那么这里的我们也要做这样的操作，保证SonarQube Scanner在容器内部也能访问的到

```bash
mv sonar-scanner-cli-4.6.0.2311-linux.zip /opt/jenkins && unzip sonar-scanner-cli-4.6.0.2311-linux.zip

```

我们进入到容器内部进行查看，已经能够看到SonarQube Scanner了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/096e6a2ef36d4b189c0c642ad8581fbc~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

*   **Jenkins全局工具配置设置**

这里一定要注意图中框住的地方，一定是容器内部的目录地址，而不是本地目录

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/116a9e20912145f89e6082bad4130eef~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 配置SonarQube服务

有了上一章的基础之后，我们这里想要将项目进行质量检查非常简单，只需要在Jenkins任务配置中添加一个执行shell脚本的命令即可

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/121e01c465664553aeafb89d6d094194~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

记住：这种方式需要将SonarQube的服务地址写在SonarScanner的配置文件中。而这样的话如果SonarQube的地址改变之后我们需要登录到服务器上修改SonarQube Scanner的配置信息比较繁琐。

那接下来我们就采用另一种方式

#### 配置SonarQube的权限

SonarQube是需要登录验证的，当然为了减少账户密码的流动，我们这里使用SonarQube的令牌机制。

进入到SonarQube的​`​配置>权限​`​下查看图中的三个选项是否开启，默认情况下都是**开启状态**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ab3f53899c4446185b5d2d8e0bf3131~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 添加令牌

随后点击​`​账户头像>我的账号>安全​`​，在令牌的框中生成想要的令牌

> 记住：及时复制保存

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0a9c551123645a48e84687cb8cbc501~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### Jenkins添加SonarQube服务

回到Jenkins中，进入到​`​系统管理>系统配置​`​​，找到​`​SonarQube servers​`​的地方添加SonarQube服务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26d99228c9f243779774fa3d5b26dd4c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

按照以往经验，在添加登录凭证的时候选择的都是​`​用户名/密码​`​​的方式，这次我们选择​`​Secret Text​`​的方式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e9379ab5c304ae1a9b20303a5bfecff~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6635db631ee8438b865de0a0d69837d0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

> 这样就完成了SonarQube服务的配置，随后再有改动就会变得很简单

### 项目配置

前置工作全部完成之后，接下来就要开始在项目中进行配置了，还记得我们之前的项目吧.

我们继续在该项目中进行才做，点击配置进行到配置页面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/672d0296e24b403ea1dfd850aa47e49d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

通过最开始一系列的配置，能够发现在​`​项目构建下>添加构建步骤​`​中多了一个选项，这个就是我们想要的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a55d5d00f944753b599df761cb970b1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7da7e7d896e546de856a12de06ab65d3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这里将这几个参数做一个简单介绍：

*   **Task to run**

这一栏什么都不用填，

*   **JDK**

JDK选择我们全局配置的就可以

*   **Path to project properties**

sonar-project.properties文件 中(-Dproject.settings) 的位置。如果未指定默认值为 sonar-project.properties，路径应该相对于模块根目录。

已当前项目为例，这里默认是项目的的根目录

*   **Analysis properties**

这里会将一些配置参数传递给SonarQube。也就是说如果我们没有sonar-project.properties的文件，那么我们可以把相关配置配置在这里。

#### 提示

著名的开源项目都是比较友好的，会给出相关的介绍和实例：

*   点击**问号**，就能够看到相关参数的介绍，并且Jenkins还给出了案例以供参考

> 注意：  
> **从4.** **1.2开始，强制要求sonar.java.binaries参数**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20322ee7a2ab47c9b8c10854b92848ba~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

那，这样就简单了。我们将上一章​`​sonar-project.properties​`​的内容复制到**Analysis properties**下，我们来执行一下

```ini

sonar.projectName=${JOB_NAME}
sonar.projectKey=${JOB_NAME}
sonar.sources=.
sonar.java.binaries=target/

```

> 相关变量参数相信不需要我做过多的介绍，之前我们也了解过

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfa1c2db2c0448bdb510b43ce620d118~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 立即构建

先来看看构建日志，看看构建成功的样子

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/501116e9d7f94d83adeffd248d116a00~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

再到SonarQube下查看，发现已经成功通过Jenkins将项目推送。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe0529fb0a8c49b3bc471101ff933ad7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

进入到项目页面，发现了SonarQube的菜单项，点击它能够直接跳转到到对应SonarQube的项目详情下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d1a3c7a073c4a37b23edd33927df139~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

最后
--

好了，实操性的文章总是这么的简短，并且相应流程也介绍的非常清晰。

大家赶快操作起来吧！！！