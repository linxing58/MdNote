# Jenkins-jenkins+maven+svn自动部署(Windows)_成长20171221的博客-CSDN博客
**生命要用于尝试, 给自己一个机会, 没有试过怎么知道不行.**

## 一: 安装所需软件

**jdk     Jenkins     Maven    svn**

**Jenkins 安装 maven integration plugin   subversion plugin   Deploy to container**

如果插件安装失败, 请参考此篇博文 [https://www.cnblogs.com/sxdcgaq8080/p/10489326.html](https://www.cnblogs.com/sxdcgaq8080/p/10489326.html)

## 二: 配置[Jenkins](https://so.csdn.net/so/search?q=Jenkins&spm=1001.2101.3001.7020)

在 Jenkins 是全局工具配置中

![](https://img-blog.csdnimg.cn/2021020316031422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### 1. 配置[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)的 setting 文件

![](https://img-blog.csdnimg.cn/20210204111255743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### 2. 配置 jdk

![](https://img-blog.csdnimg.cn/20210203160853441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### 3. 配置 maven

![](https://img-blog.csdnimg.cn/20210203160924102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

## 三: 创建项目

### **第一步: 创建 maven 项目**

![](https://img-blog.csdnimg.cn/20210203161808692.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210203161921716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

如果没有构建一个 maven 项目选项, 需要安装插件: Maven Integration plugin

![](https://img-blog.csdnimg.cn/20210203170030359.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

如果插件安装失败, 请参考此篇博文 [https://www.cnblogs.com/sxdcgaq8080/p/10489326.html](https://www.cnblogs.com/sxdcgaq8080/p/10489326.html)

### **第二步: 项目详细配置: general**

![](https://img-blog.csdnimg.cn/20210203172535380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### **第三步: 源码管理**

主要目的是从远程仓库拉取代码

![](https://img-blog.csdnimg.cn/20210204091247621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### **第四步: 构建触发器**

Jenkins 提供了 6 种构建触发器，分别是：

1.  build whenever a snapshot dependency is built , 当 job 依赖的快照版本被 build 时，执行本 job；
2.  触发远程构建 (例如, 使用脚本)；
3.  build after other projects are built 当本 job 依赖的 job 被 build 时，执行本 job；
4.  build periodically 隔一段时间 build 一次，不管版本库代码是否发生变化，通常不会采用此种方式；
5.  GitHub hook trigger for GITScm polling 通过 Github 钩子触发；
6.  poll scm 隔一段时间比较一次源代码，如果发生变更，那么久 build。否则，不进行 build，通常采用这种方式。

![](https://img-blog.csdnimg.cn/20210204091415392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### 第五步: 增加构建步骤

如果项目不是父子多模块工程, 则不需要这一步

项目有多个 module, 且都是父子 pom，所以我们构建的时候直接选择 构建最顶层 maven 目标，这里需要注意的是，因为我们的项目有多个，所以需要将每个仓库目录下的 pom 打包一次，这就需要指定特定目录的 pom，jenkins 也可以实现这一功能，在高级选项中可以指定 pom 的位置

![](https://img-blog.csdnimg.cn/20210205104427123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210205104510214.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210205104636902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### **第六步: build**

![](https://img-blog.csdnimg.cn/20210204091557174.png)

Root POM: pom.xml 文件的路径是相对于工作空间的, 如果没有下面的自定义工作空间就是相对于 Jenkins 默认的工作空间, 反之是相对于自定义的工作空间

Goals and options: 执行什么操作 

> compile
>
> clean    删除 target/
>
> test       test case junit/testNG
>
> package 打包
>
> install    把项目 install 到 local repo
>
> deploy    发本地 jar 发布到 remot         

选择自定义空间, 点击高级, 输入目录, 从 svn 检出的代码就会放在这个目录中

![](https://img-blog.csdnimg.cn/20210204134439694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

编译好的 calss 文件和打好的包会放在 pom.xml 文件同级目录 target 文件夹下

完成上述步骤就能对[svn](https://so.csdn.net/so/search?q=svn&spm=1001.2101.3001.7020)的项目进行编译, 并且打包成 war 包.

* * *

接下来如何将生成的 war 包或 class 文件自动复制到 Tomcat 中 (Tomcat 会自动将 war 包解压成项目)

### **第七步: 远程发布**

**方式①:Deploy war/ear to a container**

**配置 tomcat(config/tomcat-user.xml) 配置一个 manager 用户**

> **<role rolename="admin-gui"/>**
>
> **<role rolename="manager-gui"/>**
>
> **<role rolename="manager-script"/>**
>
> **<role rolename="manager-status"/>**
>
> **<role rolename="tomcat"/>**
>
> **<role rolename="role1"/>**
>
> **<user password="admin" roles="admin-gui,manager-gui,manager-script,manager-status,tomcat,role1" username="admin"/>**

![](https://img-blog.csdnimg.cn/20210204170512381.png)

**修改 context.xml(webapps\\manager\\META-INF\\context.xml) 注释掉访问来源受限**

![](https://img-blog.csdnimg.cn/20210204170705654.png)

**注释掉 server.xml 中的 Context 标签**

![](https://img-blog.csdnimg.cn/20210204175832237.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

**远程发布 Tomcat , 先安装 Deploy to container plugin**

![](https://img-blog.csdnimg.cn/20210204175706822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

**方式②:Execute Windows batch command**

jenkins 和项目部署在同一台服务器上可以采用这种方式, 且系统为 Windows.

自动部署的思路：设置全局变量（项目名称，构建新包路径，配置文件路径，Tomcat 路径等）-> 关闭 Tomcat-> 删除 Tomcat 中旧项目 -> 拷贝新文件到 Tomcat 应用目录 -> 替换配置文件 -> 启动 Tomcat，

![](https://img-blog.csdnimg.cn/20210205110119775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210205110657312.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

脚本文件: 此脚本文件只做参考

````null
set project_name=jenkinstestset service_name=EIP_JENKINSTESTset package_path=F:\Jenkins\workspaceset config_path=F:\Jenkins\projectConfigset startup_command=%project_base_path%\%project_name%\tomcat\bin\startup.batset shutdown_command=%project_base_path%\%project_name%\tomcat\bin\shutdown.batrem 改变进程ID，保证执行完毕后jenkins不会杀掉所有进程cd /d %project_base_path%\%project_name%\tomcat\binif exist %project_base_path%\%project_name%\eip goto exsit_back_fileif exist %project_base_path%\%project_name%\eip_back goto remove_backfile_no_copyif exist %project_base_path%\%project_name%\eip_back goto remove_backfilemd %project_base_path%\%project_name%\eip_backxcopy %project_base_path%\%project_name%\eip %project_base_path%\%project_name%\eip_back /s/erd /s/q %project_base_path%\%project_name%\eiprd /s/q %project_base_path%\%project_name%\eip_backmd %project_base_path%\%project_name%\eip_backxcopy %project_base_path%\%project_name%\eip %project_base_path%\%project_name%\eip_back /s/erd /s/q %project_base_path%\%project_name%\eiprd /s/q %project_base_path%\%project_name%\eip_backmd %project_base_path%\%project_name%\eipxcopy %package_path%\eip-web\target\eip-web %project_base_path%\%project_name%\eip /s/e/yrem 复制module(eip-application) class文件xcopy %package_path%\eip-application\target\classes %project_base_path%\%project_name%\eip\WEB-INF\classes /s/e/yrem 复制module(eip-bpm) class文件xcopy %package_path%\eip-bpm\target\classes %project_base_path%\%project_name%\eip\WEB-INF\classes /s/e/yrem 复制module(eip-business) class文件xcopy %package_path%\eip-business\target\classes %project_base_path%\%project_name%\eip\WEB-INF\classes /s/e/yrem 复制module(eip-common) class文件xcopy %package_path%\eip-common\target\classes %project_base_path%\%project_name%\eip\WEB-INF\classes /s/e/yrem 复制module(eip-core) class文件xcopy %package_path%\eip-core\target\classes %project_base_path%\%project_name%\eip\WEB-INF\classes /s/e/yrem 复制module(eip-project) class文件xcopy %package_path%\eip-project\target\classes %project_base_path%\%project_name%\eip\WEB-INF\classes /s/e/yrem 复制module(eip-system) class文件xcopy %package_path%\eip-system\target\classes %project_base_path%\%project_name%\eip\WEB-INF\classes /s/e/yxcopy %config_path%\%project_name%\eip %project_base_path%\%project_name%\eip /s/e/ycd /d %project_base_path%\%project_name%\tomcat\bin```

根据需求可以采用上面其中一种方式进行远程发布

**Q&A:**

**1.采用第二种方式,脚本启动Tomcat失败**

Jenkins执行**完**命令会杀掉进程及其衍生进程,需要在脚本中添加一行命令

> rem 改变进程ID，保证执行完毕后jenkins不会杀掉所有进程
> 
> set BUILD\_ID=dontKillMe

**2.采用第二种方式:windows下启动tomcat没有cmd窗口**

jenkins启动tomcat是直接在后台启动的，没有cmd的黑窗口,如果想要窗口的话需要自己去配置一个jenkins节点.

可以参考此篇博文:[https://blog.csdn.net/liu\_gang\_/article/details/107837710](https://blog.csdn.net/liu_gang_/article/details/107837710)

### 第八步:点击构建

![](https://img-blog.csdnimg.cn/20210204175931756.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

### Q&A:

**1.远程部署成功,启动失败**

![](https://img-blog.csdnimg.cn/20210204180102219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210204180131472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3OTc2Mjg5,size_16,color_FFFFFF,t_70)

**原因:程序包本身有问题**,比如数据库的连接等 
 [https://blog.csdn.net/qq_37976289/article/details/113610020](https://blog.csdn.net/qq_37976289/article/details/113610020)
````
