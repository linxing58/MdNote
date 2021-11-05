# CentOS安装Hadoop - 玄同太子 - 博客园
Hadoop 的核心由 3 个部分组成：

    HDFS: Hadoop Distributed File System，分布式文件系统，hdfs 还可以再细分为 NameNode、SecondaryNameNode、DataNode。

    YARN: Yet Another Resource Negotiator，资源管理调度系统

    Mapreduce：分布式运算框架

1、软件与环境

　环境：CentOS-7-x86_64-Minimal-1810

    hadoop 版本：jdk-8u221-linux-x64.tar.gz，下载地址：[https://www.apache.org/dist/hadoop/common/](https://www.apache.org/dist/hadoop/common/)

    jdk 版本：jdk-8u221-linux-x64.tar.gz，hadoop 只支持 jdk7 和 jdk8，不支持 jdk11

2、解压安装文件

    通过 ftp 等工具讲安装包上传到服务器上，并解压到 / usr/local / 目录

cd /usr/local/ tar -zxvf /var/ftp/pub/jdk-8u221-linux-x64.tar.gz
tar -zxvf /var/ftp/pub/hadoop-2.9.2.tar.gz

3、配置 JDK

    修改 ${HADOOP_HMOE}/etc/hadoop/hadoop-env.sh 文件，修改 JAVA_HOME 配置（也可以修改 / etc/profile 文件，增加 JAVA_HOME 配置）。

vi etc/hadoop/hadoop-env.sh
// 修改为
export JAVA_HOME=/usr/local/jdk1.8.0_221/

4、设置伪分布模式（Pseudo-Distributed Operation）

    修改 etc/hadoop/core-site.xml 文件，增加配置（fs.defaultFS：默认文件系统名称）：

&lt;configuration>
    &lt;property>
        &lt;name>fs.defaultFS&lt;/name>
        &lt;value>hdfs://localhost:9000&lt;/value>
    &lt;/property>
&lt;/configuration>

    修改 etc/hadoop/hdfs-site.xml 文件，增加配置（dfs.replication：文件副本数）：

&lt;configuration>
    &lt;property>
        &lt;name>dfs.replication&lt;/name>
        &lt;value>1&lt;/value>
    &lt;/property>
&lt;/configuration>

5、设置主机允许无密码 SSH 链接

ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa       // 创建公钥私钥对
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys //
chmod 0600 ~/.ssh/authorized_keys // 设置权限，owner 有读写权限，group 和 other 无权限

6、格式化文件系统

bin/hdfs namenode -format

7、启动 NameNode 和 DataNode 进程（启动 hdfs）

./sbin/start-dfs.sh // 启动 NameNode 和 DataNode 进程
./sbin/stop-dfs.sh  // 关闭 NameNode 和 DataNode 进程

![](https://img2018.cnblogs.com/blog/1031555/201909/1031555-20190910131638533-874768126.png)

    输入地址：[http://192.168.114.135:50070，可查看 HDFS](http://192.168.114.135:50070，可查看HDFS)

![](https://img2018.cnblogs.com/blog/1031555/201909/1031555-20190910133312208-1282505704.png)

8、 启动 YARN

./sbin/start-yarn.sh
./sbin/stop-yarn.sh

![](https://img2018.cnblogs.com/blog/1031555/201909/1031555-20190910174331646-1743282090.png)

     输入地址：[http://192.168.114.135:8088/，可查看 YARN](http://192.168.114.135:8088/，可查看YARN)

![](https://img2018.cnblogs.com/blog/1031555/201909/1031555-20190910173940884-762261211.png) 
 [https://www.cnblogs.com/zhi-leaf/p/11496877.html](https://www.cnblogs.com/zhi-leaf/p/11496877.html)
