# Jenkins+gitea实现自动部署_小人物也有大烦恼的博客-CSDN博客
执行shell命令：

1.  获取jenkins源文件：```null
    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    ```
    
2.  导入jenkins公钥```null
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    ```
    
3.  安装 ```null
    yum install jenkins
    ```
    
4.  启动jenkins                     

           service jenkins start    启动

           service jenkins stop    停止

            service jenkins restart    重启

假如：jenkins启动报错：

![](https://img-blog.csdnimg.cn/20200521181812298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3VuaGVqaW5n,size_16,color_FFFFFF,t_70)
查看错误信息是因为找不到java，所以需要配置java路径。将第一步的路径复制一下，编辑配置：

> vi  /etc/init.d/jenkins

![](https://img-blog.csdnimg.cn/20201126104857166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

 将路径输入在如图位置，保存退出即可访问。http://{IP地址}:8080  

* * *

为了实现前端后端的自动部署，所以我们需要对jenkins配置全局环境

1、 Maven配置

![](https://img-blog.csdnimg.cn/20201128111358277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

        ![](https://img-blog.csdnimg.cn/20201128111532924.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
2、 JDK配置

        ![](https://img-blog.csdnimg.cn/20201128111430173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
3、 Git 配置

![](https://img-blog.csdnimg.cn/20201128111506332.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
4、 Node配置

![](https://img-blog.csdnimg.cn/2020112811160544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

* * *

搜索相关使用的插件，点击选中，点直接安装就可以了，如图所示：

![](https://img-blog.csdnimg.cn/20201128112217280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
这里我安装了我需要的插件：  
1、 [Deploy to container Plugin](https://plugins.jenkins.io/deploy)                       4、[Config File Provider Plugin](https://plugins.jenkins.io/config-file-provider)        
2、 [Generic Webhook Trigger Plugin](https://plugins.jenkins.io/generic-webhook-trigger)             5、 [Publish Over SSH](https://plugins.jenkins.io/publish-over-ssh)  
3、 [NodeJS Plugin](https://plugins.jenkins.io/nodejs)                                         

* * *

针对前后端分离的项目，我们对前端进行单独的配置。首先创建任务，针对我们的前面项目和gitea的仓库名称，新建了rim-ui的任务。  
相关配置如下：  
1、 丢弃旧的构建

![](https://img-blog.csdnimg.cn/20201128113041494.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
2、源码管理配置（配置代码地址以及账号认证）

![](https://img-blog.csdnimg.cn/20201128113142356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
![](https://img-blog.csdnimg.cn/20201128113325964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
3、 构建触发器（使用[Generic Webhook Trigger Plugin](https://plugins.jenkins.io/generic-webhook-trigger)配置自动构建，当Git提交代码就会触发自动构建 ）

![](https://img-blog.csdnimg.cn/2020112811353328.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
![](https://img-blog.csdnimg.cn/20201128113620738.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

4、 Gitea中配置钩子

![](https://img-blog.csdnimg.cn/20201128113934326.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
![](https://img-blog.csdnimg.cn/20201128114101516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
![](https://img-blog.csdnimg.cn/20201128114128204.png)

5、 配置构建环境

![](https://img-blog.csdnimg.cn/20201128114158171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

6、 配置构建执行shell命令

![](https://img-blog.csdnimg.cn/20201128114409573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

创建rim后端任务，因为我们这边是发布的jar包，所以后面的方式也是以jar的方式来演示。  
例如：  
1、 丢弃旧的构建配置

![](https://img-blog.csdnimg.cn/20201128114931521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
2、 源码管理配置（跟前端一样，就不多提）

![](https://img-blog.csdnimg.cn/20201128115001113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)
  
3、 配置构建器（跟上雷同）

![](https://img-blog.csdnimg.cn/20201128115026176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

4、 配置gitea钩子，雷同，就不写了

5、 配置build

![](https://img-blog.csdnimg.cn/20201128115129241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

6、 配置post steps

![](https://img-blog.csdnimg.cn/20201128115233626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/2020112811525571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MTI4MDQ3,size_16,color_FFFFFF,t_70)