# Linux+Tomcat+Jdk1.8+jenkins环境搭建 - 走看看
1.下载jdk的rpm安装包，这里以jdk-8u191-linux-x64.rpm为例进行说明

下载地址：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 如图操作；

![](https://img2018.cnblogs.com/blog/1080118/201901/1080118-20190115175707799-1808102200.png)

2\. 将jdk-8u191-linux-x64.rpm 移动到合适的安装目录上，安装软件不要在/home路径下，这样很容易涉及到不同用户的访问权限，这样对程序的维护，是相当不利的。这里将把安装包移动到/opt目录

mv jdk-8u191-linux-x64.rpm /opt   ;把安装包移动到/opt目录，这里有个前提是要先切换到该文件的所在目录下再执行MV命令；

3、给安装包赋予运行权限

chmod 755 jdk-8u191-linux-x64.rpm

4、安装rpm文件

rpm -ivh jdk-8u191-linux-x64.rpm --force --nodeps      就可以了;nodeps的意思是忽视依赖关系,因为各个软件之间会有多多少少的联系。有了这两个设置选项就忽略了这些依赖关系，强制安装或者卸载

5、检查java版本

执行 java -version，如果显示如下图，说明安装成功；

![](https://img2018.cnblogs.com/blog/1080118/201901/1080118-20190115180553084-25325047.png)

6\. 对比环境变量

\[root@localhost opt\]# vi /etc/profile                          ;编辑系统配置文件

然后输入i ,最后增加下面内容  
\==================================================================================  
export JAVA\_HOME=/usr/java/jdk1.7.0\_04  
export CLASSPATH=.:$JAVA\_HOME/lib/dt.jar:$JAVA\_HOME/lib/tools.jar   
export PATH=$PATH:$JAVA\_HOME/bin

最后按exc键，然后输入冒号，最后输入:wq 保存退出

Tomcat7的安装

1.下载Linux版tomcat7，官网即可下载

[https://tomcat.apache.org/download-70.cgi](https://tomcat.apache.org/download-70.cgi)

![](https://img2018.cnblogs.com/blog/1080118/201901/1080118-20190115181130728-90292485.png)

第二步：解压，切换到放置的位置执行如下命令，

　　\[root@localhost ~\]# tar -zxvf /usr/java/apache-tomcat-7.0.82.tar.gz

第三步：启动

　　进入到tomcat bin目录中。

　　输入 ./startup.sh启动Tomcat，假如显示 Tomcat started 则表明启动成功。

![](https://img2018.cnblogs.com/blog/1080118/201901/1080118-20190115181644534-122434678.png)

如不成功参考如下方法；

设置环境变量  
打开profile文件：vi /etc/profile  
然后按i进入编辑模式，在文件末尾添加下面的环境变量配置：  
CATALINA\_HOME=/usr/tomcat/apache-tomcat-7.0.86  
export CATALINA\_HOME    
然后ESC退出编辑模式，然后输入:wq保存退出  
使环境变量立即生效  
输入命令：source /etc/profile

安装Jenkins
---------

1、**直接复制下面命令执行即可；**

\#    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo  
\#    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key  
如果已经导入过密钥，rpm --import会失败，忽略即可

2、**使用yum安装Jenkins；**

\#   sudo yum install jenkins

3、**启动Jenkins**

常用命令

sudo service jenkins start //启动  
sudo service jenkins stop //停止  
sudo service jenkins restart //重启  
sudo chkconfig jenkins on //开机自启

日志目录：

/var/log/jenkins/jenkins.log

如果启动失败或有其他错误可以查看Jenkins日志

**4、初始化Jenkins**

浏览器输入Jenkins地址 (192.168.1.110:8080)  
根据提示找到initialAdminPassword后输入  
选择Install suggested plugins后jenkins会自动联网安装  
设置管理员账号密码等信息  
设置JenkinsURL，默认即可  
更改Jenkins端口  
/etc/sysconfig/jenkins

该配置文件中可以更改8080端口为其他端口，如果其他主机无法访问的话尝试关闭防火墙或者配置防火墙放行端口

更改端口后进入Jenkins管理页面的系统管理会提示“反向代理设置有误”，解决方法如下：

点击系统设置->找到Jenkins URL->更改端口为你自定义的端口->点击保存

注意：在初始化Jenkins前不建议更改端口，否则会出现登录后页面空白的问题，建议使用8080端口登录成功一次后再进行更改

Jenkins用户添加Root权限
-----------------

使用Jenkins自带用户的话会出现执行脚本时没有权限的问题，下面给出解决办法

sudo vim /etc/sysconfig/jenkins

修改$JENKINS\_USER

JENKINS\_USER=“root”

修改Jenkins相关文件夹用户权限

sudo chown -R root:root /var/lib/jenkins

sudo chown -R root:root /var/cache/jenkins

sudo chown -R root:root /var/log/jenkins

重启Jenkins

service jenkins restart

将war包部署到tomcat中

下面给出一个简单示例，将已有的war包部署到tomcat中(jenkins与tomcat在同一台主机)

点击Jenkins主页的新建任务  
输入任务名称  
选择构建一个自由风格的软件项目后点击确定  
在构建内添加构建步骤，选择执行shell  
输入执行脚本，脚本见下方  
点击保存  
点击左侧的立即构建

脚本供参考：

#!/bin/sh

tomcat\_path=/opt/apache-tomcat-8.0.50  
ShutDownTomcat=${tomcat\_path}/bin/shutdown.sh  
StartTomcat=${tomcat\_path}/bin/startup.sh

echo "============删除旧的war包==================="  
rm ${tomcat\_path}/webapps/root.war

echo "============删除tomcat下旧的文件夹============="  
rm -rf ${tomcat\_path}/webapps/root

echo "======拷贝编译出来的war包到tomcat下======="  
cp /home/gavinandre/root.war ${tomcat\_path}/webapps/root.war

echo "====================关闭tomcat====================="  
${ShutDownTomcat}

echo "================sleep 10s========================="  
for i in {1..10}  
do  
echo $i"s"  
sleep 1s  
done

export BUILD\_ID=DontKillMe

echo "====================启动tomcat====================="  
${StartTomcat}  
\------------------------------------------------------------------------------------------------到此就结束-----------------------------------------------------------------------------------------------